---
title: "Java 面试题整理 (四)"
date: 2021-06-24T17:23:19+08:00
categories:
- java
tags:
- java
- Interview
---
### 1. Wrapper Classes
#### 1.1 What are Wrapper classes
Wrapper classes are classes in object-oriented programming that are used to wrap or encapsulate primitive data types so that they can be treated as objects. In Java, for example, there are several wrapper classes such as Integer, Double, and Boolean that correspond to the primitive data types int, double, and boolean, respectively.

Wrapper classes provide a way to represent primitive data types as objects, which allows them to be used in situations where objects are required, such as in collections or as method arguments. They also provide additional functionality, such as methods for converting between the wrapped value and other data types.
<!--more-->
Wrapper classes are often used in conjunction with autoboxing and unboxing, which is a feature in some programming languages that automatically converts between primitive data types and their corresponding wrapper classes. This can make the code more concise and easier to read, but it can also introduce performance overhead in some cases.

#### 1.2 Why do we need Wrapper classes in Java
- To convert primitive data types into objects: Primitive data types like int, double, boolean, etc. are not objects in Java, so they cannot be used in situations where objects are required, such as in collections, generic types, and as method arguments. Wrapper classes provide a way to represent these primitive types as objects, so they can be used in these contexts.
- To provide additional functionality: Wrapper classes provide additional functionality beyond what is available for primitive data types. For example, the Integer class provides methods for converting between integers and strings, finding the maximum and minimum values, and performing bit operations.
- To implement algorithms and data structures: Many algorithms and data structures require objects to work properly, so wrapper classes are essential for implementing them in Java. For example, the Collections framework relies heavily on wrapper classes like Integer and Double to implement sorting and searching algorithms.
- To support autoboxing and unboxing: Wrapper classes are also used to support the autoboxing and unboxing features in Java, which allow primitive types to be automatically converted to and from their corresponding wrapper classes. This makes the code more concise and easier to read.
  
Overall, wrapper classes provide an important abstraction layer in Java, allowing primitive data types to be treated as objects and providing additional functionality and flexibility to the language.

#### 1.3 What are the different ways of creating Wrapper class instances
In Java, there are two ways to create instances of a wrapper class:  
1. Using constructor: Each wrapper class has a constructor that takes a primitive value as its argument. For example, the following code creates an Integer object with the value 42:
```
Integer intObj = new Integer(42);
```
Similarly, you can create objects of other wrapper classes like Double, Boolean, etc. using their respective constructors.

2. Using valueOf() method: Each wrapper class also provides a static valueOf() method that takes a primitive value as its argument and returns an object of the wrapper class. For example, the following code creates an Integer object with the value 42 using the valueOf() method:
```
Integer intObj = Integer.valueOf(42);
```
Similarly, you can create objects of other wrapper classes like Double, Boolean, etc. using their respective valueOf() methods.

It's worth noting that since Java 9, the use of constructors to create instances of wrapper classes has been deprecated. The recommended approach is to use the valueOf() method instead, which offers better performance and memory utilization by reusing previously created objects in some cases.

#### 1.4 What are differences in the two ways of creating Wrapper classes
The two ways of creating instances of a wrapper class in Java - using a constructor and using the valueOf() method - have some differences in terms of their behavior and performance:
- Object creation: When using a constructor, a new object is always created, even if an object with the same value already exists. However, when using the valueOf() method, the JVM can reuse previously created objects for certain values, which can improve performance and reduce memory usage.
- Performance: As mentioned, using the valueOf() method can be faster than using a constructor in some cases, especially when creating a large number of objects with the same value. This is because the JVM can take advantage of object pooling to avoid creating new objects unnecessarily.
- Syntax: The syntax for creating objects using constructors and valueOf() methods is different. When using a constructor, you need to use the new keyword and provide the value as an argument. When using valueOf(), you call the method directly on the wrapper class and provide the value as an argument.
- Deprecation: Since Java 9, the use of constructors to create instances of wrapper classes has been deprecated. The recommended approach is to use the valueOf() method instead, which offers better performance and memory utilization by reusing previously created objects in some cases.

