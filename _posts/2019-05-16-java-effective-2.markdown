---
layout: post
title:  "Java Effective - II"
subtitle: "90 points to make your coffee"
date:   2019-05-17 00:00:00
categories: [programming,books,java]
---

This is a multi-part? series on the book Absolute Java 3rd edition by Joshua Bloc, enumerating in short the 90 points made in the book.

## 2. Methods common to all objects
  > All of its nonfinal methods (equals, hashCode, toString, clone, and finalize) have explicit general contracts because they are designed to be overridden. It is the responsibility of any class overriding these methods to obey their general contracts; failure to do so will prevent other classes that depend on the contracts (such as HashMap and HashSet) from functioning properly in conjunction with the class.

###  10. General contract for overriding equals
  - This deserves a page on its own.
  - Don't override equals unless there is a need to have equivalence rules, generally objects are ok with being equal with just themselves.
  - Equals is overridden usually for value/model classes to have equivalence when properties match.
  - Rules for equals overriding is to ensure objects behave as expected when interacting with others, like when retrieving from data structures.
  - Equals should follow general logical rules: reflexive,symmetric,transitive,consistent,nullable
    - Reflexive : Object must be equal to itself, hard to break unless intentionally.
    - Symmetric : a.equals(b) => b.equals(a) . This can be easily broken if a new one type tries to provide equivalence to an existing type. eg, new type CaseInsensitiveString implementing equals with String.
    - Transitive : This guy is tricky, but finally there is no way in Java that we can extend an instantiable value  class, add an additional property and uphold all equivalence needs. Start with a class point with 2 properties, x and y for coordinates, implementing equals that compare co-ordinates. Extend it into ColoredPoint adding a "color" property. A naive way to implement equals would be to compare the new color property if other object is a ColoredPoint, otherwise use super Class's equals.
      ```java
      // equals method in ColoredPoint class
      public boolean equals(Object o){
        if(O instance of ColoredPoint && !((ColoredPoint)O).getColor().equals(this.getColor()){
          return false;
        }
        if(O instance of Point){
          return super.equals(O);
        }
        return false;
      }
    ```
    - This looks fine at the first glance, but assume we have 3 points,
      a =  (1,2,Red), b=(1,2), c=(1,2,Green)
      a.equals(b) is true, b.equals(c) is true, but c.equals(a) is false, breaking transitivity.
      At this point you would ask, why should ColorPoint call super.Point(O), cant it just compare all properties it needs to compare including the super class properties the coordinates.. Well you cant, because it breaks Liskov substitution principal(read on ). If you create a set of Points, add b into it and then checks it using a, (which theoretically should work, them being the same point and all, a just has an additional property) it says a is not in the set.
    - Consistency : If 2 objects are equal, they are equal from now until the end of time. Hence do not write your equals to rely on unreliable properties. If you add an item to a set the contains with an equal object should return true as long as the item is in the set.
    - Non-nullability : something.equals(null) is always false, never an exception
  -  Whenever possible, dont write an equals method on your own, use something like AutoValue to generate it.

### 11. Override hashcode when overriding equals
  - When 2 object are equal as per their equals method, their hashcodes should be the same.
  - 2 non equal objects can have same hashcode, but this leads to bad performance on datastructures using it.
  - Same as equals in general try to avoid implementing your own hashcode method, instead use a library like AutoValue to generate it.
  - If implementing hashcode make sure all significant fields that contributed to the equals are a part of the hashcode as well.
  - Don't specify a concrete definition for what your hashcode will return (like the String class did), or your clients will end up with hard dependencies on your implementation. If your hashcode specification is generic, it leaves you free to change the implementation without cause for complaint.

### 12. Always override toString
  - Always is a string word :)
  - It isn't critical but "providing a good toString implementation makes your class much more pleasant to use and makes systems using the class easier to debug"
  - Never use toString in your business logic.

### 13. Override clone judiciously (read never)
  - Cloneable is a weird interface without any methods.  A class implementing it tells Object.clone() that this is a clonable subclass. Without it calling .clone() would cause Object.clone() to throw CloneNotSupportedException.
  - Every cloneable class is supposed to provide its own clone() method implementation, which is a "complex unenforceable thinly documented protocol". It creates objects without going through a constructor.
  - Ideally the clone is equals() to the original, but this is not a requirement. The exact nature of what a "clone" is is left to the implementer, but the general idea is that x.clone() != x and x.clone().getClass() == x.getClass(). Again these are not absolute requirements.
  - The default object clone dose a shallow copy of all properties your class has, which mean if your class has references to mutable objects this can be trouble. Your custom clone should if needed do a deep copy of those fields.
  - There is a lot of un-necessary complexity in implementing correct clones which can simply be avoided by creating either copy constructors or static factory copy methods. For example most collection implementations have copy constructs that create an instance given any collection object.

### 14. Consider implementing comparable
  - Although comparable is a different interface not provided by the default java object, consider implementing it in all value objects that have a natural ordering.
  - compareTo(Object o) > 0 if o is greater, < 0 if o is less, == 0 if o is equal.
  - In general if y.compareTo(z) == 0, y.equals(z) is true. Any violation of this has to be called out “Note: This class has a natural ordering that is inconsistent with equals”. Breaking this may be okay but it can impact how these classes behave in different contexts, for example a TreeSet will not allow 2 objects for whom compareTo == 0 while a HashSet will, but a HashSet does not allow 2 objects that are the same as per equals() but a TreeSet will.
  - The general contract for compareTo() method is similar to equals being reflexive, symmetric and transitive, but it  can throw a classcastexception if it doesnt want to support comparing to a type.
  - Being similar to equals, the same caveat applies: "there is no way to extend an instantiable class with a new value component while preserving the compareTo contract, unless you are willing to forgo the benefits of object-oriented abstraction."
  - From java 8 Comparator provides construction methods to easily define comparators with lambdas, this can be used to in compareTo as well.
    ~~~Java
    // Comparable with comparator construction methods
      private static final Comparator<PhoneNumber> COMPARATOR =
        Comparator.comparingInt((PhoneNumber pn) -> pn.areaCode)
        .thenComparingInt(pn -> pn.prefix)
        .thenComparingInt(pn -> pn.lineNum);

      public int compareTo(PhoneNumber pn) {
        return COMPARATOR.compare(this, pn);
      }
    ~~~
