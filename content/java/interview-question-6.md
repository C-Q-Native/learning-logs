---
title: "Java 面试题汇编 (七)"
date: 2021-06-27T17:23:19+08:00
categories:
- java
tags:
- java
- Interview
---
### 1. Multi threading
#### 1.1 How do you create a thread
In Java, there are two main ways to create a thread: by extending the Thread class, or by implementing the Runnable interface.

1. Extending the Thread class:
   
   To create a thread by extending the Thread class, you need to create a new class that extends Thread and overrides its run() method. The run() method contains the code that will be executed in the new thread. Here's an example:
   ```
    public class MyThread extends Thread {
        public void run() {
            // Code to be executed in the new thread
        }
    }
    ```
    To start the new thread, you can create an instance of MyThread and call its start() method:
    ```
    MyThread thread = new MyThread();
    thread.start();
    ```
2. Implementing the Runnable interface:
   
   To create a thread by implementing the Runnable interface, you need to create a new class that implements Runnable and overrides its run() method. The run() method contains the code that will be executed in the new thread. Here's an example:
   ```
    public class MyRunnable implements Runnable {
        public void run() {
            // Code to be executed in the new thread
        }
    }
    ```
    To start the new thread, you can create an instance of MyRunnable and pass it to a new instance of the Thread class. Then you can call the start() method on the new Thread instance:
    ```
    MyRunnable runnable = new MyRunnable();
    Thread thread = new Thread(runnable);
    thread.start();
    ```

#### 1.2 What are the different states of a thread
In Java, a thread can be in one of several states at any given time, depending on what it is currently doing. These states are defined by the Thread.State enum and are as follows:

- NEW: When a thread is created but has not yet started, it is in the NEW state.
- RUNNABLE: When a thread is ready to run, but the operating system has not yet started executing it, it is in the RUNNABLE state.
- BLOCKED: When a thread is waiting for a monitor lock to be released so it can enter a synchronized block or method, it is in the BLOCKED state.
- WAITING: When a thread is waiting indefinitely for another thread to perform a particular action, it is in the WAITING state. For example, a thread can be in the WAITING state if it is waiting for a notification from another thread.
- TIMED_WAITING: When a thread is waiting for a specified amount of time for another thread to perform a particular action, it is in the TIMED_WAITING state. For example, a thread can be in the TIMED_WAITING state if it is waiting for a timeout period to expire.
- TERMINATED: When a thread has completed its execution or has been terminated abruptly, it is in the TERMINATED state.

You can get the current state of a thread by calling its `getState()` method. This can be useful for debugging and troubleshooting threading issues in your program.

#### 1.3 What is priority of a thread? How do you change the priority of a thread
In Java, a thread's priority is an integer value that specifies how much the thread should be favored by the scheduler relative to other threads. Thread priority is used by the operating system to determine which threads should be executed first when there are multiple threads competing for the CPU's attention.

Java defines thread priorities as integer values between 1 and 10, with 1 being the lowest priority and 10 being the highest. By default, all threads in Java have a priority of 5.

You can change the priority of a thread using the `setPriority()` method, which is defined in the `Thread` class. The `setPriority()` method takes an integer value between 1 and 10 as an argument, representing the new priority of the thread.

Here's an example of how to set the priority of a thread:

```
Thread thread = new Thread();
thread.setPriority(8); // Set the priority to 8 (high)
```

In this example, we create a new instance of `Thread` and set its priority to 8 using the `setPriority()` method.

It's important to note that the exact behavior of thread priorities can vary depending on the operating system and JVM implementation. In general, however, higher-priority threads are given preferential treatment by the scheduler and are more likely to be executed before lower-priority threads. However, you should avoid relying too heavily on thread priorities, as they can lead to unpredictable and hard-to-debug behavior.

#### 1.4 What is executorservice
`ExecutorService` is a high-level interface in Java that provides a way to execute tasks asynchronously in a pool of threads. It extends the `Executor` interface, which is a lower-level interface that provides a way to execute tasks but doesn't provide any management of threads.

With an `ExecutorService`, you can submit tasks to be executed asynchronously, and the service will automatically manage a pool of threads to handle the tasks. When a task is submitted, the service will either reuse an existing thread from the pool or create a new thread if no threads are available. Once a thread is finished executing a task, it will be returned to the pool for reuse.

