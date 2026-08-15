<img width="1091" height="588" alt="image" src="https://github.com/user-attachments/assets/4e38a25e-30d7-4fba-9023-4936eb53561e" />


process হলো একটি চলমান Program, Thread হলো সেই Program-এর কাজ করার পথ (execution path), আর CPU core  সরাসরি Thread-এর instructions execute করে। Operating System-এর Scheduler ঠিক করে কোন Thread কখন CPU পাবে।

CPU Memory থেকে instruction Fetch করে, instruction Decode করে বুঝে, তারপর Execute করে। এরপর পরবর্তী instruction-এর জন্য একই cycle repeat করে।

CPU (Central Processing Unit)
--------------------------------
- CPU = the whole processor.
- It contains CPU cores, cache, memory controller, etc.
- CPU is a single processor unit.
- Actual instructions are executed by the CPU cores.
- A CPU can have multiple cores.

Example:
```
CPU
├── Core 1 → Executes instructions
├── Core 2 → Executes instructions
├── Core 3 → Executes instructions
├── Core 4 → Executes instructions
├── Cache
└── Memory Controller
Single-Core vs Multi-Core Processor
Single-Core Processor

A single-core processor has only one CPU core to execute instructions.

CPU
└── Core 1
    └── Executes instructions
One physical core
Executes one instruction stream at a time
Limited multitasking performance
Older processors commonly used this design
Multi-Core Processor

A multi-core processor contains multiple CPU cores inside one CPU.

CPU
├── Core 1 → Executes instructions
├── Core 2 → Executes instructions
├── Core 3 → Executes instructions
└── Core 4 → Executes instructions
Multiple physical cores
Cores can execute instructions concurrently
Better multitasking and parallel processing
Modern CPUs commonly have multiple cores
```
### CPU-Bound vs I/O-Bound Tasks
```
1. CPU-Bound Task

যে task-এ মূল কাজ CPU-এর calculation/processing, তাকে CPU-bound task বলে।

Examples:

Mathematical calculation
Sorting
Image processing
Encryption
Complex algorithms
Thread → CPU Core → Calculation

Key Point: CPU-bound task-এ bottleneck হলো CPU।
তাই বেশি Thread সবসময় performance বাড়ায় না।

2. I/O-Bound Task

যে task-এ Thread-কে Input/Output operation-এর জন্য অপেক্ষা করতে হয়, তাকে I/O-bound task বলে।

Examples:

Database query
File read/write
HTTP/API call
Network operation
Kafka message
Thread
  ↓
I/O Request
  ↓
WAITING ⏳
  ↓
Response
  ↓
Processing

Key Point: I/O-bound task-এ bottleneck হলো I/O waiting।

3. Thread & Concurrency
CPU-Bound:
Thread → CPU → Calculation


I/O-Bound:
Thread → I/O → WAITING → Response

Concurrency: একাধিক task-এর progress একসাথে manage করা।

Parallelism: একাধিক CPU core-এ একাধিক task একই সময়ে execute করা।
```
### 1 Core and 2 Thread মানে 1টি physical core এবং 2টি logical processor;
```
1. Basic Concept

একটি CPU Core-এর সাথে Physical Core, Hardware Thread, এবং Logical CPU concept জড়িত।

যদি একটি CPU-তে:

1 Physical Core
2 Hardware Threads

থাকে, তাহলে Operating System সাধারণত সেটিকে:

2 Logical CPUs / Logical Processors

হিসেবে দেখতে পারে।

2. 1 Core / 2 Thread
CPU
└── Physical Core 1
      ├── Hardware Thread 1
      └── Hardware Thread 2

OS-এর কাছে:

CPU
├── Logical CPU 1
└── Logical CPU 2

অর্থাৎ:

Physical Core = 1
Hardware Threads = 2
Logical CPUs = 2
3. Physical Core

Physical Core হলো CPU-এর আসল hardware processing core।

CPU
└── Core 1

একটি CPU-তে একাধিক physical core থাকতে পারে:

CPU
├── Core 1
├── Core 2
├── Core 3
└── Core 4

এগুলো হলো 4 Physical Cores।

4. Hardware Thread

SMT (Simultaneous Multithreading) বা Intel-এর ক্ষেত্রে Hyper-Threading ব্যবহার করলে একটি physical core একাধিক hardware execution context support করতে পারে।

Physical Core 1
      │
      ├── Hardware Thread 1
      └── Hardware Thread 2

এখানে দুইটি hardware thread একই physical core-এর অনেক hardware resources share করে।

Hardware thread ≠ Physical core

5. Logical CPU / Logical Processor

Operating System hardware threads-কে সাধারণত Logical CPUs হিসেবে দেখে।

Physical Core 1
      │
      ├── Logical CPU 1
      └── Logical CPU 2

তাই:

1 Physical Core
       ↓
2 Hardware Threads
       ↓
2 Logical CPUs
```
### CPU Cache and it level
```
CPU Cache হলো CPU- core এর খুব দ্রুত ছোট memory, যেখানে frequently used data ও instructions সাময়িকভাবে রাখা হয়।

CPU Core
   ↓
L1 Cache
   ↓
L2 Cache
   ↓
L3 Cache (data একাধিক Core-এর মধ্যে shared kore )
   ↓
RAM
Cache Levels
Level	Speed	Size	সাধারণ বৈশিষ্ট্য
L1	Fastest	Small	Core-এর সবচেয়ে কাছে
L2	Fast	Medium	L1-এর চেয়ে বড়
L3	Slower	Large	অনেক CPU-তে multiple core-এর মধ্যে shared
আপনার CPU
L1 = 256 KB
L2 = 1 MB
L3 = 6 MB
Cache Hit vs Miss
Cache Hit:
CPU → Cache → Data ✓


Cache Miss:
CPU → L1 → L2 → L3 → RAM

মনে রাখুন:

L1 = ছোট + সবচেয়ে দ্রুত
L2 = মাঝারি
L3 = বড় + তুলনামূলক ধীর
```
### Process and Thread
1. Process(Memeory Share kore na) 

