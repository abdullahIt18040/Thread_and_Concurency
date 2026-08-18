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
## Runnable vs Callable

Java concurrency-তে Runnable এবং Callable দুটোই funtional interface and task define করার জন্য ব্যবহার হয়। তবে প্রধান পার্থক্য হলো return value এবং exception handling।

1. Runnable

Runnable task execute করে, কিন্তু কোনো result return করে না।

Runnable task = () -> {
    int sum = 10 + 20;
    System.out.println(sum);
};


Thread thread = new Thread(task);
thread.start();

Flow:

Runnable
   ↓
run()
   ↓
Task execute
   ↓
No return value

Method:

void run()
2. Callable

Callable task execute করে এবং result return করতে পারে।

Callable<Integer> task = () -> {
    int sum = 10 + 20;
    return sum;
};

Callable সাধারণত ExecutorService দিয়ে ব্যবহার করা হয়:

ExecutorService executor =
        Executors.newSingleThreadExecutor();


Callable<Integer> task = () -> {
    return 10 + 20;
};


Future<Integer> future = executor.submit(task);


Integer result = future.get();


System.out.println("Result = " + result);


executor.shutdown();

Output:

30
``
### ThreadGroup,RunableInterface
``
ThreadGroup group = new ThreadGroup("Workers");


Runnable task = () -> {
    System.out.println("Doing work");
};


Thread thread = new Thread(group, task, "Thread-1");


thread.start();

Flow:

              Thread
                │
       ┌────────┴────────┐
       │                 │
 ThreadGroup        Runnable Target
       │                 │
 "Workers"          "Doing Work"
       │                 │
       └────────┬────────┘
                ↓
             Thread
                ↓
              start()
                ↓
              run()
সহজে মনে রাখুন

Runnable Target → Thread কী কাজ করবে
ThreadGroup → কোন Thread-গুলোকে একটি group-এ রাখব
Thread → কাজটি execute করবে
``
### JVM-এর ThreadGroup hierarchy

সাধারণভাবে:
``
system ThreadGroup
       │
       └── main ThreadGroup
              │
              ├── main Thread
              ├── Thread-1
              └── Thread-2
আপনার main thread

Java application শুরু হলে সাধারণত main thread থাকে:

main Thread
    ↓
main ThreadGroup

যদি আপনি নতুন thread তৈরি করেন:

Thread t = new Thread(() -> {
    System.out.println("Hello");
});

তাহলে নতুন thread সাধারণত যে thread থেকে তৈরি হয়েছে, সেই thread-এর ThreadGroup inherit করে।

অর্থাৎ:

main ThreadGroup
│
├── main
├── Thread-0
└── Thread-1
নতুন ThreadGroup তৈরি করলে
ThreadGroup group =
    new ThreadGroup("WorkerGroup");

এখানে আপনি parent না দিলে, current thread-এর group সাধারণত parent হয়।

main ThreadGroup
       │
       └── WorkerGroup
              ├── Worker-1
              └── Worker-2
সহজে মনে রাখুন

Thread-এরও একটি ThreadGroup থাকে।
নতুন Thread সাধারণত creator Thread-এর ThreadGroup inherit করে।
ThreadGroup-এরও parent ThreadGroup থাকতে পারে।
``
