<img width="789" height="411" alt="image" src="https://github.com/user-attachments/assets/eba501df-9f5a-4f74-b91f-990154c2ad15" />
<img width="684" height="513" alt="image" src="https://github.com/user-attachments/assets/d43f394e-b4c7-4e3a-b6a5-063c6dc01ea4" />


## LifeCycle of Thread

Java Thread Methods — Short Note
```
Method	কাজ	State / Result
start()	নতুন thread-এর execution শুরু করে এবং run() call করায়	NEW → RUNNABLE
run()	Thread-এর actual কাজ থাকে	শেষে TERMINATED
sleep(ms)	নির্দিষ্ট সময় current thread pause করে	TIMED_WAITING
wait()	অন্য thread-এর signal-এর জন্য অপেক্ষা করে এবং lock release করে	WAITING
wait(ms)	নির্দিষ্ট সময় পর্যন্ত wait করে	TIMED_WAITING
notify()	একটি waiting thread-কে wake করার signal দেয়	WAITING → Runnable-এর দিকে
notifyAll()	সব waiting thread-কে wake করার signal দেয়	WAITING → Runnable-এর দিকে
join()	অন্য thread শেষ হওয়া পর্যন্ত current thread অপেক্ষা করে	WAITING
join(ms)	নির্দিষ্ট সময় পর্যন্ত অন্য thread-এর জন্য অপেক্ষা করে	TIMED_WAITING
interrupt()	thread-কে interruption signal দেয়; sleep/wait/join হলে exception হতে পারে	Context অনুযায়ী
getState()	Thread-এর current state জানতে ব্যবহার হয়	NEW, RUNNABLE ইত্যাদি
isAlive()	Thread এখনো running/finished কিনা check করে	true / false
currentThread()	বর্তমানে যে thread execute করছে সেটি return করে	Thread object
State মনে রাখার Shortcut
new Thread()
     ↓
   NEW
     ↓ start()
 RUNNABLE
     ↓
 ┌─────────┬─────────┬──────────────┐
 ↓         ↓         ↓
BLOCKED  WAITING  TIMED_WAITING
 ↓         ↓         ↓
 └─────────┴─────────┘
           ↓
       RUNNABLE
           ↓
      run() শেষ
           ↓
      TERMINATED
সবচেয়ে গুরুত্বপূর্ণ 6 methods
start()       → Thread শুরু
run()         → Thread-এর কাজ
sleep()       → সময়ের জন্য pause
wait()        → অন্য thread-এর signal-এর অপেক্ষা
notify()      → একটি waiting thread-কে signal
notifyAll()   → সব waiting thread-কে signal
```
## 1. join()  && Join(1000) কী?