#### 1.5 What is auto boxing
Autoboxing is a feature in Java that allows automatic conversion between primitive data types and their corresponding wrapper classes. Specifically, when a primitive value is used in a context where an object is expected, Java automatically wraps the value in the appropriate wrapper class.

For example, consider the following code snippet:
```
int i = 42;
Integer intObj = i; // autoboxing
```

In this example, the integer value 42 is assigned to the primitive int variable "i". Then, when "i" is assigned to the Integer variable "intObj", the value is automatically wrapped in an Integer object using autoboxing.

Autoboxing also works in reverse, allowing you to assign a wrapper object to a primitive variable, as shown in the following example:
```
Integer intObj = 42;
int i = intObj; // autounboxing
```

In this example, the Integer object with the value 42 is assigned to the Integer variable "intObj". Then, when "intObj" is assigned to the primitive int variable "i", the value is automatically unwrapped using autounboxing.

Autoboxing can make code more concise and easier to read by eliminating the need to manually convert between primitive values and their wrapper classes in many situations. However, it can also introduce performance overhead, especially when creating and destroying large numbers of wrapper objects.

#### 1.6 What are the advantages of auto boxing
Autoboxing is a feature in Java that allows automatic conversion of primitive data types (such as int, float, double) to their corresponding object wrapper classes (such as Integer, Float, Double) and vice versa. Here are some advantages of autoboxing:

- Convenience: Autoboxing allows for more convenient coding since it eliminates the need to manually convert primitive types to object types and vice versa.
- Improved readability: Autoboxing can improve the readability of code since it makes the code easier to understand, especially when dealing with collections of objects.
- Flexibility: Autoboxing provides more flexibility when working with collections or generic types, as they require objects instead of primitives.
- Compiler assistance: Autoboxing is supported by the compiler, which can help identify potential errors and issues in the code.
- Reduces boilerplate code: Autoboxing reduces the amount of boilerplate code required, which can make the code more concise and easier to read.

Overall, autoboxing provides a more convenient and flexible way of working with primitive types and their corresponding object types, which can make the code easier to read and write.

#### 1.7 What is casting
Casting is a process in programming where a data type is explicitly converted to another data type. In Java, there are two types of casting: primitive casting and object casting.

1. Primitive casting: Primitive casting is the process of converting one primitive data type to another. For example, casting an int to a double, or a float to an int. This is typically done when the data type of a variable needs to be changed to perform a specific operation or when a method requires a specific data type as input.
Here's an example:
```
int i = 10;
double d = (double) i; // casting int to double
```

2. Object casting: Object casting is the process of converting one object type to another object type. This is typically done when a superclass reference needs to be converted to a subclass reference, or vice versa. For example, casting a Dog object to an Animal object or an Animal object to a Dog object.
Here's an example:
```
Animal animal = new Dog(); // upcasting
Dog dog = (Dog) animal; // downcasting

```

It's important to note that casting can result in data loss or errors if the conversion is not valid. Therefore, it's important to understand the data types involved and the potential consequences of casting before doing so.

#### 1.8 What is implicit casting
Implicit casting, also known as implicit type conversion, is the automatic conversion of a data type to another data type by the programming language itself, without the need for explicit conversion or casting operators. In other words, implicit casting happens when the compiler automatically converts a variable or expression from one data type to another data type, when it is required to perform an operation or assignment.

For example, if you have an integer variable `x` and a floating-point variable `y`, and you want to add them together, the compiler will automatically convert the integer `x` to a floating-point number before the addition is performed. This is because the addition operator requires both operands to be of the same data type.

Implicit casting can be convenient in some cases, as it allows you to write code that is more concise and easier to read. However, it can also lead to unexpected results or errors if you are not careful, so it is important to be aware of the implicit casting rules of the programming language you are using.

#### 1.9 What is explicit casting
Explicit casting, also known as type casting or type conversion, is the process of converting a value from one data type to another data type by explicitly specifying the target data type.

