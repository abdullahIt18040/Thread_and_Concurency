## instance all MEthod with description 
### code-এ start() এবং run() কীভাবে কাজ করে সেটা সহজভাবে দেখি।
'''


আপনার Code
Thread t = new Thread(() -> {
    System.out.println("hello the thread occured by " 
            + Thread.currentThread().getName());
});

t.setName("task");
t.start();
কী হচ্ছে?
1. Thread Object তৈরি
Thread t = new Thread(() -> {
    // task
});

এখানে t নামে একটি Thread object তৈরি হয়েছে।

এর initial state:

NEW

Lambda:

() -> {
    System.out.println(...);
}

এটাই Thread-এর Runnable task, যা পরে run()-এর মাধ্যমে execute হবে।

2. Thread-এর নাম set
t.setName("task");

এখন:

Thread Name = task
State       = NEW
3. start() call
t.start();

এখানে নতুন thread-এর execution শুরু করার request হয়।

Conceptual flow:

t.start()
   ↓
start0() / JVM Native Layer
   ↓
OS Thread Execution
   ↓
RUNNABLE
   ↓
OS Scheduler
   ↓
CPU
   ↓
run()
   ↓
Lambda Task Execute

তখন output হবে:

hello the thread occured by task

কারণ:

Thread.currentThread().getName()

বর্তমানে execute করা thread-এর নাম "task" return করে।

start() vs run()
start() করলে
t.start();
main Thread
     │
     └──────→ task Thread
                  ↓
                run()
                  ↓
              Lambda

নতুন thread তৈরি/শুরু হয়।

run() সরাসরি call করলে
t.run();
main Thread
     ↓
   run()
     ↓
  Lambda

নতুন thread তৈরি হয় না। main thread-ই task execute করে।

তখন output হবে:

hello the thread occured by main
মনে রাখুন

start() → নতুন Thread-এর execution শুরু করে → run() execute হয়।

run() → শুধু task execute করে → নতুন Thread তৈরি করে না।
'''
