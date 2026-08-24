### Java-এর Thread.State মোট ৬টি state। সহজভাবে বুঝলে Thread-এর lifecycle এভাবে:
```
NEW
 ↓
RUNNABLE
 ↓
 ├── BLOCKED
 ├── WAITING
 └── TIMED_WAITING
 ↓
RUNNABLE
 ↓
TERMINATED
1. NEW

Thread object তৈরি হয়েছে, কিন্তু start() এখনো call করা হয়নি।

Thread t = new Thread(task);
Thread Object
     ↓
    NEW

start() call করলে:

t.start();

→ NEW থেকে RUNNABLE

2. RUNNABLE

Thread run করার জন্য ready অথবা JVM/CPU-তে বর্তমানে executing।

RUNNABLE
   ↓
OS Scheduler
   ↓
CPU পেলে → Execute
CPU না পেলে → অপেক্ষা

খুব গুরুত্বপূর্ণ:

Java-তে RUNNABLE মানে শুধু "CPU-তে এই মুহূর্তে চলছে" নয়। Thread CPU-এর জন্য ready/eligible থাকলেও RUNNABLE state-এ থাকতে পারে।


Runing

OS Scheduler CPU দিলে → Thread Running অবস্থায় থাকে।
CPU-এর জন্য অপেক্ষা করলে → Java-এর RUNNABLE state-এই থাকে।

3. BLOCKED

Thread কোনো synchronized monitor lock পাওয়ার জন্য অপেক্ষা করছে।

Example:

synchronized (obj) {
    // critical section
}

যদি Thread-1 lock ধরে রাখে:

Thread-1 → Lock ধরে আছে

Thread-2 → Lock চাইছে
              ↓
           BLOCKED

Lock free হলে:

BLOCKED
   ↓
RUNNABLE
4. WAITING

Thread অন্য একটি Thread-এর কোনো action-এর জন্য indefinitely অপেক্ষা করছে।

যেমন:

thread.join();

অথবা:

obj.wait();

Example:

Thread-1
   ↓
join(Thread-2)
   ↓
WAITING
   ↓
Thread-2 শেষ
   ↓
RUNNABLE

অর্থাৎ কতক্ষণ অপেক্ষা করবে সেটা নির্দিষ্ট নয়।

5. TIMED_WAITING

Thread নির্দিষ্ট সময়ের জন্য অপেক্ষা করছে।

Example:

Thread.sleep(5000);
RUNNABLE
   ↓
sleep(5 seconds)
   ↓
TIMED_WAITING
   ↓
5 seconds শেষ
   ↓
RUNNABLE

আরও examples:

Thread.sleep(...)
Object.wait(timeout)
Thread.join(timeout)
LockSupport.parkNanos(...)
WAITING vs TIMED_WAITING
WAITING
→ কতক্ষণ অপেক্ষা করবে নির্দিষ্ট নেই

TIMED_WAITING
→ নির্দিষ্ট সময় পর্যন্ত অপেক্ষা করবে
6. TERMINATED

Thread-এর run() method শেষ হয়ে গেছে।

public void run() {
    System.out.println("Task completed");
}

run() শেষ:

RUNNABLE
   ↓
run() completed
   ↓
TERMINATED

একবার TERMINATED হলে একই Thread object আবার start() করা যায় না।

Complete Example
Thread t = new Thread(() -> {

    try {
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

});

Initially:

NEW

তারপর:

t.start();
NEW
 ↓
RUNNABLE
 ↓
TIMED_WAITING    ← sleep(3000)
 ↓
RUNNABLE
 ↓
TERMINATED
সবগুলো একসাথে
State	সহজ অর্থ
NEW	Thread তৈরি হয়েছে, start হয়নি
RUNNABLE	চলার জন্য ready / executing
BLOCKED	synchronized lock-এর জন্য অপেক্ষা
WAITING	অন্য Thread-এর action-এর জন্য অপেক্ষা
TIMED_WAITING	নির্দিষ্ট সময়ের জন্য অপেক্ষা
TERMINATED	Thread-এর কাজ শেষ
```
### jstack Command — Java Thread Dump
### What is jstack?
```
jstack হলো Java JDK-এর একটি command-line tool, যা running Java application-এর Thread Dump বের করতে ব্যবহার করা হয়।

কেন jstack ব্যবহার করি?

jstack ব্যবহার করে আমরা দেখতে পারি:

কোন কোন Thread চলছে
Thread-এর current state
Thread কোন method/code line-এ আছে
Thread BLOCKED, WAITING, TIMED_WAITING নাকি RUNNABLE
Deadlock বা thread blocking-এর সমস্যা খুঁজে বের করতে
Thread-এর Stack Trace / Stack Frame দেখতে
Application-এর concurrency/threading সমস্যা debug করতে
Basic Flow
Running Java Application
        ↓
       PID
        ↓
    jstack PID
        ↓
    Thread Dump
        ↓
Thread Name + State + Stack Trace
Windows Example

প্রথমে Java application-এর PID বের করি:

jps -l

Output:

15548 Main

তারপর:

jstack 15548

File-এ save করতে:

jstack 15548 > thread-dump.txt
Example Thread Dump
"Worker-1"
   java.lang.Thread.State: RUNNABLE
        at Main.lambda$main$0(Main.java:10)

"main"
   java.lang.Thread.State: TIMED_WAITING
        at java.lang.Thread.sleep(Native Method)

এখানে jstack আমাদের দেখাচ্ছে:

Worker-1
   ↓
RUNNABLE
   ↓
Main.java:10
   ↓
Current Stack Trace
Short Note

jps → Java Process-এর PID বের করে
jstack → সেই PID-এর Thread Dump বের করে

jps -l
   ↓
PID = 15548
   ↓
jstack 15548
   ↓
Thread Dump

Main purpose: Java application-এর Thread behavior, state, stack trace, blocking এবং deadlock debug করা।
```
