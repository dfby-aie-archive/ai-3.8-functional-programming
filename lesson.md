# Java Functional Programming

## Lesson Overview

This lesson introduces functional programming in Java, focusing on functional interfaces, lambda expressions, and the Stream API. You'll learn to write cleaner, more expressive code by applying functional programming concepts to common programming tasks like filtering, transforming, and processing collections.

**Prerequisites:** Basic Java knowledge (classes, interfaces, collections like ArrayList)

## Lesson Objectives

By the end of this lesson, students will be able to:

1. **Write** lambda expressions to create concise anonymous functions
2. **Use** built-in functional interfaces (Predicate, Function, Consumer, Supplier)
3. **Apply** Stream API operations to process collections declaratively
4. **Chain** multiple stream operations to create data processing pipelines
5. **Transform** imperative code into functional style for better readability

---

> 📌 **Note for students**: All code examples in this lesson go inside `public static void main(String[] args)`. The first example of each Part shows the full class and main wrapper. After that, snippets show only the relevant code — but everything runs inside `main`.

---

## Part 1 - Introduction to Functional Programming

### What is Functional Programming?

Functional Programming (FP) is a style of programming where you focus on **what to do** rather than **how to do it**. Instead of writing step-by-step instructions, you describe the operations you want to perform.

Think of it like giving directions:
- **Traditional approach**: "Go forward 100m, turn left, go 50m, turn right..."
- **Functional approach**: "Go to the nearest coffee shop"

Key ideas in functional programming:
- **Functions are values**: You can pass functions around just like numbers or strings
- **Avoid changing data**: Instead of modifying existing data, create new data
- **Chain operations**: Connect multiple operations together like building blocks

Java introduced functional programming features in **Java 8** with Lambda expressions and the Stream API.

### Why Learn Functional Programming?

Functional programming helps you write:
- **Cleaner code**: Less boilerplate, more readable
- **Safer code**: Fewer bugs from accidental data changes
- **More maintainable code**: Easier to understand and modify

### Traditional vs Functional — Side by Side

**Task**: Get all even numbers from a list.

**Traditional way (How to do it):**

```java
ArrayList<Integer> numbers = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
ArrayList<Integer> evenNumbers = new ArrayList<>();

for (int i = 0; i < numbers.size(); i++) {
    if (numbers.get(i) % 2 == 0) {
        evenNumbers.add(numbers.get(i));
    }
}

System.out.println(evenNumbers); // [2, 4]
```

You are telling Java — loop through, get each item, check if even, add to list. You control every step.

**Functional way (What you want):**

```java
List<Integer> evenNumbers = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());

System.out.println(evenNumbers); // [2, 4]
```

You are just saying — stream the numbers, keep only the even ones, collect them. Java handles the loop.

> **Key point**: The original `numbers` list is **untouched**. Functional programming creates a **new list** rather than modifying the original. This makes your code safer — other parts of your program that use `numbers` are not affected.

Create `LearnFunctionalProgramming.java` and code along.

---

## Part 2 - Functional Interfaces

### What is a Functional Interface?

A **functional interface** is an interface with **exactly one abstract method**. This is the foundation of functional programming in Java — before you can use a lambda, you need a functional interface to hold it.

Think of a functional interface as a **contract**:
- The contract says: "You must provide one specific behavior"
- Your lambda expression fulfills that contract

> **Quick preview: what is a lambda?** You'll see lambdas everywhere in the examples below, so here's just enough to follow along. A **lambda expression** is a short, unnamed function that fills in the one method a functional interface requires. It looks like this: `(parameters) -> body` — for example, `name -> System.out.println(name)` means "take `name`, and print it." That's all you need for now — we'll do a full deep dive into lambda syntax and all its formats in **Part 3**.

### A Simple Example

Here is the full working code — notice the class and main wrapper:

```java
import java.util.Arrays;
import java.util.List;

@FunctionalInterface
interface Greeting {
    void sayHello(String name);
}

public class LearnFunctionalProgramming {
    public static void main(String[] args) {

        // The lambda implements the sayHello method
        Greeting friendlyGreeting = name -> System.out.println("Hello, " + name + "!");
        Greeting formalGreeting = name -> System.out.println("Good day, " + name + ".");

        // Use them
        friendlyGreeting.sayHello("Alice"); // Output: Hello, Alice!
        formalGreeting.sayHello("Bob");     // Output: Good day, Bob.

    }
}
```