Process হলো একটি running program।

যখন কোনো program ,Applicationn চালু করি, Operating System সেটিকে একটি process হিসেবে চালায়।

Examples:

Chrome       → Process
IntelliJ     → Process
Java App     → Process
Process Memory

প্রতিটি process সাধারণত নিজের   Own Memory space পায়।

Process-1              Process-2
┌────────────┐         ┌────────────┐
│ Code       │         │ Code       │
│ Data       │         │ Data       │
│ Heap       │         │ Heap       │
│ Stack      │         │ Stack      │
└────────────┘         └────────────┘
   Own Memory             Own Memory

এক Process সাধারণত অন্য Process-এর memory সরাসরি access করতে পারে না।

এটাকে Process Isolation বলা হয়।

2. Thread (memory share kore)

Thread হলো Process-এর ভিতরের execution path।

একটি Process-এর মধ্যে এক বা একাধিক Thread থাকতে পারে।

Process
│
├── Thread-1
├── Thread-2
└── Thread-3

Java example:

Java Application
       ↓
     Process
       │
       ├── Main Thread
       ├── Worker Thread
       └── Async Thread
3. Thread Memory

একই Process-এর Thread-গুলো সাধারণত কিছু memory share করে:

Process
│
├── Shared Code
├── Shared Heap
├── Shared Data
│
├── Thread-1 → Own Stack
├── Thread-2 → Own Stack
└── Thread-3 → Own Stack
Shared
Heap
Code
Global/Static Data
Per Thread
Stack
Program Counter
Registers / execution state
``
### Concurrency vs Parallelism
``
2 ta program er 2 instruction same time 2 ta core e execute hoi , tokhon eta Parallelism , 2 ta program er 2 instruction 1 core e one by one execute hoi tohon take Concurrency.
Parallelism

যখন একাধিক task/thread একই সময়ে একাধিক CPU core-এ সত্যিকারভাবে execute হয়, তখন তাকে Parallelism বলে।

Example:

2 Tasks + 2 CPU Cores


Core 1 → Instruction 1
Core 2 → Instruction 2

দুইটি instruction একই সময়ে execute হচ্ছে।

Parallelism = Multiple tasks execute simultaneously.

Concurrency

যখন একাধিক task/thread একই সময়ে active/progress করছে, কিন্তু একটি core-এ CPU তাদের একটির পর একটি execute করে এবং প্রয়োজনে context switching করে, তখন তাকে Concurrency বলে।

Example:

1 CPU Core


Instruction 1
     ↓
Context Switch
     ↓
Instruction 2
     ↓
Context Switch
     ↓
Instruction 1

Concurrency = Multiple tasks-এর overlapping progress.
``
