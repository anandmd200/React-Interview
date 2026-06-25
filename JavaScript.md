# JavaScript Fundamental Interview Questions (4 Years Experience)

## 📋 Table of Contents

1. [Hoisting](#1-hoisting)
2. [Closures](#2-closures)
3. [Event Loop & Async](#3-event-loop--asynchronous-javascript)
4. [Prototypal Inheritance](#4-prototypal-inheritance)
5. [this Keyword](#5-this-keyword)
6. [Scope & Scope Chain](#6-scope--scope-chain)
7. [Call, Apply, Bind](#7-call-apply-bind)
8. [Promises & Async/Await](#8-promises--asyncawait)
9. [Debouncing & Throttling](#9-debouncing--throttling)
10. [Event Delegation & Propagation](#10-event-delegation--propagation)
11. [Shallow vs Deep Copy](#11-shallow-vs-deep-copy)
12. [Currying](#12-currying)
13. [Generator Functions](#13-generator-functions)
14. [WeakMap & WeakSet](#14-weakmap--weakset)
15. [Proxy & Reflect](#15-proxy--reflect)

---

## 1. Hoisting

### Theory
Hoisting is JavaScript's default behavior of moving **declarations** (not initializations) to the top of their scope during the **compilation phase** before code execution.

- `var` → hoisted and initialized with `undefined`
- `let`/`const` → hoisted but remain in the **Temporal Dead Zone (TDZ)** until declaration is reached
- `function declarations` → fully hoisted (both name and body)
- `function expressions` → only the variable is hoisted

```javascript
// ✅ What the developer writes:
console.log(a);        // undefined (var is hoisted)
console.log(b);        // ❌ ReferenceError: Cannot access 'b' before initialization
console.log(myFunc()); // "Hello!" (function declaration fully hoisted)
console.log(myExpr()); // ❌ TypeError: myExpr is not a function

var a = 10;
let b = 20;

function myFunc() {
  return "Hello!";
}

var myExpr = function() {
  return "Hi!";
};
```

```javascript
// ✅ How JavaScript interprets it internally:
var a;               // hoisted declaration
var myExpr;          // hoisted declaration (as undefined)
function myFunc() {  // fully hoisted
  return "Hello!";
}

console.log(a);        // undefined
// 'b' is in TDZ here
console.log(myFunc()); // "Hello!"
console.log(myExpr()); // TypeError

a = 10;
let b = 20;
myExpr = function() { return "Hi!"; };
```

### 🔥 Tricky Interview Question:
```javascript
var x = 1;
function foo() {
  console.log(x); // undefined (local 'x' is hoisted inside foo)
  var x = 2;
  console.log(x); // 2
}
foo();
```

---

## 2. Closures

### Theory
A **closure** is a function that **remembers** and has access to variables from its **outer (lexical) scope** even after the outer function has returned. It creates a persistent scope.

**Three scope chains:**
- Own scope
- Outer function's variables
- Global variables

```javascript
// ✅ Basic Closure
function outer() {
  let count = 0; // enclosed variable
  
  return function inner() {
    count++;
    console.log(count);
  };
}

const counter = outer();
counter(); // 1
counter(); // 2
counter(); // 3
// 'count' persists in memory because inner() has a reference to it
```

### Practical Use Cases:

```javascript
// ✅ Data Privacy / Encapsulation (Module Pattern)
function createBankAccount(initialBalance) {
  let balance = initialBalance; // private variable

  return {
    deposit(amount) {
      balance += amount;
      return `Balance: ${balance}`;
    },
    withdraw(amount) {
      if (amount > balance) return "Insufficient funds";
      balance -= amount;
      return `Balance: ${balance}`;
    },
    getBalance() {
      return balance;
    }
  };
}

const account = createBankAccount(100);
console.log(account.deposit(50));     // "Balance: 150"
console.log(account.withdraw(30));    // "Balance: 120"
console.log(account.balance);         // undefined (private!)
```

### 🔥 Classic Interview Trap:
```javascript
// ❌ Problem: All print 3
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 1000);
}
// Output: 3, 3, 3 (var is function-scoped, loop finishes before timeout)

// ✅ Fix 1: Use let (block-scoped)
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 1000);
}
// Output: 0, 1, 2

// ✅ Fix 2: Use IIFE (Closure)
for (var i = 0; i < 3; i++) {
  (function(j) {
    setTimeout(() => console.log(j), 1000);
  })(i);
}
// Output: 0, 1, 2

// ✅ Fix 3: Use setTimeout's third parameter
for (var i = 0; i < 3; i++) {
  setTimeout((j) => console.log(j), 1000, i);
}
```

---

## 3. Event Loop & Asynchronous JavaScript

### Theory
JavaScript is **single-threaded** with a **non-blocking** concurrency model. The Event Loop manages execution using:

| Component | Role |
|-----------|------|
| **Call Stack** | Executes functions (LIFO) |
| **Web APIs** | Handles async operations (setTimeout, fetch, DOM events) |
| **Microtask Queue** | Promises, MutationObserver, queueMicrotask (HIGH priority) |
| **Macrotask Queue** | setTimeout, setInterval, I/O (LOW priority) |

**Execution order:** Call Stack → **All Microtasks** → **One Macrotask** → Repeat

```javascript
console.log("1"); // Sync - Call Stack

setTimeout(() => {
  console.log("2"); // Macrotask Queue
}, 0);

Promise.resolve().then(() => {
  console.log("3"); // Microtask Queue
});

queueMicrotask(() => {
  console.log("4"); // Microtask Queue
});

console.log("5"); // Sync - Call Stack

// Output: 1, 5, 3, 4, 2
```

### 🔥 Advanced Event Loop Question:
```javascript
console.log("start");

setTimeout(() => console.log("timeout 1"), 0);

Promise.resolve()
  .then(() => {
    console.log("promise 1");
    setTimeout(() => console.log("timeout 2"), 0);
  })
  .then(() => console.log("promise 2"));

setTimeout(() => {
  console.log("timeout 3");
  Promise.resolve().then(() => console.log("promise 3"));
}, 0);

console.log("end");

// Output: start, end, promise 1, promise 2, timeout 1, timeout 3, promise 3, timeout 2
```

---

## 4. Prototypal Inheritance

### Theory
JavaScript uses **prototypal inheritance** — objects can inherit directly from other objects. Every object has an internal `[[Prototype]]` link accessible via `__proto__` or `Object.getPrototypeOf()`.

**Prototype Chain:** When accessing a property, JS looks up the prototype chain until it finds the property or reaches `null`.

```javascript
// ✅ Constructor Function Pattern
function Animal(name) {
  this.name = name;
}
Animal.prototype.speak = function() {
  return `${this.name} makes a sound`;
};

function Dog(name, breed) {
  Animal.call(this, name); // Call parent constructor
  this.breed = breed;
}

// Set up inheritance
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog; // Fix constructor reference

Dog.prototype.bark = function() {
  return `${this.name} barks!`;
};

const dog = new Dog("Rex", "German Shepherd");
console.log(dog.speak()); // "Rex makes a sound" (inherited)
console.log(dog.bark());  // "Rex barks!" (own method)
console.log(dog instanceof Dog);    // true
console.log(dog instanceof Animal); // true
```

```javascript
// ✅ ES6 Class Syntax (syntactic sugar over prototypes)
class Animal {
  constructor(name) {
    this.name = name;
  }
  speak() {
    return `${this.name} makes a sound`;
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name);
    this.breed = breed;
  }
  bark() {
    return `${this.name} barks!`;
  }
}
```

```javascript
// ✅ Prototype chain visualization
console.log(dog.__proto__ === Dog.prototype);              // true
console.log(dog.__proto__.__proto__ === Animal.prototype);  // true
console.log(dog.__proto__.__proto__.__proto__ === Object.prototype); // true
console.log(dog.__proto__.__proto__.__proto__.__proto__);   // null
```

---

## 5. `this` Keyword

### Theory
`this` refers to the **execution context** — its value depends on **how** a function is called, not where it's defined.

| Call Pattern | `this` Value |
|---|---|
| Global / Regular function | `window` (or `undefined` in strict mode) |
| Object method | The object |
| `new` keyword | New instance |
| `call/apply/bind` | Explicitly set |
| Arrow function | Lexical (from enclosing scope) |
| Event handler | The element |

```javascript
// ✅ Different contexts
const obj = {
  name: "Alice",
  
  // Regular method - 'this' = obj
  greet() {
    console.log(this.name); // "Alice"
  },
  
  // Arrow function - 'this' = enclosing scope (NOT obj)
  greetArrow: () => {
    console.log(this.name); // undefined (this = window/global)
  },
  
  // Nested function problem
  greetDelayed() {
    setTimeout(function() {
      console.log(this.name); // undefined (this = window)
    }, 100);
    
    // Fix: Arrow function inherits 'this'
    setTimeout(() => {
      console.log(this.name); // "Alice" ✅
    }, 100);
  }
};
```

### 🔥 Tricky Interview Question:
```javascript
const obj = {
  name: "Object",
  getName: function() {
    return this.name;
  }
};

const getName = obj.getName;

console.log(obj.getName());  // "Object" (method call)
console.log(getName());       // undefined (simple function call, this = window)

// Fix:
const boundGetName = obj.getName.bind(obj);
console.log(boundGetName()); // "Object"
```

```javascript
// ✅ 'this' with new keyword
function Person(name) {
  this.name = name;
  this.greet = () => {
    console.log(this.name); // Arrow captures 'this' from constructor
  };
}

const person = new Person("Bob");
const greet = person.greet;
greet(); // "Bob" ✅ (arrow function retains 'this')
```

---

## 6. Scope & Scope Chain

### Theory
**Scope** determines the accessibility of variables. JavaScript has 3 types:

| Scope Type | Keyword | Accessible Where |
|---|---|---|
| **Global** | var/let/const | Everywhere |
| **Function** | var | Within the function |
| **Block** | let/const | Within `{}` block |

```javascript
var globalVar = "global";

function outer() {
  var outerVar = "outer";
  
  function inner() {
    let innerVar = "inner";
    
    console.log(innerVar);  // ✅ "inner" (own scope)
    console.log(outerVar);  // ✅ "outer" (parent scope)
    console.log(globalVar); // ✅ "global" (global scope)
  }
  
  inner();
  console.log(innerVar); // ❌ ReferenceError
}

outer();
console.log(outerVar); // ❌ ReferenceError
```

```javascript
// ✅ Block Scope difference: var vs let/const
if (true) {
  var a = 1;   // Function-scoped (leaks out of block)
  let b = 2;   // Block-scoped
  const c = 3; // Block-scoped
}

console.log(a); // 1 ✅
console.log(b); // ❌ ReferenceError
console.log(c); // ❌ ReferenceError
```

### Lexical Scope:
```javascript
// Functions are scoped where they are DEFINED, not where they are CALLED
let x = 10;

function foo() {
  console.log(x); // 10 (lexical scope - where foo was defined)
}

function bar() {
  let x = 20;
  foo(); // Still prints 10, NOT 20
}

bar();
```

---

## 7. Call, Apply, Bind

### Theory
These methods allow you to **explicitly set `this`** for a function.

| Method | Invokes Immediately? | Arguments |
|---|---|---|
| `call()` | ✅ Yes | Comma-separated |
| `apply()` | ✅ Yes | Array |
| `bind()` | ❌ Returns new function | Comma-separated |

```javascript
const person = { name: "Alice" };

function introduce(greeting, punctuation) {
  return `${greeting}, I'm ${this.name}${punctuation}`;
}

// call - individual arguments
console.log(introduce.call(person, "Hello", "!"));
// "Hello, I'm Alice!"

// apply - array of arguments
console.log(introduce.apply(person, ["Hi", "."]));
// "Hi, I'm Alice."

// bind - returns a new function
const boundIntroduce = introduce.bind(person, "Hey");
console.log(boundIntroduce("!!"));
// "Hey, I'm Alice!!" (partial application)
```

### Practical Use Cases:
```javascript
// ✅ Borrowing methods
const numbers = [5, 6, 2, 3, 7];
const max = Math.max.apply(null, numbers); // 7
// Modern alternative: Math.max(...numbers)

// ✅ Converting array-like objects
function example() {
  const args = Array.prototype.slice.call(arguments);
  // Modern: const args = Array.from(arguments) or [...arguments]
  console.log(args);
}

// ✅ Custom bind polyfill (Interview favorite!)
Function.prototype.myBind = function(context, ...args) {
  const fn = this;
  return function(...newArgs) {
    return fn.apply(context, [...args, ...newArgs]);
  };
};
```

---

## 8. Promises & Async/Await

### Theory
Promises represent the **eventual completion or failure** of an async operation.

**States:** `pending` → `fulfilled` OR `rejected` (settled, immutable)

```javascript
// ✅ Promise Creation & Chaining
function fetchUser(id) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (id > 0) {
        resolve({ id, name: "Alice" });
      } else {
        reject(new Error("Invalid ID"));
      }
    }, 1000);
  });
}

fetchUser(1)
  .then(user => {
    console.log(user); // { id: 1, name: "Alice" }
    return fetchUser(2);
  })
  .then(user2 => console.log(user2))
  .catch(err => console.error(err.message))
  .finally(() => console.log("Done"));
```

### Promise Static Methods:
```javascript
const p1 = Promise.resolve(1);
const p2 = new Promise(resolve => setTimeout(() => resolve(2), 1000));
const p3 = Promise.resolve(3);
const pFail = Promise.reject("Error!");

// ✅ Promise.all - ALL must resolve (fails fast)
Promise.all([p1, p2, p3])
  .then(results => console.log(results)); // [1, 2, 3]

// ✅ Promise.allSettled - Waits for ALL regardless of outcome
Promise.allSettled([p1, pFail, p3])
  .then(results => console.log(results));
// [{status:"fulfilled", value:1}, {status:"rejected", reason:"Error!"}, {status:"fulfilled", value:3}]

// ✅ Promise.race - First to settle wins
Promise.race([p1, p2])
  .then(result => console.log(result)); // 1 (p1 resolves first)

// ✅ Promise.any - First to RESOLVE wins (ignores rejections)
Promise.any([pFail, p2, p3])
  .then(result => console.log(result)); // 3
```

### Async/Await:
```javascript
// ✅ Cleaner syntax with error handling
async function getUserData() {
  try {
    const user = await fetchUser(1);
    const posts = await fetchPosts(user.id); // Sequential
    return { user, posts };
  } catch (error) {
    console.error("Failed:", error.message);
    throw error; // Re-throw if needed
  }
}

// ✅ Parallel execution with async/await
async function getDashboardData() {
  const [users, posts, comments] = await Promise.all([
    fetchUsers(),
    fetchPosts(),
    fetchComments()
  ]);
  return { users, posts, comments };
}
```

### 🔥 Interview Question: Promise Execution Order
```javascript
async function foo() {
  console.log("foo start");
  await bar();
  console.log("foo end"); // This becomes a microtask
}

async function bar() {
  console.log("bar");
}

console.log("script start");
foo();
console.log("script end");

// Output: script start, foo start, bar, script end, foo end
```

---

## 9. Debouncing & Throttling

### Theory
Both are techniques to **control function execution frequency**.

| Technique | Behavior | Use Case |
|---|---|---|
| **Debounce** | Executes after caller **stops** calling for N ms | Search input, resize |
| **Throttle** | Executes at most **once** every N ms | Scroll, mousemove |

```javascript
// ✅ Debounce Implementation
function debounce(fn, delay) {
  let timerId;
  
  return function(...args) {
    clearTimeout(timerId);
    timerId = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  };
}

// Usage
const searchInput = document.getElementById("search");
const handleSearch = debounce((e) => {
  console.log("API call:", e.target.value);
}, 300);

searchInput.addEventListener("input", handleSearch);
// Only fires 300ms after the user STOPS typing
```

```javascript
// ✅ Throttle Implementation
function throttle(fn, limit) {
  let inThrottle = false;
  
  return function(...args) {
    if (!inThrottle) {
      fn.apply(this, args);
      inThrottle = true;
      setTimeout(() => {
        inThrottle = false;
      }, limit);
    }
  };
}

// Usage
const handleScroll = throttle(() => {
  console.log("Scroll event processed");
}, 200);

window.addEventListener("scroll", handleScroll);
// Fires at most once every 200ms during scrolling
```

```javascript
// ✅ Advanced Debounce with Leading Edge & Cancel
function debounce(fn, delay, { leading = false } = {}) {
  let timerId;
  
  function debounced(...args) {
    if (!timerId && leading) {
      fn.apply(this, args);
    }
    clearTimeout(timerId);
    timerId = setTimeout(() => {
      if (!leading) fn.apply(this, args);
      timerId = null;
    }, delay);
  }
  
  debounced.cancel = () => {
    clearTimeout(timerId);
    timerId = null;
  };
  
  return debounced;
}
```

---

## 10. Event Delegation & Propagation

### Theory
**Event Propagation** has 3 phases:
1. **Capturing** (top → target)
2. **Target** (the element itself)
3. **Bubbling** (target → top) ← default

**Event Delegation:** Attach ONE listener to a parent instead of multiple listeners on children.

```javascript
// ❌ Bad: Individual listeners on each button
document.querySelectorAll(".btn").forEach(btn => {
  btn.addEventListener("click", handleClick);
});

// ✅ Good: Event delegation on parent
document.getElementById("button-container").addEventListener("click", (e) => {
  if (e.target.matches(".btn")) {
    console.log("Button clicked:", e.target.textContent);
  }
  // Also works for dynamically added elements!
});
```

```javascript
// ✅ Propagation Control
element.addEventListener("click", (e) => {
  e.stopPropagation();        // Stops bubbling to parent
  e.stopImmediatePropagation(); // Stops other listeners on SAME element too
  e.preventDefault();         // Prevents default browser behavior
});

// Capturing phase
parent.addEventListener("click", handler, true); // 3rd arg = useCapture
// OR
parent.addEventListener("click", handler, { capture: true });
```

```javascript
// ✅ Practical: Dynamic list with delegation
const ul = document.getElementById("todo-list");

ul.addEventListener("click", (e) => {
  // Delete button
  if (e.target.matches(".delete-btn")) {
    e.target.closest("li").remove();
  }
  // Edit button
  if (e.target.matches(".edit-btn")) {
    const li = e.target.closest("li");
    // ... edit logic
  }
});
```

---

## 11. Shallow vs Deep Copy

### Theory
| Copy Type | Behavior | Nested Objects |
|---|---|---|
| **Shallow** | Copies first level only | Shared reference |
| **Deep** | Copies all levels | Independent |

```javascript
const original = {
  name: "Alice",
  address: { city: "NYC", zip: "10001" },
  hobbies: ["reading", "coding"]
};

// ✅ Shallow Copy Methods
const shallow1 = { ...original };
const shallow2 = Object.assign({}, original);

shallow1.name = "Bob";          // ✅ Doesn't affect original
shallow1.address.city = "LA";   // ❌ ALSO changes original!

console.log(original.address.city); // "LA" 😱
```

```javascript
// ✅ Deep Copy Methods

// Method 1: structuredClone (Modern - Best)
const deep1 = structuredClone(original);

// Method 2: JSON parse/stringify (Limitations!)
const deep2 = JSON.parse(JSON.stringify(original));
// ⚠️ Loses: Functions, undefined, Date objects, RegExp, Map, Set, Infinity, NaN

// Method 3: Custom recursive deep clone
function deepClone(obj) {
  if (obj === null || typeof obj !== "object") return obj;
  if (obj instanceof Date) return new Date(obj);
  if (obj instanceof RegExp) return new RegExp(obj);
  if (obj instanceof Map) return new Map([...obj].map(([k, v]) => [k, deepClone(v)]));
  if (obj instanceof Set) return new Set([...obj].map(v => deepClone(v)));
  
  const clone = Array.isArray(obj) ? [] : {};
  
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      clone[key] = deepClone(obj[key]);
    }
  }
  
  return clone;
}
```

```javascript
// ⚠️ JSON.stringify limitations demo
const problematic = {
  fn: () => "hello",       // ❌ Lost
  undef: undefined,        // ❌ Lost
  date: new Date(),        // ❌ Becomes string
  nan: NaN,                // ❌ Becomes null
  infinity: Infinity,      // ❌ Becomes null
  regex: /test/gi,         // ❌ Becomes {}
  map: new Map([[1, 2]]),  // ❌ Becomes {}
};

const jsonClone = JSON.parse(JSON.stringify(problematic));
console.log(jsonClone);
// { date: "2024-01-01T...", nan: null, infinity: null, regex: {}, map: {} }
```

---

## 12. Currying

### Theory
**Currying** transforms a function with multiple arguments into a sequence of functions each taking **one argument**.

`f(a, b, c)` → `f(a)(b)(c)`

```javascript
// ✅ Basic Currying
function multiply(a) {
  return function(b) {
    return function(c) {
      return a * b * c;
    };
  };
}

console.log(multiply(2)(3)(4)); // 24

// Arrow function version
const multiply = a => b => c => a * b * c;
```

```javascript
// ✅ Practical Use Cases

// 1. Reusable utility functions
const withTax = rate => price => price + price * rate;

const withGST = withTax(0.18);
const withVAT = withTax(0.20);

console.log(withGST(100)); // 118
console.log(withVAT(100)); // 120

// 2. Logger with context
const logger = level => module => message => 
  console.log(`[${level}] [${module}]: ${message}`);

const errorLogger = logger("ERROR");
const authError = errorLogger("AUTH");

authError("Invalid token");  // [ERROR] [AUTH]: Invalid token
authError("Session expired"); // [ERROR] [AUTH]: Session expired
```

```javascript
// ✅ Generic Curry Utility (Interview favorite!)
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    return function(...nextArgs) {
      return curried.apply(this, [...args, ...nextArgs]);
    };
  };
}

// Usage
function sum(a, b, c) {
  return a + b + c;
}

const curriedSum = curry(sum);

console.log(curriedSum(1)(2)(3));     // 6
console.log(curriedSum(1, 2)(3));     // 6
console.log(curriedSum(1)(2, 3));     // 6
console.log(curriedSum(1, 2, 3));     // 6
```

### 🔥 Interview Question: Infinite Currying
```javascript
// sum(1)(2)(3)...(n)() — returns total when called with no args
function sum(a) {
  return function(b) {
    if (b !== undefined) {
      return sum(a + b);
    }
    return a;
  };
}

console.log(sum(1)(2)(3)(4)()); // 10
```

---

## 13. Generator Functions

### Theory
Generators are functions that can be **paused and resumed**. They use `function*` syntax and `yield` keyword. They return an **iterator** object.

```javascript
// ✅ Basic Generator
function* numberGenerator() {
  console.log("Start");
  yield 1;
  console.log("After first yield");
  yield 2;
  console.log("After second yield");
  yield 3;
  console.log("End");
}

const gen = numberGenerator();
console.log(gen.next()); // "Start"         → { value: 1, done: false }
console.log(gen.next()); // "After first"   → { value: 2, done: false }
console.log(gen.next()); // "After second"  → { value: 3, done: false }
console.log(gen.next()); // "End"           → { value: undefined, done: true }
```

```javascript
// ✅ Infinite sequence
function* idGenerator() {
  let id = 1;
  while (true) {
    yield id++;
  }
}

const getId = idGenerator();
console.log(getId.next().value); // 1
console.log(getId.next().value); // 2
console.log(getId.next().value); // 3

// ✅ Two-way communication
function* calculator() {
  const a = yield "Enter first number";
  const b = yield "Enter second number";
  return a + b;
}

const calc = calculator();
console.log(calc.next());      // { value: "Enter first number", done: false }
console.log(calc.next(10));    // { value: "Enter second number", done: false }
console.log(calc.next(20));    // { value: 30, done: true }
```

```javascript
// ✅ Practical: Paginated API calls
function* paginatedFetch(url, pageSize) {
  let page = 1;
  while (true) {
    const response = yield fetch(`${url}?page=${page}&size=${pageSize}`);
    if (response.data.length === 0) return;
    page++;
  }
}
```

---

## 14. WeakMap & WeakSet

### Theory
| Feature | Map/Set | WeakMap/WeakSet |
|---|---|---|
| Key types | Any | **Objects only** |
| Enumerable | Yes (iterable) | No |
| Size property | Yes | No |
| Garbage Collection | Prevents GC | **Allows GC** (weak reference) |

```javascript
// ✅ WeakMap - Private data & caching
const privateData = new WeakMap();

class User {
  constructor(name, age) {
    privateData.set(this, { name, age }); // Truly private
  }
  
  getName() {
    return privateData.get(this).name;
  }
}

let user = new User("Alice", 30);
console.log(user.getName()); // "Alice"
console.log(user.name);     // undefined (private!)

user = null; // Object is garbage collected, WeakMap entry is automatically removed
```

```javascript
// ✅ WeakMap for Caching
const cache = new WeakMap();

function expensiveComputation(obj) {
  if (cache.has(obj)) {
    console.log("From cache");
    return cache.get(obj);
  }
  
  const result = /* heavy computation */ obj.data * 2;
  cache.set(obj, result);
  return result;
}

let data = { data: 42 };
expensiveComputation(data); // Computes
expensiveComputation(data); // "From cache"
data = null; // Cache entry auto-cleaned by GC
```

```javascript
// ✅ WeakSet - Track visited/processed objects
const visited = new WeakSet();

function processNode(node) {
  if (visited.has(node)) return; // Already processed
  
  visited.add(node);
  // ... process node
}
```

---

## 15. Proxy & Reflect

### Theory
**Proxy** creates a wrapper around an object to intercept and customize fundamental operations (get, set, delete, etc.).

**Reflect** provides methods for interceptable JavaScript operations — same methods as Proxy traps.

```javascript
// ✅ Basic Proxy - Validation
const validator = {
  set(target, property, value) {
    if (property === "age") {
      if (typeof value !== "number") throw TypeError("Age must be a number");
      if (value < 0 || value > 150) throw RangeError("Age must be 0-150");
    }
    target[property] = value;
    return true;
  },
  
  get(target, property) {
    if (property in target) {
      return target[property];
    }
    throw new ReferenceError(`Property '${property}' doesn't exist`);
  }
};

const person = new Proxy({}, validator);
person.age = 25;    // ✅ Works
person.age = -1;    // ❌ RangeError
person.age = "old"; // ❌ TypeError
person.foo;         // ❌ ReferenceError
```

```javascript
// ✅ Reactive System (Vue.js-like reactivity)
function reactive(obj) {
  const handlers = new Map();
  
  return new Proxy(obj, {
    get(target, key) {
      // Track dependency
      console.log(`Getting ${key}`);
      return Reflect.get(target, key);
    },
    set(target, key, value) {
      const oldValue = target[key];
      const result = Reflect.set(target, key, value);
      if (oldValue !== value) {
        console.log(`${key} changed: ${oldValue} → ${value}`);
        // Trigger re-render
      }
      return result;
    }
  });
}

const state = reactive({ count: 0, name: "Alice" });
state.count = 1; // "count changed: 0 → 1"
```

```javascript
// ✅ Negative Array Indexing (Python-like)
function createArray(...elements) {
  return new Proxy(elements, {
    get(target, prop) {
      const index = Number(prop);
      if (Number.isInteger(index) && index < 0) {
        prop = target.length + index;
      }
      return Reflect.get(target, prop);
    }
  });
}

const arr = createArray(1, 2, 3, 4, 5);
console.log(arr[-1]); // 5
console.log(arr[-2]); // 4
```

---

## 🎯 Bonus: Quick-Fire Interview Questions

```javascript
// Q1: What's the output?
console.log(typeof null);         // "object" (historical bug)
console.log(typeof undefined);    // "undefined"
console.log(typeof NaN);          // "number"
console.log(null == undefined);   // true (loose equality)
console.log(null === undefined);  // false (strict equality)
console.log(NaN === NaN);         // false

// Q2: What's the output?
console.log(0.1 + 0.2 === 0.3);  // false (floating point)
console.log(0.1 + 0.2);          // 0.30000000000000004

// Q3: What's the output?
console.log([] == ![]);           // true 🤯
// [] == ![] → [] == false → 0 == 0 → true

// Q4: What's the output?
console.log("5" - 3);   // 2 (string coerced to number)
console.log("5" + 3);   // "53" (number coerced to string)
console.log(true + true); // 2

// Q5: Object comparison
console.log({} === {});   // false (different references)
const a = {};
const b = a;
console.log(a === b);     // true (same reference)
```

---

## 📌 Summary Cheat Sheet

| Concept | Key Takeaway |
|---|---|
| **Hoisting** | Declarations move up; `let`/`const` have TDZ |
| **Closures** | Functions remember their lexical scope |
| **Event Loop** | Microtasks > Macrotasks; single-threaded |
| **Prototypes** | Objects inherit from other objects |
| **this** | Depends on HOW function is called |
| **Scope** | Lexical scoping; `let`/`const` are block-scoped |
| **call/apply/bind** | Explicit `this` binding |
| **Promises** | Async handling; microtask queue |
| **Debounce/Throttle** | Control execution frequency |
| **Event Delegation** | One parent listener instead of many |
| **Deep Copy** | Use `structuredClone()` in modern JS |
| **Currying** | Transform multi-arg to single-arg chain |
| **Generators** | Pausable functions with `yield` |
| **WeakMap/WeakSet** | Allow garbage collection |
| **Proxy** | Intercept object operations |

> 💡 **Pro Tip:** For a 4-year experienced role, interviewers expect you to not only know these concepts but also explain **real-world use cases**, **performance implications**, and **trade-offs**.
