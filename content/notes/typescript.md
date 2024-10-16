+++
title = "TypeScript"
LastModifierDisplayName = "Alain Bouchard"
LastModifierEmail = "abouchard@live.ca"
disableToc = "false"
+++

{{< toc >}}

# TypeScript

JavaScript and TypeScript are two popular programming languages for developing web applications. JavaScript is a simple and versatile language that supports dynamic typing. TypeScript extends JavaScript by adding static typing and features like interfaces, enums, and advanced type-checking.

## Node.JS Installation Quick Overview

Get current `node` version:

```bash
> node -v

v22.9.0
```

Installing `Node.js` using [brew]:

```bash
> brew install node
```

## Initializing TypeScript Project

Installing `typescript` using `npm` at the project level:

```bash
> npm install typescript --save-dev

> ls -1 

node_modules
package-lock.json
package.json

> npx tsc -v

Version 5.6.3

> cat package.json 

{
  "devDependencies": {
    "typescript": "^5.6.3"
  }
}
```

## Setting TypeScript Configuration File

A configuration file `tsconfig.json` is required at the Project root directory:

`tsconfig.json`:
```json
{
    
    "compilerOptions": {
        "outDir": "build",
        "target": "ES6" ,
        "noEmit": false,
        "experimentalDecorators": true,
        "emitDecoratorMetadata": true
    },
    "include": ["src/**/*"]
}
```

Details:

- `include`: the directories to search the TypeScript files from
- `compilerOptions.target`: the [ECMAScript] version compatibility
- `compilerOptions.outDir`: the built files directory, or where `tsc` will drop the generated `js` files
- `compilerOptions.noEmit`: `true` or `false`, if the `js` files should be generated.  The true value will make `tsc` to only check the files without generating any `js` file. 
- `compilerOptions.experimentalDecorators`: `true` or `false`, if the then the decorators (experimental) feature will be enabled. 
- `compilerOptions.emitDecoratorMetadata`: `true` or `false`, This library implements polyfills for another set of proposed ECMAScript features (experimental), requires the reflect-metadata library to be installed : `npm i reflect-metadata --save`. 

## Searching and Importing Open Source Defined Types

All defined types can be found in [DefinitelyTypes] github repository, or more easily on [npnjs.com] package locator:

For example, search for `@types jquery` within [npmjs.com] search field.

The installation command will be:

```bash
npm install --save @types/jquery
```

The imported library will be located in `node_modules/@type` directory.

## Types

### Primitive and Built-in Types

By default, Javascript will infer the variable type.  For example, this will create a `number` type by default:

```js
let x = 5
```

However, TypeScript won't infer anymore since variable type must be defined:

```ts
let x: number
let y: string
let z: boolean
let a: Date
let b: string[]
let c: any
```

The `any` type will be automatically inferred.

It is possible to use `casting`.

```ts
let b: string[]
b = "Hello World!" as any
```

Using `as any` is however avoiding the whole TypeScript purpose.

### Creating Custom Types Using Interface

It is possible to create custom types in TypeScript by using `interface`, for example:

```ts
interface Contact {
  id: number;
  name: string;
  birthDate?: Date; // Optional since ? was added
}

let contact1: Contact;

let contact2: Contract = {
  birthDate: new Date("01-01-1999"),
  id: 1234,
  name: "John Snow"
}

let contact3: Contract = {
  id: 1234,
  name: "John Snow"
}
```

When a `?` is added to the interface field then it will make this field optional.

It is possible to extend an actual interface, for example:

```ts
interface Contact extend Address {
  id: number;
  name: string;
  birthDate?: Date; // Optional since ? was added
}

interface Address {
  line1: string;
  line2: string;
  province: string;
  region: string;
  postalCode: string;
}

let contact1: Contract = {
  birthDate: new Date("01-01-1999"),
  id: 1234,
  name: "John Snow"
  postalCode: "H0H0H0"
  ...
}
```

### Creating Custom Types Using Aliases

An Alias is only a replacement name for an existing type, for example:

```ts
type ContactName = string
```

So `ContactName` is in reality a `string`, and both are now interchangeable.

### Enumerable Types

Enums allow a developer to define a set of named constants, for example:

```ts
enum ContactStatus {
  Active = "active", // assigned value "active" is optional
  Inactive = "inactive",
  New = "new"
}

interface Contact {
  id: number;
  name: string;
  birthDate?: Date;
  status: ContactStatus;
}

let contact1: Contract = {
  birthDate: new Date("01-01-1999"),
  id: 1234,
  name: "John Snow",
  status: ContactStatus.Active
}
```

## Typing a Function

This is a JavaScript function:

```js
function clone(source) {
  return Object.apply({}, source) // Apply will return an "any" type
}

const contact1: Contact = {id: 1, name: "John Do"};
const contact2 = clone(contact1); // contact2's type will be "any"
```

### Using Specific Types in a Function

This is the typed version in TypeScript:

```ts
function clone(source: Contact): Contact {
  return Object.apply({}, source) // Apply will return an "any" type, but the function's returned type will be Contact
}

const contact1: Contact = {id: 1, name: "John Do"};
const contact2 = clone(contact1); // contact2's type will be "Contact"
```

### Defining a Megatype using Generics

```ts
function clone<T>(source: T): T {
  return Object.apply({}, source)
}

const contact1: Contact = {id: 1, name: "John Do"};
const contact2 = clone(contact1);

// clone() will work for any type, e.g.
const dateRange = { startDate: Date.now(), endDate: Date.now() };
const dateRageCopy = clone(dateRange);
```

It is possible to define multiple types using Generics, for example:

```ts
function clone<T1, T2>(source: T1): T2 {
  return Object.apply({}, source)
}

// T1 and T2 must be specified at the function's call
const contact1: Contact = {id: 1, name: "John Do"};
const contact2 = clone<Contact, Date>(contact1);
```

It is possible to use `generics` with an `interface`:

```ts
interface ExternalContact<TExternalId> {
  id: number;
  name: string;
  birthDate?: Date;
  externalId: TExternalId;
  loadExternalId(): Task<TExternalId>;
}
```

## More Complex Types

### Using Aliases for Complex Types Definition

It is possible to allow multiple types for one field, for example:

```ts
interface Contact {
  id: number;
  name: string;
  birthDate: Date | number | string; // all 3 types can be used
}
```

Now it is possible to use an `alias`, for example:

```ts

type ContactBirthDate = Date | number | string;

interface Contact {
  id: number;
  name: string;
  birthDate: ; ContactBirthDate; // all 3 types can be used
}
```

Creating a new `interface` or custom type using an `alias`:

```ts
interface Contact {
  id: number;
  name: string;
  birthDate: Date;
}

interface Address {
  line1: string;
  line2: string;
  province: string;
  region: string;
  postalCode: string;
}

type AddressableContact = Contact & Address;
```

Replacing an Enum type with an alias, for example:

```ts
enum ContactStatus {
  Active = "active",
  Inactive = "inactive",
  New = "new"
}

let a: ContactStatus = ContactStatus.Active;

// Can be replaced by

type ContactStatus = "active" | "inactive" | "new";

let a: ContactStatus = "active";
```

Using `keyof` to get the available fields for a given type:

```ts
interface Contact {
  id: number;
  name: string;
  birthDate: Date;
}

type ContactFields = keyof Contact;

function getValue(source, propertyName: ContactFields) {
  return source[propertyName]
}

// With a generic
function getValue<T>(source, propertyName: keyof T) {
  return source[propertyName]
}
```

## Working with Record Type

In TypeScript, a Record is a utility type that allows you to map keys of a specific type to values of another type. It's a concise way to define objects with a uniform key-value structure. Here's a simple example:

```ts
// Syntax: Record<KeyType, ValueType>

// Example 1: Record with string keys and number values
const scores: Record<string, number> = {
  Alice: 85,
  Bob: 92,
  Charlie: 88,
};

// Example 2: Record with string keys and boolean values
const isActive: Record<string, boolean> = {
  Alice: true,
  Bob: false,
  Charlie: true,
};
```

Explanation:
- `Record<string, number>`: The keys are strings, and the values are numbers. So, each key (like "Alice") must map to a number (like 85).
- `Record<string, boolean>`: Here, the keys are also strings, but the values are booleans (true or false).

It is possible to define a list of types, e.g.:

```ts
const scores: Record<string, number | string> = {
  Alice: 85,
  Bob: 92,
  Charlie: "88",
};
```

It is possible to limit the keys to a custom type using `keyof`, e.g.:

```ts
const contact1: Record<keyof Contact, number | string> = {
  id: 85,
  name: "Jon Snow",
};
```

## Resource Management with using Keyword

As of TypeScript 5.2, a new using keyword has been introduced to work similarly to C#'s using statement. This feature allows you to automatically dispose of resources when they are no longer needed, provided the resource implements a Disposable pattern.

Here's a small example of using this feature in TypeScript 5.2+

```ts
// can safely omit the `implements Disposable` statement 
class Resource {
  constructor(private name: string) {
    console.log(`${this.name} is created`);
  }

  [Symbol.dispose]() {
    console.log(`${this.name} is disposed`);
  }

  use() {
    console.log(`Using ${this.name}`);
  }
}

function main() {
  using resource = new Resource("MyResource");
  resource.use();
}

main();
```

