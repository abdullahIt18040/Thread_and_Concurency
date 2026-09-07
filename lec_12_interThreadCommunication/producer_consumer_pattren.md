<img width="783" height="405" alt="image" src="https://github.com/user-attachments/assets/905ae4cc-430d-4bf9-ae47-0c25208d95e9" />

### Traditional Producer-Consumer Pattern .

<img width="1083" height="583" alt="image" src="https://github.com/user-attachments/assets/d82a7ed6-c942-4c17-9a58-201ba890d036" />

# Busy Waiting

## What is Busy Waiting?

**Busy Waiting** হলো এমন একটি situation যেখানে একটি thread কোনো condition `true` হওয়ার জন্য বারবার check করতে থাকে, কিন্তু কোনো useful কাজ না করেও **CPU continuously ব্যবহার করে**।

সহজভাবে:

> **Thread ঘুমাচ্ছে না, কাজও করছে না—শুধু বারবার condition check করছে।**

### Example

```java
boolean dataAvailable = false;

while (!dataAvailable) {
    // বারবার condition check
}
```

Flow:

```text
dataAvailable?
      ↓
    false
      ↓
  check again
      ↓
    false
      ↓
  check again
      ↓
    false
      ↓
     ...
```

এই loop চলার সময় thread CPU ব্যবহার করতে থাকে।

---

## Producer-Consumer Example

ধরো, Consumer queue থেকে data নেওয়ার জন্য অপেক্ষা করছে:

```java
while (queue.isEmpty()) {
    // waiting
}
```

Queue empty থাকা পর্যন্ত Consumer বারবার:

```text
queue.isEmpty()
queue.isEmpty()
queue.isEmpty()
queue.isEmpty()
...
```

check করবে।

এটাই **Busy Waiting**।

---

## Problem with Busy Waiting

Busy Waiting-এর প্রধান সমস্যা:

* CPU unnecessarily ব্যবহার হয়
* CPU time waste হয়
* Performance কমতে পারে
* Long waiting-এর ক্ষেত্রে inefficient

---

# `wait()` ব্যবহার করলে

Busy Waiting-এর পরিবর্তে `wait()` ব্যবহার করা যায়:

```java
synchronized (queue) {

    while (queue.isEmpty()) {
        queue.wait();
    }

    // consume data
}
```

এখন flow হবে:

```text
Queue Empty
     ↓
   wait()
     ↓
Thread → WAITING
     ↓
CPU released
     ↓
Producer adds data
     ↓
notify()
     ↓
Thread wakes up
     ↓
Condition check
     ↓
Consume data
```

`wait()` করার পর thread **WAITING state**-এ চলে যায় এবং CPU continuously consume করে না।

---

## Busy Waiting vs `wait()`

| Busy Waiting                       | `wait()`                           |
| ---------------------------------- | ---------------------------------- |
| বারবার condition check করে         | Waiting state-এ থাকে               |
| CPU continuously ব্যবহার করতে পারে | Waiting অবস্থায় CPU ব্যবহার করে না |
| Continuous `while` loop            | `wait()` দিয়ে অপেক্ষা করে          |
| CPU waste হতে পারে                 | More CPU efficient                 |
| Long waiting-এর জন্য inefficient   | Thread coordination-এর জন্য better |

---

## Easy Real-Life Example

### Busy Waiting

দরজার সামনে দাঁড়িয়ে প্রতি মুহূর্তে জিজ্ঞেস করছি:

> "দরজা খুলেছে?"
> "দরজা খুলেছে?"
> "দরজা খুলেছে?"
> "দরজা খুলেছে?"

অর্থাৎ **বারবার check করা**।

### `wait()`

দরজার সামনে বসে আছি এবং বললাম:

> "দরজা খুললে আমাকে ডাকবে।"

তারপর অপেক্ষা করছি।

দরজা খুললে আমাকে notify করা হবে।

---

## Key Concept

```text
Busy Waiting

Thread
  ↓
Check condition
  ↓
False
  ↓
Check again
  ↓
Check again
  ↓
Check again
  ↓
CPU usage continues
```

```text
wait()

Thread
  ↓
wait()
  ↓
WAITING state
  ↓
CPU released
  ↓
notify()
  ↓
Wake up
  ↓
Check condition
```

### ⭐ Remember

> **Busy Waiting = Continuously checking a condition while consuming CPU.**

> **`wait()` = Stop checking and enter WAITING state until notified.**

`wait()` ব্যবহার করার সময় সাধারণত condition check করার জন্য **`while`** ব্যবহার করা উচিত, কারণ thread wake up হওয়ার পর condition আবার verify করতে হয়।
<img width="1031" height="699" alt="image" src="https://github.com/user-attachments/assets/a97f73aa-87a8-4a0f-9c77-bd0fa840c678" />

