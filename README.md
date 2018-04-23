
# Effective Java 3rd Edition Summary


## Disclaimer
This is a summary of the book "Effective Java 3rd Edition" by Joshua Bloch (https://twitter.com/joshbloch)
The book is awesome and has a lot of interesting view in it. This document is only my take on what I want to really keep in mind when working with Java.
I hope it's not a copyright infringement. If it is, please contact me in order to remove this file from github.

## Creating and destroying objects

__Item 1 : Static factory methods__

Pros
 - They have a name
 - You can use them to control the number of instance (Example : Boolean.valueOf)
 - You can return a subtype of the return class 

Cons
 - You can't subclass a class without public or protected constructor
 - It can be hard to find the static factory method for a new user of the API

Example :
```java
public static Boolean valueOf(boolean b){
	return b ? Boolean.TRUE :  Boolean.FALSE;
}
```

__Item 2 : Builder__

Pros
 - Builder are interesting when your constructor may need many arguments
 - It's easier to read and write
 - Your class can be immutable (Instead of using a java bean)
 - You can prevent inconsistent state of you object

Example :
```java
public class NutritionFacts {
	private final int servingSize;
	private final int servings;
	private final int calories;
	private final int fat;
	private final int sodium;

	public static class Builder {
		//Required parameters
		private final int servingSize:
		private final int servings;
		//Optional parameters - initialized to default values
		private int calories		= 0;
		private int fat 			= 0;
		private int sodium 			= 0;

		public Builder (int servingSize, int servings) {
			this.servingSize = servingSize;
			this.servings = servings;
		}
		public Builder calories (int val) {
			calories = val;
			return this;				
		}
		public Builder fat (int val) {
			fat = val;
			return this;				
		}
		public Builder sodium (int val) {
			sodium = val;
			return this;				
		}
		public NutritionFacts build(){
			return new NutritionFacts(this);
		}
	}
	private NutritionFacts(Builder builder){
		servingSize		= builder.servingSize;
		servings 		= builder.servings;
		calories		= builder.calories;
		fat 			= builder.fat;
		sodium 			= builder.sodium;
	}
}
```

__Item 3 : Think of Enum to implement the Singleton pattern__

Example :
```java
public enum Elvis(){
	INSTANCE;
	...
	public void singASong(){...}
}
```

__Item 4 : Utility class should have a private constructor__
A utility class with only static methods will never be instantiated. Make sure it's the case with a private constructor to prevent the construction of a useless object.

Example :
```java
public class UtilityClass{
	// Suppress default constructor for noninstantiability
	private UtilityClass(){
		throw new AssertionError();
	}
	...
}
```

__Item 5 : Dependency Injection__
A common mistake is the use of a singleton or a static utility class for a class that depends on underlying ressources.
The use of dependency injection gives us more flexibility, testability and reusability

Example : 
```java
public class SpellChecker {
	private final Lexicon dictionary;
	public SpellChecker (Lexicon dictionary) {
		this.dictionary = Objects.requireNonNull(dictionary);
	}
	...
}
```

__Item 6 : Avoid creating unnecessary objects__
When possible use the static factory method instead of constructor (Example : Boolean)
Be vigilant on autoboxing. The use of the primitive and his boxed primitive type can be harmful. Most of the time use primitives

__Item 7 : Eliminate obsolete object references__
Memory leaks can happen in  :
 - A class that managed its own memory
 - Caching objects
 - The use of listeners and callback

In those three cases the programmer needs to think about nulling object references that are known to be obsolete

Example : 
In a stack implementation, the pop method could be implemented this way :

```java
public pop(){
	if (size == 0) {
		throw new EmptyStackException();
	}
	Object result = elements[--size];
	elements[size] = null; // Eliminate obsolete references.
	return result;
}
```

__Item 8 : Avoid finalizers and cleaners__
Finalizers and cleaners are not guaranteed to be executed. It depends on the garbage collector and it can be executed long after the object is not referenced anymore.
If you need to let go of resources, think about implementing the *AutoCloseable* interface.

__Item 9 : Try with resources__

When using try-finally blocks exception can occur in both the try and finally block. It result in non clear stacktrace.
Always use try with resources instead of try-finally. It's clearer and the exceptions that can occured will be clearer.

Example :
```java
static void copy(String src, String dst) throws IOException {
	try (InputStream in = new InputStream(src); 
		OutputStream out = new FileOutputStream(dst)) {
		byte[] buf = new byte[BUFFER_SIZE];
		int n;
		while ((n = in.read(buf)) >= 0) {
			out.write(buf, 0, n);
		}
	}
}
```

## Methods of the Object class

__Item 10 : equals__

The equals method needs to be overriden  when the class has a notion of logical equality.
This is generally the case for value classes

The equals method must be :
 - Reflexive (x = x)
 - Symmetric (x = y => y = x)
 - Transitive (x = y and y = z => x = z)
 - Consistent
 - For non null x, x.equals(null) should return false
 
Not respecting those rules will have impact on the use of List, Set or Map.

__Item 11 : hashCode__

The hashCode method need to be overriden if the equals method is overriden.

Here is the contract of the hashCode method :
 - hashCode needs to be consistent
 - if a.equals(b) is true then a.hashCode() == b.hashCode()
 - if a.equals(b) is false then a.hashCode() doesn't have to be different of b.hashCode()  
 
If you don't respect this contract, HashMap or HashSet will behave erratically.

__Item 12 : toString__

Override toString in every instantiable class unless a superclass already done it.
Most of the time it helps when debugging.
It needs to be a full representation of the object and every information contained in the toString representation should be accessible in some other way in order to avoid programmers to parse the String representation.

__Item 13 : clone__

When you implement Cloneable, you should also override clone with a public method whose return type is the class itself.
This method should start by calling super.clone and then also clone all the mutable objects of your class.

Also, when you need to provide a way to copy classes, you can think first of copy constructor or copy factory except for arrays.

__Item 14 : Implementing Comparable__

If you have a value class with an obvious natural ordering, you should implement Comparable.

Here is the contract of the compareTo method : 
 - signum(x.compareTo(y)) == -signum(y.compareTo(x))
 - x.compareTo(y) > 0 && y.compareTo(z) > 0 => x.compareTo(z) > 0
 - x.compareTo(y) == 0 => signum(x.compareTo(z)) == signum(y.compareTo(z))
 
It's also recommended that (x.compareTo(y) == 0) == x.equals(y).
If it's not, it has to be documented that the natural ordering of this class is inconsistent with equals.

When confronted to different types of Object, compareTo can throw ClassCastException.

## Classes and Interfaces

__Item 15 : Accessibility__

Make accessibility as low as possible. Work on a public API that you want to expose and try not to give access to implementation details.

__Item 16 : Accessor methods__

Public classes should never expose its fields. Doing this will prevent you to change its representation in the future.
Package private or private nested classes, can, on the contrary, expose their fields since it won't be part of the API.

__Item 17 : Immutability__

To create an immutable class : 
 - Don't provide methods that modify the visible object's state
 - Ensure that the class can't be extended
 - Make all fields final
 - Make all fields private
 - Don't give access to a reference of a mutable object that is a field of your class
 
As a rule of thumb, try to limit mutability.

__Item 18 : Favor composition over inheritance__

With inheritance, you don't know how your class will react with a new version of its superclass.
For example, you may have added a new method whose signature will be the same in the next release of its superclass.

Also, if there is a flaw in the API of the superclass you will suffer from it too.
With composition, you can define your own API for your class.

As a rule of thumb, to know if you need to choose inheritance over composition, you need to ask yourself if B is really a subtype of A.

Example :
```java
// Wrapper class - uses composition in place of inheritance
public class InstrumentedSet<E> extends ForwardingSet<E> {
	private int addCount = 0;
	public InstrumentedSet (Set<E> s){
		super(s)
	}

	@Override
	public boolean add(E e){
		addCount++;
		return super.add(e);
	}

	@Override
	public boolean addAll (Collection< ? extends E> c){
		addCount += c.size();
		return super.addAll(c);
	}

	public int getAddCount() {
		return addCount;
	}
}

// Reusable forwarding class
public class ForwardingSet<E> implements Set<E> {
	private final Set<E> s; // Composition
	public ForwardingSet(Set<E> s) { this.s = s ; }

	public void clear() {s.clear();}
	public boolean contains(Object o) { return s.contains(o);}
	public boolean isEmpty() {return s.isEmpty();}
	...
}
```

__Item 19 : Create of inheritance or forbid it__

First of all, you need to document all the uses of overridable methods.
Remember that you'll have to stick to what you documented for
The best way to test the design of your class is to try to write subclasses.
Never call overridable method in your constructor.

If a class is not designed and documented for inheritance it should be me made forbidden to inherit her either by making it final or making its constructors private (or package private) and use static factories.


__Item 20 : Interfaces are better than abstract classes__

Since Java 8, it's possible to implements default mechanism in an interface.
Java only permits single inheritance so you probably won't be able to extends your new abstract class to exising classes when you always will be permitted to implements a new interface.

When designing interfaces, you can also provide a Skeletal implementation. This type of implementation is an abstract class that implements the interface. 
It can help developers to implement your interfaces and since default methods are not permitted to override Object methods, you can do it in your Skeletal implementation.
Doing both allows developers to use the one that will fit their needs.


__Item 21 : Design interfaces for posterity__

With Java 8, it's now possible to add new methods in interfaces without breaking old implemetations thanks to default methods.
Nonetheless, it needs to be done carefully since it can still break old implementations that will fail at runtime.

__Item 22 : Interfaces are mean't to define types__

Interfaces must be used to define types, not to export constants.

Example :

```java
//Constant interface antipattern. Don't do it !
public interface PhysicalConstants {
	static final double AVOGADROS_NUMBER = 6.022_140_857e23;
	static final double BOLTZMAN_CONSTANT = 1.380_648_52e-23;
	...
}
//Instead use
public class PhysicalConstants {
	private PhysicalConstants() {} //prevents instatiation
	
	public static final double AVOGADROS_NUMBER = 6.022_140_857e23;
	public static final double BOLTZMAN_CONSTANT = 1.380_648_52e-23;
	...
}
```

__Item 23 : Tagged classes__

Those kinds of classes are clutted with boilerplate code (Enum, switch, useless fields depending on the enum).
Don't use them. Create a class hierarchy that will fit you needs better.


__Item 24 : Nested classes__

If a member class does not need access to its enclosing instance then declare it static.
If the class is non static, each intance will have a reference to its enclosing instance. That can result in the enclosing instance not being garbage collected and memory leaks.

__Item 25 : One single top level class by file__

Even thow it's possible to write multiple top level classes in a single file, don't !
Doing so can result in multiple definition for a single class at compile time.

## Generics

__Item 26 : Raw types__

A raw type is a generic type without its type parameter (Example : *List* is the raw type of *List\<E>*)
Raw types shouldn't be used. They exist for compatibility with older versions of java.
We want to discover mistakes as soon as possible (compile time) and using raw types will probably result in error during runtime.
We still need to use raw types in two cases : 
 - Usage of class litrals (List.class)
 - Usage of instanceof
 
Examples :

```java
//Use of raw type : don't !
private final Collection stamps = ...
stamps.add(new Coin(...)); //Erroneous insertion. Does not throw any error
Stamp s = (Stamp) stamps.get(i); // Throws ClassCastException when getting the Coin

//Common usage of instance of
if (o instanceof Set){
	Set<?> = (Set<?>) o;
}
```

__Item 27 : Unchecked warnings__

Working with generics can often create warnings about them. Not having those warnings assure you that your code is typesafe.
Try as hard as possible to eliminate them. Those warnings represents a potential ClassCastException at runtime.
When you prove your code is safe but you can't remove this warning use the annotation @SuppressWarnings("unchecked") as close as possible to the declaration.
Also comment on why it is safe.

__Item 28 : List and arrays__

Arrays are covariant and generics are invariant meaning that Object[] is a superclass of String[] when List<Object> is not for List<String>.
Arrays are reified when generics are erased. Meaning that array still have their typing right at runtime when generics don't. In order to assure retrocompatibility with previous version List<String> will be a List at runtime.
Typesafety is assure at compile time with generics. Since it's always better to have our coding errors the sooner (meaning at compile time), prefer the usage of generics over arrays

__Item 29 : Generic types__ 

Generic types are safer and easier to use because they won't require any cast from the user of this type.
When creating new types, always think about generics in order to limit casts.

__Item 30 : Generic methods__

Like types, methods are safer and easier to use it they are generics. 
If you don't use generics, your code will require users of your method to cast parameters and return values which will result in non typesafe code.

__Item 31 : Bounded wildcards__

Bounded wildcards are important in order to make our code as generic as possible. 
They allow us to permit more than a simple type but also all their sons (? extends E) or parents (? super E)

Examples :

If we have a stack implementation and what to add two methods pushAll and popAll, we should implement it this way :
```java
//We want to push in everything that is E or inherits E
public void pushAll(Iterable<? Extends E> src){
	for (E e : src) {
		push(e);
	}
}

//We want to pop out in any Collection that can welcome E
public void popAll(Collection<? super E> dst){
	while(!isEmpty()) {
		dst.add(pop());
	}
}
```