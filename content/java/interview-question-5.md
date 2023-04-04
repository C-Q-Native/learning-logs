---
title: "Java 面试题汇编 (六)"
date: 2021-06-26T17:23:19+08:00
categories:
- java
tags:
- java
- Interview
---
### 1. Collections
#### 1.1 Why do we need collections in Java
Collections in Java are a way to store and manipulate groups of objects. They are essential because they provide a way to manage data in a flexible and dynamic manner.

Here are some of the key reasons why we need collections in Java:

- Dynamic storage: Collections allow us to store and manipulate a large number of objects dynamically, without requiring us to know the exact number of objects in advance.
- Flexibility: Collections provide a wide range of data structures with different properties, such as sets, lists, maps, and queues. This flexibility allows us to choose the most appropriate data structure for the specific needs of our program.
- Convenience: Collections provide a wide range of methods for manipulating and accessing data. This makes it easier to perform operations such as adding, removing, searching, and sorting elements.
- Efficiency: Collections are optimized for performance, providing fast access and manipulation of data.
- Standardization: Collections are part of the Java Standard Library, making them widely used and accepted. This means that Java developers can rely on them and have a shared understanding of how they work.

Overall, collections are an essential part of Java programming because they provide a flexible, efficient, and standardized way to store and manipulate groups of objects.

#### 1.2 What are the important interfaces in the collection hierarchy
In Java, the collection hierarchy is composed of several important interfaces that define how collections should behave and provide a common API for working with them. Some of the most important interfaces in the collection hierarchy include:

1. Collection Interface: This interface is at the top of the collection hierarchy and defines the basic operations that all collections should support. These operations include adding, removing, and querying elements from the collection.
2. List Interface: This interface extends the Collection interface and defines operations for working with ordered collections, where elements can be accessed by their index.
3. Set Interface: This interface extends the Collection interface and defines operations for working with collections that do not allow duplicate elements.
4. Map Interface: This interface defines a mapping between keys and values and provides operations for working with key-value pairs.
5. Queue Interface: This interface defines operations for working with collections that support adding elements at one end and removing them from the other end, such as a FIFO (first-in, first-out) queue.
6. Deque Interface: This interface extends the Queue interface and defines operations for working with collections that support adding and removing elements at both ends, such as a double-ended queue.

By implementing these interfaces and following their guidelines, developers can create collections that can be used interchangeably with other collections that implement the same interfaces, allowing for greater flexibility and ease of use in Java applications.

#### 1.3 Can you explain briefly about the List interface
The `List` interface is one of the most commonly used interfaces in the Java Collections Framework. It extends the `Collection` interface and defines a collection that is ordered and allows duplicate elements. In other words, a `List` maintains the order of its elements and allows elements to be inserted or removed at any position.

By implementing the `List` interface, developers can create collections that maintain the order of their elements and allow for efficient random access and insertion/deletion at any position. Some common implementations of the `List` interface include `ArrayList`, Lin`kedList, and `Vector`.

#### 1.4 Can an ArrayList have duplicate elements
Yes, an `ArrayList` can have duplicate elements. In fact, one of the defining characteristics of an `ArrayList` is that it allows duplicate elements and maintains their insertion order.

For example, consider the following code:
```
ArrayList<String> list = new ArrayList<>();
list.add("apple");
list.add("banana");
list.add("apple");
```

#### 1.5 Can you briefly explain about the Set interface
In Java, the `Set` interface is a member of the Java Collections Framework and is used to store a collection of unique elements. It is similar to the `List` interface, but unlike a list, a set does not allow duplicate elements.

The `Set` interface extends the `Collection` interface, and defines the following methods:
- `add(E element)`: Adds the specified element to the set if it is not already present.
- `addAll(Collection<? extends E> collection)`: Adds all of the elements in the specified collection to the set if they are not already present.
- `clear()`: Removes all of the elements from the set.
- `contains(Object element)`: Returns `true` if the set contains the specified element, and `false` otherwise.
- `isEmpty()`: Returns `true` if the set contains no elements, and `false` otherwise.
- `iterator()`: Returns an iterator over the elements in the set.
- `remove(Object element)`: Removes the specified element from the set if it is present.
- `size()`: Returns the number of elements in the set.

The `Set` interface has several implementations in Java, including:
- `HashSet`: Uses a hash table to store the elements in the set. This implementation offers constant-time performance for the basic operations (add, remove, contains, and size), assuming the hash function is well-distributed.
- `TreeSet`: Uses a red-black tree to store the elements in the set. The elements are ordered using their natural ordering (i.e., they must implement the `Comparable` interface), or by a comparator provided at set creation time.
- `LinkedHashSet`: Extends `HashSet` to maintain a linked list of the elements in the set, in the order in which they were inserted.

Here's an example of how to create and use a `Set` in Java:
```
import java.util.HashSet;
import java.util.Set;

