## Thread.activeCount() 

```

Thread.activeCount() হলো Java-এর একটি static method, যা current thread-এর ThreadGroup এবং তার subgroup-গুলোতে কতগুলো active thread আছে তার আনুমানিক সংখ্যা return করে।

Short Note
Current Thread
      ↓
ThreadGroup
      ↓
SubGroups
      ↓
Active Threads Count
      ↓
Thread.activeCount()

Important: এখানে activeCount() পুরো JVM-এর সব thread count করে না; current thread-এর ThreadGroup ও তার subgroups-এর active thread count করে।

```
### Thread.enumerate() কী?
```

Thread.enumerate() হলো Java-এর একটি static method, যা current thread-এর ThreadGroup এবং তার subgroups-এর active thread-গুলোকে একটি Thread[] array-এর মধ্যে রাখে।

সহজভাবে:

activeCount() → কতগুলো active thread আছে
enumerate() → সেই active thread-গুলোর reference দেয়

Example
public class Main {
    public static void main(String[] args) {

        Thread t1 = new Thread(() -> {
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }, "Worker-1");

        Thread t2 = new Thread(() -> {
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }, "Worker-2");

        t1.start();
        t2.start();

        Thread[] threads = new Thread[Thread.activeCount()];

        int count = Thread.enumerate(threads);

        System.out.println("Active threads: " + count);

        for (Thread thread : threads) {
            if (thread != null) {
                System.out.println(thread.getName());
            }
        }
    }
}

সম্ভাব্য output:

Active threads: 3
main
Worker-1
Worker-2
```
## Thread Dump কী?
```

Thread Dump হলো একটি running Java application-এর একটি নির্দিষ্ট সময়ের snapshot, যেখানে সেই সময়ে থাকা thread-গুলোর state, stack trace, method call ইত্যাদি information থাকে।

সহজভাবে:

Thread Dump = কোন সময়ে কোন Thread কী করছে তার snapshot।

Example
Java Application
       ↓
    Running
       ↓
   Thread Dump
       ↓
 ┌───────────────────────┐
 │ main → TIMED_WAITING  │
 │ Worker-1 → RUNNABLE   │
 │ Worker-2 → BLOCKED    │
 │ Worker-3 → WAITING    │
 └───────────────────────┘
Thread Dump-এ কী দেখা যায়?
"Worker-1"
   java.lang.Thread.State: RUNNABLE
        at com.example.Task.process(Task.java:25)
        at com.example.Main.run(Main.java:10)

এখানে:

Thread Name → Worker-1
Thread State → RUNNABLE
Stack Trace → কোন method call-এর মধ্যে আছে
Code Line → কোন line-এ execution আছে
jstack দিয়ে Thread Dump

Windows-এ:

jps -l

ধরুন:

15548 Main

তারপর:

jstack 15548

অথবা file-এ:

jstack 15548 > thread-dump.txt
কেন Thread Dump ব্যবহার করি?

Thread Dump মূলত debugging-এর জন্য:

কোন thread কী করছে দেখতে
BLOCKED thread খুঁজতে
WAITING thread খুঁজতে
Deadlock detect করতে
Thread stuck হয়েছে কিনা দেখতে
Application-এর concurrency সমস্যা analyze করতে
Thread Dump ≠ Heap Dump
Thread Dump → Thread-এর information
Heap Dump   → Object / Heap memory-এর information
GitHub Short Note

Thread Dump হলো running Java application-এর নির্দিষ্ট সময়ের thread-এর snapshot। এতে Thread Name, State এবং Stack Trace দেখা যায়। jstack PID ব্যবহার করে Thread Dump নেওয়া যায়।
```
## JVM Structure
Thread  create korle jvm stack area te memory allowcate hoi.
```
Thread Stack — Short Note

One Thread → One JVM Stack → Multiple Stack Frames

প্রতিটি Thread-এর জন্য আলাদা JVM Stack থাকে।
Thread যখন কোনো method call করে, তখন Stack-এ একটি Stack Frame তৈরি হয়।
একটি Thread-এর Stack-এ একাধিক Stack Frame থাকতে পারে।
Method শেষ হলে তার Stack Frame remove হয়ে যায়।
Thread
  ↓
নিজস্ব JVM Stack
  ↓
┌──────────────┐
│ Frame: main  │
├──────────────┤
│ Frame: method1│
├──────────────┤
│ Frame: method2│
└──────────────┘

মনে রাখুন:
Thread → আলাদা Stack → Method অনুযায়ী Stack Frame
```

<img width="1140" height="601" alt="image" src="https://github.com/user-attachments/assets/998c0c94-d385-4378-a59e-b22b902e2124" />

## Summery
```
Thread
  ↓
JVM Stack
  ↓
Stack Frame (method)
  ├── Local Variables
  ├── Operand Stack
  └── Frame Data

Shortcut:

Stack Frame = একটি method চালানোর জন্য প্রয়োজনীয় local data + temporary data + execution information।
```
### Thread.getAllStackTraces();