In programming, the need for explicit casting often arises when the data type of a value needs to be changed to perform a specific operation or when storing data in a specific format. For example, when dividing two integers, the result may be a floating-point number, and if you want to store the result in an integer variable, you need to perform an explicit casting to convert the floating-point number to an integer.

In most programming languages, explicit casting is done using casting operators or functions. The syntax for explicit casting depends on the programming language you are using, but it generally involves placing the target data type in parentheses before the value you want to cast. For example, in Java, the syntax for explicit casting looks like this:
```
double x = 3.14;
int y = (int) x; // casting double to int
```
Note that explicit casting may result in the loss of precision or data, depending on the data types involved. Therefore, it is important to be careful when performing explicit casting and ensure that the data is converted correctly.

### 2. Strings
#### 2.1 Are all String’s immutable
Yes, all strings in Java are immutable.

In Java, the String class is designed to be immutable, meaning that once a String object is created, its value cannot be changed. Any operation that modifies a String object, such as concatenation or substring extraction, returns a new String object rather than modifying the original object.

For example, consider the following code:
```
String s1 = "Hello";
String s2 = s1.concat(" World");
System.out.println(s1); // Output: "Hello"
System.out.println(s2); // Output: "Hello World"
```
In this code, s1 is a string object containing the value "Hello". The concat method is called on s1 to append the string " World" to it. However, the original s1 object remains unchanged, and a new String object s2 is created that contains the concatenated string "Hello World".

The immutability of strings in Java has several advantages, such as improved security, thread safety, and ease of use. It also allows for efficient string manipulation operations, as intermediate results can be cached and reused.

#### 2.2 Where are String values stored in memory
In Java, string values are stored in the string pool, which is a special area of the heap memory allocated by the Java Virtual Machine (JVM).

When a string is created in Java, the JVM first checks if the string value already exists in the string pool. If the value exists, the JVM returns a reference to that string object. If the value does not exist, a new string object is created and added to the string pool.

Strings in Java are immutable, which means that their values cannot be changed after they are created. This allows the JVM to safely reuse string objects that have the same value, rather than creating new objects for each occurrence of a string value.

The string pool is also known as the intern pool, and it is implemented as a hash table, with string values serving as the keys and the string objects serving as the values. The hash table is maintained by the JVM and is shared across all threads in a Java application.

It is important to note that not all string objects are stored in the string pool. String objects created using the `new` operator are stored in the heap memory, just like any other object, and are not added to the string pool. This means that if you create a new string object using the `new` operator, it will not be equal to any string object in the string pool, even if the value of the string is the same.

#### 2.3 Why should you be careful about String concatenation(+) operator in loops
You should be careful about using the string concatenation (+) operator in loops because it can lead to performance issues and slow down your program.

When you use the + operator to concatenate strings in a loop, the program needs to create a new string object each time the operator is used. This means that the memory allocation and garbage collection processes will be called repeatedly, which can lead to a significant performance overhead.

Additionally, string concatenation using the + operator creates a new copy of the concatenated strings, which can also result in unnecessary memory usage.

To avoid these issues, it's recommended to use other alternatives to string concatenation in loops, such as StringBuilder. These classes allow you to append strings without creating new string objects, which can significantly improve the performance of your program.

Alternatively, you can also use string formatting techniques, such as interpolation, to create strings more efficiently in loops. Overall, it's important to be mindful of the performance implications of string concatenation in loops and use alternative approaches when necessary.

#### 2.4 What are differences between String and StringBuffer
String and StringBuffer classes are used to manipulate strings. Here are some differences between the two:
- Immutability: String objects are immutable, which means that their values cannot be changed once they are created. StringBuffer objects, on the other hand, are mutable, which means that their values can be modified.
- Thread-safety: String objects are thread-safe, which means that they can be safely accessed and modified by multiple threads. StringBuffer objects are also thread-safe, but their counterpart, StringBuilder, is not thread-safe.
- Performance: Because String objects are immutable, any operation that modifies a string actually creates a new string object. This can result in a performance overhead, especially when manipulating large strings. StringBuffer objects, being mutable, avoid this overhead and can be more efficient when manipulating strings.
- API: The API for String and StringBuffer is different. String provides methods for comparing, searching, and manipulating strings, while StringBuffer provides methods for modifying strings.
- Use cases: String objects are useful when you need to represent a fixed string of characters that will not change. StringBuffer objects are useful when you need to modify a string multiple times, such as when building a string dynamically.

