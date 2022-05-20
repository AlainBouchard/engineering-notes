+++
title = "Java Object-Oriented Programming"
LastModifierDisplayName = "Alain Bouchard"
LastModifierEmail = "alain.bouchard@appdirect.com"
disableToc = "false"
+++

## Java Object-Oriented Programming

## Class blueprint

```java
// Class blueprint
public class MyClass {
  // Can have attributes...
  private String attribute1;  // private attribute member, getter/setter are needed
  MyEnum myEnum;
  static string staticAttribute1 = "Static Attribute"; // belongs to the class
  private final Integer myFinalInteger; // can't be modified or be overridden by any subclasses

  // Constructor
  MyClass() {
    // Initiate the instance of MyClass class.
    this("'this' is used to call alternate constructors from within a constructor");
    // optional code lines...
  }

  // Constructor
  MyClass(String message) {
    // Initiate the instance of MyClass class.
  }

  // Setter for private attribute
  public void setAttribute1(attribute1) {
    this.attribute1 = attribute1;  // note: "this" keyword is needed to disambuguate the variable reference.
  }

  // Getter for private attribute
  public String getAttribute1() {
    return attribute1;
  }

  // Methods or behaviours:
  void myMethod() {
    // no modifier so the method is accessible from its package only.
    this.attribute1 = 'xyz';
  }

  private void myPrivateMethod() {
    // private so only accessible from this classs
  }

  protected void myProtectedMethod() {
    // protected method so accessible from this class and sub-classes
  }

  public void myPublicMethod() {
    // public method so accessible from anywhere;
  }

  // Static method
  static void myStaticMethod() {
    // A static method does not rely on any non-static attribute or member.
  }
}
```

### Static vs Non-Static members

- Non-Static Member: is accessible from an instance and belongs to that instance
- Static Member: is accessible through the class and belongs to that class
  - Static members can be accessed using the class name, example:

## Enum blueprint

```java
// Enum blueprint
public enum MyEnum {
  CONSTANT1,
  CONSTANT2,
  ...,
  CONSTANTN
}
```

## Main class blueprint

```java
// Main class
public class Main {
  public static void main(String[] arg) {
    MyClass myClass = new MyClass('abc'); // Create an object of type MyClass

    myClass.myMethod('xyz');  // Call a method from my object
  }

}
```

## Princibles

### Encapsulation

- allow us to bind together data and related functionality
- prevent classes from becoming tightly coupled
- easily modify the inner workings of one class without affecting the rest of the program
- restrictions
  - we need a clear interface between a given class and the rest of the program
  - everything can't have direct access
- make the class attributes hideen from other classes using encapsulation
- provide a clear interface through public methods
- benefits
  - clear pathways for classes to communicate
  - less code changes required for a refactoring change
  - less likely for an attribute to be overwritten with an invalid or null value unexpectedly
- Access Modifiers in Java:
  - private: only visible in class that the member lives in
  - no modifier: only visible in package the member lives in
  - protected: visible to the package and all subclasses
  - public: accessible everywhere within the program

### Inheritance

- allow us to create class hierarchies
- Subclass (child class)
  - inherits properties
  - referred to as the child class
- Superclass (parent class)
  - is inherited from
  - referred to as the parent class
- promotes code reusability and scalability
- leveraging inheritance:
  - a class can only have one superclass but multiple subclasses
  - if multiple super classes is needed then multilevel inheritance is required

```java
public class MySuperClass {
  protected String a1;
  private String a2;

  MySuperClass(String a1, String a2) {
    // Super Class constructor
    this.a1 = a1;
    this.a2 = a2;
  }

  public void myMethod() {
    // Method of the Super Class
  }
}

public class MySubClass extends MySuperClass {
  MySubClass(String arg1, String arg2) {
    super(arg1, arg2);  // call superclass with arguments
    // Sub Class constructor
  }

  public String getA1() {
    return super.a1;  // this.a1 would also work for this Superclass protected variable;
  }

  @Override
  public void myMethod() {
    // Override the Super Class Method within the Sub Class
  }
}

```

### Polymorphism

- the ability for an object or function to take amny different forms
- Java supports two types of polymorphism: `run time` and `compile-time` polymorphism
- it helps to reduce complexity and write reusable code

### Abstraction

- helps us with hide implementation complexity
- Java supports `abstact classes` and `interfaces`
- helps by fixing `inputs` and `outputs` and giving general idea of what the system does
- an `abstract class`:
  - almost like a template
  - can't be instencied
  - other classes can `extend` the abstract calss and implement the appropriate functionality
- an example of an abstract class:

  ```java
  public abstract class myAbstractClass {
    // The class requires the `abstract` keyword since it contains an abstract method.

    private final String myString;  // final String (aka constant)

    protected abstract void myAbstractMethod();  // an abstract method is not implemented!
  }

  public class myOtherClass extends MyAbstractClass {
    // variables, constructors... etc.

    @Override
    protected abstract void myAbstractMethod() {
      // The abstract method from the abstract class must be implemented by the sub-class.
    }
  }

  ```

