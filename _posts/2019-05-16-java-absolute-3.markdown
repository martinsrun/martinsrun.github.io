---
layout: post
title:  "Java absolute - III"
subtitle: "90 points to make your coffee"
date:   2019-05-20 00:00:00
categories: [programming,books,java]
---

## 3. Classes and Interfaces
  >Classes and interfaces lie at the heart of the Java programming language. They are its basic units of abstraction. The language provides many powerful elements that you can use to design classes and interfaces. This chapter contains guidelines to help you make the best use of these elements so that your classes and interfaces are usable, robust, and flexible.

### 15. Minimize access to class members
    - A well designed component hides all implementation details from it's API, ie abstraction. This allows components to be used,developed,tested in isolation without impacting the API.
    - A solid rule of thumb for access modifiers is to make each class or member as inaccessible as possible, lowest  level of access that allows for its functionality.
    - Classes/Interfaces are public or default(package-private). Keep classes package-private whenever possible, or if a class is used only from one another class, keep it as a static inner class. Public classes are a part of the API while package-private classes are a part of the implementation.
    - For members (fields,methods,nested classes,nested interfaces) it is private, package-private, protected or public.
    - After designing the class API, all others members should be private by default. Private and package-private are a part of implementation and not in design. Public and protected are a part of the API and needs to be supported.
    - Instance fields should almost never be public. Constants are the exception, but even constants shouldn't be a mutable type.
    - Java9 module system brings in an additional layer of access control with public/protected classes being avilable only within module unless exposed from within module-info.java. But this can be bypassed by adding jars directly to a class-path instead of the module-path. Module access protection is purely advisory.

### 16. In public classes use accessors into fields
  - If classes are accessible outside your package, members shouldn't be directly accessible.
  - Can be avoided for package-private or private nested classes.

### 17. Minimize mutability
  - Make all simple value classes Immutable. Basic Rules:
    - Don't provide methods that change object's state
    - Ensure class cannot be extended (final class or private constructors)
    - Make all fields final. This in addition protects against synchronization requirements when a new object is shared between threads.
    - Make all fields private
    - Ensure access controls to mutable objects stored within. Make defensive copies when sharing mutable items.
  - Immutable objects are simple, they exist in only one state, the state it was created in. Whereas mutable objects can exist in one of any number of complex states.
  - Immutability provides thread safety without explicit synchronization. They can be shared freely across threads without fear of being corrupted and without making defensive copies of them.
  - The disadvantage is you need a new object whenever a state changes, this can be reduced by caching and using factory methods for construction.
  - You can have a mutable companion class like StringBuilder in cases when mutability is required for a time.
  - By making constructors package-private you can allow child implementation classes within your package while keeping the parent class immutable.
  - When not possible to be immutable, limit mutability as much as possible. Eg, countdownlatch which is instantiated with all of it's invariants set and can only count down to 0.

### 18. Composition over inheritance
  - Safe to use inheritance within packages since it ensures subclasses are controlled by the same people who created the superclass.
  - Inheritance across packages violate encapsulation, a subclass depends on super-class implementation for functionality. If superclass changes, the subclass may have to change in tandem to support those changes.
  - Even just adding a new method in the subclass could be a risk, the superclass could create a method with the same name in the next release. This would make your method either override that one, or better, break if the superclass method has limited visibility.
  - Composition with forwarding is a much safer approach, say you want to create a custom map, create a class implementing the map interface, wrap the map class you want to override (HashMap?), and forward all map methods to the wrapped HashMap object.
  - You can make your internal wrapped map object a constructor parameter, making your custom map more flexible supporting any map class rather than just HashMap.
  - Wrapping and forwarding like this is the Decorator pattern, decorating a base class with functionality.
  - The bad of wrapping is it cannot be used with callback methods since the wrapped class is not aware of the wrapper and it will expose itself and pass into callback methods. - Self Problem
  - If A is inheriting B make sure A is-a B in all scenerios.

### 19. Design and document for inheritance or prohibit it
  - Class must document self-use of overridable methods. It should explain what the overridable method invokes, in what order and in what sequence. Any non-final public/protected methods are overridable.
  - @implSpec tag is to describe the implementation specification, it should explain how the parent method works not just what it does. This
  - Allowing inheritance also forces you to provide protected hooks for child classes to override, allowing child classes to modify parent functionality. Eg, AbstractList allows for a removeRange method that is meant to be overridden by implementer, removeRange is used in the AbstractList clear method hence it's performance depends on child implementation of removeRange.
  - Test inheritence design by implementing child classes to see if exposed methods are actually a necessity.
  - Constructors should never reference overridable methods, either directly or indirectly. Superclass constructor completes before child constructor, if overriden method depends on initialization done by child this will cause failure.
  - Similar rules apply if an overridable class is clonable or serializable, with clone or readObject method not invoking overridable methods. Generally its better not to make an overridable class clonable or serialiazable.
  - If your class has to allow inheritance make sure overridable methods are not invoked from it's methods to prevent child classes from affecting its behavior.

### 20. Prefer interfaces to abstract classes
  - "Existing classes can easily be retrofitted into an interface".
  - Interfaces provide "mixins", mixins are additional functionality provided beyond the core class behevior. Eg Comparable is a mixin that defines a natural ordering for a class.
  - Non-hierarchical type frameworks, multiple parents.
  - Interfaces provides so powerful enhancements/patterns
    - Supports the wrapper class idiom from 18.
    - For obvious implementations provide a default implementation with @implSpec documentation. Object methods, hashCode,equals,toString cannot have default impl.
    - Template method pattern is when you provide a companion skeletal template class to your interface, conventionally caller Abstract[InterfaceName]. Collections do this all the time with AbstractList/Set etc. To write a skeletal implementation, analyze the interface to see which methods are the primitives on top of which other methods are implemented. These primitives become the abstract methods, while the others become concrete methods in the skeletal class. Skeletal implementation needs good documenting.
    - A simple implementation is an extension to skeletal where we have a simple class that can behave as the default implementation for the interface.

### 21. Design interfaces for posterity
  - Although java 8 added default implementations which makes it possible to add additional functionality to an interface after it has been created, this is not always a good idea.
  -  Default methods are injected into the implementations without the implementer being aware, hence if it needs to be overriden to work properly in a particular implementation, the implementer has to add it.
  - SynchronizedCollections from apache wraps collections to ensure all updates happen in locked synchronized blocks. But default methods added in Java 8 do not know of this locking and the removeIf method in Collections interface which removes items from that match a given predicate removes ignoring the synchronized collection's locking invariant.
  - Default methods could cause your implementations to fail at runtime. It is possible to correct interface flaws once released but it could have hidden costs.

### 22. Use interfaces only to define types
  -  Avoid the constant interface pattern, aka filling the interface with just constants.

### 23. Prefer class hierarchies to tagged classes   
  - Using a "tagging" member to denote the flavor of a class is usually not a good idea.

### 24. Prefer static member classes to non-Static
  - Inner classes are non static member classes, local classes or anonymous classes.
  -  A static member class is the simplest version, has access to all enclosing members, and if declared private it is accessible from just the enclosing class.
  - Non-static member class needs an enclosing class instance to exist, and every inner class instance has a reference to this enclosing object. Good examples for non static members are iterators for collections or wrapped collections such as keySet for a map.
  - If your member class does not need a wrapped instance to depend on then it should be static.

### 25. Limit source files to a single top-level class
  - Never put multiple top level classes or interfaces in a single source file.  
