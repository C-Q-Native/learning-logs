---
title: "Java 面试题汇编 (五)"
date: 2021-06-25T17:23:19+08:00
categories:
- java
tags:
- java
- Interview
---
### 1. Advanced object oriented concepts
#### 1.1 What is polymorphism
Polymorphism is a fundamental concept in object-oriented programming (OOP) and refers to the ability of an object to take on multiple forms or behaviors. In Java, polymorphism allows an object to be treated as if it is of any type in its inheritance hierarchy.

Java supports two types of polymorphism: static polymorphism (also known as method overloading) and dynamic polymorphism (also known as method overriding).

Static polymorphism is achieved by defining multiple methods with the same name in a class, but with different parameters. The Java compiler will determine which method to call at compile-time based on the number and types of arguments passed to the method.

Dynamic polymorphism is achieved by defining a method in a subclass that has the same signature (name, return type, and parameters) as a method in its superclass. When an object of the subclass is used to call the method, the Java runtime system determines which implementation of the method to call based on the actual type of the object at runtime.

Polymorphism is an important concept in Java and OOP in general because it allows for greater flexibility and modularity in designing and implementing software systems.

#### 1.2  What is the use of instanceof operator in Java
The `instanceof` operator in Java is used to check if an object is an instance of a particular class or implements a particular interface. It is a binary operator that returns a boolean value: true if the object is an instance of the specified class or implements the specified interface, and false otherwise.

The `instanceof` operator is commonly used in Java for two main purposes:

- Type checking: It allows you to check the type of an object at runtime. This is useful when you have a reference to an object of unknown type and you want to perform some operations or method calls on it that are specific to a particular class or interface. By using the `instanceof` operator, you can first check the type of the object and then safely cast it to the appropriate type.
- Polymorphism: It allows you to check if an object is an instance of a superclass or interface, which is useful for implementing polymorphic behavior in Java. For example, if you have a list of objects that implement a common interface, you can use the `instanceof` operator to determine the type of each object in the list and then perform different operations or method calls based on their specific type.

Here's an example that illustrates the use of the `instanceof` operator in Java:
```
Object obj = "Hello World";
if (obj instanceof String) {
    String str = (String) obj;
    System.out.println("Length of string: " + str.length());
} else {
    System.out.println("Not a string object");
}
```

#### 1.3 What is coupling
coupling refers to the degree of interdependence between classes or components in a software system. It is a measure of how closely two or more classes are connected or dependent on each other.

Java offers several mechanisms for interclass communication, including method calls, inheritance, composition, and interface implementation. The way these mechanisms are used can affect the level of coupling between classes.

Low coupling is generally considered desirable in Java because it leads to more modular, flexible, and reusable code that is easier to test and maintain. High coupling, on the other hand, can make the code harder to modify, debug, and optimize.

Here are some examples of different types of coupling in Java:
- Tight coupling: Occurs when two or more classes are highly interdependent and communicate with each other directly or through a shared data structure. For example, if a class relies on the internal implementation details of another class, it creates a tight coupling between them.
- Loose coupling: Occurs when two or more classes are relatively independent and communicate with each other through well-defined interfaces. For example, if a class only relies on the public methods of another class, it creates a loose coupling between them.
- Content coupling: Occurs when one class accesses or modifies the internal data or methods of another class, creating a strong dependency between them.
- Common coupling: Occurs when multiple classes access or modify the same global data or resource, which can lead to conflicts and synchronization issues.
- Control coupling: Occurs when one class controls the behavior or flow of another class, creating a complex and tightly coupled system.

#### 1.4 What is cohesion
In Java, cohesion refers to the degree to which the methods and fields within a class are related and work together towards a common purpose. In other words, it's a measure of how well a class's members are organized and how focused they are on performing a specific set of tasks.

High cohesion is generally considered a good design principle, as it leads to code that is more modular, easier to maintain, and less prone to bugs. This is because highly cohesive classes tend to have methods and fields that are closely related and tightly focused on a single responsibility or purpose.

On the other hand, low cohesion can lead to code that is more difficult to understand and modify, since the class's members are not well-organized and may perform multiple, unrelated tasks. This can make it harder to identify and fix bugs, and can also make the code more difficult to reuse in other contexts.

To improve cohesion in Java, it's important to ensure that each class has a clear and well-defined purpose, and that its methods and fields are organized in a logical and coherent way. This can be achieved through techniques such as abstraction, encapsulation, and modular design.

#### 1.5 What is encapsulation
Encapsulation is a fundamental principle in object-oriented programming (OOP) that refers to the practice of hiding internal implementation details of an object from the outside world and providing a well-defined interface for interacting with the object. In other words, it involves bundling the data and methods that operate on the data within a single unit, and restricting access to the data from outside the unit.

