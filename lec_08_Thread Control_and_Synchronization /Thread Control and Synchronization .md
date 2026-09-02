### Shared Resource কী? — সহজ বাংলায়

Shared Resource হলো এমন কোনো data, object, variable বা resource, যেটা একাধিক thread একসাথে access বা ব্যবহার করতে পারে।
```
🏠 সহজ উদাহরণ

ধরুন, ২ জন মানুষ একটি shared bank account ব্যবহার করছে।

Thread 1 ──┐
           ├──→ Bank Account
Thread 2 ──┘

Bank account-টি হলো Shared Resource।

যদি দুজন একই সময়ে টাকা withdraw করে, তাহলে সমস্যা হতে পারে।

Java Example
class BankAccount {
    int balance = 1000;

    void withdraw(int amount) {
        balance = balance - amount;
    }
}

এখন:

BankAccount account = new BankAccount();

Thread t1 = new Thread(() -> {
    account.withdraw(700);
});

Thread t2 = new Thread(() -> {
    account.withdraw(500);
});

এখানে:

t1 ──→ account
          ↑
t2 ──→ account

account object-টি দুই thread ব্যবহার করছে।

তাই account হলো Shared Resource।

⚠️ সমস্যা: Race Condition

দুই thread যদি একই সময়ে:

balance = balance - amount;

execute করে, তাহলে তারা একই পুরনো balance value পড়তে পারে।

যেমন:

Initial balance = 1000

Thread 1 reads → 1000
Thread 2 reads → 1000

Thread 1 → 1000 - 700 = 300
Thread 2 → 1000 - 500 = 500

Final balance → 500 ❌

কিন্তু আসলে দুইজনকে টাকা দেওয়ার মতো পর্যাপ্ত balance-ই ছিল না।

🔒 কীভাবে Shared Resource protect করব?

synchronized ব্যবহার করতে পারি:

synchronized void withdraw(int amount) {
    balance = balance - amount;
}

তখন একই সময়ে একজন thread-ই method-টি execute করতে পারবে।

Thread 1 → Lock → withdraw() → Unlock
                         ↓
Thread 2 → Wait ─────────┘
          তারপর execute
⭐ Short Note

Shared Resource = যে resource একাধিক thread access/use করে।

Examples:

Shared variable
Shared object
Database connection/resource
File
Queue
Cache
Bank account balance

Shared Resource + Multiple Threads → Race Condition হতে পারে → Synchronization দিয়ে protect করা হয়।
```
<img width="907" height="555" alt="image" src="https://github.com/user-attachments/assets/dc7d1527-4148-4a02-ab51-ddf0d61ef195" />

### Race Condition কী?

