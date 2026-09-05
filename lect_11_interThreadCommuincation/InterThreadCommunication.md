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

```