Encapsulation provides several benefits to software development, such as increased security, modularity, and flexibility. By encapsulating the implementation details of an object, it becomes easier to maintain and update the object's behavior without affecting other parts of the code. Additionally, encapsulation allows developers to control access to the object's data, ensuring that it is only modified in a controlled and safe manner.

In Java, encapsulation is typically achieved through the use of access modifiers, such as private, public, and protected, to control the visibility of variables and methods. Private variables and methods are only accessible within the class itself, while public variables and methods can be accessed from any other class. Protected variables and methods are accessible within the class and its subclasses.

Encapsulation is a key concept in OOP, and is essential to writing clean, maintainable, and extensible code. By encapsulating implementation details and providing well-defined interfaces, developers can create software that is easier to understand, maintain, and extend over time.


#### 1.6 What is an inner class
An inner class in Java is a class that is defined inside another class. Inner classes are also known as nested classes, as they are defined within another class's scope. There are four types of inner classes in Java: static nested classes, non-static nested classes (also known as inner classes), local classes, and anonymous classes.

Static nested classes are declared as static and do not have access to the enclosing class's instance variables. Non-static nested classes, or inner classes, have access to the enclosing class's instance variables and can be declared as public, private, protected, or package-private. Local classes are defined inside a method and have access to the method's local variables. Anonymous classes are a special type of local class that does not have a name and is declared inline, typically used for event handling and callbacks.

Inner classes provide several benefits in Java programming, including encapsulation, code organization, and improved modularity. Inner classes can be used to group related code together and hide implementation details, making the code easier to read and maintain. Inner classes can also be used to implement interfaces, abstract classes, and inheritance hierarchies.

Here is an example of a non-static nested class in Java:
```
public class OuterClass {

    private int outerVar;

    public class InnerClass {
        private int innerVar;

        public void setInnerVar(int value) {
            innerVar = value;
        }

        public void printVars() {
            System.out.println("Outer var: " + outerVar);
            System.out.println("Inner var: " + innerVar);
        }
    }

    public void testInnerClass() {
        InnerClass innerObj = new InnerClass();
        innerObj.setInnerVar(10);
        innerObj.printVars();
    }
}
```
In this example, we have an outer class OuterClass with an instance variable outerVar. Inside OuterClass, we define an inner class InnerClass that has its own instance variable innerVar. The testInnerClass method creates an instance of InnerClass and sets the value of innerVar, then calls the printVars method to print both outerVar and innerVar. The inner class InnerClass has access to the instance variable outerVar of the enclosing class OuterClass.

#### 1.7 What is a static inner class
A static inner class in Java is a nested class that is declared with the static keyword. Unlike non-static inner classes, static inner classes do not have access to the instance variables of the enclosing class.

Static inner classes are often used to group related classes together and can be used to encapsulate implementation details. They are also useful when you want to create an object that does not depend on the state of its outer class.

Here is an example of a static inner class in Java:
```
public class OuterClass {
    private static int outerVar;

    public static class InnerClass {
        private int innerVar;

        public void setInnerVar(int value) {
            innerVar = value;
        }

        public void printVars() {
            System.out.println("Outer var: " + outerVar);
            System.out.println("Inner var: " + innerVar);
        }
    }

    public static void testInnerClass() {
        InnerClass innerObj = new InnerClass();
        innerObj.setInnerVar(10);
        innerObj.printVars();
    }
}
```
In this example, we have an outer class `OuterClass` with a static instance variable `outerVar`. Inside `OuterClass`, we define a static inner class `InnerClass` that has its own instance variable innerVar. The `testInnerClass` method creates an instance of `InnerClass` and sets the value of `innerVar`, then calls the printVars method to print both `outerVar` and `innerVar`. Note that `InnerClass` has access to the static `outerVar` of the enclosing class `OuterClass`, but not to any instance variables.

#### 1.8 Can you create an inner class inside a method
Yes, you can create an inner class inside a method in Java. This type of inner class is called a local inner class, and it is defined within the scope of a method. Local inner classes are similar to other inner classes in Java, but they have some restrictions, such as access to only final or effectively final variables from the enclosing method.

