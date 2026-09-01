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
