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
