## Basic concept of java thread


<img width="628" height="313" alt="image" src="https://github.com/user-attachments/assets/04275b66-6f51-428b-8639-91d973ccfdf3" />

### What is a Thread?

Thread হলো একটি Process-এর ভিতরের একটি execution path, যার মাধ্যমে program-এর instructions execute হয়।
```
সহজভাবে:

Thread = একটি কাজ execute করার পথ।

Example

একটি Java application:

Java Application
      ↓
    Process
      │
      ├── Thread-1 → Calculate Sum
      ├── Thread-2 → Read File
      └── Thread-3 → Database Operation

একটি Thread-এর কাজ:

Thread
  ↓
Instructions
  ↓
OS Scheduler
  ↓
CPU Core
  ↓
Fetch → Decode → Execute
Java Example
class MyTask implements Runnable {


    @Override
    public void run() {
        System.out.println("Task is running");
    }
}


public class Main {
    public static void main(String[] args) {


        MyTask task = new MyTask();


        Thread thread = new Thread(task);


        thread.start();
    }
}

Flow:

MyTask
  ↓
Runnable
  ↓
Thread Object
  ↓
start()
  ↓
run()
  ↓
Task Execute
Process vs Thread
Process = Running Program
Thread  = Execution Path inside Process

একটি Process-এর একাধিক Thread থাকতে পারে:

Process
├── Thread-1
├── Thread-2
└── Thread-3
```
### run() Method কী?
```
Java-তে run() method-এর ভিতরে Thread যে কাজটি করবে সেই task-এর code লেখা হয়।

সহজভাবে:

run() = Thread-এর কাজ

Simple Example
class MyTask implements Runnable {


    @Override
    public void run() {
        System.out.println("Task is running");
    }
}

এখানে:

Runnable
   ↓
run()
   ↓
"Task is running"

তারপর Thread তৈরি করি:

public class Main {


    public static void main(String[] args) {


        MyTask task = new MyTask();


        Thread thread = new Thread(task);


        thread.start();
    }
}
```
