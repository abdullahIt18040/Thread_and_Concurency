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
### Deadlock — Practical Example
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



