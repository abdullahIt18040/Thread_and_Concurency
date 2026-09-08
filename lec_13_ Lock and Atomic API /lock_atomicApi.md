
<img width="757" height="449" alt="image" src="https://github.com/user-attachments/assets/dc08b9a2-c4ce-4b45-b0c3-782c59837970" />

# Java Lock API — NOTE 
Lock API হলো Java-র এমন একটি concurrency mechanism যার মাধ্যমে আমরা একাধিক Thread-এর shared resource access-কে নিয়ন্ত্রণ করতে পারি এবং synchronized-এর চেয়ে বেশি flexible control পাই।


## 1. `Lock` কী?

একাধিক `Thread` যখন একই **shared resource** ব্যবহার করে, তখন একই সময়ে access করলে **Race Condition** হতে পারে।

উদাহরণ:

```java
int balance = 1000;
```

যদি দুইটি thread একই সময়ে `balance` পরিবর্তন করে, তাহলে unexpected result হতে পারে।

এই সমস্যা নিয়ন্ত্রণ করতে `Lock` ব্যবহার করা যায়।

```text
Thread A → Lock নিল → কাজ করল → Lock ছাড়ল
Thread B → অপেক্ষা করল
Thread B → Lock নিল → কাজ করল → Lock ছাড়ল
```

সহজভাবে:

> `Lock` নিশ্চিত করে যে একটি নির্দিষ্ট সময়ে lock-এর protected অংশে প্রয়োজনীয় synchronization মেনে thread কাজ করবে।

---

# 2. `synchronized` বনাম `Lock`

Java-তে built-in synchronization-এর জন্য `synchronized` আছে।

### `synchronized`

```java
public synchronized void withdraw(int amount) {
    balance -= amount;
}
```

এখানে Java নিজে থেকেই lock **acquire** এবং **release** করে।

### `Lock`

```java
Lock lock = new ReentrantLock();

lock.lock();

try {
    balance -= amount;
} finally {
    lock.unlock();
}
```

এখানে:

```java
lock.lock();
```

মানে → **Lock acquire করা**

```java
lock.unlock();
```

মানে → **Lock release করা**

---

# 3. কেন `Lock` ব্যবহার করব?

`synchronized` সহজ এবং নিরাপদ।

কিন্তু কিছু advanced concurrency problem-এ আমাদের lock-এর উপর বেশি control প্রয়োজন হয়।

`Lock` দিয়ে আমরা:

* কখন lock নেব তা control করতে পারি
* কখন lock ছাড়ব তা control করতে পারি
* lock না পেলে অপেক্ষা না করে অন্য কাজ করতে পারি
* timeout দিয়ে lock নেওয়ার চেষ্টা করতে পারি
* `Condition` ব্যবহার করতে পারি
* বিভিন্ন ধরনের locking strategy implement করতে পারি
* `ReadWriteLock` ব্যবহার করতে পারি

তাই:

```text
synchronized → সহজ এবং automatic
Lock         → flexible এবং বেশি control
```

---

# 4. `synchronized`-এর limitation

উদাহরণ:

```java
synchronized (object) {
    // কাজ
}
```

এখানে lock-এর lifecycle এই block-এর সাথে যুক্ত।

```text
synchronized block শুরু
        ↓
   Lock acquire
        ↓
 Critical section
        ↓
synchronized block শেষ
        ↓
   Lock release
```

Block শেষ হলে lock automatically release হয়।

এটি নিরাপদ, কারণ developer-কে আলাদাভাবে `unlock()` করতে হয় না।

কিন্তু advanced locking strategy-তে এমন হতে পারে:

```text
A lock
   ↓
কিছু কাজ
   ↓
B lock
   ↓
A unlock
   ↓
C lock
   ↓
B unlock
```

এই ধরনের flexible locking `Lock` API দিয়ে করা সহজ।

---

# 5. "Block-structured" বলতে কী বোঝায়?

Java documentation-এ `synchronized`-কে **block-structured locking** বলা হয়।

উদাহরণ:

```java
synchronized (lockObject) {

    System.out.println("Critical section");

}
```

