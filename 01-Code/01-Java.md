

https://www.geeksforgeeks.org/java-interview-questions/?ref=gcse_outind
# Data Types

Primitive Data Type:  data are single values with no special capabilities.
`boolean`, `byte`, `char`, `short`, `int`, `long`, `float`, `double`

Non-Primitive Data Type: 
`Strings`, `Array`, `Class`, `Object`

Pointer: Java does not provide the support of Pointer. Because java needs to be more secure.

Wrapper classes: Java contains 8 wrapper classes.
`Boolean`, `Byte`, `Char`, `Short`, `Integer`, `Long`, `Float`, `Double`
1. Wrapper classes are final and immutable
2. Provides methods like valueOf(), parseInt(), etc.
3. It provides the feature of autoboxing and unboxing.


# Keywords

### Transient

`Transient` is used at the time of serialization if we do not want to save the value of a specific variable.


### final



# Constructor

Constructor is a special method that is used to initialize objects.
The name of constructor is same as of the class.
Constructor is called when a object is created by `new` keyword.

```java
public class Apple {
	private String color;

	// default constructor
	public Apple() {
	}
	// Parameterized Constructor
	public Apple(String color) {
		this.color = color;
	}
}
```

If you don't provide a constructor in a class, the compiler automatically gennerates a default constructor.

# OOP
## 1. Inheritance & composition

Inheritance is a popular concept of Object-Oriented Programming, in which a class can inherit the properties and methods from any other class.

Java doesn't support multiple inheritance.


why composition is more advantageous than inheritance:
- Tight Coupling: Whenever any changes are made to the superclass, these changes can affect the behavior of all its child or Subclasses. This problem makes code less flexible and also creates issues during maintenance. This problem also leads to the Tight coupling between the classes.
- Fragile Base Class Problem:**** When the changes to the base class can break the functionality of its derived classes. This problem can make it difficult to add new features or modify the existing ones. This problem is known as the Fragile Base class problem.
- Limited Reuse:**** Inheritance in Java can lead to limited code reuse and also code duplication. As a subclass inherits all the properties and methods of its superclass, sometimes it may end up with unnecessary code which is not needed. This leads to a less maintainable codebase.



## 2. Interface



## 3. polymorphism




# Collection Framework

The collection framework contains classes(ArrayList, Vector, LinkedList, PriorityQueue, TreeSet) and multiple interfaces (Set, List, Queue, Deque) where every interface is used to store a specific type of data.


## 1. ArrayList vs. Vector


## LinkedList


## Stack


## HashMap & ConcurrentHashMap


## Priority Queue


# StringBuffer vs. StringBuilder


# Exception

 Checked Exception and Unchecked Exception



# Concurrency

## 1. Thread

Create a thread.



Thread States