The `ExecutorService` interface provides a number of methods for submitting tasks, such as `execute()`, `submit()`, and `invokeAll()`, as well as methods for shutting down the service when it is no longer needed.

Here's an example of how to create an `ExecutorService` and submit a task:

```
ExecutorService executor = Executors.newFixedThreadPool(5); // Create a pool of 5 threads

Runnable task = new Runnable() {
    public void run() {
        // Code to be executed in the new thread
    }
};

executor.submit(task); // Submit the task to the executor service
```

In this example, we create a new `ExecutorService` using the `newFixedThreadPool()` method, which creates a thread pool with a fixed number of threads. We then define a new `Runnable` instance to represent the task we want to execute, and submit it to the executor service using the `submit()` method.

Using an `ExecutorService` can be a more efficient way of managing threads in a program, as it allows you to avoid the overhead of creating and destroying threads for each task that needs to be executed.

#### 1.5 Explain different ways of creating executor services
In Java, there are several ways to create an `ExecutorService` instance, depending on your needs. Here are some common ways to create an `ExecutorService`:
1. `Executors.newFixedThreadPool(int nThreads)`: This method creates an `ExecutorService` with a fixed-size thread pool, where the number of threads is specified by the `nThreads` argument. This method is suitable for situations where you have a known number of tasks that need to be executed concurrently.
2. `Executors.newCachedThreadPool()`: This method creates an `ExecutorService` with a thread pool that can dynamically adjust the number of threads based on the workload. This method is suitable for situations where you have a large number of short-lived tasks that need to be executed.
3. `Executors.newSingleThreadExecutor()`: This method creates an ExecutorService with a single thread. This method is suitable for situations where you have a series of tasks that need to be executed sequentially, or where you need to ensure that tasks are executed in a specific order.
4. `Executors.newScheduledThreadPool(int corePoolSize)`: This method creates an `ExecutorService` that can schedule tasks to be executed at a specific time or after a specified delay. This method is suitable for situations where you need to execute tasks periodically or at a specific time in the future.
5. `ThreadPoolExecutor`: You can also create a custom `ThreadPoolExecutor` instance by directly instantiating the class and passing in the desired parameters, such as the core pool size, maximum pool size, and task queue. This method is suitable for situations where you need more fine-grained control over the thread pool configuration.

#### 1.6  How do you check whether an executionservice task executed successfully
In Java, you can use the `Future` interface to check whether an `ExecutorService` task has completed successfully or not. When you submit a task to an `ExecutorService`, it returns a `Future` object that represents the result of the task. You can use the methods of the `Future` interface to check whether the task completed successfully, retrieve the result of the task, or cancel the task if necessary.

Here's an example of how to submit a task to an `ExecutorService` and check whether it completed successfully using the `Future` interface:
```
ExecutorService executor = Executors.newSingleThreadExecutor();

Future<String> future = executor.submit(new Callable<String>() {
    public String call() {
        // Code to be executed in the new thread
        return "Task completed successfully";
    }
});

try {
    String result = future.get(); // Wait for the task to complete and retrieve the result
    System.out.println(result);
} catch (InterruptedException e) {
    // Handle the interruption
} catch (ExecutionException e) {
    // Handle the exception thrown by the task
} finally {
    executor.shutdown(); // Shut down the executor when it's no longer needed
}
```
If you need to check whether a task completed successfully without waiting for it to complete, you can use the `isDone()` method of the `Future` interface. This method returns `true` if the task has completed, whether it completed successfully or not.

#### 1.7 What is synchronization of threads?
Synchronization of threads is the process of coordinating the execution of multiple threads to ensure that they access shared resources or variables in a controlled and consistent manner. In Java, synchronization is typically used to prevent race conditions and ensure that only one thread at a time can access a shared resource or variable.

In Java, synchronization can be achieved using the `synchronized` keyword, which can be applied to methods or blocks of code. When a method or block of code is declared as `synchronized`, only one thread can execute it at a time. If another thread tries to execute the same method or block of code while it is already being executed by another thread, it will be blocked and put on hold until the first thread completes the execution.

