---
layout: post
title:  "Java Effective - VI"
subtitle: "90 points to make your coffee"
date:   2019-06-05 00:00:00
categories: [programming,books,java]
---

## 6. Lambdas and Streams

### 42. Lambdas over anonymous Classes
  - Anonymous classes worked for classic object oriented style, eg, the Comparator interface as an abstract Strategy and an on they fly anonymous Comparator as a concrete Strategy fits the Strategy pattern.
  - Java 8 started special treatment for single method interfaces, calling them functional interfaces, removing much of the boilerplate attached with anonymous classes. A comparator can be now defined on the fly as
  ```JAVA
    Collections.sort(words,(s1, s2) -> Integer.compare(s1.length(),s2.length()));
    //Even shorter using Comparator.comparingInt
    words.sort(comparingInt(String::length));
  ```
  - Type inference helps avoid specifying types in most situations.
  - But since lamdas lack names or documentation, if they exceed a few lines consider something else.

### 43. Prefer method references to Lambdas
  - Lambdas are succinct, method references more so.
  ```JAVA
    \\ Inserts a key into the map with value 1, if key already exists increment it
    map.merge(key, 1, (count, incr) -> count + incr);
    \\ can be expressed as
    map.merge(key, 1, Integer::sum);
  ```
  - Static method references are easy, refer by Class::method

### 44. Favor the use of Java functional Interfaces
  - With the introduction of lamdas, java patterns for class APIs have changed considerably. The template method pattern for example can use functions to override behaviors as needed.
  - If a standard functional interface works for your methods use it over a custom one.
  - java.util.Function has 43 functional interfaces defined, just need to know a few basic ones
    - Operator : Result and arguments are the same type, eg, UnaryOperator<T> String::toLowerCase, BinaryOperator<T> BigInteger::add
    - Predicate<T> : Takes in a type and responds with a boolean - Collection::isEmpty
    - Function<T,R> : Takes type T and returns type R - Arrays::asList
    - Supplier<T> : Takes no param, returns object of type T - Instant::now
    - Consumer<T> : Takes an object of type T as param, doesn't return anything - System.out::println
  - Other interfaces are 2 argument versions of these as well as primitive implementations for them such as ToIntBiFunction<T,U> that takes 2 parameters and returns an integer.
  - Write your own functional interface even if it matches one of the java functional interface, if it has some string use case and string contracts associated. Use @FunctionalInterface to annotate it.

### 45. Use streams judiciously
  - Java 8 introduced streams, which represent an infinite data sequence and stream pipelines which represent stages in those data sequences. Streams can contain custom objects or 3 primitives, int,double or long. Stream pipelines are lazy evaluated, and needs a terminal operation to be executed.
  - Don't overdo streams, it can be hard to understand. Use helper methods to be called from streams to make it more readable.
  - Things you cannot do with streams but with iteration:
    - Read/modify local variables, lambdas support only final or effectively final variables.
    - Returning, breaking, continuing from code blocks, throwing checked exceptions and all can be done from iterative methods to stop it's execution, but not for stream pipelines.

### 46. Prefer side effect free functions in streams
  - Lambda functions should be pure as possible, with each stream pipeline stage depending only on the output from the previous stage.
  - The forEach operation should be avoided, and used only to report the result or print values from a stream.
  - Collectors interface has 39 methods for to terminate steams without side effects.
  ```JAVA
  // From a stream of albums collect a map of artist to top selling album
  // This version of toMap takes in a keymapper a valueMapper, and a BinaryOperator which if 2 keys have the same value
  // combines them
  Map<Artist, Album> topHits = albums.collect(Collectors.toMap(Album::artist,a->a, BinaryOperator.maxBy(comparing(Album::sales))));
  ```
  - Collectors provide a 'groupingBy' method which supports grouping streams into Map<K,List<V>> with similar keys. groupingBy can take a downstream parameter which them the lists to by some function.

### 47. Prefer collections to streams as return types
  - Returning iterables as streams force consumers too follow one particular paradigm. Easier to return a collection when possible.

### 48. Use caution when making streams parallel
  - Streams when parallelizing use spliterators(similar to iterators but for parallellism) returned by Stream and Collections to split records across threads. Performance on parallel threads are better on Collection streams since they have spliterator implementations supporting, and since they provide good locality of reference when processing objects sequentially.  Locality of reference means sequential objects are close to one another in memory which is important for bulk operations, saving time for threads to copy data from memory to cpu cache.
  - The nature of stream pipeline terminal operations impact performance, if there is a significant time to combine results from stream threads, it inherently becomes sequential. Reductions are the best for paralell streams, where all threads are combined to a single result, also prepackaged reductions such as Min,Max,count,sum. Short circuit operations such as anyMatch or allMatch are good as well. Stream collect operations are generally compute intensive and are bad for paralell stream pipelines.
  - Under the right circumstances it is possible to achieve near linear speedups by paralellizing streams.
  -
