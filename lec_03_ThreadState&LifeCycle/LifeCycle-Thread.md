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
