+++
title = "JavaScript Best Practices and Principles"
LastModifierDisplayName = "Alain Bouchard"
LastModifierEmail = "abouchard@live.ca"
disableToc = "false"
+++

{{< toc >}}

It is easy to fall in a trap when coding in JavaScript.  Thus, it is highly suggested to add types and use Typescript.  Nevertheless, there are the Best Practices and Principles to follow with Javascript.  Many can be used with other languages like Typescript.

## Best Practices

Here are some best practices for javascript coding, in order to maintain clean, efficient and error-free code.

### Use const and let, avoid using var

- `const` is used for variables, or constants, whose values shouldn't change.
- `let` is used for variables that may change.
- Avoid `var` due to its function-scoping, which can cause unexpected behavior

#### Function Scope vs Block Scope

The `var `is function-scoped, meaning that it is accessible within the function in which it is declared, regardless of the block it's in, for example, within an `if` or `for` statement.

The `let` and `const` are block-scoped, meaning they are only accessible within the block they are defined in.

Example:

```js
function testScope() {
    if (true) {
        var functionScoped = "I'm accessible throughout the function";
        let blockScoped = "I'm accessible only within this block";
    }
    console.log(functionScoped); // Works
    console.log(blockScoped); // ReferenceError: blockScoped is not defined
}
```

#### Hoisting (or raised)

The `var` declarations are `hoisted` to the top of their scope, which means that Javascript moves all `var` declarations to the top of the function scope at runtime; warning: only declaration is hoisted, not the initialization.

This can lead to confusing bugs, especially if the `var` is used before it's initialization.  In this case, value will be `undefined` instead of causing an error.

Example:
```js
console.log(myVar); // undefined (hoisted declaration)
var myVar = "Hello"; // initialized here
```

By contrast, `let` and `const` are also hoisted, but they remain in a `temporal dead zone` until they are actually declared in code.  Accessing them before their declaration will cause a `ReferenceError`, meaning it easier to catch potential errors in the code.

Example:
```js
console.log(myLet); // ReferenceError: Cannot access 'myLet' before initialization
let myLet = "Hello";
```

#### Re-declaration Issues

The `var` allows re-declaration within the same scope, which can cause unintended variable overwrites and bugs in the code.

The `let` and `const` do not allow re-declaration within same scope.

Example:
```js
var message = "Hello";
var message = "World"; // Re-declaration allowed with var
console.log(message); // "World"

let greeting = "Hi";
let greeting = "Hello"; // SyntaxError: Identifier 'greeting' has already been declared
```

### Write Modular, Reusable Code

- Break down code into smaller, reusable functions or modules.
- Use the Single Responsibility Principle (SRP): each function or module should handle one task.
- Consider using the `module pattern` or `ES6 modules` to organize code.

Organizing Javascript code using the `module pattern` or `ES6 modules` helps manage code structure, making it more maintainable, modular, and reusable:

#### Module Pattern

The Module Pattern is a design pattern in javascript that allows you to group related functions, variables and objects into a single unit (or a `module`), usually returning an object that expose only what needs to be accessible externally while keeping the rest private.
- This pattern relies only  on `closures` to create private and public members within a function, providing `encapsulation` to avoid global scope pollution and project internal details from external access.

Example:
```js
const CounterModule = (function () {
    // Private variables and functions
    let counter = 0;

    function increment() {
        counter++;
    }

    function decrement() {
        counter--;
    }

    // Public API: exposed by returning an object
    return {
        getCounter: function () {
            return counter;
        },
        incrementCounter: function () {
            increment();
        },
        decrementCounter: function () {
            decrement();
        },
    };
})();

// Usage
CounterModule.incrementCounter();
console.log(CounterModule.getCounter()); // 1
``` 

