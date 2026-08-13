# Python Scripting Cheatsheet

Complete quick reference guide for Python programming — basics, functions, data structures, file handling, error handling, OOP, modules, strings, iterators, and standard libraries.

## 🎯 Basics & Syntax

### Variables & Data Types

Variables:

```python
x = 10                          # Integer
y = 3.14                        # Float
name = "John"                   # String
is_active = True                # Boolean
items = [1, 2, 3]               # List
coords = (10, 20)               # Tuple
data = {"name": "John", "age": 30}  # Dictionary
unique = {1, 2, 3}              # Set
```

Type conversion:

```python
int("42")                       # Convert to integer
str(42)                         # Convert to string
float("3.14")                   # Convert to float
list("abc")                     # Convert to list
type(x)                         # Get type
```

String operations:

```python
name = "Python"
name.upper()                    # PYTHON
name.lower()                    # python
name.replace("P", "J")          # Jython
name.split("y")                 # ['P', 'thon']
"-".join(["a", "b"])            # a-b
```

### Control Flow

If-else statements:

```python
if age >= 18:
    print("Adult")
elif age >= 13:
    print("Teenager")
else:
    print("Child")
```

For loops:

```python
for i in range(5):              # 0 to 4
    print(i)

for item in [1, 2, 3]:
    print(item)

for i, val in enumerate(items):
    print(f"{i}: {val}")
```

While loops:

```python
while x < 10:
    print(x)
    x += 1

while True:
    if condition:
        break
    if skip:
        continue
```

## ⚙️ Functions & Lambda

### Function Definition

Basic function:

```python
def greet(name):
    return f"Hello, {name}!"

result = greet("Alice")         # Hello, Alice!
```

Default arguments:

```python
def add(a, b=5):
    return a + b

add(10)                         # 15
add(10, 20)                     # 30
```

Variable arguments:

```python
def sum_all(*args):             # *args for tuple
    return sum(args)

def print_info(**kwargs):       # **kwargs for dict
    for key, value in kwargs.items():
        print(f"{key}: {value}")
```

Lambda functions:

```python
square = lambda x: x ** 2
square(5)                       # 25

numbers = [1, 2, 3, 4, 5]
squared = map(lambda x: x ** 2, numbers)
```

### Decorators

Simple decorator:

```python
def decorator(func):
    def wrapper():
        print("Before")
        func()
        print("After")
    return wrapper

@decorator
def my_func():
    print("Function")
```

Decorator with arguments:

```python
def repeat(times):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(times=3)
def greet():
    print("Hello!")
```

## 📦 Lists & Dictionaries

### List Operations

List methods:

```python
items = [3, 1, 4, 1, 5]
items.append(9)                 # Add element
items.extend([2, 6])            # Add multiple
items.insert(0, 0)              # Insert at index
items.remove(1)                 # Remove first occurrence
items.pop()                     # Remove and return last
items.sort()                    # Sort in place
items.reverse()                 # Reverse in place
```

List comprehension:

```python
squares = [x**2 for x in range(10)]
even = [x for x in range(10) if x % 2 == 0]
matrix = [[i*j for j in range(3)] for i in range(3)]
```

List slicing:

```python
items = [0, 1, 2, 3, 4, 5]
items[1:4]                      # [1, 2, 3]
items[:3]                       # [0, 1, 2]
items[3:]                       # [3, 4, 5]
items[::2]                      # [0, 2, 4] (every 2nd)
items[::-1]                     # [5, 4, 3, 2, 1, 0] (reversed)
```

### Dictionary Operations

Dictionary methods:

```python
person = {"name": "Alice", "age": 30}
person["city"] = "NYC"          # Add/update
person.get("name")              # Alice (safe access)
person.keys()                   # dict_keys(['name', 'age'])
person.values()                 # dict_values(['Alice', 30])
person.items()                  # Key-value pairs
person.pop("age")               # Remove and return
person.update({"age": 31})      # Update multiple
```

Dictionary comprehension:

```python
squares = {x: x**2 for x in range(5)}
# {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}
filtered = {k: v for k, v in person.items() if v}
flipped = {v: k for k, v in person.items()}
```