**What happened?**
- `Greeting` interface has exactly one method — `sayHello`
- `name -> System.out.println("Hello, " + name + "!")` is a lambda that fulfills that contract
- We can store different behaviors in the same interface type and call them later

### Built-in Functional Interfaces

Java provides common functional interfaces in the `java.util.function` package so you don't have to create your own for common patterns.

#### 1. Predicate\<T\> - Testing a Condition

**What it does**: Takes one input, returns true or false

**The method**: `boolean test(T t)`

```java
import java.util.function.Predicate;

// Create a predicate that tests if a number is even
Predicate<Integer> isEven = number -> number % 2 == 0;

// Test it
System.out.println(isEven.test(4));  // true
System.out.println(isEven.test(7));  // false
```

**Passing different behaviors to the same method:**

A Predicate isn't just for testing values directly — you can pass it into a method as an argument, the same way you'd pass an `int` or a `String`. The method just needs a parameter of type `Predicate<T>`.

The real power of this is that the *same method* behaves differently depending on which Predicate you hand it. The method stays generic — the rule comes from outside.

```java
public static void checkString(String str, Predicate<String> condition) {
    System.out.println(condition.test(str));
}
```

```java
// Two different definitions of "lengthy"
Predicate<String> longerThan5  = s -> s.length() > 5;
Predicate<String> longerThan10 = s -> s.length() > 10;

// Same method, same word — but different rule passed in → different result
checkString("Beautiful", longerThan5);  // true  (9 > 5)
checkString("Beautiful", longerThan10); // false (9 > 10)
```

Notice: `checkString` never changed. We only changed *which behavior we passed in*. This is what "functions are values" really means — you hand the method a rule, and it runs whatever rule it was given.

**Practical use with collections:**

```java
ArrayList<Integer> numbers = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10));

// Remove all odd numbers using a predicate
numbers.removeIf(n -> n % 2 != 0);
System.out.println(numbers); // [2, 4, 6, 8, 10]
```

`removeIf()` takes a `Predicate` — here, `n -> n % 2 != 0` ("is this number odd?"). It goes through the list and removes every element where the predicate returns `true`. So it strips out all the odd numbers, leaving `[2, 4, 6, 8, 10]` — this **mutates the original list in place**, using the predicate to decide what gets removed.

#### 2. Function\<T, R\> - Transforming Data

**What it does**: Takes one input of type T, returns output of type R

**The method**: `R apply(T t)`

```java
import java.util.function.Function;

// Takes a String, returns its length (Integer)
Function<String, Integer> getLength = str -> str.length();

System.out.println(getLength.apply("Hello")); // 5
System.out.println(getLength.apply("Java"));  // 4
```

Like Predicate, a Function can be passed into a method, and different Functions give different results from the same method:

```java
public static void convert(String str, Function<String, String> converter) {
    System.out.println(converter.apply(str));
}
```

```java
Function<String, String> toUpper = str -> str.toUpperCase();
Function<String, String> toLower = str -> str.toLowerCase();

convert("Hello World", toUpper); // HELLO WORLD
convert("Hello World", toLower); // hello world
```

#### 3. Consumer\<T\> - Performing Actions

**What it does**: Takes one input, returns nothing (just does something with the input)

**The method**: `void accept(T t)`

```java
import java.util.function.Consumer;

ArrayList<String> names = new ArrayList<>(Arrays.asList("Alice", "Bob", "Charlie"));

// Print each name with a greeting
Consumer<String> greetPerson = name -> System.out.println("Hello, " + name + "!");
names.forEach(greetPerson);
// Output:
// Hello, Alice!
// Hello, Bob!
// Hello, Charlie!
```

#### 4. Supplier\<T\> - Providing Values

**What it does**: Takes no input, returns a value

**The method**: `T get()`

```java
import java.util.function.Supplier;

// Supplier that always returns "Hello"
Supplier<String> greetingSupplier = () -> "Hello";

System.out.println(greetingSupplier.get()); // Hello
System.out.println(greetingSupplier.get()); // Hello
```