- an `interface` is:
  - a set of method signatures for `to-be-implemented` functionality
  - a specification for a set of behaviors without implementation
  - can't be instencied

  ```java
    public interface MyInterface {
      Long myMethod1();  // No method implementation
      void myMethod2();
    }

    public class myClassImplementingMyInterface implements MyInterface {
      @Override
      public Long myMethod1() {
        // the implementation for myMethod1 method
      }

      @Override
      public void myMethod2() {
        // the implementation for myMethod2 method
      }
    }

  ```

- Consider using abstract classes if any of these statements apply to your situation:
  - In the java application, there are some related classes that need to share some lines of code then you can put these lines of code within the abstract class and this abstract class should be extended by all these related classes.
  - You can define the non-static or non-final field(s) in the abstract class so that via a method you can access and modify the state of the Object to which they belong.
  - You can expect that the classes that extend an abstract class have many common methods or fields, or require access modifiers other than public (such as protected and private).
- Consider using interfaces if any of these statements apply to your situation:
  - It is a total abstraction, All methods declared within an interface must be implemented by the class(es) that implements this interface.
  - A class can implement more than one interface. It is called multiple inheritances.
  - You want to specify the behaviour of a particular data type, but not concerned about who implements its behaviour.

## Usefull build-in JAVA commands and other information

- System.out.println("string...");

### Method Reference Operator (or ::)

- a `method reference operator` (or `::`) is used to call a method by referring to it with the help of its class directly
  - like using a lambda expression, example:

  ```java
    // Get the stream
    Stream<String> stream
      = Stream.of("Geeks", "For",
                  "Geeks", "A",
                  "Computer",
                  "Portal");

    // Print the stream using lambda method:
    stream.forEach(s -> System.out.println(s));

    // Print the stream using double colon operator
    stream.forEach(System.out::println);

    // Both lambda and :: will do the same thing.
  ```

### The `Arbitrary Number of Arguments` (or `...` as a function argument)

- it means that zero or more String objects (or a single array of them) may be passed as the argument(s) for that method
- Reference : [http://java.sun.com/docs/books/tutorial/java/javaOO/arguments.html#varargs]
- important note:
  - the argument(s) passed in this way is always an array - even if there's just one. Make sure you treat it that way in the method body
  - the argument that gets the `...` must be the last in the method signature. So, myMethod(int i, String... strings) is okay, but myMethod(String... strings, int i) is not okay
- example:

```java
  public static int myFunction (int ... a) {
    int sum = 0;
    for (int i : a)
      sum += i;

    return sum;
  }
  public static void main( String args[] ) {
    int ans = myFunction(1,1,1);  // could have any number of arguments
    System.out.println( "Result is "+ ans );
  }
```

### Predicate

- a `Predicate` in general meaning is a statement about something that is either true or false. In programming, predicates represent single argument functions that return a boolean value
- example:

  ```java
    @FunctionalInterface
    public interface Predicate<T> {
      boolean test(T t);
    }
  ```

- An example with filter() since it does accept a Predicate as parameter:

  ```java
    // With lambda function:
    List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
    List<Integer> collect = list.stream().filter(x -> x > 5).collect(Collectors.toList());
    System.out.println(collect); // [6, 7, 8, 9, 10]

    // With predicate:
    Predicate<Integer> noGreaterThan5 =  x -> x > 5;
    List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
    List<Integer> collect = list.stream().filter(noGreaterThan5).collect(Collectors.toList());
    System.out.println(collect); // [6, 7, 8, 9, 10]
  ```

- More examples with (Java 8 Predicate Examples)[https://mkyong.com/java8/java-8-predicate-examples/]

### The `final` keyword

- Java final keyword is a non-access specifier that is used to restrict a class, variable, and method. If we initialize a variable with the final keyword, then we cannot modify its value
- if we declare a method as final, then it cannot be overridden by any subclasses
- if we declare a class as final, we restrict the other classes to inherit or extend it
- example:
  - final variables: to create constants
  - final classes: to prevent inheritance
  - final methods: to prevent method overriding

### Generics

- using `generics` enable types (classes and interfaces) to be parameters when defining classes, interfaces and methods.
- using `generics` give many benefits over using non-generic code
- stronger type checks at compile time:
  - Java compiler applies strong type checking to generic code and issues errors if the code violates type safety. Fixing compile-time errors is easier than fixing runtime errors, which can be difficult to find
  - example:

  ```java
    // Without Generics
    List list = new ArrayList();
    list.add("hello");

    // With Generics
    List<Integer> list = new ArrayList<Integer>();
    list.add("hello"); // will not compile
  ```

- enabling programmers to implement generic algorithms
  - by using generics, programmers can implement generic algorithms that work on collections of different types, can be customized, and are type safe and easier to read.
- elimination of casts
  - example:

  ```java
    // Without Generics:
    List list = new ArrayList();
    list.add("hello");
    String s = (String) list.get(0);  // Need to cast return value to String

    // With Generics:
    List<String> list = new ArrayList<String>();
    list.add("hello");
    String s = list.get(0); // no cast needed
  ```

- Generics type parameters:
  - T: Type
  - E: Element
  - K: Key (used in Map)
  - N: Number
  - V: Value (used in Map)
- Reference: [https://www.journaldev.com/1663/java-generics-example-method-class-interface]
