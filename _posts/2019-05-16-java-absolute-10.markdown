---
layout: post
title:  "Java absolute - X"
subtitle: "90 points to make your coffee"
date:   2019-08-20 00:00:00
categories: [programming,books,java]
---

## 10. Concurrency

### 78. Synchronize access to shared mutable data
  - Synchronization is not simply mutual exclusion where threads cannot see inconsistent states being modified by other threads. Proper synchronization guarantees that a thread that gains access to a synchronized block/lock can see all modifications performed within that lock before it.
  - JLS guarantees atomic updates to primitives and objects, excluding long and double. long/double being 64 bit could have their first 32bit and last 32bit updated independently, hence its safer to declare them as volatile if open to update from multiple threads to prevent this.
  - Synchronization is required for both mutual exclusion as well as reliable communication between threads.
  ```JAVA
  public class StopThread {
    private static boolean stopRequested;
      public static void main(String[] args)
      throws InterruptedException {
      Thread backgroundThread = new Thread(() -> {
        int i = 0;
        while (!stopRequested)
        i++;
        });
      backgroundThread.start();
      TimeUnit.SECONDS.sleep(1);
      stopRequested = true;
    }
  }
  ```
  - This fails because the check to while (!stopRequested) could be replace by while (true), in an optimization known as hoisting, resulting in liveliness failure. The solution to this is to either synchronize access to the stopRequested variable or to make it volatile.
  - value++ operations are not atomic and needs to be synchronized or better use objects from the java.util.concurrent.atomic package.
  - Keep mutations done in a thread limited to that threads. It is safe to do modifications on an object from one thread then share it with other threads, synchronizing only on the sharing, as long as it's not modified again. Such objects are effectively immutable, and sharing them across threads is safe publication. There are multiple ways to safely publish : use a volatile field, store in a static final field, store access with a lock, share witha concurrent collection.
  - If you need only inter thread communication and not mutual exclusion, volatile can be an acceptable compromise, as it guarantees the latest update done to the variable.

### 79. Avoid excessive synchronization
  - Never give control to client methods from within a synchronized block. Using observers or callback methods from within synchronized blocks can be dangerous as you cannot predict what they do. They could call other methods synchronized under the same lock, since java locks are re-entrant, bypassing your intended lock protection and mutating state while the current mutation is not complete. They may even start another thread that waits on the lock that your synchronized thread holds, causing a deadlock. Such methods are called alien methods, and having the outside synchronized blocks is known as an open call.
  - You should do as little work as possible from with synchronized blocks, since operations in synch blocks cost the opportunity to parallelise as well as VM optimizations.
  - A mutable class has 2 options, either synchronize access to state mutations or leave it open and ask clients to do so when necessary. java.util.concurrent objects provide locked objects, while most util classes are unlocked. Locked objects are said to be thread-safe.
  - StringBuffer is a thread-safe implementation that locks access with internal synchronization. Later StringBuilder had to be brought out to supplant it as a non thread-safe version. Similarly java.util.Random was supplanted by java.util.concurrent.ThreadLocalRandom. When in doubt do not synchronize, but document as non thread-safe.
  - If synchronizing internally you can use optimizations to achieve higher concurrency : lock splitting,lock striping, nonblocking concurrency control

### 80. Prefer executors, tasks and streams to threads
  ```JAVA
    ExecutorService exec = Executors.newSingleThreadExecutor();
    exec.execute(runnable);
    exec.shutdown();
  ```
  - Executors split up the task and the execution scheduling, unlike threads.
  - Executors provide different types of thread pools depending on scheduling needs.
  - There are 2 types of tasks supported, callable and runnable.