### Summary of Common Functional Interfaces

| Interface | Input | Output | Method | Use Case | Example |
|-----------|-------|--------|--------|----------|---------|
| `Predicate<T>` | T | boolean | `test(T t)` | Testing conditions | Is this number even? |
| `Function<T,R>` | T | R | `apply(T t)` | Transforming data | Convert string to uppercase |
| `Consumer<T>` | T | void | `accept(T t)` | Performing actions | Print each item |
| `Supplier<T>` | none | T | `get()` | Providing values | Generate random number |

> **Coming up in Part 4**: `Predicate` powers `filter()`, `Function` powers `map()`, and `Consumer` powers `forEach()`. These four interfaces are the building blocks of the Stream API.

### Creating Your Own Functional Interface

You can create custom functional interfaces for specific needs:

```java
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
}

public class LearnFunctionalProgramming {
    public static void main(String[] args) {

        // Different calculations using the same interface
        Calculator add = (a, b) -> a + b;
        Calculator subtract = (a, b) -> a - b;
        Calculator multiply = (a, b) -> a * b;

        System.out.println("5 + 3 = " + add.calculate(5, 3));        // 8
        System.out.println("5 - 3 = " + subtract.calculate(5, 3));   // 2
        System.out.println("5 * 3 = " + multiply.calculate(5, 3));   // 15
    }
}
```

**The `@FunctionalInterface` annotation** is optional but recommended because:
- It documents that this interface is meant for lambda expressions
- The compiler will give an error if you accidentally add a second abstract method

> **Note**: A functional interface can have only **one abstract method**, but it can have any number of `default` methods, `static` methods, and constants. Only abstract methods are limited — because the lambda needs to know which single method it is filling in.

### 🧑‍💻 Quick Activity: Functional Interfaces Practice

Try these exercises:

1. Create a Predicate to check if a string contains the letter 'a'
   ```java
   Predicate<String> containsA = // Your code here
   System.out.println(containsA.test("banana")); // true
   System.out.println(containsA.test("kiwi"));   // false
   ```

2. Create a Function that takes an integer and returns "Even" or "Odd"
   ```java
   Function<Integer, String> evenOrOdd = // Your code here
   System.out.println(evenOrOdd.apply(4));  // "Even"
   System.out.println(evenOrOdd.apply(7));  // "Odd"
   ```

3. Use Consumer to print each word with "Word: " prefix
   ```java
   List<String> words = Arrays.asList("Java", "Python", "C++");
   Consumer<String> printWithPrefix = // Your code here
   words.forEach(printWithPrefix);
   // Output: Word: Java, Word: Python, Word: C++
   ```

---

## Part 3 - Lambda Expressions

### What Exactly is a Lambda Expression?

A **lambda expression** is a short way to write a function without giving it a name. It's also called an **anonymous function**. It is how you fulfill the contract of a functional interface in the shortest possible way.

Think of it as a recipe you use once and don't need to save:
- **Named function**: Like a recipe card you keep in your recipe box
- **Lambda**: Like quickly telling someone "just mix flour and water"

### Lambda Syntax Explained

The basic structure is:

```
(parameters) -> { body }
```

Let's break this down:

```java
(name) -> System.out.println(name)
```

- `(name)` - **Parameter**: The input to your function
- `->` - **Arrow**: Means "goes to" or "becomes"
- `System.out.println(name)` - **Body**: What to do with the input

**Read it as**: "name goes to print name"

### The Four Lambda Formats

Here is the full working code showing all four formats:

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collections;

@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
}