Here is an example of a local inner class in Java:
```
public class OuterClass {
    private int outerVar = 10;

    public void testLocalInnerClass() {
        final int localVar = 20;

        class LocalInnerClass {
            public void printVars() {
                System.out.println("Outer var: " + outerVar);
                System.out.println("Local var: " + localVar);
            }
        }

        LocalInnerClass innerObj = new LocalInnerClass();
        innerObj.printVars();
    }
}
```
In this example, we have an outer class `OuterClass` with an instance variable `outerVar`. Inside the `testLocalInnerClass` method, we define a local inner class `LocalInnerClass` that has access to the method's local variable `localVar` and the instance variable `outerVar`. The `printVars` method of `LocalInnerClass` prints both `outerVar` and `localVar`. The `testLocalInnerClass` method creates an instance of `LocalInnerClass` and calls the `printVars` method to print both variables.

Note that the localVar variable is declared as final in the example because local inner classes can only access local variables that are effectively final. An effectively final variable is a variable that is not explicitly declared as final, but whose value does not change after it is initialized.

#### 1.9 What is an anonymous class
In Java, an anonymous class is a local class without a name that is defined and instantiated in a single expression. Anonymous classes are often used to create simple implementations of abstract classes or interfaces.

Here is an example of an anonymous class in Java:
```
public class AnonymousClassDemo {
    public static void main(String[] args) {
        // create an anonymous class that implements the Runnable interface
        Runnable runnable = new Runnable() {
            public void run() {
                System.out.println("Hello from anonymous class!");
            }
        };

        // create a thread using the anonymous class
        Thread thread = new Thread(runnable);
        thread.start();
    }
}
```
In this example, we create an anonymous class that implements the `Runnable` interface, which has a single method `run()`. We define the implementation of the `run()` method in the anonymous class using curly braces, and then we create an instance of the anonymous class and pass it to a Thread constructor. Finally, we start the thread using the `start()` method.

Anonymous classes can also be used to override methods of a superclass or implement methods of an interface, as shown in the following example:
```
public class AnonymousClassDemo {
    public static void main(String[] args) {
        // create an anonymous class that extends the Thread class
        Thread thread = new Thread() {
            public void run() {
                System.out.println("Hello from anonymous class!");
            }
        };

        // start the thread
        thread.start();
    }
}
```
In this example, we create an anonymous class that extends the `Thread` class and overrides its `run()` method to print a message. We create an instance of the anonymous class and start the thread using the `start()` method.

### 2. Modifiers
#### 2.1 What is default class modifier
In Java, the default class modifier is also known as package-private access or default access. This means that if a class is declared with no access modifier, it is accessible only within the same package. Classes with default access are not accessible outside the package in which they are defined.

Here is an example of a class with default access in Java:
```
package com.example.myapp;

class MyClass {
    // class members and methods go here
}
```
In this example, we define a class `MyClass` in the package `com.example.myapp` without specifying an access modifier. This means that the class has default access and is accessible only within the same package.

Default access is often used for classes that are intended to be used only within a particular package and are not part of the public API of a library or application. For example, utility classes, data access objects, or internal implementation classes may use default access to restrict access to their members and methods.

It is important to note that classes with default access can be accessed by other classes within the same package, even if they are in different source files. However, classes in different packages cannot access them directly, unless they are extended by a public class or implemented by a public interface.


#### 2.2 What access types of variables can be accessed from a class in same package
In Java, a class in the same package can access all members (variables and methods) of another class in the same package that have default (package-private) or public access modifiers. This means that the following access types of variables can be accessed from a class in the same package:
1. Default (package-private) access: If a class member is declared with no access modifier, it has default access and can be accessed by any class in the same package. For example:
   
    ```
    package com.example.myapp;

    class MyClass {
        int defaultVar; // default access

        // other class members and methods go here
    }
    ```
    In this example, `defaultVar` has default access, which means that it can be accessed by any class in the `com.example.myapp` package.

1. Public access: If a class member is declared with the `public` access modifier, it can be accessed by any class, regardless of the package it belongs to. For example:
    ```
    package com.example.myapp;

    class MyClass {
        public int publicVar; // public access

        // other class members and methods go here
    }
    ```
    In this example, `publicVar` has public access, which means that it can be accessed by any class, including those outside of the `com.example.myapp` package.

It is important to note that members declared with private or protected access modifiers cannot be accessed by other classes in the same package, unless they are declared within a nested or inner class.

#### 2.3  What access types of variables can be accessed from a class in different package
In Java, a class in a different package can access only those members (variables and methods) of another class that have public access modifier. This means that the following access types of variables can be accessed from a class in a different package:

1. Public access: If a class member is declared with the public access modifier, it can be accessed by any class, regardless of the package it belongs to. For example:
   ```
   package com.example.myapp;

    public class MyClass {
        public int publicVar; // public access

        // other class members and methods go here
    }
    ```
    In this example, `publicVar` has public access, which means that it can be accessed by any class, including those outside of the `com.example.myapp` package.