```
Race Condition হলো এমন একটি সমস্যা যেখানে একাধিক Thread একই Shared Resource একই সময়ে access/update করে, ফলে unpredictable বা incorrect or inconsintency  result হওয়া।

সহজ ভাষায়:

একই data নিয়ে একাধিক thread “কে আগে কাজ করবে” এই race করলে ভুল result হতে পারে।

🏦 সহজ উদাহরণ

ধরি:

int balance = 1000;

দুইটা thread একই account থেকে টাকা তুলছে:

Thread 1 → withdraw(700)
Thread 2 → withdraw(500)

দুই thread একই সময়ে balance পড়ল:

Initial balance = 1000

Thread 1 → reads 1000
Thread 2 → reads 1000

তারপর:

Thread 1 → 1000 - 700 = 300
Thread 2 → 1000 - 500 = 500

শেষে balance হতে পারে:

500 ❌

কিন্তু logically:

1000 - 700 - 500 = -200

অর্থাৎ দুই thread একই পুরনো value ব্যবহার করেছে।

💻 Java Example
class Counter {

    int count = 0;

    void increment() {
        count++;
    }
}

দুই thread:

Counter counter = new Counter();

Thread t1 = new Thread(() -> {
    for (int i = 0; i < 1000; i++) {
        counter.increment();
    }
});

Thread t2 = new Thread(() -> {
    for (int i = 0; i < 1000; i++) {
        counter.increment();
    }
});

t1.start();
t2.start();

আমরা আশা করি:

1000 + 1000 = 2000

কিন্তু result হতে পারে:

1987
1995
1978
...

কারণ:

count++;

একটি single atomic operation নয়।

এটা roughly:

1. count read
2. count + 1
3. count write

দুই thread একই সময়ে read করলে update হারিয়ে যেতে পারে।

🔒 Race Condition কীভাবে বন্ধ করব?

synchronized ব্যবহার করতে পারি:

synchronized void increment() {
    count++;
}

তখন:

Thread 1 → Lock → count++ → Unlock
                         ↓
Thread 2 → Wait ─────────┘
          ↓
       count++ 

এক সময়ে একটি thread shared resource modify করবে।

⭐ Short Note

Race Condition = Multiple threads একই shared resource একই সময়ে access/update করার কারণে unpredictable বা incorrect result হওয়া।

Multiple Threads
       ↓
Shared Resource
       ↓
Concurrent Access
       ↓
Race Condition
       ↓
Synchronization / Lock
       ↓
Safe Result
```
### Thread Safety কী?
```

Thread Safety মানে হলো—একাধিক thread একই shared resource একসাথে ব্যবহার করলেও যেন data ভুল বা inconsistent না হয়।

সহজ ভাষায়:

Multiple thread একসাথে কাজ করলেও যদি shared data ঠিক থাকে, তাহলে সেটি Thread-Safe।

🔴 Thread-Safe নয়
class Counter {
    int count = 0;

    void increment() {
        count++;
    }
}

যদি ২টা thread একই সময়ে count++ করে, তাহলে race condition হতে পারে এবং expected result নাও পাওয়া যেতে পারে।

🟢 Thread-Safe
class Counter {
    int count = 0;

    synchronized void increment() {
        count++;
    }
}

এখানে synchronized ব্যবহার করায় এক সময়ে একটি thread increment() execute করতে পারে।

Thread 1 → 🔒 Lock → count++ → Unlock
                              ↓
Thread 2 → Wait ──────────────┘
           ↓
        🔒 Lock → count++

তাই shared count নিরাপদে update হয়।

🔗 সম্পর্কটা মনে রাখুন
Multiple Threads
       ↓
Shared Resource
       ↓
Concurrent Access
       ↓
Race Condition হতে পারে
       ↓
Thread Safety দরকার
       ↓
synchronized / Lock / Atomic / Concurrent Collection
⭐ এক লাইনের Note

Thread Safety = Multiple threads একই resource নিয়ে কাজ করলেও data consistency এবং correctness বজায় থাকা।
```
### Synchronized Method কী?
```

Java-তে synchronized method এমন একটি method যেখানে একই সময়ে একই object-এর জন্য মাত্র একটি thread method-এর ভিতরের code execute করতে পারে।

সহজভাবে:

একজন Thread ঢুকলে অন্য Thread-কে অপেক্ষা করতে হয়।

🔴 Without synchronized
class Counter {
    int count = 0;

    void increment() {
        count++;
    }
}

ধরি:

Thread 1 ──→ increment()
Thread 2 ──→ increment()

দুই thread একই সময়ে count++ করতে পারে।

ফলে Race Condition হতে পারে।

🟢 With synchronized
class Counter {
    int count = 0;

    synchronized void increment() {
        count++;
    }
}

এখন:

Thread 1
   ↓
🔒 Lock নেয়
   ↓
increment()
   ↓
🔓 Lock release
   ↓
Thread 2
   ↓
🔒 Lock নেয়
   ↓
increment()

অর্থাৎ একই object-এর একই synchronized method-এ একই সময়ে একজন thread ঢুকতে পারে।

🧠 কোন Lock ব্যবহার হয়?

যদি method হয়:

synchronized void increment() {
    count++;
}

এটি instance synchronized method।

তাহলে lock নেওয়া হয় object-এর উপর।

Counter counter = new Counter();

এখানে:

counter object
      ↓
   🔒 Lock
      ↓
increment()
⚠️ দুইটি Object হলে?
Counter c1 = new Counter();
Counter c2 = new Counter();

এদের lock আলাদা:

c1 → 🔒 Lock 1
c2 → 🔒 Lock 2

তাই c1-এর synchronized method চলার সময় c2-এর synchronized method চলতে পারে।

🔵 Static Synchronized Method

যদি লিখি:

static synchronized void test() {
    // code
}

তাহলে object-এর lock নয়, Class-এর lock ব্যবহার হয়।

static synchronized method
          ↓
     Class Lock
          ↓
     🔒 Main.class
⭐ Short Note

Synchronized method = একই lock-এর মাধ্যমে এক সময়ে শুধু একটি thread method execute করতে পারে।

Thread 1 → 🔒 → Method → 🔓
Thread 2 → Wait → 🔒 → Method → 🔓
```
### Object Lock vs Class Lock বুঝি।
```
1️⃣ synchronized Instance Method → Object Lock

Code:

class Test {

    synchronized void test() {
        System.out.println("Test method");
    }
}

এখানে test() হলো instance method।

ধরি আমরা ২টা object বানালাম:

Test obj1 = new Test();
Test obj2 = new Test();

প্রতিটি object-এর নিজস্ব lock আছে:

obj1 → 🔒 Lock 1

obj2 → 🔒 Lock 2

তাই:

Thread t1 = new Thread(() -> obj1.test());
Thread t2 = new Thread(() -> obj2.test());

t1.start();
t2.start();

দুই thread একই সময়ে test() execute করতে পারবে।

Thread 1              Thread 2
   ↓                     ↓
obj1.test()           obj2.test()
   ↓                     ↓
🔒 Lock 1              🔒 Lock 2
   ↓                     ↓
 Execute               Execute
কিন্তু একই object হলে
Test obj = new Test();

Thread t1 = new Thread(() -> obj.test());
Thread t2 = new Thread(() -> obj.test());

এখন দুটো thread-এর target একই object:

Thread 1 ──→ obj → 🔒
Thread 2 ──→ obj → অপেক্ষা

তাই এক সময়ে একজনই execute করবে।

2️⃣ static synchronized → Class Lock

এখন:

class Test {

    static synchronized void test() {
        System.out.println("Test method");
    }
}

এখানে method হলো static।

তাই object-এর lock ব্যবহার হবে না।

Class-এর lock ব্যবহার হবে।

Test.class
   ↓
 🔒 Class Lock

ধরি:

Test obj1 = new Test();
Test obj2 = new Test();

এমনকি object দুইটি আলাদা হলেও:

obj1 ──┐
       ├──→ Test.class → 🔒
obj2 ──┘

দুই thread যদি:

obj1.test();
obj2.test();

call করে, তবুও একই Test.class lock ব্যবহার করবে।

Thread 1              Thread 2
   ↓                     ↓
obj1.test()           obj2.test()
   ↓                     ↓
   └────→ Test.class ←───┘
              🔒
              ↓
        একজন execute করবে
        অন্যজন wait করবে
🧠 সবচেয়ে সহজভাবে মনে রাখুন
Instance synchronized
synchronized void test()

মানে:

"আমার Object-এর lock নাও।"

obj1 → 🔒
obj2 → 🔒

আলাদা object → আলাদা lock → একই সময়ে কাজ করতে পারে।

Static synchronized
static synchronized void test()

মানে:

"আমার Class-এর lock নাও।"

Test.class → 🔒

সব object একই class-এর → একই class lock → এক সময়ে একজন।

⭐ One-line Note
synchronized method
      ↓
Object Lock

static synchronized method
      ↓
Class Lock
      ↓
ClassName.class

Shortcut:
👉 Instance = Object lock
👉 Static = Class lock
```
### Intrinsic Lock কী?
```
Java-তে Intrinsic Lock হলো JVM-এর দেওয়া built-in lock, যা প্রতিটি Java object-এর সাথে automatically থাকে।

সহজভাবে:

প্রতিটি object-এর সাথে একটি invisible lock থাকে। synchronized সেই lock ব্যবহার করে।

1️⃣ Instance synchronized হলে
class Test {

    synchronized void test() {
        System.out.println("Hello");
    }
}

ধরি:

Test obj = new Test();

তাহলে conceptually:

obj
┌───────────────┐
│ Object        │
│               │
│ 🔒 Intrinsic  │
│    Lock       │
└───────────────┘

যখন Thread 1 obj.test() call করে:

Thread 1
   ↓
obj.test()
   ↓
🔒 obj-এর Intrinsic Lock
   ↓
method execute
   ↓
🔓 Lock release

Thread 2 একই সময়ে obj.test() করতে চাইলে:

Thread 2
   ↓
obj.test()
   ↓
🔒 Lock already taken
   ↓
WAIT
2️⃣ দুইটি Object হলে
Test obj1 = new Test();
Test obj2 = new Test();

প্রতিটি object-এর আলাদা intrinsic lock:

obj1 → 🔒 Lock 1

obj2 → 🔒 Lock 2

তাই:

Thread 1 → obj1.test() → 🔒
Thread 2 → obj2.test() → 🔒

দুই thread একই সময়ে execute করতে পারে।

3️⃣ static synchronized হলে?
static synchronized void test() {
}

এখানে কোনো particular object-এর lock নয়।

Class object-এর intrinsic lock ব্যবহার হয়।

Test.class
    ↓
🔒 Intrinsic Lock

তাই:

obj1.test()
obj2.test()
   ↓
Test.class
   ↓
🔒 একই Lock

এক সময়ে একজন execute করবে।

🧠 Intrinsic Lock + synchronized
synchronized
     ↓
Intrinsic Lock
     ↓
এক সময়ে একজন Thread
     ↓
Shared Resource নিরাপদ রাখা
⭐ Short Note

Intrinsic Lock = Java object-এর সাথে থাকা built-in lock, যেটা synchronized ব্যবহার করে thread synchronization করে।

আরেকটি গুরুত্বপূর্ণ শব্দ হলো Monitor—Java-তে সাধারণভাবে Intrinsic Lock এবং Monitor শব্দ দুটো খুব কাছাকাছি অর্থে ব্যবহৃত হয়।
```
🟦 Stack → Method + Local Variable + Reference
🟩 Heap → Object + Array + Instance Data
###
```
🟦 Stack → Method + Local Variable + Reference
🟩 Heap → Object + Array + Instance Data
## Synchronized Instance Method

```java
class Test {
    synchronized void work() {
        // task
    }
}