public class LearnFunctionalProgramming {
    public static void main(String[] args) {

        // ── Format 1: No parameters ──
        Runnable task = () -> System.out.println("Task is running");
        task.run(); // Output: Task is running

        // ── Format 2: One parameter (parentheses optional) ──
        ArrayList<Integer> numbers = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
        numbers.forEach(num -> System.out.println(num * 2));
        // Output: 2, 4, 6, 8, 10

        // ── Format 3: Multiple parameters (parentheses required) ──
        ArrayList<String> names = new ArrayList<>(Arrays.asList("Alice", "Bob", "Charlie"));
        Collections.sort(names, (a, b) -> a.length() - b.length());
        System.out.println(names); // [Bob, Alice, Charlie]

        // ── Format 4: Multiple statements (braces + explicit return) ──
        Calculator sumWithPrint = (x, y) -> {
            int sum = x + y;
            System.out.println("Sum is: " + sum);
            return sum;
        };
        sumWithPrint.calculate(3, 4); // Sum is: 7

    }
}
```

**The key thing to remember about Format 4:**

A single-expression lambda returns its value **automatically** — no `return` keyword needed:

```java
(a, b) -> a + b        // returns a + b automatically
```

The moment you add curly braces, that stops. You must write `return` yourself:

```java
(x, y) -> {
    int sum = x + y;
    System.out.println("Sum is: " + sum);
    return sum;        // required
}
```

### Important Rule: You Cannot Change an Outside Variable

A lambda can **read** a variable from outside, but it cannot **change** it:

```java
int count = 0;
names.forEach(name -> count++);   // COMPILE ERROR
```

Error: *"local variables referenced from a lambda expression must be final or effectively final"*

Reading is fine:

```java
int multiplier = 3;   // never changed after this line
numbers.forEach(n -> System.out.println(n * multiplier));  // works
```

**Why**: a lambda might run later, or on a different thread, when that variable no longer exists. Java takes a copy of the value and locks it. If you need a counter, use a normal loop.

### When to Use Lambdas

✅ **Use lambdas when:**
- You need to pass simple behavior to a method
- The function is used only once, in that one place
- It makes your code clearer and shorter

❌ **Write a proper named method instead when:**
- The logic is more than a few lines
- You need the same behavior in several places
- You want to unit test it, or you need a readable name in error messages

> **Note**: You've already been writing lambdas throughout Part 2, and you'll get more practice in the Stream activity coming up — so there's no separate lambda exercise here.

---

## Part 4 - Stream API

### What is a Stream?

A **Stream** is like a pipeline for processing data. Imagine a conveyor belt in a factory:
- Items come in at one end
- They go through various stations (filtering, transforming, sorting)
- Final products come out at the other end

A Stream is **not** a data structure. It doesn't store data. The data stays in your original list — the stream only holds the **instructions** and pulls items through when asked.

### How Streams Work

Streams work in three steps:

1. **Create** a stream from a data source
2. **Transform** the data with intermediate operations (filter, map, sorted, etc.)
3. **Collect** or use the results with a terminal operation

```
Data Source → Intermediate Operations → Terminal Operation → Result
```

### Creating Streams

```java
// Method 1: From a collection (the common one)
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
Stream<String> nameStream = names.stream();
```

You can also create a stream from an array using `Arrays.stream(myArray)`, or from loose values using `Stream.of(1, 2, 3)`. In practice you'll use `.stream()` on a collection almost every time.

### Stream Operations: Two Types

#### 1. Intermediate Operations
- **Return** a new stream
- Are **lazy** (don't execute until a terminal operation is called)
- Can be **chained** together
- Examples: `filter()`, `map()`, `sorted()`

#### 2. Terminal Operations
- **Trigger** the stream pipeline execution
- Return a **result** or cause a side effect
- Can only be used **once** on a stream
- Examples: `collect()`, `forEach()`, `count()`

> **Important**: You can chain as many intermediate operations as you like, but only **one terminal operation**, at the end. Once it runs, the stream is used up.

### Important: Streams are Lazy

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

// This doesn't execute yet!
Stream<String> stream = names.stream()
    .filter(name -> {
        System.out.println("Filtering: " + name);
        return name.startsWith("A");
    });

System.out.println("Stream created, but nothing printed yet!");

// Now it executes!
stream.forEach(name -> System.out.println("Result: " + name));

// Output:
// Stream created, but nothing printed yet!
// Filtering: Alice
// Result: Alice
// Filtering: Bob
// Filtering: Charlie
```

Notice the order of that output: elements flow through the pipeline **one at a time**. Alice goes through filter, then straight to the result — *then* Bob starts. It is not "filter the whole list, then print the whole list."

### Intermediate Operations

#### filter() - Keep Only What You Want

**What it does**: Keeps elements that match a condition (a Predicate)

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