In summary, the main differences between String and StringBuffer are their immutability, thread-safety, performance, API, and use cases. If you need to modify a string multiple times, use StringBuffer, otherwise, use String.

#### 2.5 What are differences between StringBuilder and StringBuffer
both StringBuilder and StringBuffer are used for manipulating strings, but there are some differences between them:
- Thread Safety: `StringBuffer` is thread-safe, which means it is synchronized and can be accessed by multiple threads at the same time. On the other hand, `StringBuilder` is not thread-safe and should be used when no concurrent access is required.
- Performance: Because `StringBuffer` is synchronized, it is slower than `StringBuilder`. `StringBuilder` is faster because it is not synchronized.
- Mutability: Both `StringBuilder` and `StringBuffer` are mutable, meaning you can change the value of a string after it has been created.
- Availability: `StringBuilder` was introduced in Java 5 while `StringBuffer` has been in Java since version 1.0.

In general, if you are working with multiple threads, or require thread-safe operations, then you should use `StringBuffer`. However, if you are working in a single-threaded environment or performance is a concern, then you should use `StringBuilder`.

### 3. Object oriented programming basics
#### 3.1 What is state of an object
In object-oriented programming, the state of an object refers to the values of its properties or attributes at a particular point in time. An object is an instance of a class, and each object has its own set of properties or attributes that describe its current state.

For example, let's consider a car object. The state of the car object would include its current speed, fuel level, gear, and so on. If we change any of these properties, we are changing the state of the car object.

The state of an object can change over time as a result of the object's methods or external influences. For example, if we call the "accelerate" method of the car object, the speed property of the car object will change, and the state of the car object will be updated accordingly.

Understanding the state of an object is essential in object-oriented programming, as it enables us to model real-world systems and perform operations on objects based on their current state.

#### 3.2 What is behavior of an object
In object-oriented programming, the behavior of an object refers to the actions that the object can perform, as defined by its methods or functions. An object's behavior is determined by the code within its methods, which define what actions the object can take and how it will respond to specific events or messages.

For example, let's consider a dog object. The behavior of the dog object would include actions such as barking, wagging its tail, and fetching a ball. These actions are defined in the methods of the dog object, such as "bark()", "wagTail()", and "fetchBall()".

The behavior of an object can be triggered by internal or external events. For example, the dog object's "bark()" method might be called in response to a specific command from its owner, or it might bark in response to an unexpected noise.

Understanding the behavior of an object is critical in object-oriented programming, as it enables us to create robust, modular, and reusable code. By defining the behavior of an object within its methods, we can encapsulate its functionality and make it easier to modify, extend, and maintain the code over time.

#### 3.3 What is the super class of every class in Java
In Java, the superclass of every class is the Object class. The Object class is a root class in Java, which means it does not have a superclass. All other classes in Java, directly or indirectly, inherit from the Object class.

The Object class provides a set of common methods that all Java objects can use, including methods for getting and setting the class of an object, for comparing objects for equality, and for performing basic operations such as cloning and garbage collection.

Since every class in Java extends the Object class, it inherits all of its methods and properties. This means that any class can be treated as an Object and can be passed as a parameter to methods that take an Object argument.

The Object class serves as a foundation for object-oriented programming in Java, providing a common base for all other classes and enabling them to share a common set of behaviors and properties.

#### 3.4 Explain about toString method in java
In Java, the toString() method is a built-in method of the Object class that returns a string representation of the object. The toString() method is automatically inherited by all other classes in Java, and it is often overridden by classes to provide a more meaningful string representation of their objects.

The default implementation of the toString() method, which is inherited from the Object class, returns a string that consists of the class name of the object, followed by an "@" symbol, and then the object's hash code in hexadecimal format.