এর lifecycle:

```text
synchronized block শুরু
        ↓
   Lock acquire
        ↓
 Critical section
        ↓
synchronized block শেষ
        ↓
   Lock release
```

অর্থাৎ lock সাধারণত একটি নির্দিষ্ট block-এর মধ্যে সীমাবদ্ধ থাকে।

আপনি `synchronized` দিয়ে সহজভাবে বলতে পারবেন না:

```text
এখানে lock নাও
        ↓
অন্য জায়গায় কিছু কাজ
        ↓
অনেক পরে lock ছাড়ো
```

কিন্তু `Lock` explicit control দেয়।

---

# 6. `Lock` manually control করা যায়

উদাহরণ:

```java
Lock lock = new ReentrantLock();

lock.lock();

System.out.println("Critical section");

lock.unlock();
```

এখানে lock acquire এবং release developer-এর control-এ।

এই flexibility-এর কারণে complex concurrency logic implement করা যায়।

---

# 7. `Lock` ব্যবহারে একটি বড় ঝুঁকি

`synchronized` automatically lock release করে।

কিন্তু `Lock` ব্যবহার করলে developer-কে `unlock()` করতে হয়।

ভুল code:

```java
Lock lock = new ReentrantLock();

lock.lock();

doSomething();

lock.unlock();
```

যদি `doSomething()`-এর মধ্যে exception হয়:

```java
lock.lock();

doSomething(); // Exception

lock.unlock(); // এই line execute নাও হতে পারে
```

তাহলে lock release নাও হতে পারে।

ফলে অন্য thread lock-এর জন্য দীর্ঘ সময় বা অনির্দিষ্টকাল অপেক্ষা করতে পারে।

---

# 8. `try-finally` — সবচেয়ে গুরুত্বপূর্ণ Pattern

তাই `Lock` ব্যবহার করার সময় সাধারণত এই pattern অনুসরণ করতে হবে:

```java
Lock lock = new ReentrantLock();

lock.lock();

try {
    // Critical section
    doSomething();
} finally {
    lock.unlock();
}
```

Flow:

```text
lock.lock()
    ↓
try
    ↓
Critical section
    ↓
Exception হলেও
    ↓
finally
    ↓
lock.unlock()
```

অর্থাৎ exception হলেও `finally` block-এর মাধ্যমে lock release করার সুযোগ থাকে।

### Golden Rule

```java
lock.lock();

try {
    // Shared resource access
} finally {
    lock.unlock();
}
```

**`Lock` ব্যবহার করলে এই pattern অবশ্যই মনে রাখবেন।**

---

# 9. `ReentrantLock` কী?

`Lock` একটি interface:

```java
public interface Lock
```

এর একটি common implementation হলো:

```java
ReentrantLock
```

ব্যবহার:

```java
Lock lock = new ReentrantLock();
```

তারপর:

```java
lock.lock();

try {
    System.out.println("Working...");
} finally {
    lock.unlock();
}
```

### Reentrant বলতে কী বোঝায়?

একই thread একটি `ReentrantLock` একাধিকবার acquire করতে পারে।

```text
Thread A
   ↓
Lock acquire
   ↓
আবার একই Lock acquire
   ↓
কাজ
   ↓
Unlock
   ↓
Unlock
```

অর্থাৎ একই thread-এর জন্য lock পুনরায় acquire করা সম্ভব।

---

# 10. `tryLock()`

`Lock` API-এর একটি গুরুত্বপূর্ণ feature হলো:

```java
tryLock()
```

ধরুন Thread A lock ধরে রেখেছে।

Thread B যদি করে:

```java
lock.lock();
```

তাহলে Thread B lock পাওয়ার জন্য অপেক্ষা করবে।

কিন্তু:

```java
if (lock.tryLock()) {
    try {
        // কাজ
    } finally {
        lock.unlock();
    }
} else {
    System.out.println("Lock পাওয়া যায়নি");
}
```

এখানে Thread B lock না পেলে **অপেক্ষা না করে** অন্য কাজ করতে পারে।

Flow:

```text
Thread A
   ↓
Lock acquired
   ↓
Working...

Thread B
   ↓
tryLock()
   ↓
Lock available?
   ├── YES → কাজ
   └── NO  → অন্য কাজ
```

---

# 11. Timeout সহ `tryLock()`

`tryLock()`-এর মাধ্যমে নির্দিষ্ট সময় পর্যন্ত অপেক্ষাও করা যায়।

```java
if (lock.tryLock(5, TimeUnit.SECONDS)) {
    try {
        // Critical section
    } finally {
        lock.unlock();
    }
}
```

এর অর্থ:

> সর্বোচ্চ ৫ সেকেন্ড পর্যন্ত lock পাওয়ার জন্য অপেক্ষা করো।

৫ সেকেন্ডের মধ্যে lock না পেলে:

```java
false
```

return করবে।

---

# 12. `Condition` কী?

`Lock`-এর সাথে `Condition` ব্যবহার করা যায়।

উদাহরণ:

```java
Lock lock = new ReentrantLock();

Condition condition = lock.newCondition();
```

Thread কোনো condition-এর জন্য অপেক্ষা করতে পারে:

```java
condition.await();
```

অন্য thread signal দিতে পারে:

```java
condition.signal();
```

ধারণাটি:

```text
Thread A
   ↓
condition.await()
   ↓
অপেক্ষা

Thread B
   ↓
condition.signal()
   ↓
Thread A আবার কাজ শুরু
```

`Condition`-কে `wait()` / `notify()`-এর আরও flexible alternative হিসেবে বোঝা যায়।

---

# 13. `ReadWriteLock`

সব ক্ষেত্রে এমন নয় যে একই সময়ে শুধুমাত্র একটি thread resource access করবে।

Java-তে আছে:

```java
ReadWriteLock
```

এতে দুই ধরনের lock থাকে:

```text
Read Lock
Write Lock
```

একাধিক thread একই সময়ে read করতে পারে:

```text
Thread A → READ
Thread B → READ
Thread C → READ
```

কিন্তু write করার সময় exclusive access প্রয়োজন:

```text
Thread A → WRITE
              ↓
        অন্যরা অপেক্ষা
```

এটি এমন application-এ বেশি useful যেখানে:

> Write-এর তুলনায় Read অনেক বেশি হয়।

---

# 14. Hand-over-hand / Chain Locking

এটি একটি advanced locking technique।

ধরুন একটি linked list:

```text
A → B → C → D
```

প্রতিটি node-এর আলাদা lock আছে:

```text
A.lock
B.lock
C.lock
D.lock
```

Traversal-এর সময়:

```text
A lock
   ↓
B lock
   ↓
A unlock
   ↓
C lock
   ↓
B unlock
   ↓
D lock
   ↓
C unlock
```

অর্থাৎ পরবর্তী node-এর lock নেওয়ার পর বর্তমান node-এর lock release করা হয়।

এটিকে বলা হয়:

**Hand-over-hand locking / Chain locking**

এটি concurrent data structure-এ ব্যবহার করা যেতে পারে।

---

# 15. `synchronized` বনাম `Lock`

| বিষয়                    | `synchronized` | `Lock`                 |
| ----------------------- | -------------- | ---------------------- |
| Lock acquire            | Automatic      | Manual                 |
| Lock release            | Automatic      | Manual                 |
| Flexibility             | কম             | বেশি                   |
| `tryLock()`             | ❌              | ✅                      |
| Timeout                 | ❌              | ✅                      |
| `Condition`             | সীমিত          | ✅                      |
| Read/Write Lock         | ❌              | `ReadWriteLock` দিয়ে ✅ |
| Unlock করার দায়িত্ব     | JVM            | Developer              |
| Unlock ভুলে যাওয়ার risk | কম             | বেশি                   |
| `try-finally`           | প্রয়োজন নেই    | **Recommended**        |

---

# 16. সহজভাবে `Lock` বোঝার উপায়

`Lock`-কে একটি **room key** হিসেবে চিন্তা করতে পারেন।

