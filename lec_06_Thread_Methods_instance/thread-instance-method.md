## instance all MEthod with description 
### code-এ start() এবং run() কীভাবে কাজ করে সেটা সহজভাবে দেখি।
'''


আপনার Code
Thread t = new Thread(() -> {
    System.out.println("hello the thread occured by " 
            + Thread.currentThread().getName());
});

t.setName("task");
t.start();
কী হচ্ছে?
1. Thread Object তৈরি
Thread t = new Thread(() -> {
    // task
});

এখানে t নামে একটি Thread object তৈরি হয়েছে।

এর initial state:

NEW

Lambda:

() -> {
    System.out.println(...);
}

এটাই Thread-এর Runnable task, যা পরে run()-এর মাধ্যমে execute হবে।

2. Thread-এর নাম set
t.setName("task");

এখন:

Thread Name = task
State       = NEW
3. start() call
t.start();

এখানে নতুন thread-এর execution শুরু করার request হয়।

Conceptual flow:

t.start()
   ↓
start0() / JVM Native Layer
   ↓
OS Thread Execution
   ↓
RUNNABLE
   ↓
OS Scheduler
   ↓
CPU
   ↓
run()
   ↓
Lambda Task Execute

তখন output হবে:

hello the thread occured by task

কারণ:

Thread.currentThread().getName()

বর্তমানে execute করা thread-এর নাম "task" return করে।

start() vs run()
start() করলে
t.start();
main Thread
     │
     └──────→ task Thread
                  ↓
                run()
                  ↓
              Lambda

নতুন thread তৈরি/শুরু হয়।

run() সরাসরি call করলে
t.run();
main Thread
     ↓
   run()
     ↓
  Lambda

নতুন thread তৈরি হয় না। main thread-ই task execute করে।

তখন output হবে:

hello the thread occured by main
মনে রাখুন

start() → নতুন Thread-এর execution শুরু করে → run() execute হয়।

run() → শুধু task execute করে → নতুন Thread তৈরি করে না।
'''
### join() Method — আপনার Code দিয়ে ব্যাখ্

```
আপনার code:

Thread t = new Thread(() -> {
    System.out.println(
        "simple task " + Thread.currentThread().getName()
    );

    for (int i : arr) {
        sum += i;
    }
});

t.start();
t.join();

System.out.println("sum is " + sum);
join() কী করে?

join() মানে:

Current thread অন্য একটি thread-এর কাজ শেষ হওয়া পর্যন্ত অপেক্ষা করবে।

join() Method

join() ব্যবহার করলে calling/current thread অন্য Thread-এর কাজ শেষ হওয়া পর্যন্ত অপেক্ষা করে।

t.start();
t.join();
we can set time in melisecoud and nanosecound
join(3,20) = অন্য Thread শেষ হওয়া পর্যন্ত অপেক্ষা করো, তবে সর্বোচ্চ 3 ms + 20 ns।


```
### Deadlock occure 
 thread1 acquire lock then goto suspended and go blocked state , main thread need lock but thread1 alread acquired  lock, now thread1 said please call resume but main call please lock release ,as a result dead lock occure 
```
import jdk.jfr.StackTrace;

import javax.swing.*;
import java.util.LinkedList;
import java.util.Map;
import java.util.Objects;
import java.util.Queue;
import java.util.concurrent.Callable;

public class Main {

    static private int value = 0;
    private static int[] arr = {1,2,34,4,5,5,5};
    private static int sum=0;

    public synchronized static  void main(String[] args) throws InterruptedException {
        Thread thread1 = new Thread(()->{

            System.out.println("task start ..........");
            for(int i=0;i<100;i++)
            {
                System.out.println("i is : "+i);

            }

            System.out.println("task end ..........");

        });

        thread1.start();
 Thread.sleep(8);
      thread1.suspend();
        System.out.println("i am going to sleep  state");
      Thread.sleep(5000);
        thread1.resume();




    }
    static void sleep(int millsec)
    {
        try {
            Thread.sleep(millsec);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```
