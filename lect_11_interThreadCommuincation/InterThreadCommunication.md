### Inter-Thread Communication কী?
```
Inter-Thread Communication হলো একই process-এর একাধিক thread-এর মধ্যে এক thread থেকে অন্য thread-কে communicatiate or  signal/data সম্পর্কে জানানো বা কাজের coordination করা।
inter thread commnication Java-তে সাধারণত এগুলোর মাধ্যমে করা হয়:
---------object class-----
-------native method------
------should acqure lock---------
wait()
notify()
notifyAll()

এগুলো Object class-এর method।
wait(), notify(), notifyAll() নির্দিষ্ট Object-এর intrinsic lock/monitor-এর উপর কাজ করে। এগুলো call করার আগে সেই Object-এর lock acquired থাকতে হয়। তাই এই methods Object class-এর অন্তর্ভুক্ত, Thread class-এর নয়।
আর মনে রাখবেন:

wait()   → lock release করে → WAITING
notify() → waiting thread-কে signal করে this threat working  again.

মূল ধারণা

Java-তে প্রতিটি object-এর নিজের একটি intrinsic lock (monitor) থাকে।

Object obj = new Object();

এখানে obj-এর একটি নিজস্ব lock আছে।

wait(), notify(), notifyAll() আসলে thread-এর control করার জন্য নয়; object-এর monitor/lock-এর সাথে coordination করার জন্য।

কেন Object class-এ?

ধরো:

synchronized (obj) {
    obj.wait();
}

এখানে Thread বলছে:

"আমি obj-এর lock ধরে আছি। এখন আমি অপেক্ষা করব।"

wait() করলে:

Thread
  |
  | wait()
  ↓
obj-এর lock release
  |
  ↓
Waiting state

অন্য Thread:

synchronized (obj) {
    obj.notify();
}

বলছে:

"obj-এর জন্য যারা অপেক্ষা করছে, তাদের মধ্যে একজনকে জাগাও।"

অর্থাৎ waiting এবং notification নির্দিষ্ট object-এর monitor-এর সাথে সম্পর্কিত।

যদি এগুলো Thread class-এ থাকত?

ধরো এমন হতো:

thread.wait();
thread.notify();

তাহলে প্রশ্ন হবে:

কোন lock-এর জন্য thread wait করছে?

কারণ একই Thread অন্য অনেক object-এর lock নিয়ে কাজ করতে পারে।

যেমন:

synchronized (obj1) {
    // obj1 lock
}

synchronized (obj2) {
    // obj2 lock
}

Thread-কে শুধু দেখে বোঝা যায় না সে কোন object-এর monitor-এর জন্য অপেক্ষা করছে।

তাই Java design করেছে:

Object
 ├── intrinsic lock / monitor
 ├── wait()
 ├── notify()
 └── notifyAll()
সবচেয়ে গুরুত্বপূর্ণ লাইন

wait(), notify(), notifyAll() Thread-এর method নয়, কারণ এগুলো Thread-কে নয়—একটি নির্দিষ্ট Object-এর monitor/lock-এর সাথে coordination করে।

তাই:

synchronized (obj) {
    obj.wait();
}

এখানে Thread wait করছে obj-এর monitor-এর উপর।

এ কারণেই এগুলো Object class-এর method।

গুরুত্বপূর্ণ পার্থক্য

wait() করার সময় lock acquire থাকতে হবে, কিন্তু wait() করার পর thread lock release করে দেয়।

wait() call করার আগে
       ↓
   Lock acquired
       ↓
     wait()
       ↓
   Lock released
       ↓
   Waiting

আর notify() lock release করে না। notify() করার পর synchronized block শেষ হলে lock release হয়।

Lock acquired
     ↓
 notify()
     ↓
Still holding lock
     ↓
synchronized block শেষ
     ↓
Lock released
```