```
ধরো main thread একটি t1 thread তৈরি করল:

Thread t1 = new Thread(() -> {
    System.out.println("T1 running");
});

t1.start();

t1.join();

System.out.println("Main finished");

এখানে:

Main Thread
     |
     | t1.start()
     ↓
   T1 শুরু
     |
     |
     | t1.join()
     ↓
Main WAITING
     |
     | অপেক্ষা করছে T1 শেষ হওয়ার জন্য
     |
     ↓
T1 শেষ
     |
     ↓
Main আবার RUNNABLE
     |
     ↓
"Main finished"
গুরুত্বপূর্ণ

join() T1-কে wait করাচ্ছে না।

বরং Main thread অপেক্ষা করছে T1-এর জন্য।

2. Step-by-step

Code:

Thread t1 = new Thread(() -> {

    try {
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }

    System.out.println("T1 completed");
});

t1.start();

t1.join();

System.out.println("Main completed");

ধাপে ধাপে:

Step 1
t1.start();
T1 → RUNNABLE
Step 2

Main:

t1.join();

এখন Main বলছে:

"T1, তুমি শেষ না হওয়া পর্যন্ত আমি অপেক্ষা করব।"

তাই:

Main → WAITING
T1   → RUNNABLE
Step 3

T1 কাজ শেষ করে:

T1 → TERMINATED
Step 4

Main-এর join() শেষ হয়:

Main → RUNNABLE

তারপর:

System.out.println("Main completed");

execute হবে।

3. কেন WAITING?

কারণ:

t1.join();

এর কোনো timeout নেই।

Main অনির্দিষ্ট সময় অপেক্ষা করবে যতক্ষণ না t1 শেষ হয়।

join()
  ↓
WAITING
  ↓
T1 terminates
  ↓
RUNNABLE

তাই:

t1.join();

→ WAITING

4. join(milliseconds)

এখন:

t1.join(2000);

মানে:

"আমি maximum 2 seconds T1-এর জন্য অপেক্ষা করব।"

এখানে দুইটা possibility আছে।

Case 1: T1 2 seconds-এর মধ্যে শেষ হয়
Main
 ↓
join(2000)
 ↓
TIMED_WAITING
 ↓
T1 completes
 ↓
RUNNABLE
Case 2: T1 2 seconds-এর মধ্যে শেষ হয় না

2 seconds শেষ হয়ে গেলে:

Main
 ↓
TIMED_WAITING
 ↓
2 seconds complete
 ↓
RUNNABLE

Main আর T1-এর জন্য অপেক্ষা করবে না।

5. Example
Thread t1 = new Thread(() -> {
    try {
        Thread.sleep(5000);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }

    System.out.println("T1 completed");
});

t1.start();

t1.join(2000);

System.out.println("Main completed");

Timeline:

Time
0 sec
│
├── T1 starts
│
├── Main → join(2000)
│            ↓
│       TIMED_WAITING
│
2 sec
│
├── Main → join() শেষ
│
├── Main → RUNNABLE
│
└── Main prints "Main completed"
│
5 sec
│
└── T1 → TERMINATED

অর্থাৎ T1 5 seconds কাজ করলেও Main মাত্র 2 seconds অপেক্ষা করবে।

6. join() vs join(2000)
Code	Current Thread-এর State	অর্থ
t1.join()	WAITING	T1 শেষ না হওয়া পর্যন্ত অপেক্ষা
t1.join(2000)	TIMED_WAITING	Maximum 2 seconds অপেক্ষা
t1.join(5000)	TIMED_WAITING	Maximum 5 seconds অপেক্ষা
7. খুব গুরুত্বপূর্ণ: join() কোন thread-কে affect করে?

এটা interview-এর জন্য খুব গুরুত্বপূর্ণ।

t1.join();

এখানে:

       t1
       ↑
       │
     join()
       │
       ↑
   Main Thread

Main Thread WAITING হবে।

t1 WAITING হবে না।

সহজ ভাষায়:

যে thread join() call করে, সেই thread wait করে।
```
## yield() Method — সহজভাবে
```
Thread.yield() হলো একটি static method, যার মাধ্যমে current thread scheduler-কে ইঙ্গিত দেয়:

“আমি এখন CPU ছেড়ে দিতে রাজি; অন্য কোনো runnable thread থাকলে তাকে সুযোগ দিতে পারো।”

Example
public void run() {
    for (int i = 0; i < 5; i++) {
        System.out.println(Thread.currentThread().getName());

        Thread.yield();
    }
}

Flow:

Thread-1 → RUNNING
             |
          yield()
             ↓
      Scheduler-কে hint
             ↓
   অন্য RUNNABLE thread
   CPU পেতে পারে
গুরুত্বপূর্ণ

yield():

Current thread-কে WAITING বা BLOCKED করে না।
Thread-এর state সাধারণত RUNNABLE-ই থাকে।
CPU ছাড়ার guarantee নেই।
Scheduler চাইলে একই thread-কে আবার immediately CPU দিতে পারে।
অন্য thread অবশ্যই CPU পাবে—এমন guarantee নেই।
এটি শুধু scheduler-এর জন্য একটি hint।
yield() vs sleep()
yield()	sleep()
Scheduler-কে hint দেয়	নির্দিষ্ট সময় pause করে
State সাধারণত RUNNABLE	TIMED_WAITING
কোনো নির্দিষ্ট সময় নেই	সময় নির্দিষ্ট করা যায়
CPU ছাড়ার guarantee নেই	specified time পর্যন্ত sleep
Lock release করে না	Lock release করে না
Short Note
yield()
   ↓
Current thread scheduler-কে hint দেয়
   ↓
"অন্য Runnable thread-কে CPU দিতে পারো"
   ↓
State → RUNNABLE

মনে রাখবে: yield() মানে “CPU অবশ্যই ছেড়ে দিলাম” নয়, বরং “CPU ছেড়ে দিতে পারি” — scheduler-এর কাছে একটি hint।
```
### can we start the thread twice?
```
একই Java Thread object-কে দুইবার start() করা যায় না।

Example
Thread t = new Thread(() -> {
    System.out.println("Running");
});

t.start();   // ✅ প্রথমবার
t.start();   // ❌ দ্বিতীয়বার

দ্বিতীয়বার start() করলে:

java.lang.IllegalThreadStateException

হবে।

কেন?

একটি Thread-এর lifecycle:

NEW
 ↓
start()
 ↓
RUNNABLE
 ↓
Running
 ↓
TERMINATED

একবার start() করার পর Thread আর NEW state-এ থাকে না। তাই একই Thread object-কে আবার start() করা যায় না।

নতুন Thread দিয়ে করা যাবে
Thread t1 = new Thread(task);
Thread t2 = new Thread(task);

t1.start();  // ✅
t2.start();  // ✅

এখানে t1 এবং t2 দুটি আলাদা Thread object।

start() বনাম run()
t.start();  // নতুন thread তৈরি করে execution শুরু করে

কিন্তু:

t.run();    // শুধু normal method call

run() manually একাধিকবার call করা technically সম্ভব, কিন্তু নতুন thread তৈরি হবে না।

মনে রাখার shortcut
একই Thread object:

start() → ✅
start() → ❌ IllegalThreadStateException

নতুন Thread object:

t1.start() → ✅
t2.start() → ✅

মূল কথা: start() একটি Thread object-এর lifecycle-এ শুধু একবার call করা যায়।
```
### instance method of Thread class 
```
Java-এর Thread class-এ অনেক instance method আছে। Instance method মানে হলো Thread object-এর মাধ্যমে call করা method।

Thread t = new Thread();
t.start();       // instance method
t.getName();     // instance method
গুরুত্বপূর্ণ Thread Instance Methods
Method	কাজ
start()	Thread execution শুরু করে
run()	Thread-এর কাজ execute করে
join()	অন্য thread শেষ হওয়া পর্যন্ত wait করে
join(ms)	নির্দিষ্ট সময় পর্যন্ত wait করে
sleep(ms)	⚠️ এটি static, instance method নয়
interrupt()	Thread-কে interrupt signal দেয়
isAlive()	Thread এখনো alive কিনা check করে
getName()	Thread-এর নাম দেয়
setName()	Thread-এর নাম পরিবর্তন করে
getPriority()	Thread-এর priority দেখায়
setPriority()	Thread-এর priority সেট করে
getState()	Thread-এর current state দেয়
isInterrupted()	Thread interrupted কিনা check করে
setDaemon()	Thread-কে daemon thread হিসেবে সেট করে
isDaemon()	Thread daemon কিনা check করে
getId()	Thread-এর ID দেয়
getThreadGroup()	Thread-এর ThreadGroup দেয়
Example
Thread t = new Thread();

t.setName("Worker-1");
t.setPriority(Thread.MAX_PRIORITY);

System.out.println(t.getName());
System.out.println(t.getPriority());
System.out.println(t.getState());

t.start();

System.out.println(t.isAlive());
⚠️ Important: sleep() Instance Method নয়

যদিও তুমি এভাবে লিখতে পারো:

t.sleep(1000);

এটা দেখতে instance method-এর মতো হলেও sleep() আসলে static method।

সঠিকভাবে:

Thread.sleep(1000);

কারণ sleep() current thread-কেই sleep করায়, t-কে নয়।

Shortcut
Instance methods:
start()
run()
join()
interrupt()
isAlive()
getName()
setName()
getState()
getPriority()
setPriority()
isDaemon()
setDaemon()
Static methods:
sleep()
yield()
currentThread()

সবচেয়ে গুরুত্বপূর্ণ: Thread class-এর method দেখার সময় আগে বুঝবে method-টি instance নাকি static। sleep(), yield(), currentThread() → static; বাকিগুলোর অনেকগুলো object-এর মাধ্যমে call করা যায়।
```