In this example, counter, increment and decrement are private (can't be accessed directly outside the module), while the getCounter, incrementCounter and decrementCounter methods are public.

#### ES5 Modules

- `ES6 Modules (or ECMAScript 2015)` introduced a built-in module system for Javascript, designed to support modular code directly in the language. This system provides `import` and `export` keywords to allow easy code sharing between files.
- With ES6 modules, each file is treated as a `separated module` with its own scope.  Variables, functions and classes can be exported from one file and imported into another.  This improves code readability and allows for cleaner dependencies without global variable conflicts.

Example:
```js
// matUtils.js
// Named exports
export function add(a, b) {
    return a + b;
}

export function subtract(a, b) {
    return a - b;
}

// Default export
export default function multiply(a, b) {
    return a * b;
}
```

```js
// app.js
// Importing named exports
import { add, subtract } from './mathUtils.js';

// Importing default export
import multiply from './mathUtils.js';

console.log(add(2, 3)); // 5
console.log(subtract(5, 3)); // 2
console.log(multiply(4, 2)); // 8
```

Notes:
- ES6 modules are automatically `strict mode` meaning undeclared variables or unsafe operations are disallowed bu default.
- Unlike the module pattern, ES6 modules are asynchronous, and they use `tree-shaking` to removed unused exports, while helps optimize performance by reducing bundle size in production. `tree-shaking` is an optimization technique used in Javascript bundlers (like webpack, rollup and ESBuild) to remove unused code (dead code) from final bundle, making applications faster and more efficient.

#### When?

The `module pattern`is ideal if you're working in an environment where ES6 modules are not supported (e.g., older environments) or if you want to create an immediately usable object with private/public members without importing/exporting between files. 

The ÈS6 Modules` are recommended for most modern Javascript application, especially for object using build tools like webpack, vite, rollup, etc.  ES6 modules are now the standard way of organizing Javascript code in both client-side and server-side applications, including framework like React, Vue, and Node.js.

### Use Arrow Functions (=>)

- Arrow functions are concise and **don't have** their own `this` context, making them useful for callbacks and simpler functions.
- `Warning`: avoid arrow functions in methods if you need a function with its own `this` context.

#### Use arrows functions for simple callbacks.

Arrows functions make callbacks, especially inline ones, more concise and readable:

```js
// Good
const numbers = [1, 2, 3];
const squares = numbers.map(num => num * num);

// Avoid (excessive verbosity for a simple callback)
const squaresOld = numbers.map(function(num) {
  return num * num;
});
```

The practices help to make the code clearer, more predictable, and avoid common pitfalls with `this` binding.

#### Avoid arrow functions for methods in classes

Use regular function expressions for defining methods in classes, as arrow functions fo not have their own this context:

```js
class Counter {
  count = 0;

  // Good (using regular function for method)
  increment() {
    this.count += 1;
  }
}

// Avoid (arrow functions lack their own `this`)
class CounterBad {
  count = 0;
  increment = () => {
    this.count += 1;
  }
}
```

#### Use arrow functions to maintain `this“ context in callbacks

When working with callbacks in a class, arrow functions can be beneficial for preserving the `this` context:

```js
class Timer {
  constructor() {
    this.seconds = 0;
  }

  start() {
    setInterval(() => {
      this.seconds += 1; // `this` refers to Timer instance
    }, 1000);
  }
}
```

### Consistent Naming Conventions

- Use `camelCase` for variables and functions: `const userAge = 25;` 
- Use `PascalCase` for classes and constructors: `class UserAccount{ ... }`
- Use prefix boolean variables with `is`, `has`, `should` and `can`:
    ```js 
    // Good
    let isLoggedIn = false;
    const hasAccess = true;

    // Avoid
    let loggedIn = false;
    const accessGranted = true;
    ```

- Use descriptive names for variables, functions and classes:
    ```js
    // Good
    const calculateTotalPrice = (items) => { /* ... */ };
    let itemList = ['apple', 'orange', 'banana'];

    // Avoid (unclear purpose)
    const calcTP = (arr) => { /* ... */ };
    let arr = ['apple', 'orange', 'banana'];
    ````

- Use descriptive names for Constants, use UPPER_SNAKE_CASE for constants that represent values unlikely to change (e.g., configuration values), this helps indicate immutability:
    ```js
    // Good
    const MAX_USER_COUNT = 100;
    const API_BASE_URL = 'https://api.example.com';

    // Avoid
    const MaxUserCount = 100;
    const apiBaseUrl = 'https://api.example.com';
    ```

- Use consistent naming convention for Asynchronous Functions (`async`/`fetch`), when working with async functions, prefix the function names with `fetch`, `get`, `load` or `retrieve` to indicate the nature of the function:
    ```js
    // Good
    async function fetchUserData() { /* ... */ }
    const loadSettings = async () => { /* ... */ };

    // Avoid
    async function getData() { /* ... */ }
    ```

- Use `handle` or `on` prefixes with event handler functions to make their purpose clear:
    ```js
    // Good
    async function fetchUserData() { /* ... */ }
    const loadSettings = async () => { /* ... */ };

    // Avoid
    async function getData() { /* ... */ }
    ```

### Error Handling

- Always add error handling in asynchronous code (`try/catch` blocks with `async/await`).
- Use custom error messages to make debugging easier.

Example:
```js
// Asynchronous function with proper error handling
async function fetchUserData(userId) {
  try {
    const response = await fetch(`https://api.example.com/users/${userId}`);
    
    if (!response.ok) {
      throw new Error(`Failed to fetch data: ${response.status} ${response.statusText}`);
    }
    
    const data = await response.json();
    return data;
  } catch (error) {
    // Custom error message for debugging
    console.error(`Error in fetchUserData: ${error.message}`);
    throw new Error(`Unable to retrieve user data for ID: ${userId}`);
  }
}

// Example usage of the function
fetchUserData(123)
  .then(data => console.log(data))
  .catch(error => console.log(error.message));
```

### Use Template Literals for String Concatenation

- Instead of using + to concatenate strings, use template literals:
```js
// Good
const firstName = "Alain";
const lastName = "Bouchard";
const greeting = `Hello, ${firstName} ${lastName}! Welcome.`;

// Avoid
const greetingOld = "Hello, " + firstName + " " + lastName + "! Welcome.";
```

### Comment and Document the Code

- Write comments for complex or unclear code blocks, but avoid redundant comments.
- Use JSDoc-style comments for functions and classes to describe their purpose, parameters and return values.

Example:
```js
/**
 * Calculates the total price of items with tax.
 *
 * @param {number[]} prices - Array of item prices.
 * @param {number} taxRate - Tax rate to be applied (e.g., 0.08 for 8%).
 * @returns {number} Total price after tax.
 */
function calculateTotalPrice(prices, taxRate) {
  const subtotal = prices.reduce((total, price) => total + price, 0);
  return subtotal * (1 + taxRate);
}

// Example usage
const items = [10, 20, 30];
const total = calculateTotalPrice(items, 0.08);
console.log(total);
```

### Prefer Destructuring for Objects and Arrays

- Destructuring improves readability and reduces repetitive code:

Example:
```js
const user = { name: 'Alain', age: 30 };
const { name, age } = user;
```

### Use Promises and `async/await` for Asynchronous Code

- Avoid callback hell bu using Promises or async/await.
- handle errors with `try/catch` blocks around async functions.

Example:
```js
// Function using async/await with error handling
async function fetchData(url) {
  try {
    const response = await fetch(url);
    const data = await response.json();
    return data;
  } catch (error) {
    console.error("Error fetching data:", error);
    throw error; // Re-throw to handle it in calling code if needed
  }
}

// Example usage
fetchData("https://api.example.com/data")
  .then(data => console.log(data))
  .catch(error => console.log("Fetch failed:", error.message));
```

### Use Strict Mode

- Enable Strict Mode by adding `use strict;` at the beginning of your scripts.  This mode prevents the use of undeclared variables and other common Javascript pitfalls:

```js
"use strict";

function calculateArea(radius) {
  // Prevents usage of undeclared variables
  pi = 3.14159; // ReferenceError: pi is not defined
  return pi * radius * radius;
}

calculateArea(5);
```

Note: Strict Mode isn't required with Typescript as TS inherently enforces many of the same safeguards.

### Write Unit Tests

- Write unit tests for functions and components to verify code behavior.
- Use frameworks like Jest, Mocha, or Cypress (for FE and E2E testing).

### Use ESLint and Prettier for Code Quality and Formatting

- ESLint helps catch common errors and enforces a consistent style.
- Prettier ensures that code is formatted consistently across your project.

### Avoid Global Variables

- Avoid polluting the global namespace by declaring variables and functions at the global scope. Use closures aor modules instead.

Example:
```js
// Avoid: Global variables and functions
let count = 0; // Global variable

function increment() {
  count++;
  console.log("Count:", count);
}

function reset() {
  count = 0;
  console.log("Count reset.");
}

// Usage
increment(); // Count: 1
increment(); // Count: 2
reset();     // Count reset.
```

Example using an Immediately-Invoked Function Expression (IIFE) to create a closure:

```js
// IIFE to avoid polluting the global namespace
const CounterModule = (function () {
  let count = 0; // Private variable

  function increment() {
    count++;
    console.log("Count:", count);
  }

  function reset() {
    count = 0;
    console.log("Count reset.");
  }

  return {
    increment,
    reset,
  };
})();

// Using the module
CounterModule.increment(); // Count: 1
CounterModule.increment(); // Count: 2
CounterModule.reset();     // Count reset.
```

Note: A `closure` is a feature where an inner function has access to the outer (enclosing) function's variables, even after the outer function has finished executing.  This allows the inner function to `"remember"` the environment in which it was created. Closures are often used to create private variables and functions, which helps avoid global scope pollution:
```js
function createCounter() {
  let count = 0; // Private variable

  return function increment() {
    count++; // Accessing count from the outer scope
    console.log("Count:", count);
  };
}

const counter = createCounter(); // createCounter returns the inner function
counter(); // Count: 1
counter(); // Count: 2
```

### Optimize Performance

- Avoid excessive DOM manipulations; batch updates if possible.
- Use `Debouncing` and `Throttling` to handle frequent event calls like scrolling or resizing.
- Load only necessary libraries and assets, especially for web applications.

#### Debouncing

To delay the function execution until a specified time has passed since the last call.  Useful for events that file rapidly, like resize:

```js
// Debounce function
function debounce(func, delay) {
  let timer;
  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => func.apply(this, args), delay);
  };
}