2. Protected access: If a class member is declared with the protected access modifier, it can be accessed by any subclass, including those in a different package. For example:
   ```
    package com.example.myapp;

    public class MyClass {
        protected int protectedVar; // protected access

        // other class members and methods go here
    }
    ```
    In this example, `protectedVar` has protected access, which means that it can be accessed by any subclass, even if it is in a different package.

It is important to note that members declared with `default` (package-private) or `private` access modifiers cannot be accessed by classes in a different package. In order to access these members, you would need to either make them `public` or create a `public` method that provides access to them.

#### 2.4 What access types of variables can be accessed from a sub class in same package
In Java, a subclass in the same package can access all members (variables and methods) of its superclass that have default (package-private), protected, or public access modifiers. This means that the following access types of variables can be accessed from a subclass in the same package:

1. Default (package-private) access: If a superclass member is declared with no access modifier, it has default access and can be accessed by any subclass in the same package. For example:
   ```
   package com.example.myapp;

    class SuperClass {
        int defaultVar; // default access

        // other class members and methods go here
    }

    class SubClass extends SuperClass {
        void myMethod() {
            defaultVar = 42; // can access defaultVar in superclass
        }
    }
    In this example, defaultVar has default access in SuperClass, which means that it can be accessed by any subclass in the same package, such as SubClass.
2. Protected access: If a superclass member is declared with the protected access modifier, it can be accessed by any subclass, including those in the same package. For example:
   ```
   package com.example.myapp;

    class SuperClass {
        protected int protectedVar; // protected access

        // other class members and methods go here
    }

    class SubClass extends SuperClass {
        void myMethod() {
            protectedVar = 42; // can access protectedVar in superclass
        }
    }
    ```
    In this example, protectedVar has protected access in SuperClass, which means that it can be accessed by any subclass, including those in the same package, such as SubClass.
3. Public access: If a superclass member is declared with the public access modifier, it can be accessed by any class, including subclasses in the same package. For example:
   ```
   package com.example.myapp;

    public class SuperClass {
        public int publicVar; // public access

        // other class members and methods go here
    }

    class SubClass extends SuperClass {
        void myMethod() {
            publicVar = 42; // can access publicVar in superclass
        }
    }
    ```
    In this example, publicVar has public access in SuperClass, which means that it can be accessed by any class, including subclasses in the same package, such as SubClass.

#### 2.5 What is the use of a final modifier on a class
In Java, the `final` keyword can be used to modify the declaration of a class, indicating that the class cannot be subclassed or extended by any other class.

Declaring a class as `final` has several implications:

- Inheritance: A `final` class cannot be subclassed, which means that it cannot be extended by any other class. This is useful when you have a class that should not be modified or extended in any way.
- Polymorphism: Since a `final` class cannot be subclassed, it cannot be used polymorphically. This means that you cannot create an object of a subclass and assign it to a reference variable of the `final` class.
- Method Overriding: Any method in a final class is automatically considered final as well, which means that it cannot be overridden by any subclass.

In summary, marking a class as `final` provides an additional level of control over the class and its subclasses, making it useful in situations where you want to ensure that a class cannot be modified or extended in any way.

#### 2.6 What is the use of a final modifier on a method
In Java, the `final` keyword can also be used to modify the declaration of a method, indicating that the method cannot be overridden by any subclass.

Declaring a method as `final` has several implications:

- Inheritance: A `final` method cannot be overridden by any subclass. This means that the behavior of the method is fixed and cannot be changed by any subclass.
- Security: By marking a method as `final`, you can ensure that critical methods in a class cannot be modified or overridden by any malicious subclass.
- Efficiency: Since a `final` method cannot be overridden, the Java Virtual Machine (JVM) can optimize the method at compile time, leading to better performance.

In summary, marking a method as `final` provides an additional level of control over the behavior of a class and its subclasses, making it useful in situations where you want to ensure that a method cannot be modified or overridden by any subclass.

#### 2.7 What is a final variable
In Java, a final variable is a variable whose value cannot be changed once it has been initialized.

Declaring a variable as final has several implications:

- Immutability: Once a final variable has been assigned a value, it cannot be changed. This makes the variable immutable, meaning that its value cannot be modified or reassigned.
- Thread Safety: Since a final variable is immutable, it can be safely accessed by multiple threads without the need for synchronization.
- Performance: The Java Virtual Machine (JVM) can optimize access to final variables, which can lead to better performance.

Declaring a variable as `final` is useful when you have a value that should not be changed during the execution of a program, such as constants or configuration settings. It can also be used to ensure that a variable is initialized only once, preventing accidental changes to its value later in the program.

#### 2.8 how jvm to optimize access to final variables
In Java, the final keyword can be used to declare a variable whose value cannot be changed once it has been initialized. This allows the Java Virtual Machine (JVM) to perform certain optimizations that can lead to better performance.

One optimization that the JVM can perform on final variables is called "constant folding". This means that the JVM can replace references to the final variable with the actual value of the variable at compile time. This eliminates the need to access the variable at runtime, which can improve performance.

Another optimization that the JVM can perform on final variables is called "inlining". This means that the JVM can replace a method call that uses a final variable with the actual value of the variable. This eliminates the overhead of the method call, which can improve performance.

In addition to these optimizations, the use of final variables can also help the JVM perform other optimizations, such as escape analysis and lock elision.

Overall, the use of final variables can help the JVM optimize the performance of Java programs by reducing the need for runtime access to variables and enabling other optimizations that are not possible with mutable variables.

#### 2.9 how jvm to optimize access to final method
In Java, the final keyword can be used to declare a method that cannot be overridden by any subclass. This allows the Java Virtual Machine (JVM) to perform certain optimizations that can lead to better performance.

One optimization that the JVM can perform on final methods is called "inlining". This means that the JVM can replace a method call that uses a final method with the actual code of the method at compile time. This eliminates the overhead of the method call, which can improve performance.

Another optimization that the JVM can perform on final methods is called "static binding". This means that the JVM can determine at compile time which method to call based on the declared type of the reference variable, rather than having to determine the actual type of the object at runtime. This eliminates the need for runtime method resolution, which can improve performance.

In addition to these optimizations, the use of final methods can also help the JVM perform other optimizations, such as escape analysis and lock elision.

Overall, the use of final methods can help the JVM optimize the performance of Java programs by reducing the need for runtime method resolution and enabling other optimizations that are not possible with non-final methods.

#### 2.10 What is a final argument
In Java, a `final` argument is a method argument that is declared as `final`. This means that the value of the argument cannot be changed within the method.

Declaring a method argument as `final` has several implications:

- Immutability: Once a `final` argument has been assigned a value, it cannot be changed within the method. This makes the argument immutable, meaning that its value cannot be modified or reassigned.
- Safety: Since a `final` argument is immutable, it can be safely accessed by multiple threads without the need for synchronization.
- Clarity: By declaring a method argument as `final`, you indicate that the value of the argument should not be changed within the method. This can make the code easier to read and understand.

Declaring a method argument as `final` is useful when you have a value that should not be changed within a method, such as method parameters or local variables. It can also be used to prevent accidental changes to the value of the argument within the method, which can help improve the safety and clarity of the code.

#### 2.11 What happens when a variable is marked as volatile
In Java, the `volatile` keyword can be used to mark a variable as `volatile`. This has several implications for how the variable is accessed and modified by different threads.

When a variable is marked as `volatile`, it means that:

- Visibility: Any changes made to the variable by one thread are immediately visible to all other threads. This ensures that all threads have a consistent view of the variable's value.
- Atomicity: Accesses to the variable are atomic, meaning that they are indivisible and cannot be interrupted by other threads. This ensures that all threads see a consistent value for the variable.
- Ordering: Accesses to the variable are ordered, meaning that they occur in a predictable sequence. This ensures that all threads see a consistent order of operations.

Using the `volatile` keyword is useful when you have a variable that is accessed by multiple threads and its value needs to be shared between them. Examples of such variables include flags, counters, and status variables.

Note that the `volatile` keyword does not provide synchronization or mutual exclusion. If you need to perform compound operations on a `volatile` variable, you still need to use synchronization or other concurrency mechanisms to ensure that the operations are performed atomically and in the correct order.

#### 2.12 What is a static variable
In Java, a static variable is a class-level variable that is shared by all instances of the class. This means that if you change the value of a static variable in one instance of the class, the change will be reflected in all other instances of the class.

Declaring a variable as static has several implications:

- Memory allocation: The variable is allocated memory space in the class area of the JVM, rather than in the instance area. This means that the variable exists independently of any instances of the class.
- Initialization: Static variables are initialized only once, when the class is loaded by the JVM. This means that all instances of the class share the same initial value for the static variable.
- Accessibility: Static variables are accessible from any static method of the class, as well as from any instance method of the class.

Static variables can be useful for storing information that is common to all instances of a class, such as a counter that keeps track of the number of instances created, or a constant value that is used by all instances of the class. However, you should use static variables judiciously, as they can also create problems such as hidden dependencies and thread safety issues.