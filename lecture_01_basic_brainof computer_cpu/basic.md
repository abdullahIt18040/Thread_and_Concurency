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