Test obj = new Test();

Thread-1 → obj.work();
Thread-2 → obj.work();
```

`work()` একটি **synchronized instance method**, তাই এটি `obj`-এর **intrinsic lock** ব্যবহার করে।

### Execution

```text
Thread-1
   ↓
obj.work()
   ↓
obj lock acquire 🔒
   ↓
method execute
   ↓
Thread-2 → obj.work()
   ↓
lock already held by Thread-1
   ↓
Thread-2 → BLOCKED
```

Thread-1 method শেষ করলে:

```text
Thread-1
   ↓
work() শেষ
   ↓
lock release 🔓
   ↓
Thread-2 lock পায়
   ↓
Thread-2 → RUNNABLE
   ↓
work() execute
```

### Important Rule

```text
Same Object
     +
Same synchronized instance method
     ↓
Same Object Lock 🔒
     ↓
একসাথে একজন Thread execute করবে
```

### Different Objects

```java
Test obj1 = new Test();
Test obj2 = new Test();

Thread-1 → obj1.work();  // obj1 lock
Thread-2 → obj2.work();  // obj2 lock
```

`obj1` এবং `obj2`-এর **আলাদা intrinsic lock** আছে।

তাই দুইটি Thread **একসাথে `work()` execute করতে পারে**।

> **Instance synchronized method → Object-এর lock**
> **Same object → Same lock → One thread at a time**
### my code class level and object level lock 
```
import jdk.jfr.StackTrace;

import javax.swing.*;
import java.math.BigInteger;
import java.util.LinkedList;
import java.util.Map;
import java.util.Objects;
import java.util.Queue;
import java.util.concurrent.Callable;

public class Main {
    static int cnt=0;
  synchronized void increment()
    {
        cnt = cnt+1;
    }

    public  static  void main(String[] args) throws InterruptedException {
        Main m1 = new Main();
        Main m2 = new Main();

     Thread t1= new Thread(()->{
         for(int i=0;i<10000;i++)
         {
             m1.increment();
         }
     });

    Thread t2= new Thread(()->{
        for(int i=0;i<20000;i++)
        {
            m2.increment();
        }
    });



    t1.start();
    t2.start();
    t1.join();
    t2.join();

        System.out.println("count is "+ cnt);
}

}
```
