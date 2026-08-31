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
### Thread.checkAccess() Method

```
checkAccess() একটি security/access-checking method, যা current code-এর ওই Thread object-এর উপর access করার permission আছে কিনা যাচাই করত।

thread.checkAccess();
সহজভাবে

checkAccess() → এই Thread-এর উপর access করার অনুমতি আছে কি না check করে।

যদি access না থাকে, তাহলে সাধারণত:

SecurityException

হতে পারে।

Example
Thread t = new Thread(() -> {
    System.out.println("Running...");
});

t.checkAccess();   // access check
t.start();
Important

বর্তমান Java-তে Thread.checkAccess() deprecated (Java 17 থেকে), কারণ পুরোনো SecurityManager-ভিত্তিক security mechanism deprecated/removed হওয়ার পথে।

Short Note

Thread.checkAccess() → একটি Thread-এর উপর caller-এর access permission check করার জন্য ব্যবহৃত হতো। Permission না থাকলে SecurityException হতে পারে। বর্তমানে এটি deprecated।
```
