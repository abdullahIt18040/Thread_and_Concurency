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
### Daemon Thread
```
Daemon Thread কী?

Daemon Thread হলো এমন একটি background thread, যেটা main application-কে support করে।

সবচেয়ে গুরুত্বপূর্ণ কথা:

সব normal (non-daemon) thread শেষ হয়ে গেলে JVM বন্ধ হয়ে যায়। তখন daemon thread চলতে থাকলেও JVM তাকে বন্ধ করে দেয়।

🏠 সহজ উদাহরণ

ধরো একটি বাড়ি হলো তোমার Java application।

👨‍💼 Main/Normal Thread → বাড়ির মূল মানুষ
🧹 Daemon Thread → cleaning/support worker

মূল মানুষ সবাই বাড়ি ছেড়ে চলে গেলে cleaning worker বাড়িতে থাকলেও বাড়ি বন্ধ হয়ে যাবে।

একইভাবে:

Normal Thread শেষ
       ↓
JVM বন্ধ
       ↓
Daemon Thread-ও বন্ধ
সহজ Java Example
public class Main {

    public static void main(String[] args) {

        Thread daemonThread = new Thread(() -> {

            while (true) {
                System.out.println("Daemon thread running...");

                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        daemonThread.setDaemon(true);
        daemonThread.start();

        System.out.println("Main thread finished");
    }
}
এখানে কী হচ্ছে?

প্রথমে:

daemonThread.setDaemon(true);

এটা thread-টাকে Daemon Thread বানিয়েছে।

তারপর:

daemonThread.start();

thread শুরু হয়েছে।

Daemon thread বারবার print করতে চাচ্ছে:

Daemon thread running...
Daemon thread running...
Daemon thread running...

কিন্তু main() শেষ হয়ে গেলে:

Main thread finished
       ↓
main() শেষ
       ↓
কোনো Non-Daemon Thread নেই
       ↓
JVM shutdown
       ↓
Daemon Thread বন্ধ

তাই daemon thread while(true) থাকা সত্ত্বেও চিরকাল চলবে না।

🔴 এবার Non-Daemon Thread

Defaultভাবে Java thread হলো Non-Daemon।

Thread t = new Thread(() -> {

    while (true) {
        System.out.println("Normal thread running...");
    }
});

t.start();

এখানে setDaemon(true) নেই।

তাই:

Main Thread শেষ
      ↓
Normal Thread এখনও চলছে
      ↓
JVM বন্ধ হবে না
      ↓
Normal Thread চলতেই থাকবে
```
### যতক্ষণ কমপক্ষে একটি Non-Daemon thread চলছে, JVM চলবে এবং daemon thread-ও চলতে পারবে।
```
কিন্তু কোনো Non-Daemon thread না থাকলে → JVM বন্ধ → Daemon thread-ও বন্ধ।

মনে রাখার Note

Non-Daemon Thread = JVM-কে alive রাখে।
Daemon Thread = JVM alive থাকলে চলতে পারে; JVM shutdown হলে automatically শেষ হয়।
```
## my code is 
```
import jdk.jfr.StackTrace;

import javax.swing.*;
import java.util.LinkedList;
import java.util.Map;
import java.util.Objects;
import java.util.Queue;
import java.util.concurrent.Callable;

public class Main {

    public synchronized static  void main(String[] args) throws InterruptedException {
        System.out.println("main star ");
        Thread t1 = new Thread(()->{

            System.out.println("t1 star ");
            Thread d1 = new Thread(()->{
                System.out.println("daemon thread start .....");
                for(int k=0;k<10;k++)
                {
                    try {
                        System.out.println("deamon thead k is "+k);
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                }
                System.out.println("daemon thread end .....");
            });
            d1.setDaemon(true);
            d1.start();
            for(int j=0;j<10;j++)
            {
                System.out.println("t1 threa j ="+j);
                try {
                    Thread.sleep(200);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }

            System.out.println("t1 END ");
        });
        Thread t2 = new Thread(()->{
            System.out.println("t2 star ");
            for(int i=0;i<10;i++)
            {
                System.out.println("t2 threa j ="+i);
                try {
                    Thread.sleep(300);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            System.out.println("t2 END ");
        });
        t1.start();
        t2.start();
        System.out.println("main end ");

         }

}
```
