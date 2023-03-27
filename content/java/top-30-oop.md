---
title: "Top 30 OOPS Interview Questions in Java with Answers"
date: 2021-06-21T16:23:19+08:00
categories:
- java
tags:
- java
- Interview
---
#### 1. What is method overloading in OOP or Java? 

Method overloading in Java is an object-oriented programming concept that allows a programmer to declare two methods of the same name but with different method signatures, like a change in the argument list or a change in the type of argument. 
Original method :  
```
public void  show(String message){
      System.out.println(message);
}
```
Overloaded method: number of arguments is different:
<!--more-->
```
public void show(String message, boolean show){

        System.out.println(message); 
}

```

Overloaded method: type of argument is different:

```
public void  show(Integer message){
      System.out.println(message);
}
```

Not an Overloaded method: the only return type is different


```
public boolean show(String message){

        System.out.println(message); 
        return false; 
    
}

```

#### 2. What is the method overriding in OOP or Java?
It's one of the magic of object-oriented programming where the method is chosen based upon an object at runtime. In order for method overriding, we need Inheritance and Polymorphism, as we need a method with the same signature in both superclass and subclass. A call to such a method is resolved at runtime depending upon the actual object and not the type o variable. See the answer for a more detailed discussion.  
There are few rules which needs to be followed while *overriding* any method in Java, failure to follow these rules results in a compile-time error Java.

1. The first and most important rule regarding method overriding in Java is that you can only override a method in a subclass. You can not override the method in the same class.
2. A second important rule of method overriding in Java that name and signature of the method must be the same in Super class and Sub class or in the interface and its implementation.
3. The third rule to override a method in Java is that the overriding method can not reduce the accessibility of the overridden method in Java. For example, if the overridden method is public then the overriding method can not be protected, private or package-private;   
But the opposite is true overriding method can increase the accessibility of the method in Java, i.e. if the overridden method is protected then The overriding method can be protected or public.
4. Another worth noting rule of method overriding in Java is that the overriding  method can not throw checked Exception is higher in the hierarchy than the overridden method. This means if the overridden method throws IOException then the overriding method can not throw java.lang.Exception in its throws clause because of java.lang.Exception comes higher than IOException in Exception hierarchy. 
5. You can not override private, static and final the method in Java. private and static method are bonded during compile time using static binding in Java and doesn't resolve during runtime. the overriding the final method in Java is a compile-time error. Though private and static methods can be hidden if you declare another method with the same and signature in the subclass.
6. The overridden method is called using dynamic binding in Java at runtime based upon the type of Object. As shown in the following example of method overriding in Java.
7. If you are extending the abstract class or implementing the interface then you need to override all abstract methods unless your class is not abstract. an abstract method can only be used by using method overriding.
8. Always use @Override annotation while the overriding method in Java. Though this is not a rule it's one of the best Java coding practices to follow.

```
public class CollectionTest {
  
    public static void main(String args[]) {
  
      Runnable task = new Task();
      task.run(); //call overridden method in Task
    
      task = new PeriodicTask();
      task.run(); //calls overridden method in PeriodicTas

    }
  
  
}

class Task implements Runnable{

    @Override
    public void run() {
        System.out.println("Run method overridden in Task class");
    }
  
}

class PeriodicTask extends Task{
  
    @Override
    public void run() {
        System.err.println("overridden method run() in PeriodicTask class");
    }
}

```
#### 3. What is the method of hiding in Java? 

When you declare two static methods with the same name and signature in both superclass and subclass then they hide each other i.e. a call to the method in the subclass will call the static method declared in that class and a call to the same method is superclass is resolved to the static method declared in the super-class.

#### 4. Is Java a pure object-oriented language? if not why? 

Java is not a pure object-oriented programming language e.g. there are many things you can do without objects e.g. static methods. Also, primitive variables are not objects in Java. 

#### 5. What are the rules of method overloading and overriding in Java?

One of the most important rules of method overloading in Java is that the method signature should be different i.e. either the number of arguments or the type of arguments. Simply changing the return type of two methods will not result in overloading, instead, the compiler will throw an error. 

On the other hand, method overriding has more rules e.g. name and return type must be the same, method signature should also be the same, the overloaded method cannot throw a higher exception, etc. See the answer for a full list of rules related to method overloading and overriding in Java.

#### 6. The difference between method overloading and overriding?

Several differences but the most important one is that method overloading is resolved at compile-time and method overriding is resolved at runtime. The compiler only used the class information for method overloading, but it needs to know the object to resolved overridden method calls.

#### 7. Can we overload a static method in Java?

Yes, you can overload a static method in Java. You can declare as many static methods of the same name as you wish provided all of them have different method signatures.


```
/**    
 * Java program to show that we can overload static method in Java.
 */
public class StaticOverloadingTest {
  
    public static void main(String args[]) {
        greet("John"); //will call a static method with one String argument
        greet("John," "Good Morning"); //overloaded static method will be call    

    }
  
    /*
     * static method which will be overloaded
     */
    public static void greet(String name){
        System.out.println("Hello " + name);
    }
  
    /*
     * Another static method that overloads above Hello method
     * This shows that we can overload static method in Java
     */
    public static void greet(String name, String greeting){
        System.out.println(greeting + " " + name);
    }


}

```

#### 8. Can we override the static method in Java?

No, you cannot override a static method because it's not bounded to an object. Instead, static methods belong to a class and are resolved at compile time using the type of reference variable. But, Yes, you can declare the same static method in a subclass, which will result in method hiding i.e. if you use the reference variable of type subclass then the new method will be called, but if you use the reference variable of superclass then the old method will be called.

#### 9. Can we prevent overriding a method without using the final modifier? 

Yes, you can prevent the method overriding in Java without using the final modifier. In fact, there are several ways to accomplish it e.g. you can mark the method as private or static, those cannot be overridden.

#### 10. Can we override a private method in Java?

No, you cannot. Since the private method is only accessible and visible inside the class they are declared, it's not possible to override them in subclasses. Though, you can override them inside the inner class as they are accessible there.

#### 11. What is the covariant method overriding in Java? 

In the covariant method overriding, the overriding method can return the subclass of the object returned by the original or overridden method. This concept was introduced in Java 1.5 (Tiger) version and it's very helpful in case the original method is returning a general type like Object class, because, then by using the covariant method overriding you can return a more suitable object and prevent client-side typecasting. One of the practical use of this concept is when you override the clone() method in Java.

#### 12. Can we change the return type of method to subclass while overriding?

Yes, you can, but only from Java 5 onward. This feature is known as covariant method overriding and it was introduced in JDK 5 release. This is immensely helpful if the original method return super-class like the clone() method returns a java.lang.Object. By using this, you can directly return the actual type, preventing client-side type-casting of the result.

#### 13. Can we change the argument list of an overriding method?

No, you cannot. The argument list is part of the method signature and both overriding and overridden methods must have the same signature.

#### 14. Can we override a method that throws runtime exception without throws clause?

Yes, there is no restriction on unchecked exceptions while overriding. On the other hand, in the case of a checked exception, an overriding exception cannot throw a checked exception which comes higher in type hierarchy e.g. if the original method is throwing IOException then the overriding method cannot throw java.lang.Exception or java. lang.Throwable.