## 📄 File Handling

### File Operations

Reading files:

```python
with open("file.txt", "r") as f:
    content = f.read()          # Read entire file

with open("file.txt", "r") as f:
    lines = f.readlines()       # Read all lines

with open("file.txt", "r") as f:
    for line in f:              # Iterate lines
        print(line.strip())
```

Writing files:

```python
with open("file.txt", "w") as f:
    f.write("Hello")            # Overwrite

with open("file.txt", "a") as f:
    f.write("\nAppend")         # Append

with open("file.txt", "w") as f:
    f.writelines(["Line1\n", "Line2\n"])
```

File paths:

```python
import os
os.path.exists("file.txt")      # Check existence
os.path.isfile("file.txt")      # Check if file
os.path.isdir("folder")         # Check if directory
os.path.getsize("file.txt")     # Get file size
os.path.basename("/path/to/file.txt")  # "file.txt"
```

## ⚠️ Exceptions & Error Handling

### Try-Except Blocks

Basic exception handling:

```python
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero")
except Exception as e:
    print(f"Error: {e}")
else:
    print("Success!")
finally:
    print("Cleanup")
```

Raise exceptions:

```python
if age < 0:
    raise ValueError("Age cannot be negative")
if not name:
    raise TypeError("Name must be provided")

raise RuntimeError("Something went wrong")
```

Custom exceptions:

```python
class CustomError(Exception):
    pass

try:
    raise CustomError("Custom message")
except CustomError as e:
    print(f"Caught: {e}")
```

## 🏗️ Classes & OOP

### Class Definition

Basic class:

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def greet(self):
        return f"Hello, I'm {self.name}"

    def __str__(self):
        return f"Person({self.name}, {self.age})"

person = Person("Alice", 30)
print(person.greet())
```

Inheritance:

```python
class Animal:
    def speak(self):
        pass

class Dog(Animal):
    def speak(self):
        return "Woof!"

dog = Dog()
print(dog.speak())
```

Class methods & static methods:

```python
class Counter:
    count = 0

    @classmethod
    def get_count(cls):
        return cls.count

    @staticmethod
    def add(a, b):
        return a + b

    @property
    def value(self):
        return self.count
```

## 📦 Modules & Imports

### Import Methods

Import statements:

```python
import os                       # Import module
import os.path                  # Import submodule
from os import path             # Import specific
from os import *                # Import all
import numpy as np              # Alias
from datetime import datetime, timedelta
```

Built-in modules:

```python
import os                       # File operations
import sys                      # System functions
import time                     # Time functions
import datetime                 # Date/time objects
import json                     # JSON handling
import re                       # Regular expressions
import random                   # Random numbers
```

## 📝 Strings & Regular Expressions

### String Formatting

String formatting methods:

```python
name = "Alice"
age = 30

# Format method
"Hello, {}. You are {} years old".format(name, age)

# f-strings (Python 3.6+)
f"Hello, {name}. You are {age} years old"

# Old-style
"Hello, %s. You are %d years old" % (name, age)

# Format with alignment
f"{name:>10}"                   # Right align
f"{name:<10}"                   # Left align
f"{age:05d}"                    # Pad with zeros
```

Regular expressions:

```python
import re

text = "Email: test@example.com"
pattern = r"[a-z]+@[a-z]+\.[a-z]+"
re.search(pattern, text)        # Find first match
re.findall(pattern, text)       # Find all matches
re.sub(pattern, "***", text)    # Replace
re.split(r"\s+", text)          # Split by pattern
re.match(r"^Email", text)       # Match at start
```

## 🗂️ Data Structures

### Built-in Collections

Tuples:

```python
coords = (10, 20)               # Immutable
x, y = coords                   # Unpacking
single = (42,)                  # Single element
t = tuple([1, 2, 3])            # Convert list
t.count(1)                      # Count occurrences
t.index(2)                      # Find index
```

Sets:

```python
unique = {1, 2, 3}
unique.add(4)                   # Add element
unique.remove(2)                # Remove (error if missing)
unique.discard(2)               # Remove (silent if missing)
unique.union({3, 4})            # Union
unique.intersection({2, 3})     # Intersection
unique.difference({2, 3})       # Difference
```

Collections module:

```python
from collections import Counter, defaultdict, OrderedDict