public class ExampleSet {
    public static void main(String[] args) {
        // create a new HashSet
        Set<String> set = new HashSet<>();

        // add some elements to the set
        set.add("Alice");
        set.add("Bob");
        set.add("Charlie");

        // try to add a duplicate element
        set.add("Bob");

        // print the set
        System.out.println("Set: " + set);

        // remove an element from the set
        set.remove("Charlie");

        // print the updated set
        System.out.println("Updated Set: " + set);
    }
}
```
In this example, we create a new `HashSet` and add some elements to it using the `add()` method. We then try to add a duplicate element to the set, but because sets do not allow duplicates, the duplicate element is not added.

We then print the set to the console using the `toString()` method, and remove an element from the set using the `remove()` method. Finally, we print the updated set to the console.

The output of the above code would be:
```
Set: [Bob, Alice, Charlie]
Updated Set: [Bob, Alice]
```
This shows that we have successfully created and manipulated a `Set` in Java.

#### 1.6 What are the important interfaces related to the Set interface
There are two important interfaces related to the `Set` interface in Java:

- `SortedSet`: This interface extends the `Set` interface and defines a set that is sorted in ascending order according to the natural ordering of its elements or by a `Comparator`. It adds methods for navigating the set in order, such as `first()`, `last()`, `headSet(E toElement)`, and `tailSet(E fromElement)`. The `TreeSet` class implements this interface.
- `NavigableSet`: This interface extends the `SortedSet` interface and adds methods for navigating the set in both ascending and descending order, such as `lower(E e)`, `floor(E e)`, `ceiling(E e)`, `higher(E e)`, and `descendingSet()`. The `TreeSet` class also implements this interface.

Both of these interfaces provide additional functionality for working with sets that are sorted in a particular order, beyond what is provided by the basic `Set` interface.

#### 1.7 What is a HashSet
A HashSet is a collection class in programming that stores a set of unique objects. It is a part of the Java Collections Framework and is used to implement the set interface. In a HashSet, the order of the elements is not guaranteed, and duplicate elements are not allowed.

The HashSet class is implemented using a hash table data structure, which provides fast access and insertion of elements. It uses the hash code of an object to determine its position in the hash table. This allows for fast searching and retrieval of elements.

HashSet is commonly used in situations where uniqueness is required, such as checking if a list of items contains duplicates or in implementing a dictionary with unique keys.

In summary, a HashSet is a data structure that stores a unique set of objects using a hash table, and it is commonly used for fast and efficient operations on unique elements.

#### 1.8 What is a linkedHashSet? How is different from a HashSet?
A LinkedHashSet is another collection class in Java that extends the HashSet class and implements the Set interface. The primary difference between a HashSet and a LinkedHashSet is that the LinkedHashSet maintains the insertion order of the elements, while the HashSet does not.

In a LinkedHashSet, the order in which the elements are added to the set is preserved. When iterating over the elements of a LinkedHashSet, they will be returned in the same order in which they were added. This can be useful in situations where you need to maintain the order of elements, such as when implementing a cache or maintaining a list of recent items.

Like the HashSet, the LinkedHashSet uses a hash table to store its elements, but it also maintains a doubly linked list to maintain the order of the elements. This can make the LinkedHashSet slightly slower than the HashSet for certain operations, but it provides the benefit of maintaining the insertion order.

In summary, the LinkedHashSet is similar to the HashSet in that it stores a unique set of objects, but it also maintains the order in which the elements were added. This can be useful in certain situations where the order of elements is important.

#### 1.9 What is a TreeSet? How is different from a HashSet?
A TreeSet is another collection class in Java that implements the SortedSet interface, which extends the Set interface. The primary difference between a TreeSet and a HashSet is that the TreeSet maintains the elements in sorted order, while the HashSet does not.

In a TreeSet, the elements are stored in a sorted tree structure. The order is determined by the natural ordering of the elements, or by a Comparator that is specified when the TreeSet is created. This means that when iterating over the elements of a TreeSet, they will be returned in sorted order, which can be useful in certain situations.

In contrast, a HashSet uses a hash table to store its elements, which provides faster access and insertion time but does not maintain any order of the elements.

Another difference between the two is that the TreeSet does not allow null elements, whereas the HashSet can contain at most one null element.

TreeSet provides a number of additional methods for finding elements based on their position in the sorted order, such as first(), last(), and subSet(). These methods can be useful in certain situations where you need to work with elements in a specific order.

In summary, the TreeSet is similar to the HashSet in that it stores a unique set of objects, but it also maintains the elements in sorted order. This can be useful in certain situations where you need to work with elements in a specific order, but it can also be slower for certain operations compared to the HashSet.

#### 1.10 Explain briefly about Queue interface
The Queue interface in Java is a part of the Java Collections Framework and defines a collection that is designed for holding elements prior to processing. It is an ordered list of elements that follows the "first-in, first-out" (FIFO) principle, meaning that the first element added to the queue will be the first one to be processed or removed.

The Queue interface provides methods for adding elements to the end of the queue, removing elements from the front of the queue, and inspecting the elements at the front of the queue without removing them. The main methods of the Queue interface include:

add(E e) - Adds the specified element to the end of the queue.
offer(E e) - Adds the specified element to the end of the queue and returns true if the operation is successful.
remove() - Removes and returns the element at the front of the queue.
poll() - Removes and returns the element at the front of the queue, or returns null if the queue is empty.
element() - Returns the element at the front of the queue without removing it.
peek() - Returns the element at the front of the queue without removing it, or returns null if the queue is empty.
The Queue interface also extends the Collection interface and inherits all of its methods, such as size(), isEmpty(), contains(), and iterator().

There are several classes in Java that implement the Queue interface, such as LinkedList, PriorityQueue, and ArrayDeque. Each implementation has its own characteristics and performance trade-offs, so the choice of implementation will depend on the specific needs of your application.

#### 1.11 What are the important interfaces related to the Queue interface
There are several important interfaces related to the Queue interface in Java, including:

- Deque: The Deque interface, which stands for "double-ended queue", extends the Queue interface and adds methods for inserting and removing elements from both ends of the queue. It provides additional methods such as addFirst(), addLast(), removeFirst(), removeLast(), and peekFirst().
- BlockingQueue: The BlockingQueue interface extends the Queue interface and provides methods that will block until the queue becomes non-empty when retrieving an element, or until space becomes available in the queue when adding an element. It is commonly used in concurrent programming to synchronize access to a shared resource.
- TransferQueue: The TransferQueue interface extends the BlockingQueue interface and adds methods that allow elements to be transferred between threads. It is used in concurrent programming when one thread needs to transfer data to another thread.
- PriorityQueue: The PriorityQueue class implements the Queue interface and maintains a priority queue of elements. The elements are ordered based on their natural ordering or a specified Comparator. The head of the queue is always the element with the highest priority.
- QueueIterator: The QueueIterator interface extends the Iterator interface and provides methods for iterating over the elements in a queue. It allows access to the elements in the queue in the order they will be removed.
These interfaces provide additional functionality and flexibility when working with queues in Java, and can be useful in different scenarios depending on your specific requirements.

#### 1.12 Can you briefly explain about the Map interface
The Map interface in Java is part of the Java Collections Framework and represents a collection of key-value pairs. It is used to store and retrieve data based on a key, where each key is associated with a value. The Map interface provides methods to add, remove, and retrieve elements from the collection, as well as to iterate over the keys, values, or key-value pairs.

The main methods of the Map interface include:
- put(K key, V value) - Associates the specified value with the specified key in the map.
- get(Object key) - Returns the value associated with the specified key in the map, or null if the key is not present.
- remove(Object key) - Removes the key-value pair associated with the specified key from the map.
- containsKey(Object key) - Returns true if the map contains the specified key, false otherwise.
- containsValue(Object value) - Returns true if the map contains the specified value, false otherwise.
- keySet() - Returns a Set of all the keys in the map.
- values() - Returns a Collection of all the values in the map.
- entrySet() - Returns a Set of all the key-value pairs in the map.

There are several classes in Java that implement the Map interface, including HashMap, TreeMap, LinkedHashMap, and ConcurrentHashMap. Each implementation has its own characteristics and performance trade-offs, so the choice of implementation will depend on the specific needs of your application.

In general, the Map interface is useful for situations where you need to store and retrieve data based on a key. It is commonly used in a variety of applications, including databases, caching systems, and search algorithms.

#### 1.13 What is a TreeMap? How is different from a HashMap
A TreeMap is a class in Java that implements the Map interface and provides a sorted key-value mapping based on the natural ordering of the keys or a custom Comparator. It is implemented as a red-black tree and provides guaranteed log(n) time cost for the basic operations (put, get, and remove) where n is the size of the map.

The main difference between a TreeMap and a HashMap is that the keys in a TreeMap are ordered, while the keys in a HashMap are not. This means that a TreeMap can be useful in situations where you need to maintain a specific order of the keys or where you need to quickly find the smallest or largest key in the map. On the other hand, a HashMap is generally faster for basic operations such as put, get, and remove, especially for larger maps, because it uses a hash table to store the key-value pairs.

Another difference is that TreeMap does not allow null keys, while a HashMap can have one null key. However, both TreeMap and HashMap allow null values.

In general, the choice between TreeMap and HashMap depends on your specific requirements. If you need to maintain a sorted order of keys, then TreeMap may be the better choice. If you need fast access to key-value pairs and order is not important, then HashMap may be a better choice.

### 2. Advanced collections
#### 2.1 What is the difference between synchronized and concurrent collections in Java?
Synchronized collections and concurrent collections are two ways to provide thread-safe access to collections in Java.

Synchronized collections are collections that are wrapped with synchronization code to make them thread-safe. This means that only one thread can access the collection at a time, which can lead to contention and performance issues in multi-threaded applications. Synchronized collections are typically used in situations where thread-safety is required but the collection is accessed infrequently or by a small number of threads.

Concurrent collections, on the other hand, are designed to be used in multi-threaded applications and provide high-performance, thread-safe access to collections. They are implemented using techniques such as lock-striping and non-blocking algorithms, which allow multiple threads to access the collection simultaneously without contention. Concurrent collections are optimized for high concurrency and are typically used in situations where thread-safety is required and the collection is accessed frequently or by a large number of threads.

In summary, synchronized collections provide thread-safety by wrapping collections with synchronization code, but can lead to contention and performance issues in multi-threaded applications. Concurrent collections provide high-performance, thread-safe access to collections by using specialized algorithms and techniques optimized for high concurrency.

#### 2.2 Explain about the new concurrent collections in Java?
Java provides a set of concurrent collections in the java.util.concurrent package that are designed to be used in multi-threaded applications and provide high-performance, thread-safe access to collections. These collections use specialized algorithms and techniques optimized for high concurrency, and are generally faster and more efficient than synchronized collections.

Here are some examples of concurrent collections in Java:

- ConcurrentHashMap: A high-performance concurrent hash table that allows multiple threads to read and write concurrently without contention. It is designed to be used in situations where frequent updates are expected.
- ConcurrentSkipListMap: A concurrent map that maintains its entries in ascending key order. It uses a skip list data structure to provide high concurrency and efficient operations.
- ConcurrentSkipListSet: A concurrent set that maintains its elements in ascending order. It is implemented using a skip list data structure, and provides high concurrency and efficient operations.
- ConcurrentLinkedDeque: A concurrent deque (double-ended queue) that allows multiple threads to add and remove elements from both ends concurrently.
- ConcurrentLinkedQueue: A concurrent queue that allows multiple threads to add and remove elements concurrently.
- LinkedTransferQueue: A transfer queue that provides blocking operations and allows elements to be transferred between producers and consumers in a FIFO order. It is optimized for high throughput and low latency.

These concurrent collections provide thread-safe access to collections without the performance overhead of synchronized collections. They are optimized for high concurrency and are designed to be used in multi-threaded applications.

#### 2.3 Explain about copyonwrite concurrent collections approach?
Copy-on-write (COW) is a concurrency strategy used in Java to provide high-performance, thread-safe access to collections. The basic idea behind COW is to make a copy of the collection every time a modification is made, instead of modifying the original collection in place. This means that readers can access the collection concurrently with writers without the need for synchronization, which can improve performance in certain situations.

The copy-on-write approach is implemented in several concurrent collections in Java, such as `CopyOnWriteArrayList` and `CopyOnWriteArraySet`. These collections are implemented using an array that is copied every time a modification is made. Reads are performed on the current copy of the array, while writes are performed on a new copy of the array. This ensures that readers are always accessing a consistent view of the collection, even if writes are happening concurrently.

One advantage of the copy-on-write approach is that it can improve performance in situations where reads are much more frequent than writes. In such cases, the overhead of copying the entire collection on each write may be outweighed by the benefits of providing concurrent read access without synchronization.

However, the copy-on-write approach has some drawbacks as well. One of the main drawbacks is that it can be expensive in terms of memory usage, as multiple copies of the collection need to be maintained. Additionally, there is a delay between the time a modification is made and the time it becomes visible to readers, which can be problematic in certain situations.

Overall, the copy-on-write approach is a useful concurrency strategy in Java that can provide high-performance, thread-safe access to collections in certain situations. However, it is important to carefully consider the trade-offs and choose the appropriate collection and concurrency strategy for your specific application requirements.

#### 2.4 What is a lock? How is it different from using synchronized approach
In Java, a lock is a low-level synchronization mechanism that can be used to control access to shared resources in a concurrent program. A lock provides a way for threads to coordinate their access to a shared resource, allowing only one thread to access the resource at a time. The java.util.concurrent.locks package provides a set of classes that implement locking mechanisms in Java.

The synchronized keyword, on the other hand, is a high-level synchronization mechanism that is built into the Java language. It provides a simple and convenient way to synchronize access to shared resources by allowing only one thread to execute a block of code at a time.

The main difference between using a lock and using synchronized is that a lock provides more fine-grained control over synchronization. With a lock, you can specify which threads can access the resource, and when they can access it. This can be useful in situations where you need more flexibility or control over synchronization, or where you need to implement more complex synchronization protocols.

Another difference is that locks can be more efficient than synchronized in some situations. For example, the ReentrantLock class in the java.util.concurrent.locks package provides a more scalable and efficient locking mechanism than synchronized in situations where many threads are contending for access to a shared resource.

However, using locks can be more complex and error-prone than using synchronized, as it requires more careful attention to locking protocols and error handling. Additionally, locks can be more prone to deadlock if they are not used correctly.

In summary, locks and synchronized are both mechanisms for synchronizing access to shared resources in Java, but they have different characteristics and trade-offs. The choice between using a lock and using synchronized depends on the specific requirements and constraints of your application.

#### 2.5 What is load factor?
In Java, the load factor is a value that is used to control the re-sizing of a hash table-based collection, such as HashMap and HashSet. The load factor is a floating-point value that represents the percentage of the table's capacity that can be filled with elements before the table is resized.

For example, a load factor of 0.75 indicates that the table can be filled up to 75% of its capacity before it needs to be resized. When the number of elements in the collection exceeds the load factor times the current capacity of the table, the table is resized to a larger size to accommodate more elements.

Resizing the table is an expensive operation, as it requires allocating a new array and copying the elements from the old array to the new array. By setting an appropriate load factor, you can balance the trade-off between memory usage and performance. A higher load factor will use less memory but require more frequent resizing, while a lower load factor will use more memory but require less frequent resizing.

The default load factor for HashMap and HashSet is 0.75, which is a reasonable value for most use cases. However, you can adjust the load factor by passing a second argument to the constructor when creating a new instance of the collection. For example, new HashMap<>(16, 0.5f) creates a new HashMap with an initial capacity of 16 and a load factor of 0.5.

#### 2.6 When does a Java collection throw UnsupportedOperationException?
A Java collection can throw `UnsupportedOperationException` when an unsupported operation is attempted on the collection. The `UnsupportedOperationException` is a runtime exception that is thrown by methods that are not implemented or supported by a particular collection.

For example, the `add` and `remove` methods of an unmodifiable collection such as `Collections.unmodifiableList` or `Collections.unmodifiableSet` will throw `UnsupportedOperationException` if called, as the collection cannot be modified. Similarly, the `clear` method of an immutable collection such as `ImmutableList` from the Google Guava library will throw `UnsupportedOperationException`, as the collection cannot be cleared.

Another example is when using a read-only view of a collection, such as with `Collections.singletonList` or `Collections.emptySet`, which do not support adding or removing elements, so calling any method that modifies the collection will throw an `UnsupportedOperationException`.

It's important to check the documentation for a collection to see which operations are supported and which are not. If an operation is not supported by a collection, attempting to use that operation will result in an `UnsupportedOperationException` being thrown.

#### 2.7 What are atomic operations in Java?
In Java, atomic operations are operations that are executed as a single, indivisible unit of work, without the possibility of interruption or interference by other threads. Atomic operations are guaranteed to complete without any other thread modifying the shared data at the same time, ensuring that the result of the operation is always consistent and predictable.

The `java.util.concurrent.atomic` package provides a set of classes that implement atomic operations on primitive data types, such as `AtomicInteger`, `AtomicLong`, and `AtomicBoolean`, as well as reference types such as `AtomicReference`.

Some common atomic operations supported by these classes include:
- `get`: retrieves the current value of the atomic variable.
- `set`: sets the value of the atomic variable to a new value.
- `compareAndSet`: compares the current value of the atomic variable with an expected value, and sets the value to a new value if the comparison is true.
- `getAndIncrement`: increments the atomic variable and returns its previous value.

Atomic operations are useful in multi-threaded environments where multiple threads may access and modify the same shared data. Using atomic operations can help avoid race conditions and other synchronization issues that can arise when multiple threads access the same data concurrently.

### 3. Generics
#### 3.1 What are Generics
Generics in Java are a language feature that allows classes and methods to be parameterized with one or more types. This enables developers to write more generic and reusable code that can work with different types of objects, without having to specify the exact type at compile time.

Generics are implemented using type parameters, which are represented by one or more placeholder types enclosed in angle brackets (`<` and `>`). For example, the following code defines a generic class Box that can hold objects of any type:
```
public class Box<T> {
    private T object;

