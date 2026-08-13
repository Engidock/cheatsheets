# Groovy Scripting Cheatsheet

Complete quick-reference guide for Groovy scripting language and DSL development.

## 🎯 Groovy Fundamentals

### Installation & Basic Syntax

Install & Run Groovy:

```bash
brew install groovy                    # macOS
sudo apt-get install groovy            # Linux
groovy -version                        # Check version
groovy script.groovy                   # Run script
groovysh                               # Interactive console
groovyc script.groovy                  # Compile
java -cp $GROOVY_HOME/lib/*:. Script   # Execute compiled
```

Variables & Data Types:

```groovy
// Dynamic variables (def keyword)
def name = "Groovy"
def age = 25
def balance = 100.50
def flag = true
def items = [1, 2, 3]

// Typed variables
String greeting = "Hello"
Integer count = 10
Double price = 99.99
Boolean active = false

// Type conversion
def num = "123".toInteger()            // String to Int
def decimal = "45.6".toDouble()        // String to Double
def str = 123.toString()               // Int to String
def bool = "true".toBoolean()          // String to Boolean
```

String Types & Interpolation:

```groovy
// Single quotes (literal - no interpolation)
def single = 'Hello World'

// Double quotes (with interpolation)
def double = "Hello $name"
def result = "2+2 = ${2 + 2}"

// Slashy strings (for paths/regex - no escaping needed)
def path = /C:\path\to\file/
def regex = /\d+/

// Triple quotes (multi-line)
def multiline = '''Line 1
Line 2
Line 3'''

// String interpolation examples
println "Name: $name"
println "Length: ${name.length()}"
println "Upper: ${name.toUpperCase()}"
```

### Comments

Comment Types:

```groovy
// Single line comment

/* Multi-line comment
   spanning multiple lines
   useful for documentation */

/** JavaDoc comment
 * @param value Input value
 * @return Result value
 */
```

## 📦 Collections

### Lists - Creation & Operations

List Creation & Access:

```groovy
// Create lists
def list = [1, 2, 3, 4, 5]
def mixed = [1, "two", 3.0, true]
def empty = []
def list2 = new ArrayList()

// Access elements
println list[0]                        // First element
println list[-1]                       // Last element
println list[1..3]                     // Slice [2,3,4]
println list.first()                   // First
println list.last()                    // Last

// Add elements
list << 6                              // Append
list.add(7)                            // Add method
list.addAll([8, 9, 10])                // Add multiple

// Remove elements
list.remove(0)                         // Remove by index
list.remove(Integer.valueOf(3))        // Remove by value
list.pop()                             // Remove last
list.removeAt(2)                       // Remove at index

// List info
list.size()                            // Length
list.isEmpty()                         // Empty check
list.contains(3)                       // Contains check
3 in list                              // In operator
```

List Methods & Transformations:

```groovy
// List methods
list.reverse()                         // Reverse
list.sort()                            // Sort ascending
list.sort().reverse()                  // Sort descending
list.unique()                          // Remove duplicates
list.join(", ")                        // Join to string
list.min()                             // Minimum value
list.max()                             // Maximum value
list.sum()                             // Sum of elements

// Flatten nested lists
def nested = [[1, 2], [3, 4], [5, 6]]
def flat = nested.flatten()            // [1,2,3,4,5,6]

// Create from range
def nums = (1..10).toList()            // [1,2,3,...,10]

// Functional operations
list.collect { it * 2 }                // Transform each
list.findAll { it > 2 }                // Filter matching
list.any { it % 2 == 0 }               // Any match?
list.all { it > 0 }                    // All match?
list.each { println it }               // Iterate
```

### Maps - Creation & Operations

Map Creation & Access:

```groovy
// Create maps
def map = [key: 'value', name: 'Groovy', version: 4]
def empty = [:]
def map2 = new HashMap()
def map3 = new TreeMap()               // Sorted map

// Access entries
map['newKey'] = 'newValue'             // Bracket notation
map.anotherKey = 'value'               // Dot notation
map.put('key', 'updated')              // Put method
map.putAll([a: 1, b: 2])               // Add multiple

// Remove entries
map.remove('key')
map.clear()                            // Remove all

// Map info
map.size()
map.isEmpty()
map.containsKey('name')                // Key exists
map.containsValue('Groovy')            // Value exists

// Keys & values
map.keySet()                           // All keys
map.values()                           // All values
map.entrySet()                         // Key-value pairs

// Access with defaults
map.get('missing', 'default')
map.getOrDefault('key', 'default')
```

Map Iteration & Transformation:

```groovy
// Iterate map
map.each { key, value ->
    println "$key: $value"
}

// Iterate entries
map.each { entry ->
    println "${entry.key}: ${entry.value}"
}

// Collect keys and values
def keyList = map.collect { k, v -> k }
def valueList = map.collect { k, v -> v }

// Filter map
def filtered = map.findAll { k, v -> v.length() > 3 }

// Transform map
def transformed = map.collectEntries { k, v ->
    [(k.toUpperCase()): v.toUpperCase()]
}

// Any/All checks
def hasLong = map.any { k, v -> v.length() > 5 }
def allStrings = map.every { k, v -> v instanceof String }
```