```

এটি একটি static method, যা JVM-এর বর্তমানে চলমান/active thread-গুলোর Stack Trace সংগ্রহ করে।

Map<Thread, StackTraceElement[]> all =
        Thread.getAllStackTraces();
সহজভাবে

getAllStackTraces() → সব active thread-এর Stack Trace একসাথে দেয়।

Flow:

Thread.getAllStackTraces()
          ↓
   All Active Threads
          ↓
 ┌────────┬────────┬────────┐
 │ main   │ T1     │ T2     │
 └────────┴────────┴────────┘
    ↓        ↓        ↓
 Stack     Stack    Stack
 Trace     Trace    Trace
Example
public class Main {
    public static void main(String[] args) {

        Thread t1 = new Thread(() -> {
            while (true) {
                // doing work
            }
        }, "Worker-1");

        t1.start();

        Map<Thread, StackTraceElement[]> threads =
                Thread.getAllStackTraces();

        for (Map.Entry<Thread, StackTraceElement[]> entry
                : threads.entrySet()) {

            Thread thread = entry.getKey();

            System.out.println(
                    "Thread: " + thread.getName()
                    + " | State: " + thread.getState()
            );

            for (StackTraceElement element : entry.getValue()) {
                System.out.println("    " + element);
            }
        }
    }
}

Output-এর মতো:

Thread: main | State: RUNNABLE
    Main.main(Main.java:15)

Thread: Worker-1 | State: RUNNABLE
    Main.lambda$main$0(Main.java:7)
Return কী করে?
Map<Thread, StackTraceElement[]>

এখানে:

StackTraceElement = একটি Stack Frame-এর information।

Thread
   ↓
StackTraceElement[]
   ↓
StackTraceElement = একটি Stack Frame-এর information।
যেমন একটি thread-এর stack:

Worker-1
   ↓
methodC()
   ↓
methodB()
   ↓
methodA()
   ↓
run()

প্রতিটি method call-এর information StackTraceElement হিসেবে থাকে।

getStackTrace() vs getAllStackTraces()
thread.getStackTrace()

→ একটি নির্দিষ্ট Thread-এর stack trace।

Thread.getAllStackTraces()

→ সব active Thread-এর stack trace।

jstack-এর সাথে সম্পর্ক
Thread.getAllStackTraces()
        ↓
Java code থেকে
সব Thread-এর Stack Trace

আর:

jstack PID
    ↓
Java process-এর
Thread Dump

দুটিই thread debugging-এর জন্য useful।

GitHub Short Note

Thread.getAllStackTraces() → JVM-এর active threads-এর Thread এবং তাদের StackTraceElement[]-এর একটি Map return করে। এটি একসাথে সব thread-এর stack trace দেখার জন্য ব্যবহার করা হয়।
```
## Thread.yield() কী?
```
Thread.yield() হলো একটি static method, যা current thread-কে OS Scheduler-এর কাছে একটি hint দেয়:

“আমি এখন CPU ছেড়ে দিতে পারি; অন্য কোনো runnable thread-কে CPU দিতে পারো।”

Thread.yield();
Flow
Thread-1 Running
      ↓
Thread.yield()
      ↓
OS Scheduler-কে Hint
      ↓
অন্য Runnable Thread CPU পেতে পারে
      ↓
Thread-1 আবার CPU পেতে পারে
Example
public class Main {

    public static void main(String[] args) {

        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                System.out.println("T1: " + i);
                Thread.yield();
            }
        });

        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                System.out.println("T2: " + i);
                Thread.yield();
            }
        });

        t1.start();
        t2.start();
    }
}

Output নির্দিষ্ট নয়:

T1: 0
T2: 0
T1: 1
T2: 1
...

আবার এমনও হতে পারে:

T1: 0
T1: 1
T2: 0
T1: 2
...

কারণ yield() শুধু hint। Scheduler এটি মানতেই হবে এমন নয়।

yield() কী করে না?

yield():

❌ Thread-কে WAITING করে না
❌ Thread-কে BLOCKED করে না
❌ নির্দিষ্ট সময়ের জন্য pause করে না
❌ অন্য thread-কে CPU দেওয়ার guarantee দেয় না
❌ Thread-এর কাজ বন্ধ করে না

সাধারণভাবে thread RUNNABLE state-এই থাকে।

sleep() vs yield()
yield()
→ Scheduler-কে hint দেয়
→ নির্দিষ্ট সময় নেই
→ guarantee নেই
→ RUNNABLE থাকে

sleep(1000)
→ 1 second-এর জন্য sleep
→ TIMED_WAITING
→ 1 second পরে আবার RUNNABLE
Short GitHub Note

Thread.yield() current thread-এর CPU usage সাময়িকভাবে কমানোর জন্য OS Scheduler-কে একটি hint দেয় যাতে অন্য runnable thread CPU পেতে পারে। তবে অন্য thread CPU পাবে—এর কোনো guarantee নেই।
```

