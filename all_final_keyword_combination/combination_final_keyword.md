### Java-তে final অনেক জায়গায় ব্যবহার করা যায়। সবচেয়ে গুরুত্বপূর্ণ সব common combination নিচে দিলাম।
```
1. final Local Variable
final int x = 10;

x = 20; // ❌

একবার value assign করার পর আবার assign করা যাবে না।

final variable → value cannot be reassigned
2. final Instance Variable
class Student {

    final int age = 20;
}

প্রতিটি object-এর age একবার set হলে পরিবর্তন করা যাবে না।

তবে constructor দিয়ে set করা যায়:

class Student {

    final int age;

    Student(int age) {
        this.age = age;
    }
}
Student s = new Student(25);

এখানে age পরে পরিবর্তন করা যাবে না।

3. static final

এটা খুব গুরুত্বপূর্ণ।

public static final int MAX_SIZE = 10;

এখানে:

static → class-level
final → value পরিবর্তন করা যাবে না
Class
  |
  └── MAX_SIZE = 10

Access:

System.out.println(Student.MAX_SIZE);

সাধারণত constants বানাতে ব্যবহার করা হয়।

public static final double PI = 3.14159;
4. final Reference
final List<Integer> list = new ArrayList<>();

এখানে reference পরিবর্তন করা যাবে না:

list = new ArrayList<>(); // ❌

কিন্তু object-এর ভিতরের data পরিবর্তন করা যাবে:

list.add(10); // ✅
list.add(20); // ✅
list.remove(0); // ✅

কারণ:

list ───────────→ ArrayList Object
 ↑
final reference

final object-কে immutable করে না।

5. final Method
class Parent {

    final void show() {
        System.out.println("Hello");
    }
}

Child class এই method override করতে পারবে না।

class Child extends Parent {

    @Override
    void show() { } // ❌
}
উদ্দেশ্য

এই method-এর implementation subclass পরিবর্তন করতে পারবে না।

6. final Class
final class Parent {
}

এই class-কে extend করা যাবে না।

class Child extends Parent { // ❌
}

Example:

final class Student {
}

Student থেকে subclass তৈরি করা যাবে না।

Java-এর:

String

class-টিও final।

7. final Parameter
void calculate(final int value) {

    value = 20; // ❌
}

Method-এর ভিতরে parameter-টাকে reassign করা যাবে না।

calculate(10);
8. final Object Parameter
void add(final List<Integer> list) {

    list.add(10); // ✅

    list = new ArrayList<>(); // ❌
}

অর্থাৎ:

final reference
       ↓
reference পরিবর্তন ❌
object পরিবর্তন    ✅
9. final + Constructor
class Employee {

    final int id;

    Employee(int id) {
        this.id = id;
    }
}

এখানে id declaration-এর সময় initialize করা হয়নি।

কিন্তু constructor-এ একবার assign করা হয়েছে।

Employee e = new Employee(101);

এরপর:

e.id = 102; // ❌
10. static final Constant

সবচেয়ে common combination:

public static final int MAX_USERS = 100;

মানে:

static → class-এর একটি copy
final  → value পরিবর্তন করা যাবে না

তাই:

MAX_USERS = 200; // ❌
11. private final
private final int capacity = 10;

এখানে:

private → শুধু class-এর ভিতরে access
final   → reassign করা যাবে না

Producer-Consumer-এ:

private final int capacity = 10;

খুব common।

12. public final
public final int capacity = 10;

মানে:

public → অন্য class থেকেও access করা যাবে
final  → value reassign করা যাবে না

কিন্তু যদি এটা instance variable হয়:

public final int capacity = 10;

তাহলে প্রতিটি object-এর নিজের capacity থাকবে।

13. private static final

খুব common constant declaration:

private static final int MAX_SIZE = 10;

মানে:

private → শুধু class-এর ভিতরে
static  → class-level
final   → পরিবর্তন করা যাবে না
14. public static final

Public constant:

public static final int MAX_SIZE = 10;

অন্য class থেকেও:

System.out.println(MyClass.MAX_SIZE);

access করা যাবে।

15. final + abstract Method ❌

এটা করা যায় না:

abstract final void show();

কারণ:

abstract → subclass অবশ্যই override করবে
final    → subclass override করতে পারবে না

দুটো contradictory।

16. final + abstract Class ❌
final abstract class A {
}

❌ Invalid।

কারণ:

abstract → extend করার জন্য
final    → extend করা যাবে না
17. final + synchronized Method ✅

এটা valid:

final synchronized void test() {
}

মানে:

final       → override করা যাবে না
synchronized → একসাথে controlled access
18. final + static Method ✅
static final void test() {
}

তবে static method technically override হয় না; এটি hide করা যায়। final দিলে subclass-এ একই signature দিয়ে hide-ও করা যাবে না।

Quick Cheat Sheet
Combination	Meaning
final int x	value reassign করা যাবে না
final Object obj	reference reassign করা যাবে না
final method()	override করা যাবে না
final class	extend করা যাবে না
final parameter	parameter reassign করা যাবে না
static final	class-level constant
private final	private + cannot reassign
public final	public + cannot reassign
private static final	private class constant
public static final	public class constant
final synchronized method	cannot override + synchronization
final abstract method	❌ Invalid
final abstract class	❌ Invalid
সবচেয়ে গুরুত্বপূর্ণ ৩টা মনে রাখুন
final variable

→ reassign করা যাবে না

final method

→ override করা যাবে না

final class

→ extend করা যাবে না

আর:

static final

→ class-level constant।
```
