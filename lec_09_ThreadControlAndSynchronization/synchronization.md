<img width="1222" height="706" alt="image" src="https://github.com/user-attachments/assets/fcc1bc8c-5602-4116-a117-8f0242aa9c1a" />
## Dead lock
<img width="1123" height="773" alt="image" src="https://github.com/user-attachments/assets/e1dddc86-f35c-45fb-84b1-03ee10497cab" />
<img width="1184" height="848" alt="image" src="https://github.com/user-attachments/assets/f9254fcf-448a-461f-8f47-202304e63067" />

## Dead lock occure and my codeis 

```

import jdk.jfr.StackTrace;

import javax.swing.*;
import java.math.BigInteger;
import java.util.LinkedList;
import java.util.Map;
import java.util.Objects;
import java.util.Queue;
import java.util.concurrent.Callable;

class ServiceA{
    private int resoureA =100;
    public synchronized void proccess(ServiceB serviceB)
    {
        System.out.println(" process START by  "+Thread.currentThread().getName()+"inside serviceA");
        System.out.println("resurecr b need insedes A "+serviceB.getResoureB());
        System.out.println(" process end by  "+Thread.currentThread().getName()+"inside serviceA");
    }


   public synchronized int getResoureA()
    {
        return resoureA;
    }


}
class ServiceB{
    private int resoureB =200;
    public synchronized void proccess(ServiceA serviceA)
    {
        System.out.println(" process START by  "+Thread.currentThread().getName()+"inside serviceB");
        System.out.println("resurecr A need insedes B "+serviceA.getResoureA());
        System.out.println(" process end by  "+Thread.currentThread().getName()+"inside serviceB");
    }


    public synchronized int getResoureB()
    {
        return resoureB;
    }
}

public class Main {

    public  static  void main(String[] args) throws InterruptedException {
    ServiceA serviceA = new ServiceA();
    ServiceB serviceB  = new ServiceB();
        Thread T1 = new Thread(()->{
        serviceA.proccess(serviceB);


        });
        Thread T2 = new Thread(()->{
           serviceB.proccess(serviceA);

        });
    T1.start();
    T2.start();


        System.out.println("main task end.....");

}

}
```
### Deadlock — Practical Example and how to find it . 
```

বাস্তব জীবনে ধরুন দুইজন মানুষ আছে এবং দুইটি resource আছে:

Resource A = Pen
Resource B = Notebook
Situation

Person-1 আগে Pen নিল:

Person-1 → Pen 🔒

এখন Person-1 Notebook চায়:

Person-1 → Notebook চাইছে

কিন্তু একই সময়ে Person-2 আগে Notebook নিয়ে ফেলেছে:

Person-2 → Notebook 🔒

এখন Person-2 Pen চায়:

Person-2 → Pen চাইছে

ফলে:

Person-1
   ↓
Pen 🔒
   ↓
Notebook-এর জন্য অপেক্ষা
   ↑
Notebook 🔒
   ↑
Person-2
   ↓
Pen-এর জন্য অপেক্ষা

দুজনই অপেক্ষা করছে, কিন্তু কেউ নিজের resource ছাড়ছে না।

এটাই Deadlock।

Java-তে একই Example
public class Main {

    static Object pen = new Object();
    static Object notebook = new Object();

    public static void main(String[] args) {

        Thread person1 = new Thread(() -> {

            synchronized (pen) {
                System.out.println("Person-1: Pen acquired");

                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }

                System.out.println("Person-1: Waiting for Notebook");

                synchronized (notebook) {
                    System.out.println("Person-1: Notebook acquired");
                }
            }
        });

        Thread person2 = new Thread(() -> {

            synchronized (notebook) {
                System.out.println("Person-2: Notebook acquired");

                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }

                System.out.println("Person-2: Waiting for Pen");

                synchronized (pen) {
                    System.out.println("Person-2: Pen acquired");
                }
            }
        });

        person1.start();
        person2.start();
    }
}
কী হচ্ছে?
Person-1 → pen lock 🔒 → notebook lock চাই → BLOCKED

Person-2 → notebook lock 🔒 → pen lock চাই → BLOCKED

ফলে:

Person-1 → অপেক্ষা করছে Person-2-এর notebook-এর জন্য
Person-2 → অপেক্ষা করছে Person-1-এর pen-এর জন্য
                    ↓
                 DEADLOCK
jstack দিয়ে দেখতে পারেন

Program stuck হওয়ার পর:

jps -l

PID বের করুন, তারপর:

jstack <PID>

তখন Java thread dump-এ কোন thread কোন lock-এর জন্য অপেক্ষা করছে তা দেখতে পারবেন।

মনে রাখার সহজ rule

একজনের কাছে Lock-A, অন্যজনের কাছে Lock-B; দুজনই একে অপরের Lock-এর জন্য অপেক্ষা করলে → Deadlock।

T1 → Lock A → wants Lock B
T2 → Lock B → wants Lock A
                ↓
```

