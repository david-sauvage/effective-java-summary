
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
