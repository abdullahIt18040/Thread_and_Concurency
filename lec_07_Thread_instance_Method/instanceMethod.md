## stop method
```
Thread.stop() Method

Thread.stop() একটি Thread-এর execution জোর করে বন্ধ করার জন্য ব্যবহৃত হতো।

thread.stop(); // ❌ Deprecated
Flow
RUNNABLE
   ↓
stop()
   ↓
TERMINATED
কেন stop() Deprecated?

stop() Thread-কে হঠাৎ করে বন্ধ করে দেয়। Thread যদি তখন কোনো shared data বা lock নিয়ে কাজ করে, তাহলে data inconsistent/corrupted হতে পারে।

Example:

Thread-1
   ↓
Shared Data update করছে
   ↓
stop()
   ↓
হঠাৎ Thread বন্ধ ❌
   ↓
Data অসম্পূর্ণ অবস্থায় থাকতে পারে

আর stop() ThreadDeath exception trigger করতে পারে, যা application-এর জন্য unpredictable behavior তৈরি করতে পারে।

Modern Alternative

Thread বন্ধ করার জন্য সাধারণত:

thread.interrupt();

ব্যবহার করা হয়।

Example:

Thread t = new Thread(() -> {
    while (!Thread.currentThread().isInterrupted()) {
        System.out.println("Running...");
    }
});

t.start();

t.interrupt();
```
