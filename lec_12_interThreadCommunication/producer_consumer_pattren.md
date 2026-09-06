### Traditional Producer-Consumer Pattern .

<img width="1083" height="583" alt="image" src="https://github.com/user-attachments/assets/d82a7ed6-c942-4c17-9a58-201ba890d036" />

## Busy Waiting কী?

```
Busy Waiting হলো এমন একটি situation যেখানে একটি thread কোনো condition true হওয়ার জন্য বারবার check করতে থাকে, কিন্তু কাজ না থাকলেও CPU ব্যবহার করতে থাকে।

সহজভাবে:

Thread ঘুমাচ্ছে না, কাজও করছে না—শুধু বারবার condition check করছে।

Example
boolean dataAvailable = false;

while (!dataAvailable) {
    // বারবার check করছে
}

এখানে Thread:

dataAvailable?
     ↓
   false
     ↓
আবার check
     ↓
   false
     ↓
আবার check
     ↓
   false
     ↓
...

এই loop চলার সময় CPU continuously ব্যবহার হচ্ছে।

Practical Producer-Consumer Example

ধরো Consumer data-এর জন্য অপেক্ষা করছে:

while (queue.isEmpty()) {
    // অপেক্ষা
}

Queue empty থাকা পর্যন্ত Consumer বারবার:

queue.isEmpty()
queue.isEmpty()
queue.isEmpty()
queue.isEmpty()
...

check করছে।

এটাই Busy Waiting।

wait() ব্যবহার করলে কী হয়?

Busy waiting-এর পরিবর্তে:

synchronized (queue) {

    while (queue.isEmpty()) {
        queue.wait();
    }

    // consume
}

এখন:

Queue empty
     ↓
   wait()
     ↓
Thread WAITING
     ↓
CPU release
     ↓
Producer data দেয়
     ↓
notify()
     ↓
Thread wake up
     ↓
condition check

এখানে thread continuously CPU ব্যবহার করে condition check করছে না।

Busy Waiting vs wait()
Busy Waiting	wait()
বারবার condition check করে	অপেক্ষা করে
CPU ব্যবহার করে	CPU ব্যবহার করে না
while loop continuously চলে	Thread WAITING state-এ যায়
CPU waste হতে পারে	CPU efficient
Simple কিন্তু inefficient	Better coordination
মনে রাখার সহজ উদাহরণ

Busy Waiting:

দরজার সামনে দাঁড়িয়ে প্রতি ১ সেকেন্ডে বলছি—"দরজা খুলেছে?" 😄

wait():

দরজার সামনে বসে আছি; দরজা খুললে আমাকে ডাকতে বলেছি।

Busy Waiting
Thread → check → check → check → check → check
              CPU ব্যবহার হচ্ছে


wait()
Thread → WAITING
              ↓
         notify()
              ↓
          wake up
⭐ GitHub Note

Busy Waiting: A thread continuously checks a condition in a loop while waiting for an event, consuming CPU unnecessarily. wait() is preferred when appropriate because the thread enters WAITING state and releases the CPU until it is notified.**
```