// Usage: Resize event with debounce
window.addEventListener("resize", debounce(() => {
  console.log("Resized!");
}, 300));
```

#### Throttling

To ensure a function is only called once in a specified time interval, regardless of how many times the event occurs.  Useful for scroll events:

```js
// Throttle function
function throttle(func, limit) {
  let inThrottle;
  return function (...args) {
    if (!inThrottle) {
      func.apply(this, args);
      inThrottle = true;
      setTimeout(() => (inThrottle = false), limit);
    }
  };
}

// Usage: Scroll event with throttle
window.addEventListener("scroll", throttle(() => {
  console.log("Scrolled!");
}, 300));
```

### Minimize Use of `this` Context

- Use arrow functions and .bind() to avoid unexpected `this` binding.
- Prefer `class` for object-oriented code, which clarifies `this` usage.

In Javascript, `.bind()` is used to explicitly set the `this` context for a function, ensuring it refers to a specific object regardless of where or how the function is called. This is particularly helpful **when passing functions as callbacks**, as it prevents unexpected `this` binding.

Example of `.bind[]` to preserve `this` context:
```js
const user = {
  name: "Alain",
  greet() {
    console.log(`Hello, ${this.name}!`);
  },
};

// Without .bind(), `this` is lost in a callback
setTimeout(user.greet, 1000); // Output: "Hello, undefined!"

