---
title: "Java 面试题整理 (二)"
date: 2021-06-21T16:23:19+08:00
categories:
- java
tags:
- java
- Interview
---
#### 1. Difference between String, StringBuffer, and StringBuilder in Java? 
String is immutable while both StringBuffer and StringBuilder are mutable, which means any change e.g. converting String to upper case or trimming white space will produce another instance rather than changing the same instance. In later two, StringBuffer is synchronized while StringBuilder is not, in fact, it's a ditto replacement of StringBuffer added in Java 1.5.

<!--more-->
#### 2. Difference between extending Thread vs implementing Runnable in Java? 
The difference comes from the fact that you can only extend one class in Java, which means if you extend the Thread class you lose your opportunity to extend another class, on the other hand, if you implement Runnable, you can still extend another class.

#### 3. Difference between Runnable and Callable interface in Java?
Runnable was the only way to implement a task in Java that can be executed in parallel before JDK 1.5 adds Callable. Just like Runnable, Callable also defines a single call() method but unlike run() it can return values and throw exceptions.

#### 4. Difference between ArrayList and LinkedList in Java? 
In short, ArrayList is backed by an array in Java, while LinkedList is just a collection of nodes, similar to a linked list data structure. ArrayList also provides a random search if you know the index, while LinkedList only allows a sequential search. On other hand, adding and removing an element from the middle is efficient in LinkedList as compared to ArrayList because it only requires modifying links, and no other element is rearranged.


#### 5. What is the difference between wait and notify in Java? 
Both wait and notify methods are used for inter-thread communication, where the wait is used to pause the thread on a condition, and notify is used to send a notification to waiting threads. Both must be called from synchronized context e.g. synchronized method or block.


#### 6. Difference between HashMap and Hashtable in Java? 
Though both HashMap and Hashtable are based upon hash table data structure, there are subtle differences between them. HashMap is non-synchronized while Hashtable is synchronized and because of that HashMap is faster than Hashtable, as there is no cost of synchronization associated with it. One more minor difference is that HashMap allows a null key but Hashtable doesn't.

#### 7. Difference between TreeSet and TreeMap in Java?
Though both are sorted collections, TreeSet is essentially a Set data structure that doesn't allow duplicate, and TreeMap is an implementation of Map interface. In reality, TreeSet is implemented via a TreeMap, much like how HashSet is implemented using HashMap.

#### 8. Difference between Serializable and Externalizable in Java? 
Serializable is a marker interface with no methods defined it but Externalizable interface has two methods defined on it e.g. readExternal() and writeExternal() which allows you to control the serialization process. Serializable uses a default serialization process which can be very slow for some applications.

#### 9. Difference between transient and volatile in Java? 
the transient keyword is used in Serialization while volatile is used in multi-threading. If you want to exclude a variable from the serialization process then mark that variable transient. Similar to the static variable, transient variables are not serialized. 

On the other hand, volatile variables are signal to the compiler that multiple threads are interested in this variable and it shouldn't reorder its access. the volatile variable also follows the happens-before relationship, which means any write happens before any read in a volatile variable. You can also make non-atomic access of double and long variable atomic using volatile.

#### 10. Difference between abstract class and interface? 
From Java 8 onwards difference between abstract class and interface in Java has been minimized, now even the interface can have an implementation in terms of default and static method. BTW, in Java, you can still extend just one class but can extend multiple inheritances. An abstract class is used to provide default implementation with just something left to customize, while an interface is used heavily in API to define the contract of a class.

#### 11. What is the difference between FileInputStream and FileReader in Java? 
The main difference between FileInputStream and FileReader is that the former is used to read binary data while the latter is used to read text data, which means later also consider character encoding while converting bytes to text in Java.

#### 12. Can we have a return statement in the finally clause? What will happen?
Yes, you can use the return statement in finally block, but it will not prevent finally block from being executed. BTW, if you also used the return statement in the try block then return the value from the finally block with override whatever is returned from the try block.

#### 13. What is the difference between the calling start() and run() method of Thread?

You might have heard this question before, if calling the start() method calls the run() method eventually then why not just call the run() method? Well, the difference is, the start method also starts a new thread. If you call the run method directly then it will run on the same thread, not on a different thread, which is what the original intention would be.

#### 14. Why wait and notify methods are declared in the Object class?

This question is more to find out how much experience you really have and what is your thinking about Java API and its design decision. A similar question is why String is immutable in Java? Well, a true answer can only be given by Java designers but you can reason something. 

For example, wait and notify methods are associated with locks that are owned by the object, not thread, and that's why it makes sense to keep those methods on `java.lang.Object` class. See the [detailed](http://javarevisited.blogspot.sg/2012/02/why-wait-notify-and-notifyall-is.html) answer for more discussion and reasoning.

#### 15. How does HashSet works in Java?
HashSet is internally implemented using HashMap in Java and this is what your interviewer wants to hear. He could then quiz you with some common sense-based questions e.g. how can you use HashMap because it needs two object keys and value? what is the value in the case of HashSet? 

Well, in the case of HashSet a dummy object is used as a value and key objects are the actual element on Set point of view. Since HashMap doesn't allow duplicate keys it also follows the contract of set data structure to not allow duplicates. See [detailed](http://java67.blogspot.sg/2014/01/how-hashset-is-implemented-or-works-internally-java.html) answers for more analysis and explanation.

#### 16. What is the difference between synchronized and concurrent Collection in Java?

There was a time, before Java 1.5 when you only have synchronized collections if you need them in a multi-threaded Java program. Those classes were plagued with several issues most importantly performance because they lock the whole collection or map whenever a thread reads or writes. To address those issues, Java released a couple of Concurrent collection classes e.g. ConcurrentHashMap, CopyOnWriteArrayList, and BlockingQueue to provide more scalability and performance.

[Read more](http://javarevisited.blogspot.sg/2010/10/what-is-difference-between-synchronized.html)

#### 17. Difference between static and dynamic binding in Java?

This is usually asked as a follow-up of the previous question, static binding is related to an overloaded method and dynamic binding is related to the overridden method. A method like private, final, and static are resolved using static binding at compile time but virtual methods that can be overridden are resolved using dynamic binding at runtime.

#### 18. What is the difference between Checked and Unchecked Exception in Java?

Checked exception ensures that handling of the exception is provided and it's verified by compiler also, while for throwing unchecked exception no special provision is needed e.g. throws clause. A method can throw unchecked exceptions without any throw clause.

[Read more](http://java67.blogspot.sg/2012/12/difference-between-runtimeexception-and-checked-exception.html)