### my code for traditional producer and consumer example
```
import jdk.jfr.StackTrace;

import javax.swing.*;
import java.math.BigInteger;
import java.util.LinkedList;
import java.util.Map;
import java.util.Objects;
import java.util.Queue;
import java.util.concurrent.Callable;

class ShareQueue{
    private final int CAPACITY=10;
    private final Queue<String>queue = new LinkedList<>();
    public synchronized void produce(String task)
    {
        if (queue.size()==CAPACITY)
        {
            try {
               this.wait();

            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }


        }
        queue.add(task);

        this.notifyAll();
    }
    public synchronized String consumer()
    {
        if (queue.isEmpty())
        {
            try {
               this.wait();
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
        String task = queue.poll();
        this.notifyAll();
        return task;

    }

}
public class Main {

    public  static  void main(String[] args) throws InterruptedException {
        ShareQueue task = new ShareQueue();
        for(int i =0;i<5;i++)
        {
            new Thread(()->{
                taskProducer(task);
            }).start();
        }
        for(int i =0;i<5;i++)
        {
            new Thread(()->{
                taskConsumer(task);
            }).start();
        }



    }
static void taskProducer(ShareQueue shareQueue)
{

    int i=0;
    while (true)
    {
        i++;
        String temtask = "task "+i;
        System.out.println("PRODUCER : "+Thread.currentThread().getName()+": by task="+temtask);
        shareQueue.produce(temtask);
        try {
            Thread.sleep(200);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
    static void taskConsumer(ShareQueue shareQueue)
    {

        int i=0;
        while (true)
        {
            i++;
            String temtask = "task "+i;
            System.out.println("consumer  : "+Thread.currentThread().getName()+": by task="+temtask);
            shareQueue.consumer();
            try {
                Thread.sleep(10000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
    }

}
```
# Spurious Wakeup

## What is Spurious Wakeup and how to handle this ?

**Spurious Wakeup** হলো এমন একটি situation যেখানে `wait()` করা একটি thread **`notify()` / `notifyAll()` না হলেও** wake up করতে পারে। 
অর্থাৎ:

```text
Thread
   ↓
 wait()
   ↓
WAITING 
   ↓
Spurious Wakeup
   ↓
Thread wakes up
   ↓
Condition এখনো FALSE ❌
```

তাই `wait()` থেকে ফিরে আসার পর **condition আবার check করা বাধ্যতামূলক**।

---

# ❌ Wrong Approach: `if`

```java
synchronized (queue) {

    if (queue.isEmpty()) {
        queue.wait(); //suddenly somehow thread come waiting stage to runable stage . to handle this uisng while loop .

    }

    String task = queue.remove();
}
```

সমস্যা:

```text
queue.empty = true
      ↓
Consumer → wait()
      ↓
Spurious Wakeup
      ↓
Consumer wakes up
      ↓
queue এখনো empty ❌
      ↓
queue.remove()
      ↓
Exception ❌
```

`if` condition শুধুমাত্র **একবার** check করে।

---

# ✅ Correct Approach: `while`

```java
synchronized (queue) {

    while (queue.isEmpty()) {
        queue.wait();
    }

    String task = queue.remove();
}
```

এখন flow:

```text
Consumer
   ↓
queue empty?
   ↓
 YES
   ↓
 wait()
   ↓
Wake up
   ↓
queue empty?
   ↓
 YES → আবার wait()
   ↓
Producer adds task
   ↓
notify()
   ↓
Consumer wakes
   ↓
queue empty?
   ↓
 NO
   ↓
remove task ✅
```

---

# Why `while`?

`wait()` থেকে ফিরে আসার **প্রতিবার condition পুনরায় verify করতে হবে**।

এটি শুধু **Spurious Wakeup**-এর জন্য নয়।

অন্য কোনো thread condition পরিবর্তন করে ফেললেও `while` condition পুনরায় check করে।

তাই Java concurrency-তে সাধারণ rule:

```java
while (conditionIsNotSatisfied) {
    wait();
}
```

---

# Producer-Consumer Example

```java
class SharedQueue {

    private final Queue<String> queue = new LinkedList<>();
    private final int CAPACITY = 10;

    public synchronized void produce(String task)
            throws InterruptedException {

        while (queue.size() == CAPACITY) {
            wait();
        }

        queue.add(task);

        notifyAll();
    }

    public synchronized String consume()
            throws InterruptedException {

        while (queue.isEmpty()) {
            wait();
        }

        String task = queue.remove();

        notifyAll();

        return task;
    }
}
```

---

## Producer

```java
while (queue.size() == CAPACITY) {
    wait();
}
```

মানে:

> Queue full হলে অপেক্ষা করো। Wake up হওয়ার পর আবার check করো queue এখনো full কিনা।

```text
Queue Full
    ↓
  wait()
    ↓
 Wake up
    ↓
Check again
    ↓
Still Full? → wait()
    ↓
Not Full? → Produce ✅
```

---

## Consumer

```java
while (queue.isEmpty()) {
    wait();
}
```

মানে:

> Queue empty হলে অপেক্ষা করো। Wake up হওয়ার পর আবার check করো task এসেছে কিনা।

```text
Queue Empty
    ↓
  wait()
    ↓
 Wake up
    ↓
Check again
    ↓
Still Empty? → wait()
    ↓
Has Task? → Consume ✅
```

---

# `if` vs `while`

| `if`                               | `while`                        |
| ---------------------------------- | ------------------------------ |
| Condition একবার check করে          | Condition বারবার check করে     |
| Spurious wakeup-এর ক্ষেত্রে unsafe | Spurious wakeup handle করে     |
| Wake up → directly continue        | Wake up → condition আবার check |
| ❌ Not recommended                  | ✅ Recommended                  |

---

## ⭐ Golden Rule

```text
wait() → always use inside while
```

### Remember

> **Never assume that waking up means the condition is true.**

```java
while (conditionIsFalse) {
    wait();
}
```

**Spurious Wakeup → Wake up → Re-check condition → If still false → wait again.**