// Using .bind() to preserve `this`
setTimeout(user.greet.bind(user), 1000); // Output: "Hello, Alain!"
```

In this specific case, the arrow function `=>` isn't suggested, Example:
```js
const user = {
  name: "Alain",
  greet: () => console.log(`Hello, ${this.name}!`), // Avoid (arrow function here makes `this` undefined)
};
```

### Avoid Deeply Nested Code

- Deeply nested  ode can be hard to read and maintain. Refactor nested callbacks and `if` statements into smaller functions.

### Use Default Parameters

- Set default values for function parameters to handle undefined arguments:
```js
function greet(name = 'Guest') {
    console.log(`Hello, ${name}`);
}
```

### Write Clean and Readable Code

- Aim for readability over cleverness. Code should be understandable by others without needing additional explanation.
- Avoid adding comments, use code readability instead.

## Principles

Javascript Principles form the foundational ideas and approaches that guide how code should be structured, organized and maintained. Following these principles leads to more efficient, maintainable and error-free code.

### Single Responsibility Principle (SRP)

- Each function, module or class should have one specific responsibility.
- Bu following SRP, you make your code easier to read, test and maintain.

### Don't Repeat Yourself (DRY)

- Avoid duplicating code; instead, reuse functions, classes, or modules.
- DRY promotes reusability and helps avoid inconsistent code, especially during updates.

### Keep it simple, Stupid (KISS)

- Write simple, clear code, instead of overly complicated solutions.
- Avoid unnecessary complexity and strive to make your code easy to understand and maintain.

### Encapsulation

- Hide implementation details and expose only what's necessary.
- Encapsulation helps prevent external code from accidentally interacting with or modifying internal logic.

Example:
```js
class BankAccount {
    #balance = 0; // Private field

