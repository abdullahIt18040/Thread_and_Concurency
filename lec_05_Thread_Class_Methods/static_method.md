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
