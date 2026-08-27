## Thread.activeCount() 

```

Thread.activeCount() হলো Java-এর একটি static method, যা current thread-এর ThreadGroup এবং তার subgroup-গুলোতে কতগুলো active thread আছে তার আনুমানিক সংখ্যা return করে।

Short Note
Current Thread
      ↓
ThreadGroup
      ↓
SubGroups
      ↓
Active Threads Count
      ↓
Thread.activeCount()

Important: এখানে activeCount() পুরো JVM-এর সব thread count করে না; current thread-এর ThreadGroup ও তার subgroups-এর active thread count করে।

```