    constructor(accountHolder) {
        this.accountHolder = accountHolder; // Public property
    }

    deposit(amount) {
        if (amount > 0) this.#balance += amount;
    }

    withdraw(amount) {
        if (amount > 0 && amount <= this.#balance) this.#balance -= amount;
    }

    getBalance() {
        return this.#balance;
    }
}

// Usage
const account = new BankAccount("Alain");
account.deposit(100);
console.log(account.getBalance()); // 100
account.withdraw(30);
console.log(account.getBalance()); // 70
// console.log(account.#balance); // Error: Private field '#balance' is not accessible
```

### Separation of Concerns (SoC)

- Keep different concerns (data handling, UI rendering, business logic) separate.
- In Javascript, this often means separating HTML/CSS from JavaScript logic and organizing Javascript into cohesive modules.

### Modularity

- Break down your code into smaller, independent modules or functions.
- Modularity improves reusability, testability, and maintainability by keeping related functions grouped logically.

### Favor Composition Over Inheritance

- Use composition (combining functions to build objects) rather than inheritance (creating class hierarchies) when possible.
- Composition is more flexible than inheritance, making it easier to reuse code without creating complex class hierarchies.

Example of Composition vs Inheritance:
```js
// Behavior functions
function canFly(obj) {
  obj.fly = () => console.log("Flying!");
  return obj;
}

function canSwim(obj) {
  obj.swim = () => console.log("Swimming!");
  return obj;
}

// Composing a bird object with specific behaviors
const bird = canFly({});
bird.fly(); // Output: "Flying!"

// Composing a fish object with different behavior
const fish = canSwim({});
fish.swim(); // Output: "Swimming!"
```

Instead of creating multiple subclasses like `FlyingBird` or `SwimmingFish`, we use small, composable functions (e.g., `canFly` or `canSwim`).  This approach allows flexible combinations of behaviors, making it easier to create objects with varied capabilities without rigid class hierarchies. 

## What does TypeScript fix?

When using **TypeScript** with JavaScript principles, most concepts remain valid, but TypeScript's features can modify or enhance certain principles. 

There are the JavaScript principles that are affected.

### Encapsulation, Privacy and Minimize `this` Context

**TypeScript** provides `private` and `protected` access modifiers for class fields and methods, enabling true encapsulation without relying on JavaScript-specific patterns like closures or the # syntax for private fields, and making it easier to manage privacy without relying heavily on `this`.  Additionally, using arrow functions and bound functions in TYpeScript classes can help avoid unexpected `this` binding, especially in callbacks.

The **impact** is that it makes encapsulation more intuitive and aligned with other OOP languages.  You can avoid closures or the module pattern for encapsulation within classes, as TypeScript's access modifiers give more straightforward privacy control. Additionally, you can more easily avoid `this` in context where it's not needed or expected. Access modifiers and arrow functions reduce reliance of `this` for accessing private or protected members, which can simplify code and make it easier to follow. 

Example of a TypeScript class:
```ts
class BankAccount {
    private balance: number = 0; // private access modifier