Here's an example of how to use synchronization in Java to ensure that only one thread can access a shared resource at a time:
```
class Counter {
    private int count;

    public synchronized void increment() {
        count++;
    }

    public synchronized int getCount() {
        return count;
    }
}

class MyThread implements Runnable {
    private Counter counter;

    public MyThread(Counter counter) {
        this.counter = counter;
    }

    public void run() {
        for (int i = 0; i < 1000; i++) {
            counter.increment();
        }
    }
}

public class Main {
    public static void main(String[] args) throws InterruptedException {
        Counter counter = new Counter();

        Thread thread1 = new Thread(new MyThread(counter));
        Thread thread2 = new Thread(new MyThread(counter));

        thread1.start();
        thread2.start();

        thread1.join();
        thread2.join();

        System.out.println("Count: " + counter.getCount());
    }
}
```
In this example, we define a `Counter` class that has a `count` variable and two methods `increment()` and `getCount()` that modify and retrieve the value of the count variable, respectively. Both methods are declared as `synchronized`, which means that only one thread can execute them at a time.

We also define a `MyThread` class that implements the `Runnable` interface and takes a `Counter` object as its constructor argument. The `run()` method of MyThread increments the `count` variable of the `Counter` object by calling the `increment()` method in a loop.

In the Main class, we create two MyThread objects and two Thread objects that execute them. We start both threads using the `start()` method, wait for them to complete using the `join()` method, and print the final value of the `count` variable using the `getCount()` method of the Counter object.

Thanks to the synchronization provided by the `synchronized` keyword, only one thread can access the `increment()` method of the `Counter` object at a time, ensuring that the count variable is incremented correctly and the final value of `count` is 2000 (1000 increments from each thread).

#### 1.7 Can a static method be synchronized?
Yes, a static method can be synchronized in Java. When a static method is declared as synchronized, it is locked on the class object associated with the method's class.

Here's an example of a synchronized static method:
```
public class MyClass {
    private static int count;

    public static synchronized void increment() {
        count++;
    }

    public static int getCount() {
        return count;
    }
}
```

#### 1.8 Describe a few other important methods in threads?
Here are a few other important methods in Java threads:
- `wait()`: This method is used to make a thread wait until another thread notifies it to wake up. When a thread calls `wait()`, it releases the lock on the object and enters into a wait state until another thread calls `notify()` or `notifyAll()` on the same object to wake it up.
- `notify()`: This method is used to wake up a single thread that is waiting on the object. If multiple threads are waiting on the object, only one thread will be woken up.
- `notifyAll()`: This method is used to wake up all threads that are waiting on the object. All threads will be woken up, but only one thread at a time will be able to acquire the lock and continue executing.
- `join()`: This method is used to make a thread wait for another thread to finish its execution. When a thread calls `join()` on another thread, it waits until the other thread completes its execution before continuing.
- `yield()`: This method is used to give up the CPU to other threads that are waiting to run. When a thread calls `yield()`, it indicates that it's willing to give up the CPU to allow other threads to execute.
- `sleep()`: This method is used to make a thread sleep for a specified amount of time. When a thread calls `sleep()`, it suspends its execution for the specified time period.
- `interrupt()`: This method is used to interrupt a thread that is blocked or sleeping. When a thread is interrupted, it wakes up and throws an `InterruptedException`, allowing it to respond to the interrupt request.

#### 1.9 Can you write a synchronized program with wait and notify methods?
Yes, here's an example of a program that uses synchronization with `wait()` and `notify()` methods to allow two threads to take turns printing numbers from 1 to 10:

```
public class SynchronizedWaitNotifyExample {
    public static void main(String[] args) {
        Printer printer = new Printer();
        Thread thread1 = new Thread(new PrintOddNumbers(printer));
        Thread thread2 = new Thread(new PrintEvenNumbers(printer));
        thread1.start();
        thread2.start();
    }
}

class Printer {
    private boolean isOdd = true;

    public synchronized void printOdd(int number) {
        while (!isOdd) {
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("Odd: " + number);
        isOdd = false;
        notify();
    }

    public synchronized void printEven(int number) {
        while (isOdd) {
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("Even: " + number);
        isOdd = true;
        notify();
    }
}

class PrintOddNumbers implements Runnable {
    private Printer printer;
    private int count = 1;

    public PrintOddNumbers(Printer printer) {
        this.printer = printer;
    }

    @Override
    public void run() {
        while (count <= 10) {
            printer.printOdd(count);
            count += 2;
        }
    }
}

class PrintEvenNumbers implements Runnable {
    private Printer printer;
    private int count = 2;

    public PrintEvenNumbers(Printer printer) {
        this.printer = printer;
    }

    @Override
    public void run() {
        while (count <= 10) {
            printer.printEven(count);
            count += 2;
        }
    }
}
```
In this example, we define a `Printer` class that has two methods: `printOdd()` and `printEven()`. Each method is declared as `synchronized`, which means that only one thread can execute the method at a time. The `printOdd()` method prints an odd number and sets the `isOdd` flag to `false`, while the `printEven()` method prints an even number and sets the `isOdd` flag to `true`. Both methods then call the `notify()` method to wake up the waiting thread and release the lock.

We also define two threads, `PrintOddNumbers` and `PrintEvenNumbers`, that each take turns printing numbers from 1 to 10 using the Printer object. The threads are started using the `start()` method, which causes them to run concurrently.

The `wait()` method is called within a loop that checks the condition that needs to be met for the thread to proceed. If the condition is not met, the thread releases the lock on the object and enters into a waiting state until another thread calls `notify()` or `notifyAll()` on the same object to wake it up.

This program uses synchronization with `wait()` and `notify()` methods to ensure that the threads take turns printing odd and even numbers.

### 2. Functional Programming - Lamdba expressions and Streams
#### 2.1 What is functional programming?
Functional programming is a programming paradigm that focuses on writing programs by composing functions, rather than using mutable data and imperative statements. In functional programming, functions are treated as first-class citizens, which means that they can be assigned to variables, passed as arguments to other functions, and returned as values from functions.

The main principles of functional programming are:
- Pure functions: Functions that do not have any side effects and only depend on their input parameters to produce a result. This means that they do not modify any external state or rely on any global variables.
- Immutable data: Data that cannot be modified after it has been created. This means that any operation on the data creates a new copy, rather than modifying the original data.
- Higher-order functions: Functions that take other functions as arguments or return functions as results.

#### 2.2 Can you give an example of functional programming?
```
import java.util.Arrays;
import java.util.List;

public class FunctionalProgrammingExample {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
        
        // Using lambda expression to define a function
        Function<Integer, Integer> square = (number) -> number * number;

        // Using a stream to apply the function to each element in the list and collect the results
        List<Integer> squares = numbers.stream()
                                        .map(square)
                                        .collect(Collectors.toList());

        // Printing the list of squares
        System.out.println(squares);
    }
}
```
In this example, we define a list of numbers and a function called square that takes an integer as input and returns its square. The square function is defined using a lambda expression, which is a concise way of defining a function in Java.

We then use a stream to apply the square function to each element in the list of numbers using the map() method. The map() method applies the square function to each element in the list and returns a new stream containing the results. Finally, we collect the results using the collect() method and store them in a new list called squares.

This program demonstrates some of the core principles of functional programming, such as using functions as first-class citizens and immutability of data. By defining a function that takes input and produces output, we can easily apply it to a list of values using a stream, without modifying the original list.

#### 2.3 Explain about streams with an example?

```
import java.util.Arrays;
import java.util.List;

public class StreamExample {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

        // Using streams to filter even numbers and collect the results
        List<Integer> evenNumbers = numbers.stream()
                                            .filter(number -> number % 2 == 0)
                                            .collect(Collectors.toList());

        // Printing the list of even numbers
        System.out.println(evenNumbers);
        
        // Using streams to find the sum of all odd numbers
        int sum = numbers.stream()
                            .filter(number -> number % 2 != 0)
                            .mapToInt(Integer::intValue)
                            .sum();

        // Printing the sum of all odd numbers
        System.out.println(sum);
    }
}
```

#### 2.4 what are intermediate operations in streams?
Intermediate operations in Java streams are operations that transform or filter the elements of a stream. They return a new stream that can be used for further operations. These operations are "lazy", meaning that they do not execute until a terminal operation is called on the stream.

