---
layout: post
title:  "Java absolute - I"
subtitle: "90 points to make your coffee"
date:   2019-05-16 00:00:00
categories: [programming,books,java]
---

This is a multi-part? series on the book Absolute Java 3rd edition by Joshua Bloc, enumerating in short the 90 points made in the book.

## 1. Creating and destroying objects

###  1. Static factory methods over constructors
 * Factory methods have names identifying their behavior.
 * They don't have to return a new instance every time unlike constructors. This allows caching instances (flyweight pattern).
 * They can return objects of any subtype unlike constructors.  The java.util.Collections class contain multiple static factory methods for various collection subtypes. Collections.unmodifiableCollection() returns an instance of Collection interface after wrapping provided collection into an unmodifiable, without exposing the unmodifiable implementation.
 * Returned class can vary based on input parameters. Based on parameters the returned object could be one or the other subclass of a common interface.\
 * Classes returned need not exist when static method is written. This is ideal for service provider frameworks when the returned class could be the interface returned by services, but services might be non existent at present. For example it could return a jdbc Connection calling DriverManager.getConnection() even though no driver has registerd calling the DriverManager.registerDriver().

### 2. Consider Builders when faced with many or optional parameters.
  * Builders can be used with class hierarchies with a parallel hierarchy of builders.
  * Builders can produce immutable objects as well.
  * On the flip-side creating  a parallel  object just to build an object is not exactly performance or memory efficient.

### 3. Declaring singletons
  * The bad about singletons is they cannot be tested since mock objects cannot be created unless it has an interface.
  * Singletons are usually implemented by either a public static field or a private static field and a public "getInstance()" method.
  * If your singleton needs to be serialized ensure it's instance fields are transient and it provides a "readResolve" method to prevent duplicate instances.
  * Can also use a single item Enum for singletons but it does not look really good.

### 4. Enforce non insatiability with private constructor

### 5. Prefer dependency injection to hard-wiring resources
  * This is necessary when behavior is parameterized by an underlying dependency reference.
  * Simple pattern in to pass dependencies with constructor

### 6. Avoid unnecessary object creation
  * String primitives "abc" over new String("abc")
  * Static factory methods to control obj creation over using constructors.
  * Caching repeated use single objects such as regex compiled patterns
  * Avoid boxed or autoboxed objects since they are immutable, every operation creates multiple objects.

### 7. Eliminate obsolete object references
  * Whenever a class manages its own memory it should take care of memory leaks.
  * Prefer WeakHashMaps to automatically GC objects when they loose reference.
  * Care for memory references when using listeners and callbacks, listeners and callbacks have to be de-registered.

### 8. Avoid finalizers and cleaners
  * They are unpredictable with no guarantee of execution time.
  * They have significant performance and security issues. Performance since they can delay normal GC operations.
  * If your class has resources it needs to free up, use AutoClosable. Finalize or cleaners can be a last ditch backup for non critical resources.

### 9. Use try-with-resources over try-finally
  * Just concise and better in every way. It can shorten nested try-finally blocks into a single block without suppressing inner exception.