### Ranges

Range Operations:

```groovy
// Create ranges
def range = 1..5                       // 1,2,3,4,5 (inclusive)
def range2 = 1..<5                     // 1,2,3,4 (exclusive)
def letters = 'a'..'z'                 // a to z
def desc = 5..1                        // 5,4,3,2,1

// Range methods
range.first()                          // First element
range.last()                           // Last element
range.size()                           // Range size

// Iterate range
range.each { println it }
for (i in 1..10) { println i }

// Range transformations
range.collect { it * 2 }               // [2,4,6,8,10]
range.findAll { it > 2 }               // [3,4,5]

// Range checks
range.contains(3)
3 in range

// Convert to list
def list = (1..10).toList()            // [1,2,3,...,10]

// Step through range
(1..10).step(2).each { println it }    // 1,3,5,7,9
```

## 🔧 Control Flow

### Conditional Statements

If-Else Statements:

```groovy
// Simple if
if (age >= 18) {
    println "Adult"
}

// If-else
if (age >= 18) {
    println "Adult"
} else {
    println "Minor"
}

// If-else if-else
if (age < 13) {
    println "Child"
} else if (age < 18) {
    println "Teenager"
} else {
    println "Adult"
}

// Ternary operator
def status = age >= 18 ? "Adult" : "Minor"

// Elvis operator (?:)
def name = person?.name ?: "Unknown"

// Safe navigation (?.)
def length = text?.length()            // null if text is null
```

Switch Statements:

```groovy
// Switch-case
switch (value) {
    case 1:
        println "One"
        break
    case 2..5:
        println "Two to Five"
        break
    case "text":
        println "String"
        break
    case ~/\d+/:                       // Regex match
        println "Number"
        break
    default:
        println "Other"
}

// Multiple cases same action
switch (grade) {
    case 'A':
    case 'B':
        println "Good"
        break
    case 'C':
        println "Fair"
        break
}
```

### Loops

For Loops:

```groovy
// Range for
for (i in 1..5) {
    println i
}

// List for
for (item in list) {
    println item
}

// Map for
for (key in map.keySet()) {
    println "$key: ${map[key]}"
}

// Traditional C-style for
for (int i = 0; i < 5; i++) {
    println i
}

// Nested for loops
for (i in 1..3) {
    for (j in 1..3) {
        println "$i-$j"
    }
}
```

While & Do-While:

```groovy
// While loop
int i = 0
while (i < 5) {
    println i
    i++
}

// Do-while loop
int j = 0
do {
    println j
    j++
} while (j < 5)

// Break statement
for (i in 1..10) {
    if (i == 5) break
    println i                          // 1,2,3,4
}

// Continue statement
for (i in 1..10) {
    if (i == 5) continue
    println i                          // 1,2,3,4,6,7,8,9,10
}
```

Closure Iteration:

```groovy
// Each iteration
list.each { item ->
    println item
}

// Each with implicit 'it'
list.each { println it }

// Each with index
list.eachWithIndex { item, index ->
    println "$index: $item"
}

// Map iteration
map.each { key, value ->
    println "$key: $value"
}

// Collect (map/transform)
def doubled = list.collect { it * 2 }

// Find first matching
def first = list.find { it > 2 }

// Find all matching
def filtered = list.findAll { it > 2 }

// Any match?
def hasEven = list.any { it % 2 == 0 }

// All match?
def allPositive = list.every { it > 0 }
```

## ⚡ Methods & Closures

### Method Definition

Basic Methods:

```groovy
// No parameters
def greet() {
    println "Hello"
}

// With parameters
def add(a, b) {
    return a + b
}

// With type declarations
def multiply(int x, int y) {
    return x * y
}

// Default parameters
def power(base, exp = 2) {
    return base ** exp
}

// Variable arguments
def sum(int... numbers) {
    int total = 0
    for (num in numbers) {
        total += num
    }
    return total
}

// Named parameters
def person(name, age, city = "Unknown") {
    return "$name, $age, $city"
}

// Implicit return (last expression)
def max(a, b) {
    a > b ? a : b
}
```

### Closures

Closure Definition & Usage:

```groovy
// Simple closure
def greeting = { println "Hello!" }
greeting()

// Closure with parameter
def add = { a, b -> a + b }
println add(5, 3)                      // 8

// Closure with implicit 'it'
def square = { it * it }
println square(5)                      // 25

// Multi-line closure
def compute = { x ->
    x = x * 2
    x = x + 10
    x
}
println compute(5)                     // 20

// Closure in variable
def operations = [
    add: { a, b -> a + b },
    subtract: { a, b -> a - b },
    multiply: { a, b -> a * b }
]
println operations.add(10, 5)          // 15

// Accessing outer scope
def x = 10
def closure = { y -> x + y }
println closure(5)                     // 15

// Modifying outer variable
def counter = 0
def increment = { counter++ }
increment()
println counter                        // 1
```

