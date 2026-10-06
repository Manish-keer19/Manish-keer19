# Core Java Syntax Practice — Before DSA

> **Purpose:** Rebuild Java coding muscle memory through hands-on practice.
> Open a Java editor → read a task → write the code yourself.
> No theory. No solutions. No DSA. Just code.

---

## 1. Java Program Structure

### What to practice

* Writing a complete Java class from scratch
* `public`, `private`, `static`, `final` keywords
* The `main` method signature
* Single-line and multi-line comments
* Import statements
* Basic file structure

### Practice

1. Write a complete Java program that prints `Hello, World!` to the console.
2. Write a class called `App` with a `main` method that prints your name.
3. Add a single-line comment above the `main` method describing what it does.
4. Add a multi-line comment at the top of a Java file with your name and date.
5. Write a class with a `public static final` integer constant called `MAX_SIZE` set to `100`, and print it from `main`.
6. Write a class with a `private static` String field called `appName`, assign it a value, and print it from `main`.
7. Write a class with two methods: a `static` method `greet()` that prints a greeting, and `main` that calls `greet()`.
8. Write a class with a `private` field, a `public` method, and a `static` method. Call both methods from `main`.
9. Import `java.util.Scanner` and `java.util.ArrayList` at the top of a file (you don't need to use them — just practice the import syntax).
10. Write a program with a `public` class, a `private static final` constant, a `private static` method, and a `public static void main` method that calls the private method.
11. Create two classes in your mind: `Main` and `Helper`. In `Main`, create an object of `Helper` and call a method on it. Write both classes.
12. Write a class with a Javadoc comment (`/** ... */`) above a method.
13. Write a complete program that declares a `final` local variable inside `main`, assigns it, and prints it.

---

## 2. Variables and Data Types

### What to practice

* All 8 primitive types: `byte`, `short`, `int`, `long`, `float`, `double`, `char`, `boolean`
* `String` declaration and initialization
* `final` variables
* Type casting (widening and narrowing)
* Literal suffixes: `L`, `f`, `d`
* Overflow behavior

### Practice

1. Declare an `int` variable, assign it `42`, and print it.
2. Declare a `double` variable, assign it `3.14`, and print it.
3. Declare a `char` variable, assign it `'A'`, and print it.
4. Declare a `boolean` variable, assign it `true`, and print it.
5. Declare a `String` variable, assign it `"Java"`, and print it.
6. Declare a `byte` variable with the maximum value a byte can hold, and print it.
7. Declare a `short` variable with the value `30000`, and print it.
8. Declare a `long` variable using the `L` suffix with the value `10000000000L`, and print it.
9. Declare a `float` variable using the `f` suffix with the value `2.5f`, and print it.
10. Declare a `final int` variable called `SPEED_LIMIT`, assign it `120`, and print it.
11. Declare an `int` and a `double`. Assign the `int` to the `double` (widening cast) and print the `double`.
12. Declare a `double` with value `9.99`. Cast it to an `int` (narrowing cast) and print the `int`.
13. Declare a `char` with value `'Z'`. Cast it to an `int` and print the numeric value.
14. Declare an `int` with value `65`. Cast it to a `char` and print the character.
15. Declare two `int` variables on the same line, assign them different values, and print both.
16. Declare a variable of each primitive type (all 8) and print each one.
17. Declare an `int` with value `2_000_000` (using underscores) and print it.
18. Assign `Integer.MAX_VALUE` to an `int`, add `1` to it, and print the result. Observe the overflow.
19. Declare a `long` and assign it the result of multiplying two large `int` values. Use `(long)` cast to avoid overflow.
20. Declare a `String`, reassign it to a different value, and print the new value.

---

## 3. Operators

### What to practice

* Arithmetic: `+`, `-`, `*`, `/`, `%`
* Assignment: `=`, `+=`, `-=`, `*=`, `/=`, `%=`
* Comparison: `==`, `!=`, `>`, `<`, `>=`, `<=`
* Logical: `&&`, `||`, `!`
* Increment/Decrement: `++`, `--` (pre and post)
* Ternary: `? :`
* Basic bitwise: `&`, `|`, `^`, `~`, `<<`, `>>`

### Practice

1. Declare two `int` variables and print their sum, difference, product, quotient, and remainder.
2. Declare an `int` variable with value `10`. Use `+=` to add `5`, then print it.
3. Declare an `int` variable with value `20`. Use `-=`, `*=`, `/=`, and `%=` one at a time, printing after each.
4. Declare two `int` variables. Print the result of each comparison operator (`==`, `!=`, `>`, `<`, `>=`, `<=`).
5. Declare two `boolean` variables. Print the result of `&&`, `||`, and `!` on them.
6. Declare an `int` variable. Print `x++` (post-increment), then print `x`. Then print `++x` (pre-increment), then print `x`.
7. Declare an `int` variable. Print `x--` (post-decrement), then print `x`. Then print `--x` (pre-decrement), then print `x`.
8. Use the ternary operator to assign `"Even"` or `"Odd"` to a `String` variable based on an `int`, then print it.
9. Write a boolean expression that checks: `x > 0 && x < 100`. Print the result for `x = 50`.
10. Write a boolean expression that checks: `x < 0 || x > 100`. Print the result for `x = 150`.
11. Declare `int a = 5, b = 3`. Compute and print `a & b`, `a | b`, `a ^ b`.
12. Declare `int x = 1`. Compute and print `x << 3` (left shift by 3).
13. Declare `int x = 40`. Compute and print `x >> 2` (right shift by 2).
14. Print the result of `~5` (bitwise NOT).
15. Without running the code, predict the output of: `int a = 5; int b = a++ + ++a;`. Then write the code, print `a` and `b`, and verify.
16. Use the ternary operator nested: if `x > 0` print `"positive"`, else if `x == 0` print `"zero"`, else `"negative"`. Test with `x = -3`.
17. Write an expression that evaluates `(10 + 5) * 2 - 3 / 1` and print the result.
18. Declare `int x = 7`. Use `x %= 3` and print the result.
19. Write a boolean expression: `!(a > b && c < d)` with concrete values and print the result.
20. Compute `10 / 3` (integer division) and `10.0 / 3` (double division) and print both results.

---

## 4. Input and Output

### What to practice

* `System.out.print`, `System.out.println`, `System.out.printf`
* `Scanner` for various types
* `BufferedReader` + `InputStreamReader`
* Parsing: `Integer.parseInt`, `Long.parseLong`, `Double.parseDouble`
* `StringTokenizer`

### Imports needed

```java
import java.util.Scanner;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.IOException;
import java.util.StringTokenizer;
```

### Practice

1. Print `Hello` using `System.out.print` and `World` using `System.out.println` on the same line.
2. Print your name and age on separate lines using `System.out.println`.
3. Use `System.out.printf` to print a formatted string: `"Name: %s, Age: %d%n"`.
4. Use `System.out.printf` to print a `double` with exactly 2 decimal places.
5. Use `System.out.printf` to print an `int` padded to 10 characters width.
6. Create a `Scanner` object, read an `int` from the user, and print it.
7. Use `Scanner` to read a `double` from the user and print it.
8. Use `Scanner` to read a full line of text using `nextLine()` and print it.
9. Use `Scanner` to read a single word using `next()` and print it.
10. Use `Scanner` to read two integers on the same line and print their sum.
11. Use `Scanner` to read a `String` and an `int` on separate lines, then print them together.
12. Create a `BufferedReader` using `new BufferedReader(new InputStreamReader(System.in))`. Read a line and print it.
13. Use `BufferedReader` to read a line, then use `Integer.parseInt` to convert it to an `int` and print it.
14. Use `BufferedReader` to read a line, then use `Long.parseLong` to convert it to a `long` and print it.
15. Use `BufferedReader` to read a line, then use `Double.parseDouble` to convert it to a `double` and print it.
16. Use `BufferedReader` to read a line containing two space-separated integers. Use `StringTokenizer` to parse them. Print their sum.
17. Use `StringTokenizer` to split the string `"apple,banana,cherry"` using `","` as delimiter. Print each token.
18. Use `Scanner` to read 3 integers in a loop and print their sum.
19. Use `System.out.printf` to print a table row: left-align a name in 15 chars and right-align a number in 10 chars.
20. Close the `Scanner` object after use. Write the complete program showing proper resource management.

---

## 5. if / else / switch

### What to practice

* `if`, `if/else`, `else if`
* Nested `if`
* `switch`, `case`, `default`, `break`
* `return` inside conditions

### Practice

1. Write an `if` statement that prints `"Positive"` if a number is greater than 0.
2. Write an `if/else` that prints `"Even"` or `"Odd"` based on an integer.
3. Write an `if/else if/else` chain that prints `"A"`, `"B"`, `"C"`, or `"F"` based on a score (90+, 80+, 70+, else).
4. Write a nested `if`: if a number is positive, check if it's even or odd. Print accordingly.
5. Write a `switch` statement on an `int` variable `day` (1–7) that prints the day name. Include `default`.
6. Write a `switch` statement on a `char` variable for vowels (`a, e, i, o, u`). Print `"Vowel"` or `"Consonant"`.
7. Write a `switch` statement on a `String` variable with values `"start"`, `"stop"`, `"pause"`. Print a message for each.
8. Write a `switch` without `break` statements and observe fall-through behavior. Print messages in each case.
9. Write a method that takes an `int` and returns `"Positive"`, `"Negative"`, or `"Zero"` using `if/else if/else` with `return`.
10. Write a nested `if` that checks: if age >= 18, then check if age >= 65. Print `"Senior"`, `"Adult"`, or `"Minor"`.
11. Write an `if` statement that checks two conditions with `&&`: temperature > 30 AND humidity > 80. Print `"Very hot and humid"`.
12. Write an `if/else` using `||`: if day is `"Saturday"` or `"Sunday"`, print `"Weekend"`, else `"Weekday"`.
13. Write a `switch` on an `int` month (1–12) that prints the number of days in that month (ignore leap year).
14. Write a method that takes a `char` and uses `if/else` to check if it's uppercase, lowercase, digit, or special character.
15. Write a `switch` with multiple case labels falling to the same block: cases 1, 3, 5, 7, 8, 10, 12 print `"31 days"`.
16. Write an `if` that checks if a `String` variable is not `null` and not empty before printing it.
17. Write a nested `if`: check if a number is between 1 and 100, then check if it's divisible by both 3 and 5.
18. Write a `switch` on an enum value (define a simple enum `Season` with 4 values, switch on it).
19. Write a method that takes two `int` values and a `char` operator (`+`, `-`, `*`, `/`). Use `switch` to perform the operation and return the result.
20. Write an `if/else if` chain that classifies a BMI value: underweight (<18.5), normal (18.5–24.9), overweight (25–29.9), obese (30+).

---

## 6. Loops

### What to practice

* `for` loop
* `while` loop
* `do-while` loop
* Enhanced `for` loop
* Nested loops
* `break` and `continue`

### Practice

1. Write a `for` loop that prints numbers 1 to 10.
2. Write a `for` loop that prints numbers 10 down to 1.
3. Write a `for` loop that prints even numbers from 2 to 20.
4. Write a `while` loop that prints numbers 1 to 10.
5. Write a `while` loop that starts at 100 and halves the value each iteration, printing it, until it reaches 0.
6. Write a `do-while` loop that prints numbers 1 to 5.
7. Write a `do-while` loop that executes at least once even when the condition is false from the start.
8. Write a `for` loop that prints the characters of a `String` one per line using `charAt`.
9. Write an enhanced `for` loop over an `int` array `{10, 20, 30, 40, 50}` and print each element.
10. Write a `for` loop with `break`: print numbers 1 to 100 but stop when you reach 15.
11. Write a `for` loop with `continue`: print numbers 1 to 20 but skip multiples of 3.
12. Write nested `for` loops that print a 5×5 grid of `*` characters.
13. Write nested `for` loops that print a right triangle of `*` with 5 rows.
14. Write a `for` loop that iterates over a `String[]` array and prints each element with its index.
15. Write a `while` loop that reads integers from `Scanner` until the user enters `-1`, then print how many numbers were entered.
16. Write a `for` loop that prints the multiplication table for 7 (7×1=7 through 7×10=70).
17. Write a `do-while` loop that keeps asking the user to enter a positive number. Exit when they do.
18. Write nested loops: outer loop 1 to 3, inner loop 1 to 3. Print pairs like `(1,1) (1,2) (1,3)...`.
19. Write a `for` loop that sums the numbers 1 to 100 and prints the sum.
20. Write a `while` loop with a `boolean` flag variable. Set the flag to `false` to exit the loop.
21. Write a `for` loop that iterates backward through a `String` and prints each character.
22. Write a `for` loop using `break` with a label to exit an outer loop from inside a nested loop.
23. Write a `for` loop that prints every third number from 0 to 30.
24. Write a `while(true)` infinite loop with a `break` condition inside.
25. Write an enhanced `for` loop over a `String[]` array. Use `continue` to skip any string that equals `"skip"`.

---

## 7. Methods

### What to practice

* Method declaration, parameters, return types
* `void` methods and `return`
* `static` methods and instance methods
* Method calls
* Method overloading
* Variable scope
* Varargs (`...`)

### Practice

1. Write a `static` method `sayHello()` that prints `"Hello"`. Call it from `main`.
2. Write a `static` method `add(int a, int b)` that returns the sum. Call it and print the result.
3. Write a `static void` method `printLine(String text)` that prints the given text. Call it.
4. Write a `static` method `max(int a, int b)` that returns the larger value.
5. Write a `static` method `isEven(int n)` that returns `true` if `n` is even, `false` otherwise.
6. Write a `static` method `greet(String name)` that returns `"Hello, " + name`. Print the result.
7. Write two overloaded methods: `multiply(int, int)` and `multiply(double, double)`. Call both.
8. Write three overloaded methods: `print(int)`, `print(String)`, `print(boolean)`. Call all three.
9. Write a `static` method that takes an `int` array and returns the length of the array.
10. Write a `void` method that takes a `String` and an `int` count, and prints the string `count` times.
11. Write a method that takes no parameters and returns the current time as a `String` (use `new java.util.Date().toString()`).
12. Write a `static` method that takes three `int` parameters and returns the middle value (not min, not max).
13. Write an instance method: create a class with a non-static method `introduce()` that prints a message. Create an object and call it.
14. Write a `static` method with a local variable. Try to access that variable from `main` — observe the compile error (then fix by removing the access).
15. Write a method `power(int base, int exp)` that computes `base` raised to `exp` using a loop (not `Math.pow`).
16. Write a method `concat(String a, String b)` that returns `a + " " + b`.
17. Write a varargs method `sum(int... numbers)` that returns the sum of all arguments. Call it with 2, 3, and 5 arguments.
18. Write a varargs method `printAll(String... words)` that prints each word on a new line.
19. Write a method that takes a `char` and returns `true` if it is a vowel.
20. Write a method that takes an `int[]` array and an `int` value, and returns `true` if the value exists in the array (use a simple loop).
21. Write two overloaded methods: `format(String name)` returns `"Name: name"`, and `format(String name, int age)` returns `"Name: name, Age: age"`.
22. Write a `static` method and an instance method in the same class. Call the static method from `main` directly, and call the instance method through an object.
23. Write a method that takes a `String` and returns the same string reversed (using a `for` loop and `charAt`).
24. Write a method that takes two `String` parameters and returns the longer one.
25. Write a method that takes an `int` and returns a `String`: `"small"` for < 10, `"medium"` for 10–99, `"large"` for >= 100.

---

## 8. Classes and Objects

### What to practice

* Class declaration, fields, methods
* Object creation with `new`
* Object references
* Constructors and constructor overloading
* `this` keyword

### Practice

1. Create a class `Dog` with fields `String name` and `int age`. Create an object in `main` and set the fields directly.
2. Create a class `Car` with fields `String brand` and `int speed`. Print the fields of an object.
3. Create a class `Book` with a method `display()` that prints the book's title and author fields. Create an object and call the method.
4. Create a class `Circle` with a field `double radius` and a method `area()` that returns `Math.PI * radius * radius`.
5. Create a class `Counter` with an `int count` field and methods `increment()`, `decrement()`, and `getCount()`.
6. Write a class `Person` with a constructor that takes `String name` and `int age`.
7. Write a class `Rectangle` with a constructor that takes `width` and `height`, and a method `getArea()`.
8. Write a class `Student` with two constructors: one that takes only `name`, and one that takes `name` and `grade`.
9. Write a class where the constructor uses `this.name = name` to distinguish between the parameter and the field.
10. Write a class with three constructors (no-arg, one-arg, two-arg). Create an object with each constructor.
11. Create two objects of the same class. Assign one reference to another. Change a field through one reference and print through the other.
12. Create an object and assign it to a variable. Assign `null` to that variable. Print the variable.
13. Create a class `BankAccount` with fields `String owner` and `double balance`. Add a method `deposit(double amount)` and `getBalance()`.
14. Create a class `Point` with fields `int x` and `int y`. Add a method `distanceTo(Point other)` that returns the distance between two points.
15. Write a class that has a `toString()` method returning a custom string. Create an object and print it with `System.out.println`.
16. Create a class `Color` with `int r, g, b` fields and a constructor. Create 3 different color objects.
17. Write a class with a constructor that calls another constructor using `this(...)`.
18. Create a class `Employee` with fields and a method `raiseSalary(double percent)` that modifies the salary field.
19. Create an array of 3 `Dog` objects. Set fields for each and print them in a loop.
20. Write a class `Pair` with two `int` fields. Add a method `swap()` that swaps the two fields. Print before and after.
21. Create a class with a `static` field `int count` that increments in the constructor. Print the count after creating 5 objects.
22. Write a class with both `static` and instance methods. Demonstrate calling each appropriately.
23. Create a class `Temperature` with a `double celsius` field. Add methods `toFahrenheit()` and `toKelvin()`.
24. Create two classes: `Engine` and `Car`. `Car` has an `Engine` field. Create a `Car` with an `Engine` object.
25. Write a class with a `final` field that is set in the constructor and cannot be changed afterward.

---

## 9. Encapsulation

### What to practice

* `private` fields
* `public` getters and setters
* Validation in setters
* Constructor initialization
* Controlled access to data

### Practice

1. Create a class `User` with a `private String name`. Add a `public` getter and setter. Use them from `main`.
2. Create a class `Product` with `private String name` and `private double price`. Add getters and setters for both.
3. Write a setter for `age` that only allows values between 0 and 150. If invalid, print an error and don't set the value.
4. Write a setter for `email` that only allows values containing `"@"`.
5. Create a class where all fields are `private`. The only way to set them is through the constructor.
6. Create a class `BankAccount` with a `private double balance`. Add `deposit` and `withdraw` methods (no direct access to balance).
7. Write a `withdraw` method that checks if the amount is valid and the balance is sufficient before withdrawing.
8. Create a class with `private` fields and a `public toString()` method that returns a formatted string of the fields.
9. Create a class `Student` with `private` name and grades. Add a method `getAverage()` that computes and returns the average grade.
10. Create a class with a `private` field and a getter that returns a copy of the data (e.g., return a new array instead of the reference).
11. Write a class `Config` where the setter for `maxRetries` only accepts values 1–10.
12. Write a class where the constructor validates all parameters and throws `IllegalArgumentException` for invalid input.
13. Create a class with a `private` list field. Add methods `addItem()`, `getItem(int index)`, and `getSize()` — no direct access to the list.
14. Create a class `Clock` with `private int hours` and `private int minutes`. Setters should enforce valid ranges.
15. Write a class that has `private` fields, a constructor, getters, setters with validation, and a `toString` method.

---

## 10. Inheritance

### What to practice

* `extends`
* Parent/child relationship
* `super` keyword
* Constructor chaining
* Method overriding

### Practice

1. Create a class `Animal` with a field `String name`. Create a class `Dog` that extends `Animal`. In `main`, create a `Dog` and set `name`.
2. Create a parent class `Vehicle` with a method `start()`. Create a child class `Car` that inherits `start()`. Call it from a `Car` object.
3. Override a method: `Animal` has `speak()` that prints `"..."`. `Dog` overrides it to print `"Woof"`.
4. Use `@Override` annotation when overriding a method.
5. Create a parent class with a constructor that takes `String name`. Create a child class whose constructor calls `super(name)`.
6. Create a parent class `Shape` with a method `area()` returning `0`. Create `Circle` extending `Shape` and override `area()`.
7. Create a chain: `Animal` → `Dog` → `Puppy`. Each class adds one field. Create a `Puppy` and access all inherited fields.
8. Write a child class constructor that calls `super(...)` and also initializes its own fields.
9. Write a child class method that calls `super.methodName()` to invoke the parent version before adding its own behavior.
10. Create `Person` with `name` and `age`. Create `Student` extending `Person` with an additional `grade` field.
11. Create `Employee` extending `Person`. Override `toString()` in both `Person` and `Employee`.
12. Create a class hierarchy: `Shape` → `Rectangle` → `Square`. Each constructor calls the parent constructor.
13. Write a child class that overrides a method and also adds a new method not in the parent.
14. Create `Animal` with `eat()` and `sleep()`. Create `Cat` and `Dog` that both extend `Animal` and override `eat()`.
15. Create parent and child classes where the child constructor takes more parameters than the parent, calling `super` for the parent's parameters.
16. Write a class where the child overrides `toString()` and includes `super.toString()` in the returned string.
17. Create a 3-level hierarchy. In the bottom class's constructor, verify that all parent constructors are called (add print statements in each constructor).
18. Create `Vehicle` with `speed`. Create `Car` extending it with `numDoors`. Create objects of both and print their fields.
19. Override `equals()` in a child class. Compare two child objects.
20. Write a parent class with a `final` method. Verify that the child class cannot override it (write the code, observe the error, then comment out the override).

---

## 11. Polymorphism

### What to practice

* Parent reference holding child object
* Upcasting
* Method overriding with polymorphism
* `instanceof`
* Downcasting

### Practice

1. Create a `Animal` reference that holds a `Dog` object. Call an overridden method.
2. Create an `Animal[]` array containing a `Dog`, `Cat`, and `Bird` object. Loop through and call `speak()` on each.
3. Write a method that takes an `Animal` parameter. Pass a `Dog` and a `Cat` to it.
4. Use `instanceof` to check if an `Animal` reference is actually a `Dog`.
5. Downcast an `Animal` reference to `Dog` after checking with `instanceof`. Call a `Dog`-specific method.
6. Create a `Shape` reference holding a `Circle`. Call `area()`.
7. Create an `ArrayList<Animal>` containing different animal types. Iterate and call `speak()` on each.
8. Write a method `describeAnimal(Animal a)` that prints the animal's type using `instanceof` checks.
9. Write a parent class `Vehicle` with `move()`. Create `Car` and `Bike` subclasses. Use a `Vehicle` reference to call `move()` on each.
10. Create a `Shape` array with `Circle`, `Rectangle`, `Triangle`. Call `area()` on each using polymorphism.
11. Attempt an invalid downcast (cast `Cat` to `Dog`) inside a try-catch and catch `ClassCastException`.
12. Write a method that returns an `Animal` but actually returns a `Dog`. In the caller, use `instanceof` and downcast.
13. Create a class hierarchy with 3 levels. Store the bottom-level object in a top-level reference. Call an overridden method.
14. Write a program with a `Printable` method in a parent class. Override it differently in 3 child classes. Use a parent array to print all.
15. Create an `Object` reference holding a `String`. Downcast it back to `String` and call `length()`.

---

## 12. Abstract Classes

### What to practice

* `abstract class` declaration
* Abstract methods and concrete methods
* Constructors in abstract classes
* Extending and implementing abstract methods

### Practice

1. Declare an `abstract class Shape` with an `abstract double area()` method.
2. Create a `Circle` class extending `Shape` that implements `area()`.
3. Create a `Rectangle` class extending `Shape` that implements `area()`.
4. Try to instantiate `Shape` directly — observe the error, then comment it out.
5. Add a concrete method `describe()` in the abstract class `Shape` that prints `"I am a shape"`. Call it from a child object.
6. Add a constructor to the abstract class `Shape` that takes `String color`. Call it from child constructors with `super`.
7. Create an abstract class `Animal` with an abstract method `speak()` and a concrete method `breathe()`.
8. Create `Dog` and `Cat` extending `Animal`, each implementing `speak()` differently.
9. Create a `Shape[]` array with `Circle` and `Rectangle` objects. Loop through and call `area()`.
10. Create an abstract class with two abstract methods. Implement both in a child class.
11. Create an abstract class `Vehicle` with abstract `fuelType()` and concrete `start()`. Extend it with `ElectricCar`.
12. Write an abstract class with a `final` concrete method that child classes cannot override.
13. Create a two-level abstract hierarchy: `Shape` (abstract) → `Polygon` (abstract) → `Triangle` (concrete).
14. Create an abstract class with a field, a constructor that sets it, and an abstract method. Extend it.
15. Write an abstract class `Converter` with an abstract method `convert(double value)`. Create `KmToMiles` and `CelsiusToFahrenheit` subclasses.

---

## 13. Interfaces

### What to practice

* Interface declaration
* `implements`
* Implementing multiple interfaces
* Default methods
* Interface references

### Practice

1. Declare an interface `Printable` with a method `void print()`.
2. Create a class `Document` that implements `Printable`. Implement the `print()` method.
3. Create a `Printable` reference holding a `Document` object. Call `print()`.
4. Create an interface `Drawable` with `void draw()`. Create a class that implements both `Printable` and `Drawable`.
5. Create an interface with two method declarations. Implement both in a class.
6. Create an interface `Calculable` with a default method `double square(double x)` that returns `x * x`.
7. Create a class that implements an interface with a default method. Call the default method without overriding it.
8. Override a default method in an implementing class.
9. Create a `Comparable`-like interface `Rankable` with `int rank()`. Implement it in two classes.
10. Create an interface `Movable` with `void moveUp()`, `void moveDown()`. Implement in a `Player` class.
11. Write a method that takes an interface type as a parameter. Pass different implementing objects.
12. Create an array of interface type references holding different implementing objects. Loop and call the interface method.
13. Create an interface with a `static` method. Call it using the interface name.
14. Create a class that implements 3 different interfaces.
15. Create two interfaces with default methods of the same name. Implement both in a class and resolve the conflict by overriding.

---

## 14. Arrays — Core Java Syntax Only

### What to practice

* Declaration, initialization, `new`, indexing, `length`
* Modifying and reading elements
* `for` and enhanced `for` loops
* 2D arrays, arrays of objects
* `Arrays` utility methods

### Imports needed

```java
import java.util.Arrays;
```

### Practice

1. Declare an `int` array of size 5 using `new int[5]`.
2. Declare and initialize an `int` array: `{10, 20, 30, 40, 50}`.
3. Print the first and last element of an `int` array.
4. Print the `length` of an array.
5. Change the third element of an `int` array to `99` and print the array.
6. Iterate over an `int` array using a normal `for` loop and print each element.
7. Iterate over a `String` array using an enhanced `for` loop and print each element.
8. Declare a `double[]` array of size 3, assign values using index, and print each value.
9. Declare a `char[]` array with `{'J', 'a', 'v', 'a'}` and print it using `System.out.println(charArray)`.
10. Declare a `boolean[]` array of size 4 and print the default values.
11. Use `Arrays.toString()` to print an `int` array.
12. Use `Arrays.sort()` to sort an `int` array and print it.
13. Use `Arrays.fill()` to fill an `int` array of size 5 with the value `7`. Print it.
14. Use `Arrays.copyOf()` to copy an array into a new array of size 8. Print the new array.
15. Use `Arrays.copyOfRange()` to copy elements at indices 1 to 3 from an array. Print the result.
16. Use `Arrays.equals()` to compare two `int` arrays and print the result.
17. Declare a 2D `int` array of size 3×3.
18. Initialize a 2D array: `{{1,2,3}, {4,5,6}, {7,8,9}}`.
19. Print all elements of a 2D array using nested `for` loops.
20. Print a 2D array using `Arrays.deepToString()`.
21. Declare a `String[]` array with 4 names. Sort it and print.
22. Create an array of `int` with values `{5, 3, 8, 1, 9}`. Sort it. Print the sorted array.
23. Create a 2D array where each row has a different number of columns (jagged array). Print it.
24. Create an array of `Dog` objects (define a simple `Dog` class). Set fields for each and print.
25. Copy an `int` array into a new array manually using a `for` loop.
26. Declare a `String[]` array. Set one element to `null`. Loop through the array and print only non-null elements.
27. Create a 2D `int` array. Set values in a nested loop (e.g., `arr[i][j] = i * 3 + j`). Print it.
28. Use `Arrays.fill()` on a portion of an array: fill indices 1 to 3 with value `0`.
29. Create two `int` arrays with the same values. Use `Arrays.equals()` to check equality. Then change one element and check again.
30. Create a `char[]` from a `String` using `toCharArray()`. Modify one character. Create a new `String` from the `char[]` and print it.

---

## 15. Strings — Core Java Syntax Only

### What to practice

* String creation and common methods
* `length`, `charAt`, `substring`, `equals`, `contains`
* `indexOf`, `replace`, `split`, `trim`, `strip`
* Case conversion, `toCharArray`, `String.valueOf`

### Practice

1. Create a `String` literal `"Hello"` and print it.
2. Create a `String` using `new String("Hello")` and print it.
3. Print the length of a `String`.
4. Print the character at index 0 and the last index of a `String`.
5. Extract a substring from index 2 to 5 and print it.
6. Extract a substring from index 3 to the end and print it.
7. Compare two strings using `equals()` and print the result.
8. Compare two strings using `equalsIgnoreCase()` and print the result.
9. Check if a string contains `"Java"` using `contains()`.
10. Check if a string starts with `"Hello"` using `startsWith()`.
11. Check if a string ends with `"World"` using `endsWith()`.
12. Find the index of `"is"` in `"This is Java"` using `indexOf()`.
13. Find the last index of `'a'` in `"banana"` using `lastIndexOf()`.
14. Replace all occurrences of `"cat"` with `"dog"` in a string using `replace()`.
15. Replace all digits in a string with `"#"` using `replaceAll()` and a regex.
16. Split `"apple,banana,cherry"` by `","` and print each part.
17. Split `"one two three"` by spaces and print each part.
18. Trim whitespace from `"   Hello World   "` using `trim()` and print the result.
19. Strip whitespace from a string using `strip()` and print the result.
20. Convert a string to uppercase using `toUpperCase()` and print it.
21. Convert a string to lowercase using `toLowerCase()` and print it.
22. Convert a string to a `char[]` using `toCharArray()`. Print each character.
23. Convert an `int` to a `String` using `String.valueOf()`.
24. Convert a `double` to a `String` using `String.valueOf()`.
25. Concatenate two strings using `+` and print the result.
26. Concatenate two strings using `concat()` and print the result.
27. Compare two `String` references using `==` and `.equals()`. Explain the difference to yourself through the code.
28. Create an empty string `""`. Check if it is empty using `isEmpty()`.
29. Create a string. Check if it is blank (only whitespace) using `isBlank()`.
30. Chain multiple `String` methods: take a string, trim it, convert to lowercase, then replace a word, and print the result.

---

## 16. StringBuilder

### What to practice

* Creating `StringBuilder`
* `append`, `insert`, `delete`, `deleteCharAt`
* `setCharAt`, `charAt`, `length`
* `reverse`, `toString`

### Practice

1. Create a `StringBuilder` with initial value `"Hello"`. Print it.
2. Create an empty `StringBuilder`. Append `"Java"` to it. Print it.
3. Append multiple values: a string, an int, a char, a double. Print the result.
4. Insert `"Beautiful "` at index 6 in `StringBuilder("Hello World")`. Print it.
5. Delete characters from index 5 to 11 in a `StringBuilder`. Print it.
6. Delete the character at index 0 using `deleteCharAt()`. Print it.
7. Set the character at index 0 to `'J'` using `setCharAt()`. Print it.
8. Get the character at index 3 using `charAt()`. Print it.
9. Print the `length()` of a `StringBuilder`.
10. Reverse a `StringBuilder` and print it.
11. Convert a `StringBuilder` to a `String` using `toString()`. Print the `String`.
12. Build a comma-separated string from an array using `StringBuilder` and `append`.
13. Use `StringBuilder` to build a string by appending numbers 1 to 10 separated by spaces.
14. Create a `StringBuilder`, append `"abcdef"`, then replace characters at index 2–4 with `"XY"` using `replace()`. Print it.
15. Create a `StringBuilder` with `"Hello World"`. Find the index of `"World"` using `indexOf()`.
16. Chain multiple `StringBuilder` operations: `append`, `insert`, `reverse`.
17. Build a `StringBuilder` in a loop: append `"Line i\n"` for i from 1 to 5. Print it.
18. Create a `StringBuilder` from a `String`. Modify it. Show that the original `String` is unchanged.
19. Compare performance concept: concatenate 1000 strings using `+` in a loop, then do the same with `StringBuilder`. Print both results.
20. Use `StringBuilder` to build a formatted table row: left-pad a name and right-pad a number.

---

## 17. Wrapper Classes

### What to practice

* `Integer`, `Long`, `Double`, `Float`, `Boolean`, `Character`
* Autoboxing and unboxing
* Parsing strings to numbers
* `valueOf`

### Practice

1. Create an `Integer` object from an `int` value (autoboxing). Print it.
2. Assign an `Integer` to an `int` variable (unboxing). Print it.
3. Use `Integer.parseInt("123")` to convert a `String` to `int`. Print it.
4. Use `Long.parseLong("999999999999")` to convert a `String` to `long`. Print it.
5. Use `Double.parseDouble("3.14")` to convert a `String` to `double`. Print it.
6. Use `Float.parseFloat("2.5")` to convert a `String` to `float`. Print it.
7. Use `Integer.valueOf("42")` to get an `Integer` object. Print it.
8. Use `Integer.MAX_VALUE` and `Integer.MIN_VALUE`. Print both.
9. Use `Double.MAX_VALUE` and `Double.MIN_VALUE`. Print both.
10. Use `Character.isLetter('A')`, `Character.isDigit('5')`, `Character.isWhitespace(' ')`. Print each result.
11. Use `Character.toUpperCase('a')` and `Character.toLowerCase('Z')`. Print each.
12. Use `Boolean.parseBoolean("true")` and `Boolean.parseBoolean("false")`. Print both.
13. Create an `ArrayList<Integer>`. Add `int` values (autoboxing). Retrieve values as `int` (unboxing).
14. Convert an `int` to a `String` using `Integer.toString(42)`. Print it.
15. Compare two `Integer` objects using `==` and `.equals()`. Test with values both inside and outside the cached range (−128 to 127).

---

## 18. ArrayList

### What to practice

* Declaration, initialization
* `add`, `get`, `set`, `remove`
* `contains`, `size`, `isEmpty`, `clear`
* Iteration: `for`, enhanced `for`, `Iterator`
* `Collections.sort`

### Imports needed

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.Iterator;
```

### Practice

1. Create an `ArrayList<String>` and add three names to it. Print the list.
2. Create an `ArrayList<Integer>` and add five numbers. Print the list.
3. Get the element at index 2 from an `ArrayList` and print it.
4. Set the element at index 1 to a new value. Print the list.
5. Remove the element at index 0. Print the list.
6. Remove a specific element by value (e.g., remove `"Alice"`). Print the list.
7. Check if an `ArrayList` contains a specific value using `contains()`.
8. Print the `size()` of an `ArrayList`.
9. Check if an `ArrayList` is empty using `isEmpty()`.
10. Clear an `ArrayList` using `clear()`. Print its size.
11. Iterate over an `ArrayList<String>` using a normal `for` loop with `get()`.
12. Iterate over an `ArrayList<Integer>` using an enhanced `for` loop.
13. Iterate over an `ArrayList<String>` using an `Iterator`.
14. Sort an `ArrayList<Integer>` using `Collections.sort()`. Print the sorted list.
15. Sort an `ArrayList<String>` using `Collections.sort()`. Print the sorted list.
16. Create an `ArrayList<String>`, add 5 items, then remove items in a loop using `Iterator` and `remove()`.
17. Add elements at a specific index using `add(index, value)`. Print the list after each insertion.
18. Create an `ArrayList<Double>`. Add values. Compute and print the sum using a loop.
19. Convert an `ArrayList<String>` to a `String[]` array using `toArray()`.
20. Create an `ArrayList<Integer>`, add values, then use `Collections.reverse()` to reverse it. Print the result.
21. Use `Collections.min()` and `Collections.max()` on an `ArrayList<Integer>`. Print both.
22. Create an `ArrayList` and add duplicate values. Use `indexOf()` and `lastIndexOf()` to find positions.
23. Use `Collections.frequency()` to count how many times a value appears in an `ArrayList`.
24. Create a copy of an `ArrayList` using `new ArrayList<>(original)`. Modify the copy and show the original is unchanged.
25. Use `subList()` to get a portion of an `ArrayList`. Print it.

---

## 19. LinkedList

### What to practice

* Declaration, initialization
* `add`, `addFirst`, `addLast`, `get`
* `remove`, `removeFirst`, `removeLast`
* `size`, iteration

### Imports needed

```java
import java.util.LinkedList;
```

### Practice

1. Create a `LinkedList<String>`. Add three elements. Print the list.
2. Add an element at the beginning using `addFirst()`. Print the list.
3. Add an element at the end using `addLast()`. Print the list.
4. Get the element at index 1 using `get()`. Print it.
5. Remove the first element using `removeFirst()`. Print the list.
6. Remove the last element using `removeLast()`. Print the list.
7. Remove an element by value using `remove("value")`. Print the list.
8. Print the `size()` of the `LinkedList`.
9. Check if the `LinkedList` contains a value using `contains()`.
10. Iterate over the `LinkedList` using an enhanced `for` loop. Print each element.
11. Iterate using a `for` loop with `get(i)`. Print each element.
12. Get the first element using `getFirst()` and the last using `getLast()`. Print both.
13. Use `peek()` and `poll()` on a `LinkedList` (Queue behavior). Print results.
14. Clear the `LinkedList` and print its size.
15. Create a `LinkedList<Integer>`, add values, sort it using `Collections.sort()`, and print.

---

## 20. HashSet

### What to practice

* Declaration, initialization
* `add`, `remove`, `contains`
* `size`, `isEmpty`, `clear`
* Iteration

### Imports needed

```java
import java.util.HashSet;
```

### Practice

1. Create a `HashSet<String>`. Add three values. Print the set.
2. Add a duplicate value to the `HashSet`. Print the set and observe the size.
3. Check if the set contains a value using `contains()`.
4. Remove a value from the set using `remove()`. Print the set.
5. Print the `size()` of the set.
6. Check if the set is empty using `isEmpty()`.
7. Clear the set using `clear()`. Print the size.
8. Iterate over a `HashSet<Integer>` using an enhanced `for` loop.
9. Iterate over a `HashSet<String>` using an `Iterator`.
10. Create a `HashSet<Integer>` and add values 1 to 10 using a loop. Print the set.
11. Create two `HashSet<String>` sets. Add some overlapping values. Print both.
12. Use `addAll()` to compute the union of two sets. Print the result.
13. Use `retainAll()` to compute the intersection of two sets. Print the result.
14. Use `removeAll()` to compute the difference of two sets. Print the result.
15. Create a `HashSet` of custom objects. Override `hashCode()` and `equals()` in the class. Add objects and check `contains()`.

---

## 21. TreeSet

### What to practice

* Declaration, initialization
* `add`, `remove`, `contains`
* `first`, `last`, `higher`, `lower`, `ceiling`, `floor`
* Iteration, custom `Comparator`

### Imports needed

```java
import java.util.TreeSet;
import java.util.Comparator;
```

### Practice

1. Create a `TreeSet<Integer>`. Add values `{30, 10, 50, 20, 40}`. Print the set (observe sorted order).
2. Add a duplicate value to the `TreeSet`. Print the set and observe.
3. Remove a value from the `TreeSet`. Print the set.
4. Check if the `TreeSet` contains a value using `contains()`.
5. Print the first (smallest) element using `first()`.
6. Print the last (largest) element using `last()`.
7. Use `higher(25)` to find the smallest element strictly greater than 25. Print it.
8. Use `lower(25)` to find the largest element strictly less than 25. Print it.
9. Use `ceiling(25)` to find the smallest element >= 25. Print it.
10. Use `floor(25)` to find the largest element <= 25. Print it.
11. Iterate over a `TreeSet<String>` using an enhanced `for` loop.
12. Create a `TreeSet<String>`. Add names. Print them (observe alphabetical order).
13. Create a `TreeSet<Integer>` with a custom `Comparator` for descending order. Add values and print.
14. Create a `TreeSet<String>` with a case-insensitive `Comparator`. Add `"Apple"`, `"apple"`, `"APPLE"`. Print the set.
15. Use `headSet(30)`, `tailSet(30)`, and `subSet(20, 40)` on a `TreeSet<Integer>`. Print each.

---

## 22. HashMap

### What to practice

* Declaration, initialization
* `put`, `get`, `remove`, `containsKey`, `containsValue`
* `size`, `isEmpty`, `clear`
* `keySet`, `values`, `entrySet`, `Map.Entry`
* `getOrDefault`, `putIfAbsent`, `merge`

### Imports needed

```java
import java.util.HashMap;
import java.util.Map;
```

### Practice

1. Create a `HashMap<String, Integer>`. Put three key-value pairs. Print the map.
2. Get a value by key using `get()`. Print it.
3. Get a value for a key that doesn't exist using `get()`. Print the result (observe `null`).
4. Use `getOrDefault("key", defaultValue)` for a missing key. Print the result.
5. Remove a key-value pair using `remove("key")`. Print the map.
6. Check if a key exists using `containsKey()`. Print the result.
7. Check if a value exists using `containsValue()`. Print the result.
8. Print the `size()` of the map.
9. Check if the map is empty using `isEmpty()`.
10. Clear the map using `clear()`. Print the size.
11. Iterate over keys using `keySet()`. Print each key.
12. Iterate over values using `values()`. Print each value.
13. Iterate over entries using `entrySet()` and `Map.Entry`. Print each key-value pair.
14. Use `putIfAbsent("key", value)`. Put a key that already exists and one that doesn't. Print the map.
15. Update a value: get the current value, modify it, and put it back.
16. Use `replace("key", newValue)` to update a value. Print the map.
17. Use `merge("key", value, (oldVal, newVal) -> oldVal + newVal)`. Print the result.
18. Create a `HashMap<Integer, String>`. Put 5 entries. Print all entries in `key = value` format.
19. Create a `HashMap<String, String>` for a phone book. Add names and numbers. Look up a name and print the number.
20. Remove all entries from a `HashMap` where the value meets a condition (use `entrySet().removeIf()`).
21. Create a `HashMap`. Iterate over it and build a `String` with all key-value pairs.
22. Check the return value of `put()` when overwriting an existing key. Print the old value.
23. Copy one `HashMap` to another using `putAll()`. Print the new map.
24. Create a `HashMap<String, ArrayList<String>>`. Put a key with an `ArrayList` value. Add items to the list.
25. Use `forEach((key, value) -> ...)` to iterate over a `HashMap`. Print each pair.

---

## 23. TreeMap

### What to practice

* Declaration, initialization
* `put`, `get`, `remove`, `containsKey`
* `keySet`, `entrySet`
* `firstKey`, `lastKey`, `higherKey`, `lowerKey`, `ceilingKey`, `floorKey`
* Custom `Comparator`

### Imports needed

```java
import java.util.TreeMap;
import java.util.Comparator;
import java.util.Map;
```

### Practice

1. Create a `TreeMap<String, Integer>`. Put three entries. Print the map (observe sorted key order).
2. Get a value by key using `get()`. Print it.
3. Remove an entry by key. Print the map.
4. Check if a key exists using `containsKey()`.
5. Print the first key using `firstKey()`.
6. Print the last key using `lastKey()`.
7. Use `higherKey("B")` to get the next key after `"B"`. Print it.
8. Use `lowerKey("B")` to get the key before `"B"`. Print it.
9. Use `ceilingKey("B")` — the smallest key >= `"B"`. Print it.
10. Use `floorKey("B")` — the largest key <= `"B"`. Print it.
11. Iterate over keys using `keySet()`. Print each key.
12. Iterate over entries using `entrySet()`. Print each entry.
13. Create a `TreeMap<Integer, String>`. Put entries. Use `headMap(5)`, `tailMap(5)`, `subMap(2, 5)`. Print each.
14. Create a `TreeMap<String, Integer>` with a custom `Comparator` for reverse alphabetical order. Print the map.
15. Create a `TreeMap<String, String>`. Put 5 entries. Print in both ascending and descending key order.

---

## 24. Queue

### What to practice

* `Queue` declaration with `LinkedList` and `ArrayDeque`
* `offer`, `add`, `poll`, `remove`, `peek`, `element`
* `isEmpty`, `size`

### Imports needed

```java
import java.util.Queue;
import java.util.LinkedList;
import java.util.ArrayDeque;
```

### Practice

1. Create a `Queue<String>` using `LinkedList`. Add three elements using `offer()`. Print the queue.
2. Create a `Queue<Integer>` using `ArrayDeque`. Add three elements using `add()`. Print the queue.
3. Use `peek()` to look at the front element without removing it. Print it.
4. Use `poll()` to remove and return the front element. Print it. Print the queue.
5. Use `element()` to look at the front element. Print it.
6. Use `remove()` to remove the front element. Print the queue.
7. Check if the queue is empty using `isEmpty()`.
8. Print the `size()` of the queue.
9. Add 5 elements. Remove them one by one using `poll()` in a loop, printing each.
10. Add elements to a queue. Drain the queue using a `while (!queue.isEmpty())` loop.
11. Create a `Queue<String>`, offer 4 names, peek, poll 2, print remaining size.
12. Use `offer()` on a full `ArrayDeque` (it won't be full in practice, but write the code).
13. Use `poll()` on an empty queue and print the result (observe `null`).
14. Use `remove()` on an empty queue inside a try-catch. Catch `NoSuchElementException`.
15. Create a queue, add elements, convert it to an `ArrayList` using `new ArrayList<>(queue)`. Print the list.

---

## 25. Deque

### What to practice

* `ArrayDeque` as a double-ended queue
* `addFirst`, `addLast`, `offerFirst`, `offerLast`
* `removeFirst`, `removeLast`, `pollFirst`, `pollLast`
* `peekFirst`, `peekLast`

### Imports needed

```java
import java.util.Deque;
import java.util.ArrayDeque;
```

### Practice

1. Create a `Deque<String>` using `ArrayDeque`. Add elements with `addFirst()` and `addLast()`. Print.
2. Use `offerFirst()` and `offerLast()` to add elements. Print.
3. Use `peekFirst()` to see the first element. Print it.
4. Use `peekLast()` to see the last element. Print it.
5. Use `removeFirst()` to remove the first element. Print the deque.
6. Use `removeLast()` to remove the last element. Print the deque.
7. Use `pollFirst()` to remove and return the first element. Print it.
8. Use `pollLast()` to remove and return the last element. Print it.
9. Use `pollFirst()` on an empty deque. Print the result (observe `null`).
10. Create a `Deque<Integer>`. Add numbers at both ends alternately. Print the deque.
11. Use a `Deque` as a stack: `push()` 3 elements, then `pop()` them one by one. Print each.
12. Use a `Deque` as a queue: `offerLast()` to enqueue, `pollFirst()` to dequeue.
13. Add 5 elements to a deque. Print its `size()`. Remove from both ends. Print size again.
14. Iterate over a `Deque` using an enhanced `for` loop. Print each element.
15. Iterate over a `Deque` in reverse using `descendingIterator()`. Print each element.

---

## 26. PriorityQueue

### What to practice

* Declaration, default min-heap behavior
* `offer`, `add`, `peek`, `poll`
* `size`, `isEmpty`
* Max-heap using `Comparator`
* Custom `Comparator`

### Imports needed

```java
import java.util.PriorityQueue;
import java.util.Comparator;
import java.util.Collections;
```

### Practice

1. Create a `PriorityQueue<Integer>`. Add `{30, 10, 50, 20, 40}`. Print using `poll()` in a loop (observe min-heap order).
2. Use `peek()` to see the smallest element without removing it.
3. Use `poll()` to remove and return the smallest element.
4. Use `add()` and `offer()` to insert elements. Print the queue.
5. Print the `size()` of the priority queue.
6. Check if the priority queue is empty using `isEmpty()`.
7. Create a max-heap `PriorityQueue` using `Collections.reverseOrder()`. Add values and drain with `poll()`.
8. Create a max-heap using `Comparator.reverseOrder()`. Add values and drain.
9. Create a `PriorityQueue<String>`. Add names. Drain and print (observe alphabetical order).
10. Create a `PriorityQueue<String>` with a `Comparator` that sorts by string length.
11. Create a `PriorityQueue` with a custom `Comparator` using a lambda.
12. Add elements to a `PriorityQueue`. Remove a specific element using `remove(value)`.
13. Convert a `PriorityQueue` to an array using `toArray()`. Print the array (note: array order is NOT sorted).
14. Create a `PriorityQueue` of `int[]` arrays sorted by the first element using a custom `Comparator`.
15. Drain a `PriorityQueue` completely using a `while` loop, printing each element.

---

## 27. Comparable

### What to practice

* `implements Comparable<T>`
* `compareTo` method
* Natural ordering
* Sorting custom objects

### Practice

1. Create a class `Student` that implements `Comparable<Student>`. Compare by `name` alphabetically.
2. Create an `ArrayList<Student>`, add students, and sort using `Collections.sort()`. Print the sorted list.
3. Create a class `Product` that implements `Comparable<Product>`. Compare by `price`.
4. Sort an array of `Product` objects using `Arrays.sort()`.
5. Create a class `Employee` that implements `Comparable<Employee>`. Compare by `id`.
6. In `compareTo`, return a negative, zero, or positive value manually (don't use `Integer.compare` or `Double.compare` — write the logic yourself).
7. Now rewrite the `compareTo` using `Integer.compare()`.
8. Now rewrite the `compareTo` using `Double.compare()`.
9. Create a `TreeSet<Student>` (where `Student` is `Comparable`). Add students and print (observe sorted order).
10. Create a class that implements `Comparable` and is consistent with `equals()`.

---

## 28. Comparator

### What to practice

* `Comparator` interface, `compare` method
* Lambda expressions for comparators
* Anonymous comparators
* `Comparator.comparing`, `thenComparing`, `reversed`, `reverseOrder`

### Imports needed

```java
import java.util.Comparator;
import java.util.Collections;
import java.util.ArrayList;
import java.util.Arrays;
```

### Practice

1. Write a `Comparator<String>` that compares strings by length. Sort an `ArrayList<String>` with it.
2. Write the same comparator using a lambda expression.
3. Write a `Comparator<Integer>` that sorts in descending order. Sort a list.
4. Write an anonymous `Comparator` class inline when calling `Collections.sort()`.
5. Use `Comparator.comparing(Student::getName)` to sort a list of students by name.
6. Use `Comparator.comparing(Student::getAge)` to sort by age.
7. Use `Comparator.comparing(Student::getName).thenComparing(Student::getAge)` to sort by name, then by age.
8. Use `.reversed()` to reverse a comparator. Sort a list in descending order.
9. Use `Comparator.reverseOrder()` to sort a `List<String>` in reverse alphabetical order.
10. Sort an `int[][]` 2D array by the first element of each row using `Arrays.sort()` with a `Comparator`.
11. Create a `Comparator` that sorts `String` values by their last character.
12. Create a `Comparator` that sorts `null` values to the end using `Comparator.nullsLast()`.
13. Create a `Comparator` that sorts `null` values to the beginning using `Comparator.nullsFirst()`.
14. Chain three levels of sorting using `thenComparing`.
15. Write a `Comparator` for a `Map.Entry<String, Integer>` that sorts by value.
16. Sort an `ArrayList` of `String[]` arrays by the second element.
17. Sort a list of objects using `Comparator.comparingInt()`.
18. Sort a list of objects using `Comparator.comparingDouble()`.
19. Use `Comparator.naturalOrder()` explicitly.
20. Write a `Comparator` that compares two strings ignoring case, using `String.CASE_INSENSITIVE_ORDER`.

---

## 29. References and null

### What to practice

* Object references and assignment
* Multiple references to the same object
* `null` assignment and null checks
* `==` vs `.equals()`
* `NullPointerException`

### Practice

1. Create an object. Assign it to a variable. Assign that variable to another variable. Change a field through one and print through the other.
2. Create an object reference and assign `null` to it. Print the reference.
3. Try to call a method on a `null` reference inside a `try-catch`. Catch `NullPointerException`.
4. Write a null check: `if (obj != null)` before calling a method.
5. Compare two object references using `==`. Test when they point to the same object and when they don't.
6. Compare two `String` objects using `==` and `.equals()`. Show the difference.
7. Create two `new String("Hello")` objects. Show that `==` returns `false` but `.equals()` returns `true`.
8. Assign `null` to a reference. Then reassign it to a new object. Call a method.
9. Create a method that returns `null` under some condition. Call the method and handle the `null` return.
10. Pass a `null` argument to a method. Inside the method, check for `null` before proceeding.
11. Create an array of objects. Leave some elements as `null`. Loop through and print only non-null elements.
12. Create two references to the same `ArrayList`. Add an element through one reference. Print through the other.
13. Set a field of an object to `null`. Then check if the field is `null` before using it.
14. Create a method `safeLength(String s)` that returns the length if `s` is not null, or `0` if it is.
15. Create an `ArrayList<String>` and add a `null` element. Iterate and handle the `null`.
16. Compare two custom objects using `==` (reference equality) and then override `.equals()` for value equality.
17. Create a chain: `A` holds reference to `B`, `B` holds reference to `C`. Set `B`'s reference to `null` and show `A` can no longer reach `C` through `B`.
18. Show that assigning a reference to `null` does NOT destroy the object (another reference can still access it).
19. Write a method that takes an `Object` parameter. Inside, check `if (param == null)` and `if (param instanceof String)`.
20. Create a `HashMap`. Use `get()` on a missing key. Handle the `null` return gracefully.

---

## 30. Custom Classes

### What to practice

* Custom class with fields, methods, constructor
* Object creation and references
* Array of custom objects
* `ArrayList` of custom objects
* `HashMap` with custom objects

### Imports needed

```java
import java.util.ArrayList;
import java.util.HashMap;
```

### Practice

1. Create a `Student` class with `String name` and `int age`. Add a constructor and `toString()`.
2. Create 3 `Student` objects and print each.
3. Create an array of `Student` objects. Set each in a loop. Print all.
4. Create an `ArrayList<Student>`. Add 4 students. Print the list.
5. Create a `HashMap<String, Student>`. Put students with their name as key. Get a student by name and print.
6. Create a `Course` class with `String name` and `ArrayList<Student> students`. Add methods to add a student and list all students.
7. Create a `Team` class with a `String name` and an array of `Player` objects.
8. Create a `Library` class with an `ArrayList<Book>`. Add methods `addBook()` and `findByTitle(String title)`.
9. Create a `Contact` class with `name`, `phone`, `email`. Store contacts in an `ArrayList`. Print all.
10. Create a `Product` class with `name` and `price`. Create an `ArrayList<Product>`. Find and print the product with the highest price using a loop.
11. Create a `Grade` class with `String subject` and `int score`. Create an `ArrayList<Grade>`. Compute the average score.
12. Create a `City` class with `name` and `population`. Store in a `HashMap<String, City>`. Print all cities.
13. Create a `Wallet` class with a `double balance` and methods `add(double)`, `spend(double)`, `getBalance()`.
14. Create a `Playlist` class with an `ArrayList<String>` of song names. Add methods `addSong()`, `removeSong()`, `display()`.
15. Create a `Matrix` class that wraps a 2D `int` array. Add a method `display()` to print it.
16. Create a class `Pair<A, B>` with two fields of generic types. Create objects with different type combinations.
17. Create a `Registry` class that uses a `HashMap<Integer, String>` internally. Add methods `register(int id, String name)` and `lookup(int id)`.
18. Create a `Student` class with a `List<String> courses` field. Add methods to enroll in and list courses.
19. Create an `Inventory` class with a `HashMap<String, Integer>` (item name → quantity). Add methods `addStock()`, `removeStock()`, `getQuantity()`.
20. Create a `Scoreboard` class with an `ArrayList<Player>`. Add a method `getTopPlayer()` that returns the player with the highest score.

---

## 31. Node / TreeNode Syntax Only

### What to practice

* `Node` class with `data` and `next`
* `TreeNode` class with `val`, `left`, `right`
* Constructor syntax
* Object references between nodes

**STRICT: No traversal, no algorithms. Only class/reference syntax.**

### Practice

1. Create a `Node` class with `int data` and `Node next`. Add a constructor that takes `data`.
2. Create a `Node` object with data `10`. Print its `data`.
3. Create two `Node` objects. Link them: `node1.next = node2`. Print `node1.next.data`.
4. Create three nodes and chain them: `a → b → c`. Print `a.next.next.data`.
5. Create a `Node` and set its `next` to `null`. Print `node.next`.
6. Create a `TreeNode` class with `int val`, `TreeNode left`, `TreeNode right`. Add a constructor.
7. Create a `TreeNode` with value `5`. Print its `val`.
8. Create three `TreeNode` objects. Set one as root, the other two as `left` and `right` children.
9. Print `root.left.val` and `root.right.val`.
10. Create a `TreeNode` where `root.left.left` exists (a chain of 3 levels). Print the deepest node's value.
11. Create a `Node` constructor that takes both `data` and `next`. Use it to create a chain in one statement.
12. Create a `TreeNode` constructor that takes `val`, `left`, and `right`. Build a small tree in one nested constructor call.
13. Create a `Node` with a `String data` field instead of `int`. Chain two nodes.
14. Create a generic `Node<T>` class. Create a `Node<Integer>` and a `Node<String>`.
15. Check if `root.left` is `null` before accessing `root.left.val`. Write the null check.

---

## 32. Exceptions

### What to practice

* `try`, `catch`, `finally`
* `throw`, `throws`
* Custom exceptions
* Multiple catch blocks

### Practice

1. Write a `try-catch` that catches `ArithmeticException` when dividing by zero.
2. Write a `try-catch` that catches `ArrayIndexOutOfBoundsException`.
3. Write a `try-catch` that catches `NumberFormatException` when parsing `"abc"` as an integer.
4. Write a `try-catch-finally` block. Print messages in each section.
5. Write a `finally` block that executes even when no exception occurs.
6. Write a `try-catch` with multiple catch blocks for different exception types.
7. Write a method that `throws IOException`. Call it inside a `try-catch`.
8. Use `throw new IllegalArgumentException("message")` inside a method when a parameter is invalid.
9. Create a custom exception class `InsufficientFundsException` extending `Exception`.
10. Create a method that throws your custom exception. Call it in a `try-catch`.
11. Write a `try-catch` that catches `NullPointerException`.
12. Write a `try-catch` that catches the general `Exception` class.
13. Write a method that declares `throws` for two different exceptions.
14. Use `throw` to re-throw a caught exception.
15. Write a `try-catch` where the `catch` block prints `e.getMessage()`.
16. Write a custom exception with a constructor that takes a message and an error code.
17. Write a `try-with-resources` block (e.g., with `Scanner` or `BufferedReader`).
18. Write a method that catches an exception and throws a different exception (exception chaining using `new Exception("msg", cause)`).
19. Write nested `try-catch` blocks.
20. Write a program that uses `finally` to close a resource.

---

## 33. Enum

### What to practice

* Enum declaration and values
* `values()`, `valueOf()`
* Enum in `switch`
* Enum with fields and methods

### Practice

1. Declare an `enum Day` with all 7 days. Print one value.
2. Use `Day.values()` to iterate over all enum values and print each.
3. Use `Day.valueOf("MONDAY")` to get an enum constant from a string. Print it.
4. Write a `switch` statement on a `Day` variable. Print a message for each day.
5. Declare an `enum Season` with `SPRING, SUMMER, FALL, WINTER`.
6. Create a variable of type `Season`. Assign it a value and print it.
7. Declare an `enum Color` with fields `String hex` and a constructor. Print the hex value of a color.
8. Declare an `enum Size` with `SMALL, MEDIUM, LARGE` and an `int value` field (e.g., 1, 2, 3). Add a getter.
9. Use `ordinal()` on an enum value. Print the ordinal.
10. Use `name()` on an enum value. Print the name.

---

## 34. Generics

### What to practice

* Generic classes, methods, interfaces
* Type parameters: `<T>`, `<K, V>`
* Bounded generics
* Generic constructors

### Practice

1. Create a generic class `Box<T>` with a field `T value`, a getter, and a setter. Create a `Box<String>` and a `Box<Integer>`.
2. Create a generic method `<T> void printItem(T item)` that prints the item. Call it with different types.
3. Create a generic method `<T> T getFirst(T[] array)` that returns the first element of an array.
4. Create a generic class `Pair<A, B>` with two fields. Create a `Pair<String, Integer>`.
5. Create a generic interface `Transformer<T, R>` with a method `R transform(T input)`. Implement it.
6. Create a generic method that takes a `List<T>` and prints all elements.
7. Create a generic class `Stack<T>` with `push(T)`, `pop()`, and `peek()` methods using an `ArrayList<T>` internally.
8. Use bounded generics: `<T extends Number>`. Write a method that takes a `T` and calls `doubleValue()`.
9. Write a generic method `<T extends Comparable<T>> T max(T a, T b)` that returns the larger value.
10. Create a `Map<K, V>` wrapper class `SimpleMap<K, V>` with `put`, `get`, and `size` methods.
11. Create a generic method that takes a `List<? extends Number>` and computes the sum using `doubleValue()`.
12. Create a generic class `Triple<A, B, C>` with three fields of different types.
13. Write a generic method that creates and returns an `ArrayList<T>` from varargs: `<T> ArrayList<T> listOf(T... items)`.
14. Create a generic interface `Repository<T>` with methods `save(T item)`, `findById(int id)`, `findAll()`. Implement it for a `Student` class.
15. Write a generic constructor: a non-generic class with a constructor `<T> ClassName(T param)`.
16. Use `<T extends Comparable<T>>` in a generic class declaration.
17. Create a generic method that swaps two elements in an array: `<T> void swap(T[] arr, int i, int j)`.
18. Create a `Converter<F, T>` interface with `T convert(F from)`. Implement `StringToInteger` and `IntegerToString`.
19. Use wildcard `<?>` to write a method that prints any `List<?>`.
20. Write a generic method `<K, V> void printMap(Map<K, V> map)` that prints all entries.

---

## 35. Utility Classes

### What to practice

* `Math` methods
* `Arrays` methods
* `Collections` methods
* `Objects` methods
* `Integer` / `Character` methods

### Imports needed

```java
import java.util.Arrays;
import java.util.Collections;
import java.util.Objects;
import java.util.ArrayList;
```

### Practice

1. Use `Math.max(a, b)` and `Math.min(a, b)`. Print the results.
2. Use `Math.abs(-15)`. Print the result.
3. Use `Math.pow(2, 10)` and `Math.sqrt(144)`. Print both.
4. Use `Math.ceil(4.3)` and `Math.floor(4.7)`. Print both.
5. Use `Math.round(4.5)` and `Math.round(4.4)`. Print both.
6. Use `Math.random()` to generate a random double between 0 and 1. Print it.
7. Generate a random integer between 1 and 100 using `Math.random()`.
8. Use `Arrays.sort()`, `Arrays.toString()`, `Arrays.fill()` on an `int[]`. Print after each.
9. Use `Collections.sort()`, `Collections.reverse()`, `Collections.shuffle()` on an `ArrayList`. Print after each.
10. Use `Collections.unmodifiableList()` to create a read-only list. Try to add to it (catch the exception).
11. Use `Objects.equals(a, b)` to safely compare two objects (one may be `null`).
12. Use `Objects.hash(field1, field2)` to generate a hash code.
13. Use `Objects.requireNonNull(obj, "message")` to validate a parameter.
14. Use `Character.isLetter()`, `Character.isDigit()`, `Character.isLetterOrDigit()` on various characters.
15. Use `Integer.toBinaryString()`, `Integer.toHexString()`, `Integer.toOctalString()`. Print each.

---

## 36. Java Performance / Syntax Awareness

### What to practice

* Integer overflow awareness
* `long` arithmetic with `1L`
* String vs StringBuilder in loops
* Primitive vs wrapper types
* ArrayList initial capacity

### Practice

1. Multiply two large `int` values (e.g., `100000 * 100000`). Print the result. Observe the overflow.
2. Fix the overflow by casting one operand to `long` before multiplying: `(long) 100000 * 100000`. Print the result.
3. Use `1L * a * b` to avoid overflow in a multiplication. Print the result.
4. Concatenate 100 strings in a `for` loop using `+`. Then do the same with `StringBuilder`. Print both results.
5. Create an `ArrayList<Integer>` with initial capacity `1000`: `new ArrayList<>(1000)`.
6. Declare an `int` variable and an `Integer` variable. Assign between them (autoboxing/unboxing). Print both.
7. Show that `new Integer(5) == new Integer(5)` is `false` (reference comparison). Use `.equals()` instead.
8. Declare a `long` variable. Assign `Integer.MAX_VALUE + 1L` to it. Print the result.
9. Create a `String` in a loop using `+=`. Measure with `System.currentTimeMillis()` before and after. Print elapsed time.
10. Do the same as above with `StringBuilder`. Compare the elapsed time.

---

## 37. Mixed Core Java Practice

Small programs combining multiple Core Java features.

### Practice

1. Create a `Person` class with a constructor, fields, and `toString()`. Create 5 objects, store them in an `ArrayList`, and print all.
2. Create a `BankAccount` class with deposit and withdraw methods. Use encapsulation. Test from `main`.
3. Create a `Student` class implementing `Comparable<Student>` by GPA. Sort an `ArrayList<Student>`.
4. Create an interface `Playable` with `play()`. Implement it in `Guitar` and `Piano`. Store in a `Playable[]` and loop.
5. Create an abstract class `Shape` with abstract `area()`. Create `Circle` and `Rectangle`. Store in `Shape[]`, print areas.
6. Create a `HashMap<String, ArrayList<String>>` representing courses and their students. Add data and print.
7. Create a `TreeSet<String>` of names. Print the first, last, and a name using `ceiling()`.
8. Create a `PriorityQueue<Integer>` (min-heap). Add numbers. Drain and print in order.
9. Create a `Deque<String>` used as a stack. Push 5 items. Pop and print all.
10. Create a `Student` class with a `List<Integer> grades`. Add a method `getAverage()`. Create students and print averages.
11. Create an `Employee` class extending `Person`. Override `toString()`. Create objects of both. Store in `Person[]`.
12. Use `Scanner` to read a name and age. Create a `Person` object and print it.
13. Create a `Node` class. Create 3 nodes and link them: `n1 → n2 → n3`. Print `n1.next.next.data`.
14. Create a custom exception `InvalidAgeException`. Throw it from a `Person` constructor if age < 0. Catch in `main`.
15. Create an `enum Priority` with `LOW, MEDIUM, HIGH`. Use it in a `Task` class. Print tasks with their priority.
16. Create a generic `Box<T>` class. Create `Box<String>` and `Box<Integer>` objects. Print their contents.
17. Create a `Comparator<String>` that sorts by length. Sort an `ArrayList<String>` and print.
18. Create a `HashMap<String, Integer>` for a word counter. Use `getOrDefault` to safely increment. Print all entries.
19. Create a `LinkedList<String>`. Use `addFirst` and `addLast`. Print. Remove from both ends. Print.
20. Create a `2D int array`. Fill it using nested loops. Print it using `Arrays.deepToString()`.
21. Create an `ArrayList<Integer>`. Use `Collections.sort()`, `Collections.reverse()`, `Collections.min()`, `Collections.max()`. Print after each.
22. Create a class with a `private` field, a getter with validation, and a constructor. Test from `main`.
23. Write a `try-catch` that reads an integer from the user. If parsing fails, print an error and ask again using a loop.
24. Create a `StringBuilder`. Append 10 items in a loop. Reverse. Convert to `String`. Print.
25. Create two `HashSet<String>` sets. Compute and print their union and intersection.
26. Create a `Queue<Integer>` using `ArrayDeque`. Offer 5 elements. Poll all and print.
27. Create an abstract class `Notification` with `send()`. Create `EmailNotification` and `SMSNotification`. Send from a `Notification[]`.
28. Use `Math.max`, `Math.min`, `Math.abs`, `Math.pow` in one program. Print all results.
29. Create a class implementing two interfaces. Call methods from both interface references.
30. Create a `TreeMap<String, Integer>`. Put entries. Print `firstKey()`, `lastKey()`, and iterate.
31. Create a `wrapper class` style class: a class `IntWrapper` with a `private int` value, constructor, getter, and `toString()`.
32. Create a method that takes `int... args` (varargs). Call it with 0, 1, 3, and 5 arguments.
33. Use `Comparator.comparing().thenComparing()` to sort a list of custom objects by two fields.
34. Create an `ArrayList<String>`. Remove all strings shorter than 4 characters using `removeIf()`. Print.
35. Create a `HashMap<String, Student>`. Iterate using `entrySet()` and print each student.
36. Create a `TreeSet<Integer>` with a reverse-order comparator. Add values. Print.
37. Use `Objects.requireNonNull()` in a constructor to validate parameters.
38. Create a class with a `static` counter. Each constructor call increments it. Create 5 objects and print the count.
39. Create an enum `Operation` with `ADD, SUBTRACT, MULTIPLY, DIVIDE`. Add a method `apply(int a, int b)` that performs the operation.
40. Create a generic method `<T> ArrayList<T> repeat(T item, int count)` that creates a list with `count` copies of `item`. Test with `String` and `Integer`.
41. Create a `Node<T>` generic node class. Create nodes of `Integer` and `String` type. Link two `Integer` nodes.
42. Write a program that uses `BufferedReader` to read two integers, creates `Point` objects, and prints the distance.
43. Create a `PriorityQueue<String>` sorted by string length using a custom comparator. Add strings and drain.
44. Create a `Deque<Integer>`. Use it to add elements at both ends. Print using `descendingIterator()`.
45. Create a class `Config` with a `HashMap<String, String>` for key-value settings. Add methods `set(key, value)`, `get(key)`, `has(key)`.

---

## 38. Final Blind Syntax Test

No hints. No imports shown. No explanations. No solutions.

Open a blank Java file and write each program from memory.

### Practice

1. Write a complete Java program that prints `"Ready for DSA"`.
2. Declare all 8 primitive types, assign values, and print each.
3. Read two integers using `Scanner`, compute and print their product.
4. Read a line using `BufferedReader` and `InputStreamReader`. Parse it as an `int`. Print it.
5. Write an `if/else if/else` that classifies a temperature: cold (< 10), warm (10–30), hot (> 30).
6. Write a `switch` on a `String` variable with 4 cases and a default.
7. Write a `for` loop that prints numbers 1 to 50.
8. Write a `while` loop that doubles a number starting from 1 until it exceeds 1000. Print each value.
9. Write a `do-while` loop that prints numbers 10 down to 1.
10. Write nested loops that print a right-aligned triangle of `*` with 5 rows.
11. Write a `static` method `multiply(int a, int b)` that returns the product. Call it from `main`.
12. Write two overloaded methods `display(int)` and `display(String)`. Call both.
13. Write a varargs method `average(double... values)` that returns the average. Call it with 3 and 5 arguments.
14. Create a class `Car` with fields, a constructor, and `toString()`. Create an object and print it.
15. Create a class with constructor overloading (no-arg and two-arg). Create objects with both.
16. Create a class with `private` fields, getters, setters with validation, and a constructor.
17. Create `Animal` and `Dog extends Animal`. Override a method. Call it polymorphically.
18. Create an abstract class `Shape` with abstract `area()`. Implement it in `Circle`. Print the area.
19. Create an interface `Greetable` with `greet()`. Implement it. Call via interface reference.
20. Create a class implementing two interfaces.
21. Declare an `int[]` array. Fill it with values 1–10 using a loop. Print with `Arrays.toString()`.
22. Declare a 2D `int[][]` array. Fill and print with `Arrays.deepToString()`.
23. Use `String` methods: `substring`, `toUpperCase`, `replace`, `split` on a sentence.
24. Use `StringBuilder` to build a comma-separated string from an array. Print the result.
25. Parse `"123"` to `int`, `"45.67"` to `double`, `"true"` to `boolean`. Print each.
26. Create an `ArrayList<String>`. Add 5 items. Remove one by index, one by value. Sort. Print.
27. Create a `LinkedList<Integer>`. Use `addFirst`, `addLast`, `removeFirst`, `removeLast`. Print after each.
28. Create a `HashSet<String>`. Add values including duplicates. Print the set and its size.
29. Create a `TreeSet<Integer>`. Add values. Print `first()`, `last()`, `higher(x)`, `lower(x)`.
30. Create a `HashMap<String, Integer>`. Put 5 entries. Get by key. Check `containsKey`. Iterate with `entrySet`.
31. Create a `TreeMap<String, Integer>`. Print `firstKey()`, `lastKey()`. Iterate.
32. Create a `Queue<String>` using `ArrayDeque`. Offer, peek, poll. Print after each.
33. Create a `Deque<Integer>`. Add at both ends. Remove from both ends. Print.
34. Create a `PriorityQueue<Integer>`. Add values. Drain with `poll()` and print each.
35. Create a max-heap `PriorityQueue<Integer>`. Add values. Drain and print.
36. Create a class implementing `Comparable`. Sort an `ArrayList` of those objects.
37. Sort an `ArrayList<String>` using `Comparator.comparing(String::length).thenComparing(Comparator.naturalOrder())`.
38. Create two object references pointing to the same object. Modify through one, print through the other.
39. Handle `NullPointerException`: call a method on a `null` reference in a `try-catch`.
40. Create a `Node` class. Create 3 nodes chained together. Print the last node's data via the first.
41. Create a `TreeNode` class. Build a root with left and right children. Print child values.
42. Write a `try-catch-finally` block catching `NumberFormatException`.
43. Create a custom exception. Throw it from a method. Catch it in `main`.
44. Create an `enum Color` with 5 values. Iterate with `values()`. Use in a `switch`.
45. Create a generic class `Container<T>` with `set(T)` and `get()`. Use with `String` and `Integer`.
46. Write a generic method `<T> void printArray(T[] arr)` that prints all elements.
47. Use `Math.max`, `Math.min`, `Math.sqrt`, `Math.pow`, `Math.abs` in one program.
48. Create a `Student` class. Store students in a `HashMap<Integer, Student>` (id → student). Look up by id.
49. Create a class with a `static` field counting instances. Create 3 objects. Print the count.
50. Create a program that uses at least 5 different collections (`ArrayList`, `HashMap`, `HashSet`, `Queue`, `TreeMap`). Add data to each and print.

---

# Completion Checklist

- [ ] I can create a Java class from memory
- [ ] I can write `main()` from memory
- [ ] I can declare all primitive data types
- [ ] I can use operators without looking up syntax
- [ ] I can take input using Scanner
- [ ] I can take input using BufferedReader
- [ ] I can write if/else without looking up syntax
- [ ] I can write switch without looking up syntax
- [ ] I can write for loops without looking up syntax
- [ ] I can write while loops without looking up syntax
- [ ] I can write methods without looking up syntax
- [ ] I can create classes and objects
- [ ] I can create constructors
- [ ] I understand and can write `this`
- [ ] I can use encapsulation
- [ ] I can write inheritance syntax
- [ ] I can use `super`
- [ ] I can write method overriding
- [ ] I can use polymorphism
- [ ] I can write abstract classes
- [ ] I can write interfaces
- [ ] I can create and use arrays
- [ ] I can use common String methods
- [ ] I can use StringBuilder
- [ ] I can use wrapper classes
- [ ] I can use ArrayList
- [ ] I can use LinkedList
- [ ] I can use HashSet
- [ ] I can use TreeSet
- [ ] I can use HashMap
- [ ] I can use TreeMap
- [ ] I can use Queue
- [ ] I can use Deque
- [ ] I can use PriorityQueue
- [ ] I can implement Comparable
- [ ] I can write Comparator syntax
- [ ] I understand Java object references
- [ ] I can handle null safely
- [ ] I can create Node/TreeNode classes
- [ ] I can handle exceptions
- [ ] I can use enums
- [ ] I can use generics
- [ ] I can use common Java utility classes
- [ ] I can combine multiple Core Java features
- [ ] I can write normal Java code from a blank file without looking up basic syntax