Explanations:
- Resource class: Implements `[Symbol.dispose]` to clean up resources.
- using keyword: Automatically calls Symbol.dispose when the block ends.
- In main(): Creates a resource, uses it, and automatically disposes it when done.
- The `implements Disposable` part is optional; the critical part is implementing the [Symbol.dispose]() method for the using keyword to automatically dispose of the resource.

## Extending and modifying existing types

### The Partial helper type

Partial<T> is a TypeScript utility type that makes all properties of a given type T optional. This allows you to create objects where only some properties of the original type are required. This is an example:

```ts
interface User {
  name: string;
  age: number;
  email: string;
}

// Using Partial to make all fields optional
const updateUser = (user: Partial<User>) => {
  console.log(user);
};

// Example usage
updateUser({ name: "Alice" });
updateUser({ age: 30, email: "alice@example.com" });
```

Explanation:
- `Partial<User>`: Makes all properties in the `User` interface optional.
- `updateUser`: Can accept an object with any subset of `User` properties (e.g., just `name`, `age`, or `email`).

### The Omit helper type

`Omit<T, K>` is a TypeScript utility type that constructs a new type by removing the specified keys `K` from type `T`. This allows you to create a type without certain properties.  This is an example:

```ts
interface User {
  name: string;
  age: number;
  email: string;
  address: string;
}

// Using Omit to exclude 'email' and 'address'
const createBasicUser = (user: Omit<User, 'email' | 'address'>) => {
  console.log(user);
};

// Example usage
createBasicUser({ name: "Alice", age: 30 });
```

Explanation:
- `Omit<User, 'email' | 'address'>`: Excludes both `email` and `address` from the `User` type.
- `createBasicUser`: Accepts an object with only `name` and `age`, omitting `email` and `address`.

### The Pick helper type

`Pick<T, K>` is a TypeScript utility type that creates a new type by selecting specific keys `K` from type `T`. It allows you to include only the desired properties. There is an example:

```ts
interface User {
  name: string;
  age: number;
  email: string;
  address: string;
}

// Using Pick to select only 'name' and 'email'
const getUserContactInfo = (user: Pick<User, 'name' | 'email'>) => {
  console.log(user);
};

// Example usage
getUserContactInfo({ name: "Alice", email: "alice@example.com" });
```

Explanation:
- `Pick<User, 'name' | 'email'>`: Selects only `name` and `email` from the `User` type.
- `getUserContactInfo`: Accepts an object with just `name` and `email`, ignoring other properties like `age` and `address`.

### The Required helper type

`Required<T>` is a TypeScript utility type that makes all properties of a given type `T` required, removing any optional modifiers. This is an example:

```ts
interface User {
  name?: string;
  age?: number;
}

// Using Required to make all fields mandatory
const createUser = (user: Required<User>) => {
  console.log(user);
};

// Example usage (now both 'name' and 'age' are required)
createUser({ name: "Alice", age: 30 });
```

Explanation:
- `Required<User>`: Converts all optional properties in `User` to required.
- `createUser`: Now expects both `name` and `age` to be provided.

## Decorators

A TypeScript `decorator` is a special kind of declaration that can be attached to `classes`, `methods`, `properties`, or `parameters` to modify their behavior. Decorators are essentially functions that take the target (like a `class` or `method`) as an argument and allow you to apply additional logic to it. They are commonly used for things like logging, validation, or adding metadata.

Decorators are written with the @ symbol followed by the decorator function name and can be applied to:
- Classes
- Methods
- Properties
- Accessors
- Parameters

To enable `decorators` in TypeScript, you need to add the `experimentalDecorators` option in your `tsconfig.json`.  

More details may be found in the [Decorators] official documentation.

### Decorator with no arguments

This is an example:

```ts
function log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  descriptor.value = function (...args: any[]) {
    console.log(`Method ${propertyKey} was called with args: ${args}`);
    return originalMethod.apply(this, args);
  };
}

class Example {
  @log
  greet(name: string) {
    console.log(`Hello, ${name}!`);
  }
}

// Example usage
const example = new Example();
example.greet("Alice");
```

In this example, the `@log` decorator adds logging behavior to the `greet` method, printing when the method is called and the arguments passed to it.

### Decorator with no arguments

This is an example:

```ts
function logWithMessage(message: string) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    descriptor.value = function (...args: any[]) {
      console.log(`${message} - Method ${propertyKey} was called with args: ${args}`);
      return originalMethod.apply(this, args);
    };
  };
}

class Example {
  @logWithMessage("Custom log")
  greet(name: string) {
    console.log(`Hello, ${name}!`);
  }
}

// Example usage
const example = new Example();
example.greet("Alice");
```