For example, consider the following code:
```
public class Person {
  private String name;
  private int age;

  public Person(String name, int age) {
    this.name = name;
    this.age = age;
  }
}

public class Main {
  public static void main(String[] args) {
    Person p = new Person("John", 30);
    System.out.println(p.toString());
  }
}
```
The output of this code will be something like: `Person@6d06d69c`. This is the default implementation of the toString() method inherited from the Object class.

To provide a more meaningful string representation of an object, the toString() method can be overridden in a class. The new implementation can return any string that represents the object in a more human-readable way.

For example, let's override the toString() method in the Person class to return a string that includes the person's name and age:

```
public class Person {
  private String name;
  private int age;

  public Person(String name, int age) {
    this.name = name;
    this.age = age;
  }

  @Override
  public String toString() {
    return "Person{name='" + name + "', age=" + age + "}";
  }
}

public class Main {
  public static void main(String[] args) {
    Person p = new Person("John", 30);
    System.out.println(p.toString());
  }
}
```

The output of this code will be: Person{name='John', age=30}. This new implementation of the toString() method provides a more meaningful string representation of the Person object.

#### 3.5 What is the use of equals method in Java
In Java, the `equals()` method is used to compare the equality of two objects. Specifically, it is used to compare the content of two objects to determine whether they are equal or not. The `equals()` method is defined in the Object class, which is the base class for all classes in Java.

When you want to compare the equality of two objects, you can use the `equals()` method to perform the comparison. The `equals()` method takes an Object as a parameter and returns a boolean value that indicates whether the two objects are equal or not.

The `equals()` method is often used in conjunction with the hashCode() method to implement hash tables, which are used for efficient data lookup in many programming applications. When implementing the `equals()` method, it is important to follow the contract defined by the Java specification, which states that if two objects are equal according to the `equals()` method, they must also have the same hash code.

It is also worth noting that many classes in Java override the `equals()` method to provide their own implementation of object equality. For example, the String class defines `equals()` to compare the contents of two strings, while the Integer class defines `equals()` to compare the numerical values of two integers.

#### 3.6 Can super class reference variable can hold an object of sub class

Yes,  a super class reference variable can hold an object of a subclass. This is possible because a subclass is a type of its superclass, and therefore can be treated as an instance of its superclass.

Here is an example:
```
class Animal {
    public void makeSound() {
        System.out.println("The animal makes a sound");
    }
}

class Dog extends Animal {
    public void makeSound() {
        System.out.println("The dog barks");
    }
}

public class Main {
    public static void main(String[] args) {
        Animal animal = new Animal();
        Dog dog = new Dog();

        // super class reference variable holds an object of subclass
        Animal dogAsAnimal = new Dog();

        animal.makeSound();         // Output: The animal makes a sound
        dog.makeSound();            // Output: The dog barks
        dogAsAnimal.makeSound();    // Output: The dog barks
    }
}
```
In this example, we have a `Animal` class and a `Dog` class which extends the `Animal` class. The `Animal` class has a `makeSound` method that prints "The animal makes a sound", while the `Dog` class overrides this method with its own implementation that prints "The dog barks".

In the `main` method, we create an instance of the `Animal` class and an instance of the Dog class. We also create a reference variable of type `Animal` that holds an instance of the `Dog` class. We can then call the `makeSound` method on each object and see that the `Dog` object calls its own implementation of `makeSound`, while the `Animal` object and `dogAsAnimal` object call the `makeSound` method of the `Animal` class.

This demonstrates that a superclass reference variable can hold an object of a subclass, and that the method that is called depends on the actual object that the reference variable is referring to.

#### 3.7 When do you use an abstract class
You should use an abstract class in Java when you want to create a common base class that provides some common functionality, but allows its subclasses to provide their own implementations for some methods. Here are some specific situations where you might use an abstract class:

- When you want to define a contract: An abstract class can be used to define a contract or an interface for a group of related classes. By defining abstract methods that must be implemented by the subclass, you can ensure that all the subclasses conform to a common interface.
- When you want to provide a default implementation: An abstract class can also provide default implementations for some methods, which can be shared by all the subclasses. This can help avoid code duplication and make your code more maintainable.
- When you want to provide a template method: An abstract class can define a template method, which is a method that provides a high-level algorithm or flow of control, but delegates some steps to its subclasses. This can help ensure consistency and reuse in your code.
- When you want to encapsulate common behavior: An abstract class can encapsulate common behavior that can be reused by its subclasses. By providing some concrete methods, an abstract class can help simplify the implementation of its subclasses.
- When you want to control the subclassing: An abstract class can also be used to control the subclassing, by making the constructor protected and providing factory methods to create instances of the subclass. This can help ensure that the subclass is created and used correctly.

#### 3.8 What is the use of this()
In Java, `this()` is a special keyword that is used to call one constructor from another constructor in the same class. It is used to avoid code duplication and to ensure that the object is properly initialized before any other methods are called on it.

The `this()` keyword must be the first statement in the constructor, and it can be used with or without arguments. When used without arguments, it calls the default constructor of the same class. When used with arguments, it calls another constructor in the same class that matches the argument list.

Here's an example of how to use `this()` to call another constructor in the same class:
```
public class MyClass {
    private int value;

    public MyClass() {
        this(0);
    }

    public MyClass(int value) {
        this.value = value;
    }
}
```
In this example, we have a class called `MyClass` with two constructors. The first constructor takes no arguments and calls the second constructor with a value of 0 using `this()`. The second constructor takes an integer argument and initializes the `value` field.

By calling the second constructor from the first constructor, we avoid duplicating the code to initialize the value field. This ensures that the object is properly initialized before any other methods are called on it.

#### 3.9 Can a constructor be called directly from a method in java

Yes, a constructor can be called directly from a method in Java using the `this()` keyword. The `this()` keyword is used to call a constructor from another constructor in the same class. When used inside a method, `this()` can be used to call the constructor of the same class.

Here's an example of how to call a constructor from a method using `this()`:
```
public class MyClass {
    private int value;

    public MyClass(int value) {
        this.value = value;
    }

    public void setValue(int value) {
        this.value = value;
        this.myConstructor(); // Call constructor from method
    }

    private void myConstructor() {
        System.out.println("Value set to " + this.value);
    }
}
```
In this example, we have a class called `MyClass` with a private field called `value`, and a constructor that takes an integer argument and initializes the `value` field. We also have a method called `setValue` that sets the value of `value` and calls `myConstructor()` using `this()`.

The `myConstructor()` method is a private method that simply prints a message indicating that the value has been set.

We can create an instance of `MyClass` and call the `setValue` method to see the constructor being called from the method:
```
MyClass myObj = new MyClass(10);
myObj.setValue(20); // Output: Value set to 20
```

In this example, we have created an instance of `MyClass` with an initial value of 10, and then called the `setValue` method to set the value to 20. The `setValue` method sets the value and calls the constructor through` this()`, which prints a message indicating that the value has been set to 20.

#### 3.10  Is a super class constructor called even when there is no explicit call from a sub class constructor

If a subclass constructor does not explicitly call a superclass constructor using the `super()` keyword, the default constructor of the superclass will be called automatically. This means that a superclass constructor is always called, even if there is no explicit call from the subclass constructor.

Here's an example to illustrate this behavior:
```
public class SuperClass {
    public SuperClass() {
        System.out.println("SuperClass constructor");
    }
}

public class SubClass extends SuperClass {
    public SubClass() {
        System.out.println("SubClass constructor");
    }
}

public class Main {
    public static void main(String[] args) {
        SubClass obj = new SubClass();
    }
}
```
In this example, we have a SuperClass with a default constructor that prints a message, and a SubClass that extends SuperClass and has a default constructor that also prints a message. We then create an instance of SubClass in the main method.

When we run the program, we get the following output:
```
SuperClass constructor
SubClass constructor
```
This output shows that the constructor of the superclass SuperClass is called automatically, even though we did not explicitly call it from the SubClass constructor.

It's important to note that if the superclass does not have a default constructor, the subclass constructor must explicitly call one of the superclass constructors using super(). If the subclass constructor does not explicitly call a superclass constructor and the superclass does not have a default constructor, the compiler will generate an error.

