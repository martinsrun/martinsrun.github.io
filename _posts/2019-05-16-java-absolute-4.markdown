---
layout: post
title:  "Java absolute - IV"
subtitle: "90 points to make your coffee"
date:   2019-05-24 00:00:00
categories: [programming,books,java]
---

## 4. Generics
 > Before generics, you had to cast every object you read from a collection. If someone accidentally inserted an object of the wrong type, casts could fail at runtime. With generics, you tell the compiler what types of objects are permitted in each collection. The compiler inserts casts for you automatically and tells you at compile time if you try to insert an object of the wrong type.

### 26. Don't use raw types
  - Raw types or generic types without type specified (new List()) is left for backward compatibility, don't use it. It has no type safety at compile time.
  - Use List<Object> over raw List. List<Object> is safer than raw List as a raw Lists are not type safe and can  accept List<String> or List<Integer>, but a List<Object> accepts only List<Objects>. The compiler knows it contains objects.
  - For unknown type parameters use unbounded wildcards over raw types. List<?> over List. Unbounded wildcard type, aka list of some type, constraints the compiler such that the list defined as such will not allow anything to be added to it other than null.
  - Instance of comparisons does not work on parameterized types except for unbounded wildcards.
    ```Java
      // Legitimate use of raw type - instanceof operator
      if (o instanceof Set) {
      // Raw type
      Set<?> s = (Set<?>) o;
      // Wildcard type
      ...
      }
    ```

### 27. Eliminate unchecked warnings
  - Making sure there are not unchecked warnings ensures your code is as type-safe as it can be with no surprise ClassCastExceptions.
  - If there is absolutely no way to remove a type warning, but you can prove the code is type safe, use @SupressWarnings("unchecked") to suppress it. But ensure the annotation is used on the smallest possible scope, if it works, on a single variable.

### 28. Use Lists over Arrays
  - Generic types are invariant and erased, meaning List<String> cannot be passed to a List<Object> and that type information is removed at runtime.
  - Arrays are covariant and reified, meaning String[] can be passed to a Object[] and that type information is maintained and enforced at runtime. Arrays allowing supertypes to reference subtypes have implications on type checking, an Object[] variable can store a String[] which can then have an integer set into it since its an Object[] variable. This throws an ArrayStoreException at runtime.
  - Due to the loss of type information in generics, arrays and generic types dont intermingle well with Arrays not supporting generic objects or types.

### 29. Favor generic types  
  - If your current objects are generic, generify them. It doesn't break your existing users.
  - Making your types generic means your users do not have to do an additional cast on the object you return, hence no risk of a classcastexception.
  - If you need to store your generic type in an array inside your class, cast an Object array into your generic type array, but ensure the array stays within your class.
  - Use bounded type parameters if you need to restrict the type of variables possible.
     class DelayQueue<E extends Delayed> implements BlockingQueue<E> : Limits the possible values of E to Delayed and its subclasses.

### 30. Favor generic types
  - Type parameter list is specified between method modifiers and return type eg, public static <E> Set<E> union()
  - Due to erasure we can have Generic methods returning the same immutable object since at runtime the type information is lost - generic singleton factory. For example if you want to return a function that works generically, you can return the same function object with a generic param.
  - You can have a type parameter bounded by an expression with the same type param, Comparable being a classic example. Since comparable always compares to objects of same type, you can define methods that take comparable objects with recursive bounds : public static <E extends Comparable<E>> E max(Collection<E> c);

### 31. Use bounded wildcards to increase API flexibility
  - List<Integer> cannot be passed into a List<Number> although integers are just numbers. Any method that takes a list  of Number hence cannot support a list of numbers unless bounded by a wildcard <? extends E>
  - Consider a pushAll method on a custom generic stack defined as pushAll(Iterable<E> src). This supports only iterables of the same type as the stack datatype even though the stack can contain subtypes of E.
    - pushAll(Iterable<? extends E> src) permits pushAll to take iterables with subtypes of E
    - within the pushAll method src is a producer, ie it provides values to be pushed
  - Consider a popAll method that takes a collection and pops all values of our set into it. The default implementation again supports popping into collections with same type as our set, popAll(Collection<E> dest)
    - popAll(Collection<? super E> dest) permits popAll to pop into a collection that holds supertypes of E.
    - within the context of the popAll method dest is a consumer, ie takes values being popped.
  - When passing producers use <? extends E>, for consumers <? super E> : PECS producers-extend consumers-super
  - When implemented properly wildcards are invisible to the consumer and yet provide flexibility in your API. Never use a wildcard in the returntype as it will force your consumers to use wildcards in their code as well.
  - With bounder wildcards Comparable type constraint can now be updated
    - public static <T extends Comparable<? super T>> T max (List<? extends T> list);
    - Comparable is a consumer within the context of T, it can be anything that can compare to itself or it's supers
    - This works in situations when you have a Comparable implemented in one of your parent classes and you want all of it's subclasses to use the same natural ordering.
  - If a generic type param occurs only once in a method declaration replace it with a wildcard.

### 32. Combine varargs and Generics judiciously
  - Although its not possible to create generic type arrays, its possible to create generic type varargs, which are the same thing. This was done for convenience since many Java API methods use it (Ex: Arrays.asList(T... a)).
  - Heap pollution occurs when a variable of parameterized type refers to an object of a different type, and is a classcastexception waiting to happen as java will auto cast it when referenced.
  - The @SafeVarargs annotation is used to say a method although using generic varargs is typesafe and uses it without causing heap pollution.
  - If using generic varargs, ensure you do not mix objects types being put into the array, and never return the generic array reference.

### 33. Consider typesafe heterogeneous containers
  - General collections support a limited number of types in an instance. A List can be a list of any type, but to hold multiple types it has to be defined as a List of Object. Same for other collections and containers.
  - The Class object in java is generic, String.class is the type of Class<String>. When a class literal is passed along with a method to communicate with compile and runtime checks it's a type token.
  - Consider map like collection that can store one instance of a different class type. The key would be of type Class<T>, and to put a String into it we would use Class<String> as the key.
  - Similarly for database rows, a single row can have multiple values of different types which mean multiple classes in the same row.