List<Integer> evenNumbers = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());

System.out.println(evenNumbers); // [2, 4, 6, 8, 10]
```

#### map() - Transform Each Element

**What it does**: Transforms each element using a Function

```java
List<String> names = Arrays.asList("alice", "bob", "charlie");

List<String> upperNames = names.stream()
    .map(name -> name.toUpperCase())
    .collect(Collectors.toList());

System.out.println(upperNames); // [ALICE, BOB, CHARLIE]
```

#### sorted() - Arrange in Order

```java
List<Integer> numbers = Arrays.asList(5, 2, 8, 1, 9, 3);

// Natural order (ascending)
List<Integer> sorted = numbers.stream()
    .sorted()
    .collect(Collectors.toList());

System.out.println(sorted); // [1, 2, 3, 5, 8, 9]

```

### Terminal Operations

#### collect() - Gather Results

**What it does**: Converts the stream back into a collection

```java
import java.util.stream.Collectors;

List<String> names = Arrays.asList("Alice", "Bob", "Alice", "Charlie");

// Collect to List - keeps duplicates and order
List<String> nameList = names.stream()
    .collect(Collectors.toList());

// Collect to Set - removes duplicates
Set<String> nameSet = names.stream()
    .collect(Collectors.toSet());
```

#### forEach() - Do Something With Each Element

**What it does**: Performs an action on each element

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

names.stream()
    .forEach(name -> System.out.println("Hello, " + name + "!"));
```

> **Note**: `forEach` produces nothing. It is for *doing* something (printing, logging, saving). If you want changed data back, use `map()` then `collect()`.

#### count() - Count Elements

**What it does**: Counts how many elements are in the stream

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

long count = numbers.stream()
    .filter(n -> n % 2 == 0)
    .count();

System.out.println("Even numbers: " + count); // 5
```

### Chaining Operations - The Real Power

The real power of streams comes from chaining operations together:

```java
List<String> names = Arrays.asList("alice", "BOB", "Charlie", "DAVID", "eve");

List<String> result = names.stream()
    .filter(name -> name.length() > 3)     // Only names with 4+ letters (removes "BOB" and "eve")
    .map(n -> n.toLowerCase())             // Convert to lowercase
    .sorted()                              // Sort alphabetically
    .collect(Collectors.toList());         // Collect to list

System.out.println(result); // [alice, charlie, david]
```

> **Order matters**: put `filter()` before `map()` so you only transform the elements that survive. Be aware that order can also change the **result**, not just the speed — mapping to uppercase before filtering for names starting with "A" would catch a lowercase "alice", but filtering first would miss it.

### 🧑‍💻 Quick Activity: Stream Practice

Try these exercises:

1. From a list of numbers 1-20, get all numbers divisible by 3
   ```java
   List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20);
   // Your code here to get [3, 6, 9, 12, 15, 18]
   ```

2. From a list of names, get all names in uppercase that are longer than 4 characters
   ```java
   List<String> names = Arrays.asList("John", "Alice", "Bob", "Charlie", "Eve");
   // Your code here to get [ALICE, CHARLIE]
   ```

---

## Part 5 - Extras (Optional / Self-Study)

> This section is optional. It gives you exposure to things you may meet in other people's code. There is no activity for this part.

### Method References

Method references are a shorter way to write a lambda **when the lambda only calls a single existing method**, using the `::` operator.

```java
// Lambda
names.forEach(name -> System.out.println(name));

// Method reference — same thing, shorter
names.forEach(System.out::println);
```

**Simple rule**: whenever your lambda looks like `x -> someMethod(x)`, you can usually replace it with `someClass::someMethod`. Java passes the item automatically. If your lambda does anything extra — like `name -> System.out.println("Hi " + name)` — the shortcut does not apply.



### Parallel Streams

Swapping `.stream()` for `.parallelStream()` splits the work across multiple CPU cores:

```java
List<String> result = names.parallelStream()
    .filter(n -> n.length() > 4)
    .collect(Collectors.toList());
```

This works safely **because** functional code doesn't change shared data — each element is processed independently. Only worth it for large amounts of data; on small lists it is actually slower, because splitting and coordinating the work costs more than it saves.

---

END