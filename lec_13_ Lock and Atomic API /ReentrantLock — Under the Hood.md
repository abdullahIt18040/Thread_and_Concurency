# ReentrantLock — Under the Hood

Java-এর `ReentrantLock` বুঝতে হলে মূলত ৩টি concept বুঝতে হবে:

```text
ReentrantLock
      ↓
     AQS
(AbstractQueuedSynchronizer)
      ↓
┌───────────────┐
│ state         │
│ owner thread  │
│ wait queue    │
└───────────────┘
      ↓
     CAS
```

---

## 1. ReentrantLock কী?

```java
ReentrantLock lock = new ReentrantLock();

lock.lock();

try {
    // Critical Section
} finally {
    lock.unlock();
}
```

`ReentrantLock` একটি explicit lock implementation যা internally **AQS (AbstractQueuedSynchronizer)** ব্যবহার করে।

---

# 2. AQS কী?

**AQS = AbstractQueuedSynchronizer**

AQS হলো Java concurrency-এর একটি framework, যার মাধ্যমে synchronization mechanism তৈরি করা হয়।

যেমন:

```text
ReentrantLock
Semaphore
CountDownLatch
ReadWriteLock
```

`ReentrantLock`-এর ক্ষেত্রে AQS মূলত manage করে:

```text
1. Lock state
2. Owner thread
3. Waiting threads
```

---

# 3. AQS `state`

AQS-এর গুরুত্বপূর্ণ একটি field:

```java
private volatile int state;
```

সহজভাবে:

```text
state = 0
    ↓
Lock free

state = 1
    ↓
Lock acquired

state = 2
    ↓
Same thread আবার lock করেছে

state = 3
    ↓
Same thread আরও একবার lock করেছে
```

অর্থাৎ `state` দিয়ে **reentrant hold count** বোঝানো হয়।

---

# 4. First Thread Lock নিলে

শুরুতে:

```text
state = 0
owner = null
```

Thread A:

```java
lock.lock();
```

করলে AQS lock acquire করার চেষ্টা করে।

Conceptually:

```text
CAS(state, 0, 1)
```

অর্থাৎ:

```text
Expected = 0
New      = 1
```

যদি `state` এখনও `0` থাকে:

```text
CAS → SUCCESS
```

তারপর:

```text
state = 1
owner = Thread A
```

Thread A এখন lock-এর owner।

---

# 5. CAS কেন ব্যবহার করা হয়?

**CAS = Compare-And-Swap**

CAS একটি atomic operation।

ধরি:

```text
state = 0
```

Thread A এবং Thread B একই সময়ে lock নিতে চাচ্ছে:

```text
Thread A → CAS(0 → 1)
Thread B → CAS(0 → 1)
```

কিন্তু atomic CAS-এর কারণে একজনই সফল হবে।

```text
Thread A → SUCCESS
Thread B → FAIL
```

তাই দুইটি thread একই সময়ে lock owner হতে পারে না।

---

# 6. Reentrant কীভাবে কাজ করে?

ধরি Thread A lock নিয়েছে:

```text
state = 1
owner = Thread A
```

Thread A আবার:

```java
lock.lock();
```

করল।

Thread A already owner, তাই তাকে block করা হবে না।

AQS check করে:

```text
currentThread == ownerThread
```

যদি true হয়:

```text
state = state + 1
```

তাই:

```text
First lock  → state = 1
Second lock → state = 2
Third lock  → state = 3
```

এটাই **Reentrant** behavior।

---

# 7. ReentrantLock Example

```java
lock.lock(); // state = 1

lock.lock(); // state = 2

lock.lock(); // state = 3
```

তারপর:

```java
lock.unlock(); // state = 2

lock.unlock(); // state = 1

lock.unlock(); // state = 0
```

অর্থাৎ:

> প্রতিটি successful `lock()`-এর জন্য একটি corresponding `unlock()` করতে হবে।

---

# 8. অন্য Thread Lock নিতে চাইলে

ধরি:

```text
owner = Thread A
state = 1
```

এখন Thread B:

```java
lock.lock();
```

করল।

Thread B owner নয়:

```text
Thread B != Thread A
```

তাই B lock acquire করতে পারবে না।

এরপর AQS Thread B-কে waiting queue-তে রাখতে পারে।

```text
Lock Owner
    │
    ▼
Thread A
    │
    ▼
Waiting Queue
    │
    ├── Thread B
    │
    └── Thread C
```

---

# 9. AQS Waiting Queue

সহজভাবে AQS queue:

```text
             AQS
              │
              ▼
       ┌─────────────┐
       │ Thread A    │ ← Lock Owner
       └─────────────┘
              │
              ▼
       ┌─────────────┐
       │ Thread B    │ ← Waiting
       └─────────────┘
              │
              ▼
       ┌─────────────┐
       │ Thread C    │ ← Waiting
       └─────────────┘
```