    deposit(amount: number) {
        if (amount > 0) this.balance += amount;
    }

    getBalance(): number {
        return this.balance;
    }
}
```

### Type Safety and `const` Usage

**TypeScript** enforces types, meaning that developers don't need to rely as heavily on `const` to prevent reassignment. The type system ensures that values of specific types remain consistent throughout the code.

The **impact** is that the use of `const` for immutability is still recommended, but sightly less critical for type safety, as TypeScript will enforce types at compile time.

### Avoiding Global Scope Pollution

**TypeScript has its own module system and strict scoping, which naturally avoids global pollution bu requiring explicit imports and exports.

The **impact** is that the emphasis on avoiding global scope pollution is less relevant in TypeScript, as it enforces a module-based approach.  Each file is treated as a module by default, so variables and functions aren't accidentally exposed globally.

### Type Safety and Immutability

**TypeScript** provides the `readonly` keyword for properties.  It adds a layer of immutability by preventing properties from being modified after initialization. This is especially helpful in enforcing immutability for objects and class properties.

The **impact** is that the immutability is still important, but TypeScript's `readonly` keyword gives more control over immutability directly, reducing the need to use libraries or additional patterns to enforce immutability manually.

Example:
```ts
class User {
    readonly name: string;

    constructor(name: string) {
        this.name = name;
    }
}
```

### Error Handling and Fail Fast

**TypeScript**'s type checking at compile time reduces certain runtime errors, enforcing stricter code that prevents many errors, enforcing stricter code that prevents many errors before they happen. Additionally, TypeScript modules automatically run in `strict mode`, enforcing safer coding by default

The **impact** is that while `fail fast` is still a good principle, TypeScript's static typing minimizes runtime errors, especially those related to type mismatches or missing properties. This reduces reliance on `try/catch` for type errors, though runtime errors (like network errors) still require handling.

JavaScript vs TypeScript example:
```js
// javascript version with type error handling
function divide(a, b) {
    try {
        if (typeof a !== 'number' || typeof b !== 'number') {
            throw new Error("Both arguments must be numbers");
        }
        if (b === 0) {
            throw new Error("Cannot divide by zero");
        }
        return a / b;
    } catch (error) {
        console.error("Error:", error.message);
    }
}

// Usage
console.log(divide(10, 0)); // Error: Cannot divide by zero
console.log(divide("10", 2)); // Error: Both arguments must be numbers
```
```ts
// typescript version with no type error handling
function divide(a: number, b: number): number | void {
    try {
        if (b === 0) {
            throw new Error("Cannot divide by zero");
        }
        return a / b;
    } catch (error) {
        if (error instanceof Error) {
            console.error("Error:", error.message);
        }
    }
}

// Usage
console.log(divide(10, 0)); // Error: Cannot divide by zero
console.log(divide(10, 2)); // 5
``` 

### Documentation and Readability

**TypeScript** types provide a level of `self-documentation`, as types clarify what a function, variable, or class is expected to handle. For example, seeing `getUser(id: number): User` clearly communicates what `getUser` does.

The **impact** is that this principle still relevant, but `enums` and TypeScript's type checking make it easier to avoid magic numbers and use constants consistently.

Example:
```ts
enum Status {
    Active = 1,
    Inactive = 0,
}
```

## Conclusion

Most JavaScript principles remain valid in TypeScript, but TypeScript's features like `private`, `readonly`, modules, enums and static typing improve and simplify many best practices. With TypeScript, code can be more organized, self-documenting and type-safe, which can reduce reliance on specific patterns used in pure JavaScript.
