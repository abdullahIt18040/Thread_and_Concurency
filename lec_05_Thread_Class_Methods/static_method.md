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
### Thread.enumerate() কী?
```

Thread.enumerate() হলো Java-এর একটি static method, যা current thread-এর ThreadGroup এবং তার subgroups-এর active thread-গুলোকে একটি Thread[] array-এর মধ্যে রাখে।

সহজভাবে:

activeCount() → কতগুলো active thread আছে
enumerate() → সেই active thread-গুলোর reference দেয়

Example
public class Main {
    public static void main(String[] args) {

        Thread t1 = new Thread(() -> {
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }, "Worker-1");

        Thread t2 = new Thread(() -> {
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }, "Worker-2");

        t1.start();
        t2.start();

        Thread[] threads = new Thread[Thread.activeCount()];

        int count = Thread.enumerate(threads);

        System.out.println("Active threads: " + count);

        for (Thread thread : threads) {
            if (thread != null) {
                System.out.println(thread.getName());
            }
        }
    }
}

সম্ভাব্য output:

Active threads: 3
main
Worker-1
Worker-2
```