### 81. Prefer concurrency utils over wait-notify
  - Since java 5 the concurrency package has evolved with multiple classes supporting higher level concurrency abstractions. Using wait-notify is complex in comparison.
  - There are 3 main categories of utils, executors, concurrent collections, synchronizers
  - Concurrent collections implement collection interfaces to provide high performant concurrent impls. They provide state-dependent modify operations which combine several primitives into a single atomic operation, and where added in  Java 8 into the collection interface with default method impls. eg, the Map interface's putIfAbsent(key, value)
  - Concurrent collections make synchronized collections obsolete, use ConcurrentHashMap instead of Collections.synchronizedMap for far better performance.
  - Synchoronizers are objects that enable threads to wait on one another allowing them to co-ordinate their activities. The most commonly used synchronizers are Countdownlatch and Semaphore, but java also provides cyclicbarrier, exchange and phasers.
  - Countdownlatch is a single-use barrier that allow threads to wait for one or more threads to perform some activity, the sole constructor taking the number of threads it needs to wait on.
  ``` JAVA
  // Simple framework for timing concurrent execution
  public static long time(Executor executor, int concurrency, Runnable action) throws InterruptedException
  {
    CountDownLatch ready = new CountDownLatch(concurrency);
    CountDownLatch start = new CountDownLatch(1);
    CountDownLatch done = new CountDownLatch(concurrency);
    for (int i = 0; i < concurrency; i++) {
      executor.execute(() -> {
        ready.countDown(); // Tell timer we're ready
        try {
          start.await(); // Wait till peers are ready
          action.run();
        } catch (InterruptedException e) {
          Thread.currentThread().interrupt();
        } finally {
          done.countDown(); // Tell timer we're done
        }
      });
    }
    ready.await();
    // Wait for all workers to be  ready
    long startNanos = System.nanoTime();
    start.countDown(); // And they're off!
    done.await();
    // Wait for all workers to finish
    return System.nanoTime() - startNanos;
  }
  ```
  - The method spawns a number of threads specfied by a param, each with the given executor, waits for all threads to get ready and then times how long it takes to process in parallel. If has 3 latches, ready: which marks that a thread is ready to proceed, start:which stops the thread from then continuing until others have been ready, done: which marks all threads as complete.
  - There is seldom if any reason to use wait-notify in code these days.

### 82. Document thread safety
  - The presence of synchronized modifier in a method declaration is not a part of it's API and does not guarantee thread safety.
  - Document one of the following levels of thread safety:
    - Immutable : Instances of the class are constant, there is no need for an external synchronization
    - Un-conditionally thread safe : Mutable, but has sufficient internal synchronization that it can be used without external synchronization. eg, AtomicLong, ConcurrentHashMap
    - Conditionally thread safe : Some methods need external synchronization to be used in a thread safe manner.  eg, Collections.synchronized wrapped collections need external synchronize for its iterators.
    - Not thread safe : Instances of this class are mutable and to use them concurrently the client must surround method invocations with external synchronization, eg ArrayList, HashMap
    - Thread-Hostile: Class is unsafe for concurrent use even if method accesses have external synchronization. This is usually the result of modifying static data without synchronization.
  - Lock objects should be declared as final to prevent inadvertent updates which could be catastrophic.
  - Lock objects should be private on unconditonally thread-safe classes to prevent clients from using the same lock causing locks. For conditionally thread-safe classes the document should contain details on what lock object the client should externally sycronize on.

### 83. Use lazy initialization judiciously
  - Lazy initialization is primarily an optimization but can help break harmful circular references as well. It is a double edged sword since the cost to access goes up. Lazy initialization can be useful if the field is accessed in very few instances.
  - Under most circumstances normal initialization is preferred.
  - Using a synchronized access to a field handles lazy initialization  to prevent multiple threads from initializing the same field, but this is not performant. Using double check-lock will make access better. Make sure the field used for double check locking is volatile.
  - An easier way to have lazy initialization is to use a static holder class idiom. Since the getField simply returns access, it does not need Synchronization. The holder class gets loaded on first reference.
  ```JAVA
  // Lazy initialization holder class idiom for static fields
  private static class FieldHolder {
    static final FieldType field = computeFieldValue();
  }
  private static FieldType getField() return { FieldHolder.field; }
  ```
### 84. Do not depend on thread scheduling
  - Programs should not depend on thread scheduling behavior for correctness or performance