    public void set(T object) {
        this.object = object;
    }

    public T get() {
        return object;
    }
}
```
In this example, the type parameter `T` represents the type of object that the `Box` class can hold. The `set` and `get` methods use the type parameter `T` to specify the type of the object being set or retrieved.

Generics provide several benefits, including:
- Type safety: The use of generics helps to ensure type safety at compile time, by allowing the compiler to detect and report errors if a type mismatch occurs.
- Reusability: Generics make it possible to write generic and reusable code that can work with different types of objects, without having to write separate code for each type.
- Abstraction: Generics provide a level of abstraction that allows developers to write code that is more generic and easier to understand.

Generics are widely used in Java collections, as well as in other areas of the Java language such as streams, lambdas, and functional interfaces.

#### 3.2 What are the restrictions in using generic type that is declared in a class declaration?
When you declare a generic type in a class declaration in Java, there are certain restrictions on how you can use that type within the class. Here are some of the main restrictions:

- You cannot create an instance of the generic type directly: When you declare a generic class like `Box<T>`, you cannot create an instance of `T` directly using the `new` operator. Instead, you need to create an instance of `Box<T>` and specify the type argument for T when you create the instance.
- You cannot use primitive types as type arguments: In Java, you cannot use primitive types like `int` or `double` as type arguments for a generic class. Instead, you need to use the corresponding wrapper classes like `Integer` or `Double`.
- You cannot create arrays of the generic type: In Java, you cannot create an array of a generic type like `T[]`. Instead, you need to use a collection class like `ArrayList<T>`.
- You cannot create a static field of the generic type: In Java, you cannot create a static field of the generic type `T` in a generic class declaration. This is because the type `T` is associated with a specific instance of the class, and static fields are shared among all instances of the class.
- You cannot catch an exception of the generic type: In Java, you cannot catch an exception of the generic type `T` in a catch block. This is because the type `T` is erased at runtime, so the catch block would not be able to determine the actual type of the exception. Instead, you need to catch a specific type of exception or use a wildcard catch block.

#### 3.3 How can we restrict Generics to a subclass of particular class?
In Java, you can restrict generics to a subclass of a particular class by using the `extends` keyword in the generic declaration. This is known as bounded type parameters.

Here's an example:
```
public class Box<T extends Number> {
    private T value;

