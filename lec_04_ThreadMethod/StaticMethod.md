## All Static method.  Thread priority mustbe with in (1 to 10) 

 THREAD MAX_PRIORITY :   
```
Thread.MAX_PRIORITY
System.out.println(Thread.MAX_PRIORITY);

Output:

10

Java-তে Thread priority-এর range:

MIN_PRIORITY = 1
NORM_PRIORITY = 5
MAX_PRIORITY = 10
তিনটি গুরুত্বপূর্ণ Priority Constant
Constant	Value	অর্থ
Thread.MIN_PRIORITY	1	সর্বনিম্ন priority
Thread.NORM_PRIORITY	5	Default priority
Thread.MAX_PRIORITY	10	সর্বোচ্চ priority

উদাহরণ:

Thread t = new Thread();

t.setPriority(Thread.MAX_PRIORITY);

System.out.println(t.getPriority());

Output:

10
গুরুত্বপূর্ণ

MAX_PRIORITY CPU পাওয়ার guarantee দেয় না।

MAX_PRIORITY = 10
        ↓
Scheduler-এর কাছে higher-priority hint
        ↓
কিন্তু অবশ্যই আগে execute হবে — এমন guarantee নেই

তাই মনে রাখবেন:

Thread.MAX_PRIORITY = priority-এর সর্বোচ্চ মান (10), কিন্তু এটি static method নয়; static final constant।

```
## Thread Priority Inheritance — 
```
নতুন Thread তৈরি হলে, সে সাধারণত creating thread-এর priority inherit করে।
Creating thread-কে এখানে informalভাবে parent thread বলা যায়।
Default priority: NORM_PRIORITY = 5
Maximum: MAX_PRIORITY = 10
Minimum: MIN_PRIORITY = 1
Thread.currentThread().setPriority(8);

Thread child = new Thread(() -> {
    System.out.println(Thread.currentThread().getPriority());
});

Flow:

Parent Thread (Priority 8)
          ↓ creates
Child Thread (Priority 8)

New Thread-এর initial priority = creating Thread-এর priority।
```
##  Thread.sleep() && Thread.interrapte();
```

sleep() current thread-কে নির্দিষ্ট সময়ের জন্য pause করে।

Thread.sleep(2000);

মানে current thread প্রায় 2 seconds sleep করবে।

Thread state
RUNNABLE
   ↓
Thread.sleep()
   ↓
TIMED_WAITING
   ↓
সময় শেষ
   ↓
RUNNABLE

Example:

public class Main {
    public static void main(String[] args) throws InterruptedException {

        System.out.println("Start");

        Thread.sleep(2000);

        System.out.println("End");
    }
}

Output:

Start
   ↓
2 seconds wait
   ↓
End
Important

sleep():

Current thread-কে pause করে
TIMED_WAITING state-এ যায়
নির্দিষ্ট সময় পরে আবার runnable হয়
কোনো lock release করে না
এটি static method
Thread.sleep(1000);
2. Thread.interrupt()

interrupt() অন্য একটি thread-কে interrupt করার request পাঠায়।

thread.interrupt();

এটা সরাসরি thread-কে kill করে না।

Example
Thread t = new Thread(() -> {

    try {
        Thread.sleep(10000);
    } catch (InterruptedException e) {
        System.out.println("Thread interrupted!");
    }

});

t.start();

t.interrupt();

এখানে:

Thread t
   ↓
sleep(10 seconds)
   ↓
TIMED_WAITING
   ↓
main → t.interrupt()
   ↓
InterruptedException
   ↓
catch block

অর্থাৎ sleep() অবস্থায় থাকা thread-কে interrupt করলে তার sleep আগেই শেষ হয়ে যায় এবং InterruptedException হয়।

sleep() + interrupt() একসাথে
Thread worker = new Thread(() -> {

    try {
        System.out.println("Sleeping...");
        Thread.sleep(10000);

    } catch (InterruptedException e) {
        System.out.println("Interrupted!");
    }
});

worker.start();

Thread.sleep(2000);

worker.interrupt();

Flow:

Worker Thread
     │
     ├── sleep(10 sec)
     │
     │    TIMED_WAITING
     │
Main Thread
     │
     └── worker.interrupt()
              ↓
       InterruptedException
              ↓
          catch block
সবচেয়ে গুরুত্বপূর্ণ পার্থক্য
sleep()	interrupt()
Current thread-কে sleep করায়	অন্য thread-কে interrupt request পাঠায়
TIMED_WAITING state হয়	Thread-এর interrupt status পরিবর্তন করে
সময় শেষ হলে জেগে ওঠে	sleep()/wait()/join()-এ থাকলে InterruptedException হতে পারে
Static method	Instance method
Thread.sleep()	thread.interrupt()
মনে রাখার shortcut

sleep() = "আমি কিছুক্ষণ অপেক্ষা করছি"

interrupt() = "তুমি যদি wait/sleep করো, তাহলে তোমার অপেক্ষা ভেঙে দাও"

আর একটা গুরুত্বপূর্ণ বিষয়: interrupt() thread-কে forcefully stop করে না। Thread কীভাবে interrupt handle করবে, সেটা thread-এর code-এর ওপর নির্ভর করে।
```
