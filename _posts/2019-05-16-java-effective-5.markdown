---
layout: post
title:  "Java Effective - V"
subtitle: "90 points to make your coffee"
date:   2019-05-30 00:00:00
categories: [programming,books,java]
---

## 5. Enums and Annotations
 > JAVA supports two special-purpose families of reference types: a kind of class called an enum type, and a kind of interface called an annotation type.

### 34. Use enums over int Constants
  - Before enums where added the usual way to represent such a pattern was to declare a group of int constants, one for each type, technique called int enum pattern. This makes your enum basically integers which means you could end up with integer operations on enums. Also int enums don't make sense when printing out or comparing.
  - String enum pattern where you have string constants moonlighting as enumerations do not work either. They put dependency on hardcoded string constants which could break based on typo errors, and rely on low performance string equals.
  - Enums are simple types that export one implementation of an enumeration constant via a public static final field. They are generalized singletons. Enums provide type safety, in a parameter which expects an Apple enum, it ensures only Apple types are accepted.
  ```Java
  public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
  ```
  - Enums let you extend functionality by adding fields and methods to your enumeration type. Just declare instance fields and write constructors that utilize those instance fields, add a few methods along and you are good to go..
  - You can add constant specific method implementations to enum types  if needed, method implementations specific to a particular type of Apple.
  - If you override toString() of an enum to change the default String representation which is the type name of the enum, consider overriding the fromString() as well which provides the enum type given a String representation.
  - It is not necessary for the set of constants in an enum be permanent, the types of Apples could change as botanists create new succulent colorful ones or if a particular type perishes to a sudden outburst of tech changes. Your enum list could change to reflect those and your clients will get to know it with anyone referencing the now expired Apple will get a healthy compile time error explaining why investing in defunct stocks is a bad idea.

### 35. Use instance fields instead of Ordinals.
  - Enums have an ordinal method which represents a type's ordering in an enumeration. Don't use it.. If you have the burning need to represent some property in an enum type, use an instance field.

### 36. Use enumsets instead of bit-fields.
  - To collect and operate on int enum patterns as a set int enums were declared in multiples of 2, which meant they could be combined and represented as a bitfield. You could easily do bitwise arthemetic on them to find combinations and intersections of those enum bitwise sets.
  - But finding what a combined bitwise set cnotained was difficult since it meant going back and finding what constant each integer in those sets meant. Also the size of a bitwise set had to be known in advance to determine if an int or a long was needed to represent them.
  - The optimal solution is to use a enumset instead which internally uses a similar performant bitwise representation. It lets you iterate through your set as well as do whatever operation you could do with a bitwise  field.

### 37. Use EnumMaps when needed to partition enums
  - You have a bunch of objects identified by some enum, say a group of accounts each with an enum property identifying their role, and you wish to partition accounts by role.
  - The old school way off doing this was to partition into a map with the role enum's ordinal as key and an array of corresponding accounts as values.
  - The current generation of enums support EnumMaps which basically implement the same thing, but provide type safety. It is better performant than a standard map since EnumMaps know in advance the number of keys it has.

### 38. Emulate enum extensiblity with Interfaces
  - While enumerations cannot be extended and for most cases don't need extension, if required can have a parent interface which can be used as a parent type.
  - This is useful if you have an enum type which you would like to allow clients to extend and add to.
  - If you have an emum of allowed operations, ADD, MULTIPLY, SUBTRACT, DIVIDE, and want to allow your clients to add to possible operations, create a parent interface called Operations, extend it in an enum to support your basic operations.. Now your clients can extend the same interface in their enum to add to possible operations as long as your classes are dependent on the interface and not the enum type.

### 39. Use annotations to naming patterns
  - Naming patterns were used before annotations became a thing to indicate that something required special treatment, like test methods began with 'test'.
  - Instead a standard test annotation works, you could define your own test annotation as:
  ```Java
    import java.lang.annotation.*;
    /**
    * Indicates that the annotated method is a test method.
    * Use only on parameterless static methods.
    */
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    public @interface Test {
    }
    ```
    @Retention and @Target are meta-annotations which indicate special properties of this annotation. @Retention(RetentionPolicy.RUNTIME) indicates that this annotation is available at runtime and @Target(ElementType.METHOD) indicates this annotation can be applied to methods.
    - Use predefined annotations that libs provide

### 40. Consistently use the @Overrides annotations
  - Use overrides annotation on every method that is meant to override a superclass declaration. It gives you nice compile time errors if you did mess up the override.

## 41. Use marker interfaces to define types
    - Marker interfaces have no methods but merely marks the implementing class as having some property. Consider serialiazble, it means an implementing class can be serialized.
    - Marker annotations can do the same but using iterfaces as markers have 2 benifits:
      - Marker interface provide checking at compile time, marker annotations work only at runtime.
      - Marker interfaces can be targeted in an interface hierarchy by being defined as a subtype of another interface. Marker annotations will work on any class or method if it is defined as ElementType.TYPE.
