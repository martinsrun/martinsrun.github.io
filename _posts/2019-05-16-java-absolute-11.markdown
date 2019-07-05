---
layout: post
title:  "Java absolute - XI"
subtitle: "90 points to make your coffee"
date:   2019-06-24 00:00:00
categories: [programming,books,java]
---

## 11. Serialization

### 85. Prefer alternatives to Java Serialization
  - Serialization has multiple security vulnerabilities. The attack surface on serialization is too big to be protected as any serializable class can be open. Deserializing an untruested stream source can cause multiple  issues.
  - Object graphs are deserialized by invoking the readObject method on an ObjectInputStream, which is essentially a magic constructor. Attackers and security researchers study the serializable types in the Java libraries and in commonly used third-party libraries, looking for methods invoked during deserialization that perform potentially dangerous activities. Such methods are called gadgets, and multiple gadgets can be used in concert to form a gadget chain. There have been attacks with vulnerable gadget chains.
  - You can easily mount a DOS attack with a  short stream that takes long to deserialize. This class makes 2^100 hashCode method calls when deserializing, making it impossible to complete.
  ```JAVA
  // Deserialization bomb - deserializing this stream  takes forever
  static byte[] bomb() {
    Set<Object> root = new HashSet<>();
    Set<Object> s1 = root;
    Set<Object> s2 = new HashSet<>();
    for (int i = 0; i < 100; i++) {
    Set<Object> t1 = new HashSet<>();
    Set<Object> t2 = new HashSet<>();
    t1.add("foo"); // Make t1 unequal to t2
    s1.add(t1); s1.add(t2);
    s2.add(t1); s2.add(t2);
    s1 = t1;
    s2 = t2;
  }
  return serialize(root); // Method omitted for   brevity
  }
  ```
  - The best way to avoid these issues is to avoid serialization, to store files or transfer in such a form use JSON or protobuffers. Protobuffers being binary is far more performant, but Json is human redable.
  - Never deserialize untruested data,  read only from white listed sources.

### 86. Implement serializable with great caution
  - Implementing serializable decreases class flexibility once it has been released. You cannot change the classes internals once released without causing version mismatches. It is possible to support a different version using ObjectOutputStream.putFields and ObjectOutputStream.readFields but its not worth it.
  - Serialization introduces a whole suite of bugs and security errors. Since deserialization does not use a 'hidden constructor' you may end up missing out on the invariants that a constructor generally provides, especially if using the default serialization implementation.
  - Making a class serializable makes it diffucult to test on changes since all scenarios have to be tested during serialization-deserialization to make sure a faithful replication is made.
  - Classes designed for inheritance and interfaces should rarely be serializable. Doing so forces future implementers to support it as well.  Throwable class although is extendable is serializable, this is to allow sending exceptions across RMI client-servers.

### 87. Consider using a custom serialization form.
  - The default serialization is usually very wrong, capturing and calling private fields on the class. If your class is a simple model containing plain data fields, ie its object fields match its logical representation, this may be okay. But for a complex object such, a custom list, the default serialization is not the way to go. You would want a simple stream containing just the size, and the objects stored in the list.
  - Serialization has not object of the object graph and hence does a full traversal which is usually too expensive. Using default serialization for a complex object has multiple issues,
    - Permanently ties the API
    - Excessive space and time to work on
    - Can cause memory overflows
  -  Have your own writeObject and readObject implementations

### 88. Make readObject methods defensive
  - For classes with object reference fields that should remain private, defensive copy the deserialized reference, then validate before storing the reference.

### 89. For instance control prefer enum types to readResolve.  

### 90. Consider serialization proxies to instance serialization
  - A private static nested class that is serializable instead of the actual one.