```text
             Lock / Key
                 │
        ┌────────┴────────┐
        ↓                 ↓
    Thread A           Thread B
        │                 │
     Key পেল           অপেক্ষা
        │                 │
      কাজ করল             │
        │                 │
     Key ফেরত             │
                          ↓
                       Key পেল
                          │
                        কাজ
```

যে thread lock-এর ownership পেয়েছে, সে protected critical section-এ কাজ করতে পারে।

---

# 17. পুরো বিষয়টি এক নজরে

```text
                  Java Concurrency
                        │
                        ↓
                  synchronized
                        │
                        ↓
                      Lock
                        │
              ┌─────────┴─────────┐
              ↓                   ↓
       ReentrantLock        ReadWriteLock
              │
       ┌──────┴──────┐
       ↓             ↓
   tryLock()      Condition
       │
       ↓
  Timed Lock
```

---

# 18. সবচেয়ে গুরুত্বপূর্ণ Code Pattern

এই pattern-টি অবশ্যই মনে রাখুন:

```java
Lock lock = new ReentrantLock();

lock.lock();

try {
    // Critical Section

} finally {
    lock.unlock();
}
```

### মনে রাখার Formula

```text
lock()
  ↓
try
  ↓
Critical Section
  ↓
finally
  ↓
unlock()
```

---

# 19. Short Summary

```text
synchronized
    ↓
সহজ এবং automatic locking

Lock
    ↓
Manual এবং flexible locking

ReentrantLock
    ↓
Lock-এর common implementation

tryLock()
    ↓
Lock না পেলে wait না করে return

tryLock(timeout)
    ↓
নির্দিষ্ট সময় পর্যন্ত অপেক্ষা

Condition
    ↓
Thread coordination

ReadWriteLock
    ↓
Multiple reader + exclusive writer
```

### মূল কথা

> **`synchronized` সহজ, কিন্তু `Lock` বেশি flexible। `Lock` ব্যবহার করলে lock management-এর দায়িত্ব developer-এর, তাই `try-finally` ব্যবহার করে `unlock()` করা গুরুত্বপূর্ণ।**
# Java Reentrant Lock — সহজ বাংলা নোট

## 1. একটি Object-এর Lock

Java-তে প্রতিটি object-এর সাথে একটি **Intrinsic Monitor Lock** থাকে।

```text
Object
  │
  └── Monitor Lock
```

একই সময়ে **একাধিক thread এই lock acquire করতে পারে না**।

```text
Thread-1 → Lock পেল → কাজ করছে

Thread-2 → Lock পাওয়ার জন্য অপেক্ষা করছে
```

---

## 2. Reentrant কী?

একটি গুরুত্বপূর্ণ বিষয় হলো:

> **একই thread একই lock একাধিকবার acquire করতে পারে।**

এটাকেই বলা হয় **Reentrant**।

উদাহরণ:

```java
synchronized void methodA() {
    methodB();
}

synchronized void methodB() {
    // কাজ
}
```

যদি `Thread-1` `methodA()`-তে ঢোকে:

```text
Thread-1
   ↓
Lock acquire
count = 1
   ↓
methodA()
   ↓
methodB()
   ↓
আবার একই Lock acquire
count = 2
```

এখানে দ্বিতীয়বার lock নেওয়ার সময় thread আটকে যায় না, কারণ **lock-এর owner একই Thread-1**।

---

## 3. Lock কখন Free হবে?

`methodB()` শেষ হলে:

```text
count = 2
   ↓
unlock
   ↓
count = 1
```

তারপর `methodA()` শেষ হলে:

```text
count = 1
   ↓
unlock
   ↓
count = 0
```

এখন lock পুরোপুরি free।

```text
count = 0
    ↓
Lock available
```

---

## 4. সহজ Example

```text
Thread-1
   │
   ├── Lock acquire → count = 1
   │
   ├── methodA()
   │      │
   │      └── methodB()
   │              │
   │              └── Lock acquire → count = 2
   │
   ├── methodB() শেষ → count = 1
   │
   └── methodA() শেষ → count = 0
```

---

