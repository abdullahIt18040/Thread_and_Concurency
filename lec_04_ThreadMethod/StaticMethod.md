## All Static method.
 THREAD MAX_PRIORITY :   
```
Thread.MAX_PRIORITY
System.out.println(Thread.MAX_PRIORITY);

Output:

10

Java-তে Thread priority-এর range:

MIN_PRIORITY = 1
NORM_PRIORITY = 5
MAX_PRIORITY = 10
তিনটি গুরুত্বপূর্ণ Priority Constant
Constant	Value	অর্থ
Thread.MIN_PRIORITY	1	সর্বনিম্ন priority
Thread.NORM_PRIORITY	5	Default priority
Thread.MAX_PRIORITY	10	সর্বোচ্চ priority

উদাহরণ:

Thread t = new Thread();

t.setPriority(Thread.MAX_PRIORITY);

System.out.println(t.getPriority());

Output:

10
গুরুত্বপূর্ণ

MAX_PRIORITY CPU পাওয়ার guarantee দেয় না।

MAX_PRIORITY = 10
        ↓
Scheduler-এর কাছে higher-priority hint
        ↓
কিন্তু অবশ্যই আগে execute হবে — এমন guarantee নেই

তাই মনে রাখবেন:

Thread.MAX_PRIORITY = priority-এর সর্বোচ্চ মান (10), কিন্তু এটি static method নয়; static final constant।

```
## Thread Priority Inheritance — 
```
নতুন Thread তৈরি হলে, সে সাধারণত creating thread-এর priority inherit করে।
Creating thread-কে এখানে informalভাবে parent thread বলা যায়।
Default priority: NORM_PRIORITY = 5
Maximum: MAX_PRIORITY = 10
Minimum: MIN_PRIORITY = 1
Thread.currentThread().setPriority(8);

Thread child = new Thread(() -> {
    System.out.println(Thread.currentThread().getPriority());
});

Flow:

Parent Thread (Priority 8)
          ↓ creates
Child Thread (Priority 8)

New Thread-এর initial priority = creating Thread-এর priority।
```
