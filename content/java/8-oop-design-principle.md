---
title: "8 Object-Oriented Design Principle and pattern"
date: 2021-06-24T16:23:19+08:00
categories:
- java
tags:
- java
- Interview
---
#### 1. What problem is solved by the Strategy pattern in Java?

Strategy pattern allows you to introduce a new algorithm or new strategy without changing the code which uses that algorithm. For example, the Collections.sort() method which sorts the list of the object uses the Strategy pattern to compare objects. Since every object uses a different comparison strategy you can compare various objects differently without changing the sort method.
<!--more-->
#### 2. Which OOP concept Decorator design Pattern is based upon?

The decorator pattern takes advantage of Composition to provide new features without modifying the original class. A very good to-the-point question for the telephonic round. This is quite clear from the UML diagram of the Decorator pattern, as you can see the Component is associated with a Decorator.
![image](https://2.bp.blogspot.com/-jnzC4Kx48Oc/Vmg-K8LVjqI/AAAAAAAAES8/sYAEghzm688/s16000/Decorator%2BDesign%2BPattern%2Bin%2BJava%2BUML.png)

#### 3. When to use the Singleton design pattern in Java? 

When you need just one instance of a class and want that to be globally available then you can use the Singleton pattern. It's not free of cost though because it increases the coupling between classes and makes them hard to test. This is one of the oldest design pattern questions from Java interviews. Please see the answer for a more detailed discussion.

#### 4. What is the difference between State and Strategy Patterns? 
Though the structure or class diagram of the State and Strategy pattern is the same, their intent is completely different. The state pattern is used to do something specific depending upon the state while Strategy allows you to switch between algorithms without changing the code which uses it.


#### 5. What is the difference between Association, Aggregation, and Composition in OOP? 
When an object is related to another object is called association. It has two forms, aggregation, and composition. the former is the loose form of association where the related object can survive individually while later is a stronger form of association where a related object cannot survive individually. For example, the city is an aggregation of people but is the composition of body parts.

#### 6. What is the difference between Decorator, Proxy, and Adapter patterns in Java? 
Again they look similar because their structure or class diagram is very similar but their intent is quite different. The Decorator adds additional functionality without touching the class, Proxy provides access control, and an Adapter is used to make two incompatible interfaces work together. 


#### 7. What is the 5 objects oriented design principle from SOLID? 
SOLID is the term given by Uncle Bob in his classic book, the Clean Code, one of the must-read books for programmers. In SOLID each character stands for one design principle:
S for Single Responsibility Principle
O for Open closed design principle
L for Liskov substitution principle
I for Interface segregation principle
D for Dependency inversion principle


#### 8. What is the difference between Composition and Inheritance in OOP? 
This is another great OOPS concept question because it tests what matters, both of them are very important from a class design perspective. Though both Composition and Inheritance allow you to reuse code, formerly is more flexible than later. 

Composition allows the class to get an additional feature at runtime, but Inheritance is static. You can not change the feature at runtime by substitution of a new implementation. See the answer for a more detailed discussion.


That's all about in this list of object-oriented programming or OOPS concept interview questions. We have seen questions from various OOPS concepts like Abstraction, Encapsulation, Inheritance, Composition, Aggregation, Association, Class, Object, Interface, etc.

We have also seen questions from Object-oriented design principles also known as SOLID principles and GOF design patterns like Strategy, State, and Factory, which are based upon both the object-oriented programming concept and OOP design principle.
