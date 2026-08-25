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
