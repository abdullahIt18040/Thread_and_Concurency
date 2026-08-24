## Three Levels of Thread
```
1. Application-Level Thread
Java application-এ আমরা যে Thread তৈরি করি।
Example: new Thread(task)
2. OS-Level Thread
JVM/native layer-এর মাধ্যমে application thread OS thread-এর সাথে mapped হয়।
OS এই thread-গুলো manage ও schedule করে।
3. Hardware-Level Thread
CPU-এর logical processor / hardware execution context।
OS Scheduler OS thread-কে এখানে schedule করে।
এরপর CPU core instructions execute করে।
Overall Flow

Application Thread → JVM/Native → OS Thread → OS Scheduler → Hardware/Logical Processor → CPU Core → Instruction Execute
```
## Java Thread → OS Thread Mapping
```
Application-level Thread → JVM-এর মাধ্যমে native method/code ব্যবহার করে OS-level thread-এর সাথে যুক্ত হয়।
Thread.start() → JVM-এর internal/native mechanism-এর মাধ্যমে OS thread শুরু করার ব্যবস্থা করে।
OS Scheduler → OS-level thread-কে logical processor-এ schedule করে।
CPU Core → instruction execute করে।
Result → application-এর thread execution-এর মাধ্যমে পাওয়া যায়।

Java Thread → Native/JVM → OS Thread → OS Scheduler → Logical Processor → CPU Core → Execute
```
## private native void start0() ki ki kore ;
```
1.os ke bole ekta thread lagbe.os ekta thread creat kore
2.application thread sate os thread map or assign kore
 3.ei thread jonnno memory allocate kore.
 4.ei thread ke runable state niye jai
```
## start() vs run() in Java Thread
```
start() → নতুন Thread শুরু করে → run() execute করে

run() → শুধু task-এর code execute করে; নিজে নতুন Thread তৈরি করে না।

start()
  ↓
New Thread
  ↓
run()
  ↓
Task Execute

কিন্তু:

run()
  ↓
Current Thread
  ↓
Task Execute
```
## What Happens When start() Is Called? 
```
start() → start0() → JVM/Native Layer-এর মাধ্যমে OS-level thread execution শুরু হয় → Thread Runnable State-এ যায় → OS Scheduler CPU-তে schedule করে → run() method execute হয়।
```