B এবং C lock-এর জন্য অপেক্ষা করছে।

---

# 10. Unlock করলে কী হয়?

ধরি:

```text
owner = Thread A
state = 1

Queue:
B → C
```

Thread A:

```java
lock.unlock();
```

করল।

তখন conceptually:

```text
state = 0
owner = null
```

এরপর AQS waiting thread-কে signal করে।

Thread B আবার lock acquire করার চেষ্টা করবে:

```text
Thread B
    ↓
tryAcquire()
    ↓
CAS(0 → 1)
    ↓
SUCCESS
```

তারপর:

```text
owner = Thread B
state = 1
```

---

# 11. Complete Flow

```text
Thread A
   │
   │ lock()
   ▼
AQS
   │
   │ CAS(0 → 1)
   ▼
SUCCESS
   │
   ▼
state = 1
owner = A
```

Thread A আবার lock করলে:

```text
Thread A
   │
   │ lock()
   ▼
owner == currentThread
   │
   ▼
state++
   │
   ▼
state = 2
```

অন্য Thread B এলে:

```text
Thread B
   │
   │ lock()
   ▼
owner != B
   │
   ▼
Wait Queue
```

A unlock করলে:

```text
Thread A
   │
   │ unlock()
   ▼
state--
   │
   ▼
state = 0
   │
   ▼
Signal waiting thread
   │
   ▼
Thread B
   │
   │ CAS(0 → 1)
   ▼
Lock acquired
```

---

# 12. Fair vs Non-Fair ReentrantLock

Default:

```java
new ReentrantLock();
```

এটি **non-fair**।

Fair lock:

```java
new ReentrantLock(true);
```

### Non-Fair

নতুন thread waiting queue-তে থাকা thread-এর আগে lock পেয়ে যেতে পারে।

```text
Queue:
B → C

New Thread D
     ↓
Lock পাওয়ার চেষ্টা
     ↓
D আগে পেয়ে যেতে পারে
```

### Fair

Waiting order-কে বেশি গুরুত্ব দেওয়া হয়:

```text
B → C → D
```

B আগে সুযোগ পাবে।

---

# 13. ReentrantLock Internal Structure

Simplified structure:

```text
ReentrantLock
      │
      ▼
     Sync
      │
      ├──────────────┐
      ▼              ▼
NonfairSync       FairSync
      │              │
      └──────┬───────┘
             ▼
            AQS
             │
     ┌───────┼────────┐
     ▼       ▼        ▼
   state   owner    Queue
             │
             ▼
            CAS
```

---

# 14. `synchronized` vs `ReentrantLock`

### synchronized

```text
synchronized
      ↓
JVM Monitor
      ↓
Intrinsic Lock
```

### ReentrantLock

```text
ReentrantLock
      ↓
AQS
      ↓
state + queue + CAS
```

`ReentrantLock` বেশি flexible API দেয়।

যেমন:

```java
lock.tryLock();

lock.tryLock(5, TimeUnit.SECONDS);

lock.lockInterruptibly();
```

---

# 15. Important Methods

```java
lock.lock();
```

Lock acquire করার চেষ্টা করে।

```java
lock.unlock();
```

Lock release করে।

```java
lock.tryLock();
```

Block না করে lock নেওয়ার চেষ্টা করে।

```java
lock.lockInterruptibly();
```

Waiting অবস্থায় thread interrupt করা যায়।

---

# 🧠 Mental Model

`ReentrantLock` মনে রাখার সবচেয়ে সহজ উপায়:

```text
                 ReentrantLock
                       │
                       ▼
                      AQS
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
        state        owner        Queue
          │            │            │
          │            │            ├── Thread B
          │            │            └── Thread C
          │            │
          ▼            ▼
     Hold Count    Owner Thread
          │
          ▼
         CAS
          │
          ▼
    Lock Acquisition
```

### Reentrant Flow

```text
Thread A

lock()     → state = 1
lock()     → state = 2
lock()     → state = 3

unlock()   → state = 2
unlock()   → state = 1
unlock()   → state = 0
```

### Final Summary

```text
ReentrantLock
     ↓
     AQS
     ↓
 ┌───────────────┐
 │ state         │ → Lock/hold count
 │ owner         │ → কে lock ধরে আছে
 │ wait queue    │ → কারা অপেক্ষা করছে
 └───────────────┘
     ↓
    CAS
     ↓
Atomic lock acquisition
```

> **ReentrantLock internally AQS ব্যবহার করে lock state ও waiting threads manage করে। CAS ব্যবহার করে atomicভাবে lock acquisition করার চেষ্টা করে, আর একই thread আবার lock করলে state/hold count বাড়িয়ে দেয়—এটাই reentrant behavior।**