# Counter
counts = Counter([1, 1, 2, 2, 2, 3])
counts.most_common(2)

# defaultdict
d = defaultdict(list)
d["key"].append(1)

# OrderedDict
od = OrderedDict([("a", 1), ("b", 2)])
```

## ⚡ Iterators & Generators

### Generators & Iterators

Generator functions:

```python
def count_up(n):
    i = 0
    while i < n:
        yield i
        i += 1

for num in count_up(5):
    print(num)

gen = (x**2 for x in range(5))  # Generator expression
```

Iterator protocol:

```python
class CountUp:
    def __init__(self, n):
        self.n = n
        self.i = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self.i < self.n:
            self.i += 1
            return self.i
        raise StopIteration
```

Useful functions:

```python
map(lambda x: x**2, [1, 2, 3])
filter(lambda x: x > 2, [1, 2, 3, 4])
enumerate([a, b, c])            # Index and value
zip([1, 2], [a, b])             # Pair elements
reversed([1, 2, 3])
```

## 📚 Useful Standard Libraries

### Common Libraries

JSON handling:

```python
import json

data = {"name": "Alice", "age": 30}
json_str = json.dumps(data)     # Python to JSON
parsed = json.loads(json_str)   # JSON to Python

with open("file.json", "r") as f:
    data = json.load(f)
```

DateTime:

```python
from datetime import datetime, timedelta

now = datetime.now()
today = datetime.today()
formatted = now.strftime("%Y-%m-%d %H:%M:%S")
delta = timedelta(days=5)
future = now + delta
```

Logging:

```python
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)
logger.debug("Debug message")
logger.info("Info message")
logger.warning("Warning message")
logger.error("Error message")
```

Subprocess:

```python
import subprocess

result = subprocess.run(["ls", "-l"], capture_output=True)
output = result.stdout.decode()
subprocess.call(["echo", "hello"])
```

## ✅ Best Practices & Tips

- Use virtual environments — `python -m venv venv`
- Follow PEP 8 style guide — use consistent formatting and naming
- Write meaningful variable names — avoid single letters (except loops)
- Add docstrings — document functions and classes
- Use type hints — add type annotations for clarity
- Handle exceptions properly — don't use bare `except` clauses
- Keep functions small — single responsibility principle
- Use context managers — use `with` for resource management

### Debugging Tips

Debug with print:

```python
print(f"Variable: {x}, Type: {type(x)}")
```

Use debugger:

```python
import pdb; pdb.set_trace()
```

Check variables:

```python
dir(object)                     # List attributes
help(function)                  # Get help
```

### Python Tips

- Use `assert` for debugging
- Use `if __name__ == "__main__":` for scripts
- Use `sys.exit(code)` to exit with status

## 📋 Quick Reference

| Concept | Syntax | Example |
|---|---|---|
| List | `[item1, item2]` | `[1, 2, "three"]` |
| Dict | `{"key": value}` | `{"name": "Alice"}` |
| Tuple | `(item1, item2)` | `(1, 2, 3)` |
| Set | `{item1, item2}` | `{1, 2, 3}` |
| Lambda | `lambda x: x+1` | `map(lambda x: x*2, [1,2])` |
| List Comp | `[expr for x in iter]` | `[x**2 for x in range(5)]` |
| Try-Except | `try: ... except: ...` | Catch errors safely |
| With | `with open() as f:` | Auto close files |

## 🎓 Python Concepts

**Data Types:** `int`, `float`, `str`, `bool`, `list`, `dict`, `tuple`, `set`

**Built-in Functions:** `len()`, `range()`, `enumerate()`, `zip()`, `map()`, `filter()`, `sorted()`, `sum()`

**String Methods:** `.upper()`, `.lower()`, `.strip()`, `.replace()`, `.split()`, `.join()`, `.format()`, `.startswith()`

**Useful Imports:** `os`, `sys`, `json`, `re`, `math`, `random`, `time`, `datetime`

---

*Source: adapted from the Python Scripting cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