## 🏛️ Object-Oriented Programming

### Classes & Objects

Class Definition:

```groovy
// Basic class
class Person {
    String name
    int age
}

// Using the class
def person = new Person()
person.name = "John"
person.age = 30

// Class with constructor
class Book {
    String title
    String author
    int year

    Book(title, author, year) {
        this.title = title
        this.author = author
        this.year = year
    }
}
def book = new Book("1984", "George Orwell", 1949)

// Class with methods
class Calculator {
    def add(a, b) {
        a + b
    }

    def subtract(a, b) {
        a - b
    }
}
def calc = new Calculator()
println calc.add(5, 3)                 // 8
```

Inheritance & Static Members:

```groovy
// Base class
class Animal {
    String name
    def speak() {
        "Some sound"
    }
}

// Inheritance
class Dog extends Animal {
    def speak() {
        "Woof!"
    }
}
def dog = new Dog()
dog.name = "Buddy"
println dog.speak()                    // Woof!

// Static members
class MathUtils {
    static final double PI = 3.14159

    static double circleArea(radius) {
        return PI * radius * radius
    }
}
println MathUtils.PI                   // 3.14159
println MathUtils.circleArea(5)        // 78.5
```

## 📝 String & Regular Expressions

### String Operations

String Methods & Manipulation:

```groovy
// String creation
def text = "Hello World"
def upper = text.toUpperCase()         // HELLO WORLD
def lower = text.toLowerCase()         // hello world

// String methods
println text.length()                  // 11
println text.size()                    // 11
println text.capitalize()              // Hello World
println text.reverse()                 // dlroW olleH

// Substring & search
println text.substring(0, 5)           // Hello
println text.substring(6)              // World
println text.contains("World")         // true
println text.startsWith("Hello")       // true
println text.endsWith("World")         // true
println text.indexOf("o")              // 4

// Replacement
println text.replace("World", "Groovy")
println text.replaceAll("o", "0")      // Hell0 W0rld

// Trim & split
def padded = "  hello world  "
println padded.trim()                  // hello world
def csv = "apple,banana,orange"
def fruits = csv.split(",")            // [apple, banana, orange]

// String multiplication
println "ab" * 3                       // ababab
```

Regular Expressions:

```groovy
// Regex patterns
def pattern = ~/\d+/                   // Match numbers
def email = ~/^\w+@\w+\.\w+$/

// Pattern matching
def text = "Hello 123 World"
def matches = text =~ /\d+/
println matches[0]                     // 123

// Check if matches
if (text =~ /\d+/) {
    println "Contains numbers"
}

// Replace with regex
println text.replaceAll(/\d/, "X")     // Hello XXX World

// Split with regex
def result = "a,b;c:d".split(/[,;:]/)
println result                         // [a, b, c, d]

// Validate patterns
def isEmail(email) {
    email =~ /^\w+@\w+\.\w+$/ ? true : false
}
def isPhone(phone) {
    phone =~ /^\d{3}-\d{3}-\d{4}$/ ? true : false
}
```

## 🛡️ Exception Handling

### Try-Catch-Finally

Exception Handling:

```groovy
// Basic try-catch
try {
    def result = 10 / 0
} catch (ArithmeticException e) {
    println "Error: Cannot divide by zero"
}

// Multiple catch blocks
try {
    def number = "abc".toInteger()
} catch (NumberFormatException e) {
    println "Invalid number format"
} catch (Exception e) {
    println "Unexpected error"
}

// Try-catch-finally
try {
    def file = new File("nonexistent.txt")
    file.text
} catch (IOException e) {
    println "File error"
} finally {
    println "Cleanup code"
}

// Throwing exceptions
void validateAge(age) {
    if (age < 0) {
        throw new IllegalArgumentException("Invalid age")
    }
}
```

## 📋 Quick Reference

| Operation | Example | Result |
|---|---|---|
| Arithmetic | `2 + 3 * 4 ** 2` | `50` |
| String interpolation | `"Value: $x"` | Interpolates variable |
| List access | `list[-1]` | Last element |
| Range | `1..5` | `[1,2,3,4,5]` |
| Ternary | `a > b ? a : b` | Max value |
| Elvis operator | `a ?: b` | `a` if truthy |
| Map access | `map.key` | Value |
| Power | `2 ** 8` | `256` |

## ✅ Best Practices

### ✓ Code Quality
- Use meaningful variable names
- Keep methods small and focused
- Use closures for simple operations
- Leverage dynamic features wisely
- Write unit tests for complex logic

### ✓ Groovy Features
- Embrace string interpolation
- Use closures extensively
- Leverage safe navigation (`?.`) operator
- Use elvis operator (`?:`) for defaults
- Use spread operator (`*.`) for collections

### ⚠️ Common Pitfalls
- Don't forget null checks
- String comparison uses `.equals()`
- Groovy truthiness differs from Java
- Be careful with untyped parameters
- Avoid expensive regex in loops

---
*Source: adapted from the Groovy Scripting cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