    public Box(T value) {
        this.value = value;
    }

    public T getValue() {
        return value;
    }

    public void setValue(T value) {
        this.value = value;
    }
}
```

In this example, the generic type parameter `T` is restricted to be a subclass of the `Number` class. This means that when you create an instance of Box, you can only use `Number` or its subclasses as the type argument for `T`.

For example, you can create an instance of `Box<Integer>` or `Box<Double>`, but you cannot create an instance of `Box<String>` because `String` is not a subclass of `Number`.

By using bounded type parameters, you can ensure that the generic type parameter satisfies certain constraints and avoid errors at compile-time rather than runtime.

#### 3.4 How can we restrict Generics to a super class of particular class
In Java, you can restrict generics to a superclass of a particular class by using the `super` keyword in the generic declaration. This is known as a lower bounded wildcard.

Here's an example:
```
public class Box<T super Integer> {
    private T value;

    public Box(T value) {
        this.value = value;
    }

    public T getValue() {
        return value;
    }

    public void setValue(T value) {
        this.value = value;
    }
}
```
In this example, the generic type parameter `T` is restricted to be a superclass of the `Integer` class or `Integer` itself. This means that when you create an instance of Box, you can only use Integer or its superclasses as the type argument for `T`.

For example, you can create an instance of `Box<Number>`, `Box<Object>`, or `Box<Integer>,` but you cannot create an instance of `Box<Double>` because Double is not a superclass of Integer.

By using a lower bounded wildcard, you can ensure that the generic type parameter satisfies certain constraints and allow more flexibility in the type argument.









