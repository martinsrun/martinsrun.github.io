---
layout: post
title:  "Java Effective - VIII"
subtitle: "90 points to make your coffee"
date:   2019-06-13 00:00:00
categories: [programming,books,java]
---

## 8. General

### 57. Minimize scope of local variables
  - Define local variables when you need them, and try to have an initialized for all local variables declared.
  - Prefer for loops over while loops since for loops provide a declaration for the loop variable and defines it's scope.

### 58. Use for-each over for loops
  - For each has no performance penalty over for loops but provide an awesome and clean representation of the loop.
  - There are some situations where for each loop cannot be used,
    - Destructive filtering - when removing items from a group.
    - Transforming - Transform all items in  a group
    - Parallel iteration - Iterate through multiple groups in a single loop

### 59. Understand and use libraries
  - Flaws of using a standard lib without understanding it can be critical. Consider this code to generate random numbers below a max n. These do not produce even random numbers, giving repeated patterns if n is a small power of 2. If n is large this a large simulation of random numbers generated will end up with about 2/3rd numbers less than half of n.
  ```JAVA
  // Common but deeply flawed!
  static Random rnd = new Random();
  static int random(int n) {
    return Math.abs(rnd.nextInt()) % n;
  }
  ```
  - Instead of this implementation use Random.nextInt(int), which does the job without the issues. Use libraries written by experts who know what they are doing. Post java 7, Use ThreadLocalRandom for random number generation, for streams and fork join pools use SplittableRandom.
  - Most libraries are rewritten over years fixing issues and improving performance to the point of being better than anything you can manually implement in a short period.
  -  Understand java.util, and java.io packages, with java.util.collections and java.util.concurrent being extreamly important.

### 60. Avoid float or double when you need exact values
  - Do not use float or double for calculations need precision, especially monetary calculations.
  - Instead use BigDecimal with string constructors. BigDecimal has a performance penalty, but that's the trade-off for accuracy.

### 61. Primitive types to boxed primitives  
  - Boxing and unboxing has a cost attached even if they are being done in the background (auto-boxing, auto-unboxing).
  - When mixing primitives and boxed types in operations it gets auto-unboxed, extracting values.
  - Comparing primitives and boxed types with == fails since boxed types are references.
  - Prefer primitives unless boxed types cannot be avoided, such as for collections.

### 62. Avoid strings when other types are more appropriate
  - Strings are poor substitutes for value types, value types have their API which strings dont provide.

### 63. Be aware of string concat performance costs
  - (+) string concat oparator is a quick and easy way to combine strings, but doing so over n strings have a quadratic cost over n.
  - Since strings are immutable when 2 strings are concated both are copied. Hence use concat only if you havve very few string to copy, otherwise use a string builder.

### 64. Refer to objects by their Interfaces
  - If an appropriate interface type exists, then parameters,returns, variables, fields all should declare to the interface type. Implementation should be used only for constructing the run-time object. This makes your classes super flexible.
  - If there is no appropriate interface stick with the least specific class in the hierarchy that provides the required functionality.

### 65. Prefer interfaces to reflection
  - Reflection allows one class to refer to another even if it did not exist when the other was compiled. Doing so pays the price in compile time checks, code verbosity, and performance.
  - You can refer to an interface at runtime and the create the class at runtime reflectively.
   ```JAVA
   public static void main(String[] args) {
     // Translate the class name into a Class object
     Class<? extends Set<String>> cl = null;
     try {
       cl = (Class<? extends Set<String>>)
       //
       Unchecked cast!
       Class.forName(args[0]);
     } catch (ClassNotFoundException e) {
       fatalError("Class not found.");
       }
       // Get the constructor
       Constructor<? extends Set<String>> cons = null;
     try {
       cons = cl.getDeclaredConstructor();
     } catch (NoSuchMethodException e) {
       fatalError("No parameterless constructor");
     }
     // Instantiate the set
     Set<String> s = null;
     try {
       s = cons.newInstance();
     } catch (IllegalAccessException e) {
       fatalError("Constructor not accessible");
     } catch (InstantiationException e) {
       fatalError("Class not instantiable.");
     } catch (InvocationTargetException e) {
       fatalError("Constructor threw " + e.getCause());
     } catch (ClassCastException e) {
       fatalError("Class doesn't implement Set");
     }
     // Exercise the set
     s.addAll(Arrays.asList(args).subList(1,
     args.length));
     System.out.println(s);
     }
   private static void fatalError(String msg) {
     System.err.println(msg);
     System.exit(1);
  }
   ```
   - As you can see, requires handling quiet a lot of runtime exceptions since compile time checks are bypassed.
   - This technique is the base for the service provider pattern/framework.

### 66. Use native methods judiciously
  - Java native interface(JNI) allows calling methods in native languages (C/C++). But modern versions of Java has very little legitimate reasons to use them.
  - Using native methods break java's memory corruption immunity, as well as garbage collection guarantees. They are also platform dependent.

### 67. Optimize judiciously
  >> We should forget about small efficiencies, say about 97% of the time: premature optimization is the root of all evil.  
  - Don Knuth

  - Strive to write good programs rather than performant ones, performance usually follows good code.
  - Strive to avoid design decisions that limit performance. API and design decisions are the hardest to change and  care has to be taken not to constraint performance.
  - Measure performance before and after every attempt to optimize.

### 68. Adhere to generally accepted naming conventions
  -
