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
my java code 
```
import javax.swing.*;
import java.util.LinkedList;
import java.util.Queue;
import java.util.concurrent.Callable;

public class Main {
    static private  int value =0;

    public static void main(String[] args) throws InterruptedException {


        Thread t = new Thread(() -> {
            System.out.println("task start ...........");
            try {
                Thread.sleep(10000);
            } catch (InterruptedException e) {
                System.out.println("Interrupted");
            }
            System.out.println("vale is "+value);

        });
      t.start();
     Thread.sleep(1000);

      value =10;

   t.interrupt();

        System.out.println("terminate task end ..............");



    }
}
```
## Thread.currentThread();

```

Thread.currentThread() হলো Java-এর একটি static method, যেটা বর্তমানে যে thread code execute করছে সেই Thread object-টি return করে।

সহজভাবে

Thread.currentThread();

মানে:

"এই মুহূর্তে যে thread কাজ করছে, আমাকে সেই thread-টা দাও।"

Example
public class Main {
public class Main {
    private static int value =0;
    public static void main(String[] args) throws InterruptedException {
        Thread t = new Thread(()->{
            System.out.println("This is a task start ..........");
            Thread t2 = Thread.currentThread();
            System.out.println("hash code of curren thread is "+t2.hashCode());
            try {
//                Thread.sleep(10000);
                long count =0;
                 while (!Thread.interrupted())
                {
                   count++;
                }
                System.out.println("count "+count);
            } catch (Exception e) {
               e.printStackTrace();
            }
            System.out.println("resourec / value is "+value);
            System.out.println("task end ");
        });

        t.start();
//        t.sleep(1000);
        value = 15;
//        Thread.interrupted();
   t.interrupt();


        System.out.println("Hello, World!");
    }
}

Output হতে পারে:

Thread[main,5,main]
main
1

এখানে:

Thread t = Thread.currentThread();

main() method বর্তমানে main thread দ্বারা execute হচ্ছে। তাই currentThread() → main thread-এর object return করবে।

```
### Thread.interrupted(); and t.isInterrupted()
```
দুটোই thread-এর interrupt status check করে, কিন্তু সবচেয়ে গুরুত্বপূর্ণ পার্থক্য হলো:

Thread.interrupted() → status check করে এবং status clear করে।
t.isInterrupted() → status check করে কিন্তু status clear করে না।

1. Thread.interrupted()

এটি static method।

boolean result = Thread.interrupted();

এটি current thread-এর interrupt status check করে।

যদি status true হয়:

true → return করবে
true → তারপর false করে দেবে

Example:

Thread.currentThread().interrupt();

System.out.println(Thread.interrupted()); // true
System.out.println(Thread.interrupted()); // false

কারণ প্রথমবার interrupted() status clear করে দিয়েছে।

2. t.isInterrupted()

এটি instance method।

boolean result = t.isInterrupted();

এটি t thread-এর interrupt status check করে, কিন্তু clear করে না।

Thread t = new Thread(() -> {
    while (true) {
        if (Thread.currentThread().isInterrupted()) {
            System.out.println("Interrupted!");
            break;
        }
    }
});

t.start();

t.interrupt();

System.out.println(t.isInterrupted()); // true
System.out.println(t.isInterrupted()); // true

দুইবারই true, কারণ isInterrupted() status clear করে না।

মূল পার্থক্য
Method	Type	কোন Thread check করে?	Interrupt status clear করে?
Thread.interrupted()	static	Current thread	✅ হ্যাঁ
t.isInterrupted()	instance	t thread	❌ না
সহজে মনে রাখুন
Thread.interrupted()
        ↓
"আমি কি interrupted?"
        ↓
Current Thread
        ↓
Check + Clear


t.isInterrupted()
        ↓
"t কি interrupted?"
        ↓
Thread t
        ↓
Check only

একটি গুরুত্বপূর্ণ বিষয়: interrupt() নিজে thread-কে forcefully বন্ধ করে না। এটি সাধারণত thread-এর interrupt flag/status সেট করে, আর thread-কে সেই signal handle করতে হয়।
```
