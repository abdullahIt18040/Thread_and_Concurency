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
```
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
```
### ThreadGroup,RunableInterface
```
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
আপনার output-টা আসলে JVM-এর ThreadGroup hierarchy দেখাচ্ছে। একদম সহজভাবে ভেঙে দেখি।

1. পুরো Structure

আপনার output:

system ThreadGroup
│
├── Reference Handler
├── Finalizer
├── Signal Dispatcher
├── Attach Listener
├── Notification Thread
│
├── main ThreadGroup
│   ├── main
│   └── Monitor Ctrl-Break
│
└── InnocuousThreadGroup
    └── Common-Cleaner

অর্থাৎ সব Thread একই ThreadGroup-এর মধ্যে নেই।

2. system ThreadGroup
java.lang.ThreadGroup[name=system,maxpri=10]

এটা JVM-এর একটি top-level ThreadGroup।

এর মধ্যে JVM-এর কিছু internal/system thread আছে।

যেমন:

Reference Handler
Finalizer
Signal Dispatcher
Attach Listener
Notification Thread

এগুলো আপনার application-এর business logic-এর thread নয়; JVM/runtime-এর বিভিন্ন internal কাজের জন্য ব্যবহৃত হয়।

3. main ThreadGroup
java.lang.ThreadGroup[name=main,maxpri=10]

এর মধ্যে:

Thread[main,5,main]
Thread[Monitor Ctrl-Break,5,main]
Thread[main,5,main]

এটাই আপনার Java program-এর main thread।

Format:

Thread[name, priority, group]

তাই:

Thread[main,5,main]

মানে:

Name     = main
Priority = 5
Group    = main
4. Monitor Ctrl-Break
Thread[Monitor Ctrl-Break,5,main]

এটা আপনার IDE/JVM environment-এর একটি monitoring-related thread হতে পারে।

এটাও main ThreadGroup-এর মধ্যে আছে।

5. InnocuousThreadGroup
java.lang.ThreadGroup[name=InnocuousThreadGroup,maxpri=10]

এর মধ্যে:

Thread[Common-Cleaner,8,InnocuousThreadGroup]

Common-Cleaner হলো JVM-এর cleanup-related background thread।


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
```
### Thread Stack Memory
```
প্রতিটি Thread-এর জন্য JVM আলাদা Stack Memory তৈরি করে।

এই Stack-এ Thread-এর:

Method call
Local variables
Method parameters
Execution information

রাখা হয়।

Process
│
├── Heap (Shared)
│
├── Thread-1
│    └── Stack (Own)
│
├── Thread-2
│    └── Stack (Own)
│
└── Thread-3
     └── Stack (Own)
গুরুত্বপূর্ণ

Heap → একই Process-এর Thread-গুলো share করে
Stack → প্রতিটি Thread-এর নিজের আলাদা থাকে

প্রতিটি method call-এর সময় Stack-এ একটি Stack Frame তৈরি হয়।

Thread-1 Stack
┌──────────────┐
│ methodB()    │
├──────────────┤
│ methodA()    │
├──────────────┤
│ main()       │
└──────────────┘

Short Note:

JVM creates a separate stack for each thread, which stores method calls, local variables, parameters, and execution state.


Stack Frame-এর মধ্যে কী থাকে?

সাধারণভাবে একটি Stack Frame-এ থাকে:

Stack Frame
┌──────────────────────┐
│ Local Variables      │
│ Method Parameters    │
│ Return Information   │
│ Operand Stack        │
│ Reference to runtime │
└──────────────────────┘
Example
public static void main(String[] args) {
    int a = 10;
    int b = 20;


    int result = sum(a, b);
}


static int sum(int x, int y) {
    int total = x + y;
    return total;
}

যখন main() শুরু হয়:

Thread Stack
┌─────────────────┐
│ main() Frame    │
│ a = 10          │
│ b = 20           │
└─────────────────┘

তারপর:

sum(a, b);

call হলে নতুন Frame তৈরি হয়:

Thread Stack
┌─────────────────┐
│ sum() Frame     │
│ x = 10          │
│ y = 20          │
│ total = 30      │
├─────────────────┤
│ main() Frame    │
│ a = 10          │
│ b = 20          │
└─────────────────┘

sum() শেষ হলে তার Frame Stack থেকে remove/pop হয়ে যায়:

Thread Stack
┌─────────────────┐
│ main() Frame    │
│ a = 10          │
│ b = 20          │
│ result = 30     │
└─────────────────┘
সহজভাবে মনে রাখুন

Method call → Stack Frame তৈরি
Method-এর local data → Frame-এ থাকে
Method শেষ → Frame remove/pop
```
## how to thread create and work another path
```
class Task implements Runnable{
    @Override
    public void run() {
        for(int i=0;i<10;i++)
        {
            System.out.println(i+" executed by "+Thread.currentThread().getName());
        }
    }
}
public class Main {
    public static void main(String[] args) {
   //main thread executed
        System.out.println(" start executed by "+Thread.currentThread().getName());

//        task
        Runnable target = new Task();
        // create new path to execute task
        Thread thread = new Thread(target); // different thread executed
        thread.start();// different thread executed  start here

        //main thread executed
        for(int i=0;i<10;i++)
        {
            System.out.println(i+" executed by "+Thread.currentThread().getName());
        }
        System.out.println("end executed by "+Thread.currentThread().getName());
    }
}
```