Here are some examples of intermediate operations in Java streams:
- `filter()`: This operation takes a predicate as an argument and returns a new stream that contains only the elements that satisfy the predicate.
- `map()`: This operation takes a lambda expression as an argument and returns a new stream that contains the result of applying the lambda expression to each element of the original stream.
- `flatMap()`: This operation takes a function that returns a stream as an argument and returns a new stream that contains the flattened results of applying the function to each element of the original stream.
- `distinct()`: This operation returns a new stream that contains only the distinct elements of the original stream.
- `sorted()`: This operation returns a new stream that contains the elements of the original stream in sorted order, based on a comparator.
- `peek()`: This operation takes a consumer as an argument and returns a new stream that performs the consumer's operation on each element of the original stream.
- `limit()`: This operation returns a new stream that contains only the first n elements of the original stream, where n is a given number.
- `skip()`: This operation returns a new stream that skips the first n elements of the original stream, where n is a given number.

#### 2.5  What are terminal operations in streams?
Terminal operations in Java streams are operations that produce a non-stream result. They are the final operation in a stream pipeline, meaning that after a terminal operation is called, the stream can no longer be used.

Here are some examples of terminal operations in Java streams:

- `forEach()`: This operation performs an action on each element of the stream. The action is - passed as a lambda expression.
- `count()`: This operation returns the number of elements in the stream as a long value.
- `reduce()`: This operation performs a reduction on the elements of the stream. It takes a binary operator as an argument and returns an Optional object that may or may not contain the result of the reduction.
- `collect()`: This operation collects the elements of the stream into a collection, such as a list or a set. It takes a Collector object as an argument.
- `min()`: This operation returns the minimum element of the stream, based on a comparator.
- `max()`: This operation returns the maximum element of the stream, based on a comparator.
- `anyMatch()`: This operation returns true if any element in the stream matches a given predicate.
- `allMatch()`: This operation returns true if all elements in the stream match a given predicate.
- `noneMatch()`: This operation returns true if none of the elements in the stream match a given predicate.

Using these terminal operations, we can perform a wide range of operations on the elements of a stream and obtain a final result or side-effect. It's important to note that terminal operations are "lazy," meaning that they do not execute until they are explicitly called.

#### 2.6 What are method references?
Method references are a shorthand notation for lambdas that refer to a method instead of providing an implementation directly. They are used to simplify the code when a lambda expression is used only to invoke a single method.

Method references are composed of three parts: the class or object containing the method, the name of the method, and optionally, the argument list. There are four types of method references:
- Reference to a static method: This type of method reference refers to a static method of a class. It has the form `ClassName::staticMethodName`.
- Reference to an instance method of an object: This type of method reference refers to an instance method of an object. It has the form `object::instanceMethodName`.
- Reference to an instance method of a class: This type of method reference refers to an instance method of a class. It has the form `ClassName::instanceMethodName`.
- Reference to a constructor: This type of method reference refers to a constructor of a class. It has the form `ClassName::new`.

For example, let's say we have a list of strings and we want to convert each string to uppercase using the `String.toUpperCase()` method. We can use a method reference instead of a lambda expression as follows:
```
List<String> strings = Arrays.asList("apple", "banana", "cherry");
strings.stream()
       .map(String::toUpperCase)
       .forEach(System.out::println);
```

#### 2.7 Can you explain the relationship between lambda expression and functional interfaces?
Yes, lambda expressions are closely related to functional interfaces in Java. A functional interface is an interface that has exactly one abstract method, and can be used as the target for a lambda expression.

When we use a lambda expression, we are essentially providing an implementation of the single abstract method of a functional interface. The compiler automatically infers the target type of the lambda expression based on the context in which it appears.

For example, consider the following functional interface:
```
interface MyInterface {
    void myMethod();
}
```
We can use a lambda expression to provide an implementation for this interface:
```
MyInterface myLambda = () -> System.out.println("Hello, world!");
```
In this example, we define a lambda expression that takes no arguments and prints "Hello, world!" to the console. The compiler infers that the lambda expression should be assigned to a variable of type MyInterface, since it has only one abstract method that matches the signature of the lambda expression.

Lambda expressions make it much easier to work with functional interfaces, since they allow us to write more concise and readable code. They are particularly useful when working with streams and collections, since they allow us to define complex transformations and filters in a more streamlined way.



