# Java Lock API — বাংলা নোট

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