Explanation:
- `logWithMessage(message: string)`: A decorator factory that takes a message argument and returns a decorator function.
- `@logWithMessage("Custom log")`: Applies the decorator with the custom log message to the `greet` method, enhancing it with additional logging behavior.

## Using modules in typescript

It is recommended to work with modules. Basically, the exported functions and properties require the `export` keyword in front.  The importing module require to define the imported functions and properties.

This is an example:

file: math.ts (exporting module)

```ts
export function add(a: number, b: number): number {
  return a + b;
}

export const PI = 3.14;
```

file: app.ts (importing module)
```ts
import { add, PI } from './math';

console.log(add(2, 3)); // Output: 5
console.log(PI);        // Output: 3.14
```

Explanation:
- `math.ts`: Exports a function (`add`) and a constant (`PI`).
- `app.ts`: Imports the `add` function and `PI` constant from `math.ts` and uses them in the code.

## Using the globals.d.ts file

The `globals.d.ts` file is a TypeScript declaration file used to define `global types`, `interfaces`, or `variables` that can be accessed throughout your project without imports. It's useful for:
- Declaring global variables.
- Extending global objects (e.g., Window).
- Sharing global types or interfaces.

This is an example of `globals.d.ts`:

```ts
declare const API_URL: string;

interface Window {
  myCustomProperty: string;
}

type UserRole = 'admin' | 'user' | 'guest';
```

This file allows these types and variables to be used globally across your project and is typically placed in the `src` or `types` directories. 

### The declare global {} statement

The `declare global {}` statement in TypeScript is used to extend or modify the global scope by adding new types, interfaces, or variables that will be accessible globally throughout the project. This is useful when you need to add custom properties to global objects (like `Window`, `Document`, etc.) or declare new global variables and types.

It is typically used in `.d.ts` files to make these global declarations available project-wide.

This is an example:

```ts
// globals.d.ts
export {};

declare global {
  interface Window {
    myCustomProperty: string;
  }

  declare const API_URL: string;
}
```

Meaning:
- `declare global {}`: Everything inside this block is added to the global scope, meaning you can use it without importing in other files.
- Extending `Window`: Adds a custom property `myCustomProperty` to the global `Window` object.
- Declaring global `API_URL`: Makes the `API_URL` constant available globally, with its type specified.

This approach is useful when working with global objects or variables that are shared across different parts of your application.

### Using or not using the declare global {} statement?

The difference between using `declare global {}` and not using it lies in where the declarations are scoped and how they affect the global namespace. Here's a concise comparison:

#### 1. Without declare global {}

When you use declare directly without declare global {}, the declared entities are treated as global only within the specific file where the declaration exists.

Example (without `declare global {}`):

```ts
// globals.d.ts
declare const API_URL: string;

interface Window {
  myCustomProperty: string;
}
```

These declarations are global, but only if the `.d.ts` file is included by TypeScript.
You don’t explicitly tell TypeScript that this is meant to modify the global namespace; TypeScript assumes the declarations are part of the global scope.

#### 2. With declare global {}
By wrapping the declarations inside `declare global {}`, you are explicitly modifying the global namespace and ensuring that these changes apply project-wide.

Example (with `declare global {}`):

```ts
// globals.d.ts
export {}; // Ensures the file is treated as a module

declare global {
  interface Window {
    myCustomProperty: string;
  }

  declare const API_URL: string;
}
```

- `declare global {}` is used when the file is treated as a module (via `export {}` or similar) and you still want to modify the global scope.
- This makes it explicit that you're extending the global namespace from within a module, ensuring the declarations are globally available, even if the `.d.ts` file contains export statements.

#### Key Differences
- Without `declare global {}`: The file is implicitly considered part of the global namespace. These declarations work fine in traditional `.d.ts` files with no imports or exports.
- With `declare global {}`: Explicitly extends the global namespace when the file is a module (i.e., it contains `import` or `export` statements). This is necessary if you want to mix global declarations with modular code.

You would typically use `declare global {}` in module-based projects where you are exporting/importing other things but still want to add or modify the global scope.




<!-- References and links -->

[brew]: https://brew.sh/
[Decorators]: https://www.typescriptlang.org/docs/handbook/decorators.html
[DefinitelyTyped]: https://github.com/DefinitelyTyped/DefinitelyTyped
[ECMAScript]: https://en.wikipedia.org/wiki/ECMAScript_version_history
[npmjs.com]: https://www.npmjs.com/


