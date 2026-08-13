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

CPU
├── Core 1 → Executes instructions
├── Core 2 → Executes instructions
├── Core 3 → Executes instructions
├── Core 4 → Executes instructions
├── Cache
└── Memory Controller