### starvation and it cause
```
Starvation কী?

Starvation হলো এমন একটি অবস্থা যেখানে কোনো Thread অনেকক্ষণ CPU বা required resource পাওয়ার সুযোগ না পেয়ে অপেক্ষা করতে থাকে, কারণ অন্য Thread-গুলো বারবার resource/CPU পেয়ে যাচ্ছে।

সহজভাবে:

একটি Thread কাজ করার সুযোগ পাচ্ছে না, অন্য Thread বারবার সুযোগ পাচ্ছে → Starvation

Practical Example

ধরো ৩টা Thread আছে:

Thread-1 → CPU পেয়ে কাজ করছে
Thread-2 → CPU পেয়ে কাজ করছে
Thread-3 → CPU পাওয়ার জন্য অপেক্ষা করছে

যদি Thread-1 এবং Thread-2 বারবার CPU পায় এবং Thread-3 দীর্ঘসময় CPU না পায়:

T1 → Run → Run → Run → Run
T2 → Run → Run → Run
T3 → Wait → Wait → Wait → Wait
                    ↑
                Starvation
```
## Starvation-এর কারণ
```

Unfair Thread Scheduling
Scheduler যদি Thread-গুলোর মধ্যে CPU fairly distribute না করে, তাহলে কোনো Thread বারবার CPU না পেয়ে অপেক্ষা করতে পারে।

T1 → CPU → CPU → CPU → CPU
T2 → Wait → Wait → Wait → Wait
                 ↑
             Starvation
High Thread Priority
High-priority Thread বেশি CPU opportunity পেলে low-priority Thread দীর্ঘসময় অপেক্ষা করতে পারে।
Unfair Lock
Lock fair না হলে একই Thread বারবার lock পেতে পারে এবং অন্য Thread অপেক্ষা করতে পারে।
Long-running Thread
কোনো Thread দীর্ঘসময় CPU বা resource ব্যবহার করলে অন্য Thread কম সুযোগ পেতে পারে।

Short note:

Starvation = কোনো Thread দীর্ঘসময় CPU/resource না পেয়ে অপেক্ষা করা।
Main causes: Unfair thread scheduling, high priority, unfair lock, long-running
```
### Livelock কী?
```

Livelock হলো এমন একটি অবস্থা যেখানে একাধিক Thread blocked নয়, তারা continuously কাজ করছে বা state পরিবর্তন করছে, কিন্তু কাজের কোনো progress হচ্ছে না।

সহজভাবে:

🔴 Deadlock: Thread-গুলো আটকে আছে এবং অপেক্ষা করছে।
🟡 Livelock: Thread-গুলো কাজ করছে, কিন্তু কোনো কাজ শেষ করতে পারছে না।

Practical Example: দুইজন মানুষ দরজায়

ধরো, একজন আরেকজনের সামনে দিয়ে দরজা পার হতে যাচ্ছে।

Person A → ডানে সরে
Person B → ডানে সরে

Person A → বামে সরে
Person B → বামে সরে

Person A → ডানে সরে
Person B → ডানে সরে

দুজনেই একে অপরকে সুযোগ দেওয়ার চেষ্টা করছে, কিন্তু দুজন একই direction-এ move করছে, তাই কেউ দরজা পার হতে পারছে না।

এটাই Livelock।

Java Example
while (true) {

    if (lock.tryLock()) {
        try {
            // কাজ করার চেষ্টা
        } finally {
            lock.unlock();
        }
    } else {
        // অন্য Thread-কে সুযোগ দেওয়ার জন্য lock ছেড়ে দিল
        Thread.yield();
    }
}

ধরো দুই Thread-ই একইভাবে কাজ করছে:

T1 → Lock নিতে চেষ্টা → conflict → ছেড়ে দিল
T2 → Lock নিতে চেষ্টা → conflict → ছেড়ে দিল

T1 → আবার চেষ্টা → ছেড়ে দিল
T2 → আবার চেষ্টা → ছেড়ে দিল

       ↓
  No progress
       ↓
    Livelock
```
### spoon spouse problelm livelock
## my code is
```
import jdk.jfr.StackTrace;

import javax.swing.*;
import java.math.BigInteger;
import java.util.LinkedList;
import java.util.Map;
import java.util.Objects;
import java.util.Queue;
import java.util.concurrent.Callable;

class Spoon{

    private Spouse owner;
    public Spoon(Spouse owner)
    {
        this.owner= owner;
    }
    public synchronized void eaten()
    {
        System.out.println("eating.........."+owner);

    }

    public Spouse getOwner() {
        return owner;
    }

    public synchronized void setOwner(Spouse owner) {
        this.owner = owner;
    }
}

class Spouse{

    private  String name;
    private boolean hungry= true;

    public Spouse(String name){
       this.name=name;
    }
    public void dinnerMyDear(Spouse spouse,Spoon spoon)
    {
        while (true)
        {
            if(spoon.getOwner()!=this)
            {
                try {
                    Thread.sleep(1000);
                    continue;
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            if (spouse.hungry)
            {
                spoon.setOwner(spouse);
                System.out.printf("my dear %s eat first  please ,status :: %s \n",spouse.name,Thread.currentThread()
                        .getState().name());
                continue;
            }
            spoon.eaten();
            hungry= false;

            break;
        }

    }

    @Override
    public String toString() {
        return "Spouse{" +
                "name='" + name + '\'' +
                '}';
    }
}


public class Main {

    public  static  void main(String[] args) throws InterruptedException {

Spouse wife = new Spouse("Hashiyara Khatun.....");
Spouse husband =new Spouse("ABDULLAH ");
Spoon husbandspoon = new Spoon(husband);
Thread T1= new Thread(()->
{
    wife.dinnerMyDear(husband,husbandspoon);
});
Thread T2 = new Thread(()->{
    husband.dinnerMyDear(wife,husbandspoon);
});

T2.start();
T1.start();

        System.out.println("main task end.....");

}

}
```