# 5. `ReentrantLock`-এও একই Concept

`ReentrantLock`-এর ক্ষেত্রেও একই thread একই lock বারবার acquire করতে পারে।

```java
ReentrantLock lock = new ReentrantLock();

lock.lock();    // count = 1

lock.lock();    // count = 2

lock.unlock();  // count = 1

lock.unlock();  // count = 0
```

এখানে:

```text
lock.lock()   → count + 1
lock.unlock() → count - 1
```

---

# 6. সবচেয়ে গুরুত্বপূর্ণ Rule

> **যতবার `lock()` করবেন, ততবার `unlock()` করতে হবে।**

```text
lock()       → 1
lock()       → 2
unlock()     → 1
unlock()     → 0
```

একটি `lock()` করলে একটি `unlock()` প্রয়োজন।

---

# 7. অন্য Thread কী করবে?

ধরুন:

```text
Thread-1 → Lock acquire
```

এখন:

```text
Thread-2 → একই Lock acquire করতে চায়
```

Thread-2 lock পাবে না।

```text
Thread-1
   ↓
Lock acquired
   ↓
Working...

Thread-2
   ↓
Lock চাই
   ↓
Waiting...
```

Thread-1 lock release করার পর Thread-2 lock পেতে পারে।

---

# 8. মূল বিষয় এক নজরে

```text
একটি Object
     │
     └── একটি Monitor Lock
              │
              ├── Thread-1 → acquire
              │
              └── অন্য Thread → wait
```

কিন্তু একই thread:

```text
Thread-1
   ↓
Lock acquire → count = 1
   ↓
আবার acquire → count = 2
   ↓
unlock → count = 1
   ↓
unlock → count = 0
```

### মনে রাখুন

> **Lock একটি, কিন্তু একই owning thread সেটিকে multiple times acquire করতে পারে।**

এটাই **Reentrant Locking**।

```text
একই Thread
    ↓
একই Lock
    ↓
Multiple acquire
    ↓
Multiple release
 after relase other thead can acquire lock. 
```

আর **অন্য thread একই সময়ে সেই lock acquire করতে পারে না**; তাকে অপেক্ষা করতে হয়।
<img width="746" height="275" alt="image" src="https://github.com/user-attachments/assets/2b91302e-79ab-459b-a270-f3e769aa143f" />
<img width="1314" height="822" alt="image" src="https://github.com/user-attachments/assets/95c6990d-5902-4121-a22b-f6aa47534ddd" />

## Fair lock
```
৯. Fair Lock

তুমি চাইলে:

Lock lock = new ReentrantLock(true);

এখানে true মানে fairness।

ধরো:

Thread A → অপেক্ষা করছে
Thread B → অপেক্ষা করছে
Thread C → অপেক্ষা করছে

Fair lock সাধারণত waiting order অনুসরণ করার চেষ্টা করে:

A → B → C

অর্থাৎ যে আগে অপেক্ষা করছে, তাকে আগে সুযোগ দেওয়ার চেষ্টা করা হয়।
```
# Atomic Operation — Java Concurrency

## 🔹 What is Atomic Operation?

**Atomic operation** হলো এমন একটি operation যা **একটি single, indivisible step** হিসেবে সম্পন্ন হয়।

অর্থাৎ operation-এর মাঝখানে অন্য কোনো thread এসে এটাকে ভেঙে দিতে পারে না।

> **Atomic = পুরো operation একসাথে সম্পন্ন হবে, মাঝখানে partial state থাকবে না।**

---

## 🔴 `count++` কেন Atomic নয়?

```java
count++;
```

দেখতে একটি operation মনে হলেও internally এটি প্রায়:

```text
READ → MODIFY → WRITE
```

অর্থাৎ:

```text
count++
   ↓
Read count
   ↓
Add 1
   ↓
Write count
```

তাই `count++` **atomic নয়**।

### Race Condition Example

ধরি:

```text
count = 0
```

দুইটি thread একই সময়ে কাজ করলে:

```text
Thread A              Thread B
   │                     │
   │ Read = 0            │
   │                     │ Read = 0
   │                     │
   │ 0 + 1 = 1           │ 0 + 1 = 1
   │                     │
   │ Write = 1           │ Write = 1
```

Final result:

```text
count = 1
```

কিন্তু expected:

```text
count = 2
```

এটাই **Race Condition**।

---

# 🟢 AtomicInteger

Java-তে atomic operation করার জন্য:

```java
AtomicInteger count = new AtomicInteger(0);
```

তারপর:

```java
count.incrementAndGet();
```

ব্যবহার করা যায়।

### Example

```java
import java.util.concurrent.atomic.AtomicInteger;

public class Main {

    static AtomicInteger count = new AtomicInteger(0);

    static void increment() {
        for (int i = 0; i < 300; i++) {
            count.incrementAndGet();
        }
    }

    public static void main(String[] args) throws InterruptedException {

        Thread t1 = new Thread(Main::increment);
        Thread t2 = new Thread(Main::increment);
        Thread t3 = new Thread(Main::increment);

        t1.start();
        t2.start();
        t3.start();

        t1.join();
        t2.join();
        t3.join();

        System.out.println(count.get());
    }
}
```

Output:

```text
900
```

কারণ:

```text
Thread 1 → 300
Thread 2 → 300
Thread 3 → 300

Total → 900
```

---

# 🔥 CAS — Compare And Swap

`AtomicInteger` সাধারণত **CAS (Compare-And-Swap)** mechanism ব্যবহার করে।

ধরি:

```text
count = 10
```

Thread A চায়:

```text
10 → 11
```

CAS ধারণাটি:

```text
Expected value = 10
New value      = 11
```

CAS বলবে:

```text
যদি current value এখনও 10 থাকে
        ↓
তাহলে 11 করে দাও
```

কিন্তু অন্য thread যদি এর মধ্যে value পরিবর্তন করে:

```text
10 → 20
```

তাহলে:

```text
Expected = 10
Actual   = 20

CAS → FAIL
```

Thread আবার চেষ্টা করবে।

---

# 🔹 Common Atomic Operations

```java
AtomicInteger count = new AtomicInteger(0);

count.get();

count.set(10);

count.incrementAndGet();

count.decrementAndGet();

count.getAndIncrement();

count.getAndDecrement();

count.addAndGet(10);

count.compareAndSet(10, 20);
```

---

# ⚠️ Important

**Atomic operation ≠ পুরো code thread-safe**

উদাহরণ:

```java
if (count.get() < 10) {
    count.incrementAndGet();
}
```

এখানে:

```text
get()
 ↓
condition check
 ↓
increment
```

প্রতিটি individual operation atomic হলেও **পুরো sequence atomic নয়**।

একাধিক thread একই সময়ে condition pass করতে পারে।

---

# 🧠 Atomic vs Non-Atomic

| Operation                         | Atomic? |
| --------------------------------- | ------- |
| `count++`                         | ❌ No    |
| `count--`                         | ❌ No    |
| `AtomicInteger.incrementAndGet()` | ✅ Yes   |
| `AtomicInteger.decrementAndGet()` | ✅ Yes   |
| `AtomicInteger.compareAndSet()`   | ✅ Yes   |

---

# 🎯 Key Points

```text
Atomic Operation
      ↓
Indivisible operation
      ↓
Concurrency-এর মধ্যে safe individual operation
      ↓
Race condition এড়াতে সাহায্য করে
      ↓
Java → AtomicInteger / AtomicLong / AtomicReference
      ↓
CAS (Compare-And-Swap) ব্যবহার করতে পারে
```

### One Line Definition

> **Atomic operation হলো এমন operation যা concurrency-এর মধ্যে একটি single, indivisible action হিসেবে সম্পন্ন হয়।**
## my code is 
<img width="1269" height="632" alt="image" src="https://github.com/user-attachments/assets/39bd421a-98c7-40c9-9e62-c140f0f1e0ec" />
<img width="1386" height="683" alt="image" src="https://github.com/user-attachments/assets/d5a6fff5-b42d-4f07-8ae9-90b460b6b194" />





