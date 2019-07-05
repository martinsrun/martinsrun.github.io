---
layout: post
title:  "Java absolute - IX"
subtitle: "90 points to make your coffee"
date:   2019-06-18 00:00:00
categories: [programming,books,java]
---

## 9. Exceptions

### 69. Use exceptions only in exceptional situations
  - Never assume API implementation behavior
  ```JAVA
      // Horrible abuse of exceptions. Don't ever do this!
    try {
      int i = 0;
      while(true)
      range[i++].climb();
    }
    catch (ArrayIndexOutOfBoundsException e) {
    }
  ```
  - An API should not force its clients to use exception handling for ordinary control flows. A state dependent method such as iterator.next() should have a corresponding state checking method iterator.hasNext() to ensure it gets invoked in the correct circumstances.

### 70. Use checked exceptions for recoverable conditions and runtime exceptions for programming errors
  - By confronting the user with a checked exception, the API forces it to be handled and recovered.
  - Runtime exceptions are for programming errors such as failed preconditions.

### 71. Avoid unnecessary checked exceptions  
  - When used properly checked exceptions make your API clearer by making the consumer handle error situations.
  - But using it too much will require the cost of having try catch blocks wrapping all methods. If your callers can recover from a failure, checked exceptions are okay.

### 72. Use standard Exceptions
  - IllegalArgumentException : Non-null parameter value is inappropriate
  - IllegalStateException : Object state is inappropriate for method invocation
  - NullPointerException : Parameter value is null where prohibited
  - IndexOutOfBoundsException : Index parameter value is out of range
  - ConcurrentModificationException : Concurrent modification of an object has been detected where it is prohibited
  - UnsupportedOperationException : Object does not support method

### 73. Throw exceptions appropriate to abstraction
  - Higher layers should catch lower layer exceptions and throw exceptions from their context.
  - Exception chaining : useful when lower layer log is useful to debug the error from highrt layer context.

### 74. Document exceptions thrown from all Methods
  - Always document checked exceptions and the conditions under which they occur individually with the @throws tag. Although not mandated, it is wise to document as unchecked exceptions as well since they are preconditions for your methods.
  - Document which exceptions are checked and which is unchecked since the callers responsibility is different for each.

### 75. Include failure capture information in detail messeges.
  - Failure messages may be the only information to analyze and debug a failure after it occurs, hence should contain details of why the failure happened. It should include values of params and fields that contributed to the exception.
  - IndexOutOfBoundsException should contain details of lowerbound, upperbound and index that failed.

### 76. Strive for failure atomicity
  - When an exception gets thrown, keep the object state as it was prior to the invocation that caused failure.
  - Easiest way to achieve is to work on immutable objects. Failure will simply result in an object creation failing.
  - When working on mutable objects keep failure checks and throws in methods before state modification is done.
  - Failure atomicity can also be maintained by working on a copy object of the internal state within the method. This is a more expensive approach, but the method can simply discard the copy if failure occurs during modification.
  - Writing recovery code is difficult and is used to rollback for disk based data stores.
  - Failure atomicity is not achievable in all situations. When 2 threads modify state without proper synchronization it may leave objects in an inconsistent state, and catching ConcurrentModificationException will not make it usable.
  - If failure atomicity is violated, specify so in the API.

### 77. Don't ignore Exceptions
  - An empty catch block is usually wrong, if you have one, keep an explanation of why so in comments.
