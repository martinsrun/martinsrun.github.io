---
layout: post
title:  "Java Effective - VII"
subtitle: "90 points to make your coffee"
date:   2019-06-09 00:00:00
categories: [programming,books,java]
---

## 7. Methods

### 49. Check parameters for validity
  - Checking params ensures failures happen as soon as possible. Also your API can export exact details of what params are expected and what exception gets thrown if they are violated.
  - Objects.requireNonNull helps with null check validations.
  - For private methods since API isn't exported this is not as important.
  - Checking params are extremely important for constructors and static factories since they might not use the passed in params at the moment. This could cause functionality to break when those params are actually used.

### 50. Make defensive copies as needed  
 - Always assume your clients are out to break your class invariants.
 - If your constructor accepts mutable types, make defensive copies of them before storing them in your class. Do validity checks on the defensive copies, not the actual parameters, to prevent time-of-check/time-out-use attacks.
 - Do not use clone methods to make defensive copies, the object you received may be an untrustworthy subclass of your original param which could have it's own bad clone implementation.
 - Return defensive copies of mutable internal fields.
 - Defensive copies can be avoided in certain cases where your clients can be trusted to not mess up, but your doc should call out the risks of modifying the objects in question. For example when using wrapper patterns, you usually wrap and return the original object. If client then modifies the wrapped object it may break the wrapper behavior but that impacts the client since they are using the wrapper.

### 51. Design method signatures carefully
  - Choose method names carefully - names should have meanings and be consistent within the package.
  - Avoid too many convenience methods - You will have to support and maintain them.
  - Avoid long parameter lists - Long sequences of identically typed parameters are especially harmful. To reduce param sizes
    - Break params into multiple methods
    - Create static helper member classes to hold params
    - Use builders.
  - For param types use interfaces over inplementation classes.
  - Prefer 2 element enum types over boolean when the param mean something. eg, if the param is which temperature scale to use, create an enum with { FAHRENHEIT, CELSIUS }, instead of passing a boolean to choose between.


### 52. Use overloading judiciously
  - Overloading determines which method to call at compile time, which goes against most instincts since overriding works at runtime. The following class with 3 overloaded classify methods prints "Unknown collection" 3 times, since which overloaded classify is to be called is determined based on the compile time parameter type.
   ```JAVA
   public class CollectionClassifier {
     public static String classify(Set<?> s) {
        return "Set";
    }
    public static String classify(List<?> lst) {
      return "List";
    }
    public static String classify(Collection<?> c) {
      return "Unknown Collection";
    }

    public static void main(String[] args) {
      Collection<?>[] collections = {
        new HashSet<String>(),
        new ArrayList<BigInteger>(),
        new HashMap<String, String>().values()
      };
      for (Collection<?> c : collections)
      System.out.println(classify(c));
    }
  }
   ```
  - A safe and conservative way to do overloading is to never have 2 overloadings with the same number of parameters. You can always give distinct method names without overloading.
  - This is not always possible for constructors as renaming is not an option. In that case make sure it is clear which constructor gets called in each instance, ie have radically different overloadings. It should be impossible to cast 2 non null values into the parameters of both overloadings. eg, ArrayList has a constuctor that takes an int, and another constructor that takes a Collection, that isn't confusing.
  - Lambdas, autoboxing and generic types  bring in more confusion into which overloaded method gets invoked. List has a remove method which is overloaded to take an int and to take an object of type stored in the list. If you have a list of Integers, and call l.remove(2), it removes object from index 2, not the integer 2 stored within the list.
  ExecutorService submit is overloaded to take both Runnable and Callable types, println is an overloaded method as well hence the call to exec submit with a println method reference is confusing to the compiler. Avoid overloading methods to take functional arguments in the same position.
    ```JAVA
    \\ this compiles
    new Thread(System.out::println).start();
    \\ this fails compilation
    ExecutorService exec = Executors.newCachedThreadPool();
    exec.submit(System.out::println);
    ```

### 53. Use varargs judiciously
  - Varargs shouldnt be used in performance critical situations since, cost of array creation and allocation.
  - If most of your use cases use 3 or fewer params, write methods like :
  ```JAVA
    public void foo foo() {}
    public void foo foo(int a1) {}
    public void foo foo(int a1, int a2) {}
    public void foo foo(int a1, int a2, int a3) {}
    public void foo foo(int a1, int a2, int a3, int... rest) {}
  ```

### 54. Return empty collections or arrays, never nulls
  - Handling null response will fall on your consumers, and is not worth it.
  - If performance of allocating and returning an empty collection seems to be a problem, cache and return the same reference, or use Collections.emptyList, Collections.emptySet, Collections.emptyMap

### 55. Use optionals judiciously
  - Optionals are a better option than returning null or throwing an exception when a return value is not present. Exceptions represent an exception scenerio which no values are not, and returning nulls is usually harder to handle.  Having optionals force your consumers to face the possibility that the method may not return a value and to handle it, similar to having a checked exception.
  - Optional::isPresent return true if an optional contain a value, but this is a fallback, optional methods usually take functions that can be used to work on it's values. Methods such as map, orElse, orElseGet, orElseCompute
  - When you have to deal with a stream of optionals, to convert them into values, Java 9 has a neat API streamOfOptionals.flatMap(Optional::stream)
  - Don't wrap collections in options, but simply return empty collections. Understand that options are not free and require allocation and comparisons to work, hence use when performance is not a constraint.

### 56. Write doc comments for exposed applies
  - Doc comment should make succinct the contract between client and API. It should say what a method does, its pre-conditions and its post-conditions. Typically the @throws tag and @param tag represnt preconditions and their violations.
  - Method should document every side-effect it causes, such as starting a background thread.
  - Method should have an @param tag for every param, and @returns unless void return, and @throws for every checked and unchecked exception. @param and @return tags should be a phrase describing the value represented, @throws should have the exception and an 'if' describing when it is thrown. These tag phrases don't have to be terminated by a period.
  ```JAVA
  /**
* Returns the element at the specified position in this
list.
*
* <p>This method is <i>not</i> guaranteed to run in  constant
* time. In some implementations it may run in time proportional
* to the element position.
*
* @param index index of element to return; must be  non-negative and less than the size of this list
* @return the element at the specified position in this list
* @throws IndexOutOfBoundsException if the index is out of range
*
({@code index < 0 || index >= this.size()})
*/
E get(int index);
  ```
  - The @code tag renders the portion as a code snippet, preventing doc html from rendering special chars (<, |) as html.
  - The first sentence of a doc comment becomes its summary description. Ensure no 2 members/constructors have the same summary description.
  - When documenting a generic typed class make sure to document all type parameters.
  ```JAVA
  /**
    * @param <K> the type of keys maintained by this map
    * @param <V> the type of mapped values
    */
  ```
  -
