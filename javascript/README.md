# JavaScript Cheatsheet

Complete quick reference guide for all JavaScript topics — from fundamentals to advanced patterns.

## Table of Contents

1. [Variables & Data Types](#-variables--data-types)
2. [Operators](#-operators)
3. [Control Flow](#-control-flow)
4. [Functions](#-functions)
5. [Objects & Arrays](#-objects--arrays)
6. [String Methods](#-string-methods)
7. [Array Methods](#-array-methods)
8. [Object Methods](#-object-methods)
9. [Promises & Async](#-promises--async)
10. [Classes & OOP](#-classes--oop)
11. [ES6+ Features](#-es6-features)
12. [DOM Manipulation](#-dom-manipulation)
13. [Regular Expressions](#-regular-expressions)
14. [Error Handling](#-error-handling)
15. [Advanced Patterns](#-advanced-patterns)
16. [Tips & Tricks](#-tips--tricks)

## 🔤 Variables & Data Types

Variable declaration:

```javascript
var x = 5;    // function-scoped
let y = 10;   // block-scoped
const z = 15; // block-scoped, immutable
```

Prefer `const` by default, then `let`, and avoid `var`.

Primitive data types:

```javascript
typeof 'hello'    // 'string'
typeof 42         // 'number'
typeof true       // 'boolean'
typeof undefined  // 'undefined'
typeof Symbol()   // 'symbol'
typeof 123n       // 'bigint'
```

Object/reference data types:

```javascript
typeof {}     // 'object'
typeof []     // 'object'
typeof null   // 'object' (quirk)
```

## ⚙️ Operators

Comparison operators:

```javascript
5 == '5'   // true (type coercion)
5 === '5'  // false (strict equality)
5 != 6     // true
5 !== '5'  // true
5 > 3      // true
```

Arithmetic operators:

```javascript
5 + 3   // 8
10 - 2  // 8
3 * 4   // 12
15 / 3  // 5
17 % 5  // 2
2 ** 3  // 8
```

Assignment operators:

```javascript
let x = 5;

x += 3; // x = 8
x -= 2; // x = 6
x *= 2; // x = 12
x /= 3; // x = 4
```

Logical operators:

```javascript
true && false // false (AND)
true || false // true (OR)
!true         // false (NOT)
```

## 🔀 Control Flow

If/Else:

```javascript
if (condition) {
  // code
} else if (other) {
  // code
} else {
  // code
}
```

Ternary operator:

```javascript
condition ? value1 : value2

const status = age >= 18 ? 'adult' : 'minor';
```

For loop:

```javascript
for (let i = 0; i < 5; i++) {}
```

While loop:

```javascript
while (condition) {}
```

Do-while:

```javascript
do {} while (condition);
```

For-of (iterate values):

```javascript
for (const item of array) {}
```

For-in (iterate keys):

```javascript
for (const key in object) {}
```

Switch statement:

```javascript
switch (value) {
  case 1:
    // code
    break;
  case 2:
    // code
    break;
  default:
    // code
}
```

## 🧩 Functions

Regular function:

```javascript
function add(a, b) {
  return a + b;
}
```

Arrow function:

```javascript
const add = (a, b) => a + b;
```

Async function:

```javascript
async function fetchData() {
  return await request();
}
```

Default parameters:

```javascript
function greet(name = 'World') {}
```

Rest parameters:

```javascript
function sum(...numbers) {}
```

Spread operator:

```javascript
const arr = [1, 2, 3];

[...arr, 4] // [1, 2, 3, 4]
```

Closures:

```javascript
function outer() {
  const x = 10;
  return function inner() {
    return x; // x is accessible via closure
  };
}

const closure = outer();
closure(); // 10
```

This binding:

```javascript
// Regular function
function test() {
  console.log(this); // window/global
}

// Arrow function (lexical this)
const test = () => {
  console.log(this); // parent context
};

// Method
const obj = {
  method() {
    console.log(this); // obj
  }
};
```

## 📦 Objects & Arrays

Creating arrays:

```javascript
// Literal
const arr = [1, 2, 3];

// Constructor
const arr2 = new Array(3);

// From an array-like/iterable
const arr3 = Array.from(arrayLike);

// Of specific values
const arr4 = Array.of(1, 2, 3);
```

Creating objects:

```javascript
// Literal
const obj = { name: 'John' };

// Constructor
const obj2 = new Object();

// Object.create
const obj3 = Object.create(proto);

// Class
class Person {}
const obj4 = new Person();
```

Spread operator:

```javascript
// Copy array
const copy = [...original];

// Merge arrays
const merged = [...arr1, ...arr2];

// Copy object
const copyObj = { ...obj };

// Merge objects
const mergedObj = { ...obj1, ...obj2 };
```

Destructuring:

```javascript
// Array destructuring
const [a, b, c] = [1, 2, 3];

// Object destructuring
const { name, age } = { name: 'John', age: 25 };

// With defaults
const { name = 'Unknown' } = obj;

// Nested
const { address: { city } } = user;
```

## 🔠 String Methods

String length:

```javascript
'hello'.length // 5
```

Convert to uppercase:

```javascript
'hello'.toUpperCase() // 'HELLO'
```

Convert to lowercase:

```javascript
'HELLO'.toLowerCase() // 'hello'
```

First occurrence index:

```javascript
'hello'.indexOf('l') // 2
```

Check if contains:

```javascript
'hello'.includes('ell') // true
```

Check start:

```javascript
'hello'.startsWith('he') // true
```

Check end:

```javascript
'hello'.endsWith('lo') // true
```

Extract substring — `slice(start, end)`:

```javascript
'hello'.slice(1, 4) // 'ell'
```

Extract substring — `substring(start, end)`:

```javascript
'hello'.substring(1, 4) // 'ell'
```

Extract substring — `substr(start, length)`:

```javascript
'hello'.substr(1, 3) // 'ell'
```

Split to array:

```javascript
'a,b,c'.split(',') // ['a', 'b', 'c']
```

Replace first match:

```javascript
'hello'.replace('l', 'L') // 'heLlo'
```

Replace all matches:

```javascript
'hello'.replaceAll('l', 'L') // 'heLLo'
```

Remove whitespace:

```javascript
'  hello  '.trim() // 'hello'
```

Repeat string:

```javascript
'ab'.repeat(3) // 'ababab'
```

Pad from start:

```javascript
'5'.padStart(3, '0') // '005'
```

Pad from end:

```javascript
'5'.padEnd(3, '0') // '500'
```

## 📚 Array Methods

Iteration methods:

```javascript
// forEach
arr.forEach((item, i) => {});

// map
arr.map(x => x * 2); // new array

// filter
arr.filter(x => x > 5);

// reduce
arr.reduce((acc, x) => acc + x, 0);

// find
arr.find(x => x > 3);

// findIndex
arr.findIndex(x => x > 3);
```

Add/remove elements:

```javascript
const arr = [1, 2, 3];

// Add
arr.push(4);    // [1, 2, 3, 4]
arr.unshift(0); // [0, 1, 2, 3, 4]

// Remove
arr.pop();   // removes last
arr.shift(); // removes first
arr.splice(1, 2); // remove 2 elements starting at index 1
```

Combine/transform:

```javascript
// join
arr.join(',') // "1,2,3,4,5"

// reverse
arr.reverse() // reverses in place

// sort
arr.sort((a, b) => a - b);

// flat
[[1, 2], [3]].flat() // [1, 2, 3]

// flatMap
arr.flatMap(x => [x, x * 2]);
```

Search & check:

```javascript
const arr = [1, 2, 3, 4, 5];

// indexOf
arr.indexOf(3) // 2

// includes
arr.includes(3) // true

// some
arr.some(x => x > 3) // true

// every
arr.every(x => x > 0) // true

Array.isArray([1]);      // true
Array.of(1, 2, 3);       // [1, 2, 3]
Array.from('abc');       // ['a', 'b', 'c']
```

Slice/concat/spread:

```javascript
// slice
arr.slice(1, 3) // [2, 3]

// concat
arr.concat([6, 7]) // [1, 2, 3, 4, 5, 6, 7]

// spread
[...arr] // copy

// Array.from
Array.from(arrayLike);
```

## 🧱 Object Methods

Copying objects:

```javascript
// Shallow copy
Object.assign({}, obj);
const shallow = { ...obj };

// Deep copy
JSON.parse(JSON.stringify(obj));
```

Enumerating properties:

```javascript
const obj = { a: 1, b: 2 };

Object.keys(obj);    // ['a', 'b']
Object.values(obj);  // [1, 2]
Object.entries(obj); // [['a', 1], ['b', 2]]
```

Check/Has:

```javascript
// Has property
obj.hasOwnProperty('x');
'x' in obj;

// Get property descriptor
Object.getOwnPropertyDescriptor(obj, 'x');
```

Create/Extend:

```javascript
// Create
Object.create(proto);

// Define property
Object.defineProperty(obj, 'x', {
  value: 10,
  writable: false
});

// Is frozen
Object.isFrozen(obj);

// Freeze
Object.freeze(obj);
```

## ⏳ Promises & Async

Basic promise chaining:

```javascript
promise
  .then(result => {})
  .catch(error => {})
  .finally(() => {});
```

Creating a promise:

```javascript
const promise = new Promise((resolve, reject) => {
  if (success) {
    resolve(value);
  } else {
    reject(error);
  }
});
```

Resolve/Reject shortcuts:

```javascript
Promise.resolve(value);
Promise.reject(error);
```

Multiple promises:

```javascript
Promise.all([p1, p2, p3]);
Promise.race([p1, p2, p3]);
Promise.allSettled([p1, p2]);
```

Parallel execution:

```javascript
const [a, b] = await Promise.all([
  fetch('/a'),
  fetch('/b')
]);
```

Sequential execution:

```javascript
const a = await fetch('/a');
const b = await fetch('/b');
```

Race:

```javascript
await Promise.race([timeout, fetch()]);
```

Async/await with try/catch:

```javascript
async function getData() {
  try {
    const result = await promise;
    return result;
  } catch (error) {
    console.error(error);
  }
}
```

## 🏛️ Classes & OOP

Class definition:

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }

  speak() {
    console.log(`${this.name} makes a sound`);
  }

  static info() {
    return 'Animal class';
  }
}

const dog = new Animal('Dog');
dog.speak();
```

Inheritance:

```javascript
class Dog extends Animal {
  constructor(name, breed) {
    super(name);
    this.breed = breed;
  }

  speak() {
    super.speak();
    console.log('Woof!');
  }
}
```

Private fields & methods:

```javascript
class BankAccount {
  #balance = 0;

  deposit(amount) {
    this.#balance += amount;
  }

  #calculateFee() {
    return this.#balance * 0.01;
  }
}
```

Getters & setters:

```javascript
class User {
  constructor(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }

  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  }

  set fullName(name) {
    [this.firstName, this.lastName] = name.split(' ');
  }
}
```

## ✨ ES6+ Features

Template literals:

```javascript
const name = 'World';
const greeting = `Hello ${name}!`;

// Multi-line
const text = `
Line 1
Line 2
`;

// Tagged template
function tag(strings, ...values) {
  return strings[0] + values[0];
}
```

For-of loop:

```javascript
// Iterate values
for (const item of array) {}
for (const char of 'hello') {}
for (const [key, value] of Object.entries(obj)) {}
```

Default parameters:

```javascript
function greet(name = 'World', greeting = 'Hello') {
  console.log(`${greeting} ${name}!`);
}

greet();             // Hello World!
greet('Alice');      // Hello Alice!
greet('Bob', 'Hi');  // Hi Bob!
```

Rest & spread:

```javascript
// Rest
function sum(...numbers) {
  return numbers.reduce((a, b) => a + b);
}

// Spread
const arr = [1, 2, 3];
const copy = [...arr];

// Object spread
const merged = { ...obj1, ...obj2 };
```

Optional chaining & nullish coalescing:

```javascript
// Safe navigation
obj?.prop
obj?.[0]
func?.()

// Nullish coalescing
value ?? defaultValue

// Example
const port = config?.database?.port ?? 5432;
```

Symbols:

```javascript
const sym = Symbol('description');
const obj = {
  [sym]: 'value'
};

obj[sym]; // 'value'

// Well-known symbols
Symbol.iterator
Symbol.toStringTag
```

## 🖥️ DOM Manipulation

Text & HTML content:

```javascript
// Text
element.textContent = 'Text';
element.innerText = 'Text';

// HTML
element.innerHTML = '<span>HTML</span>';
```

Classes:

```javascript
element.classList.add('class');
element.classList.remove('class');
element.classList.toggle('class');
```

Attributes:

```javascript
element.setAttribute('attr', 'value');
element.getAttribute('attr');
element.removeAttribute('attr');
```

Selecting a single element:

```javascript
document.getElementById('id');
document.querySelector('.class');
document.querySelector('#id > div');
```

Selecting multiple elements:

```javascript
document.querySelectorAll('.class');
document.getElementsByClassName('class');
document.getElementsByTagName('div');
```

Event listeners:

```javascript
// Add listener
element.addEventListener('click', handler);

// Remove listener
element.removeEventListener('click', handler);

// Trigger event
element.dispatchEvent(new Event('custom'));
```

Common events: `click`, `input`, `change`, `submit`, `focus`, `blur`, `mouseover`, `mouseout`, `keydown`, `keyup`

Creating & inserting elements:

```javascript
// Create
const el = document.createElement('div');
const text = document.createTextNode('text');

// Add
parent.appendChild(el);
parent.insertBefore(el, ref);

// Remove
el.remove();
parent.removeChild(el);
```

## 🔍 Regular Expressions

Match & search:

```javascript
// Match
string.match(regex);
string.matchAll(regex);

// Search
string.search(regex);

// Replace
string.replace(regex, replacement);
string.replaceAll(regex, replacement);

// Split
string.split(regex);
```

Creating a regex:

```javascript
const regex = /pattern/flags;
const regex2 = new RegExp('pattern', 'flags');
```

Flags:

```
g - global (all matches)
i - ignore case
m - multiline
s - dot matches newline
u - unicode
```

Test/Exec:

```javascript
regex.test(string);
regex.exec(string);
```

Named groups, lookahead, lookbehind, word boundary:

```javascript
// Named groups
/(?<year>\d{4})/

// Lookahead
/\d+(?=\$)/

// Lookbehind
/(?<=\$)\d+/

// Word boundary
/\bword\b/

// Escape special characters
/\./ // literal dot
```

Common pattern syntax:

```
.        - any character
*        - 0 or more
+        - 1 or more
?        - 0 or 1
{n}      - exactly n
{n,m}    - n to m
^        - start
$        - end
[abc]    - a, b, or c
[a-z]    - range
[^abc]   - not a, b, c
()       - group
|        - or
```

## 🚨 Error Handling

Built-in error types:

```javascript
new Error('message');
new TypeError('not a number');
new ReferenceError('undefined variable');
new SyntaxError('invalid syntax');
new RangeError('out of range');
new URIError('invalid URI');
```

Try/catch/finally:

```javascript
try {
  // code that might throw
  throw new Error('message');
} catch (error) {
  console.error(error.message);
  console.error(error.stack);
} finally {
  // cleanup code
}
```

Async error handling:

```javascript
async function fetchData() {
  try {
    const res = await request();
    return res;
  } catch (error) {
    console.error(error);
  }
}
```

Custom errors:

```javascript
class ValidationError extends Error {
  constructor(message) {
    super(message);
    this.name = 'ValidationError';
  }
}

throw new ValidationError('Invalid input');
```

## 🧠 Advanced Patterns

Module pattern (IIFE):

```javascript
const module = (function() {
  const privateVar = 'secret';
  return {
    public: 'visible',
    method() {
      return privateVar;
    }
  };
})();

module.public;  // 'visible'
module.private; // undefined
```

Closures for private state:

```javascript
function createCounter() {
  let count = 0;
  return {
    increment: () => ++count,
    decrement: () => --count,
    get: () => count
  };
}

const counter = createCounter();
counter.increment(); // 1
```

Currying:

```javascript
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn(...args);
    }
    return (...nextArgs) => curried(...args, ...nextArgs);
  };
}

const add = curry((a, b) => a + b);
add(1)(2); // 3
```

Decorator/higher-order function:

```javascript
function logged(fn) {
  return function(...args) {
    console.log(`Calling ${fn.name}`);
    return fn(...args);
  };
}

const add = logged((a, b) => a + b);
```

Compose & pipe:

```javascript
const compose = (...fns) => (x) =>
  fns.reduceRight((acc, fn) => fn(acc), x);

const pipe = (...fns) => (x) =>
  fns.reduce((acc, fn) => fn(acc), x);

const add10 = x => x + 10;
const mult2 = x => x * 2;

compose(mult2, add10)(5); // (5+10)*2 = 30
```

Memoization:

```javascript
function memoize(fn) {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      return cache.get(key);
    }
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
}
```

## 💡 Tips & Tricks

- **Use `const` by default** - Prevents accidental reassignment
- **Use `let` for loops** - Block-scoped, prevents closure issues
- **Use arrow functions for callbacks** - Lexical `this` binding
- **Use `===` instead of `==`** - Avoid type coercion bugs
- **Destructure when possible** - More readable and concise
- **Use optional chaining** - Safer property access
- **Avoid modifying prototypes** - Can cause unexpected behavior
- **Avoid callback hell** - Use Promises or async/await
- **Don't use `var`** - Use `let` or `const` instead
- **Avoid global variables** - Use modules or closures

---
*Source: adapted from the JavaScript cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
