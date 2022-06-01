![Ironhack Logo](https://i.imgur.com/1QgrNNw.png)

# TypeScript

## Learning Goals
After this lesson, you will be able to:

* Write classes, interfaces, and decorators using TypeScript.
* Compile TypeScript code to valid JavaScript code.
* Explain Typescript to a novice.

## Introduction

![TypeScript logo](https://i.imgur.com/VLqgCu4.png)

As hardware has advanced technologically, so too has the need for sophisticated software applications to run on multiple devices. JavaScript is a natural solution to this cross-platform model since almost all modern browsers are capable of running it.

Unfortunately, creating large applications in JavaScript can be difficult, especially when it comes to structuring the code in a maintainable way.

TypeScript, an open source language project created by Microsoft, aims to take JavaScript development to the next level.

TypeScript is just a **superset of JavaScript** that gets compiled into plain JavaScript. This means that anything you can do with ES5, you can do in ES6. And everything you can do with ES6, you can also use in TypeScript:

![Venn diagram of ES5 as a subset of ES6 and ES6 as a subset of TypeScript](https://i.imgur.com/3VSdcMQ.png)


TypeScript additionally introduces a number of important concepts from a few different object-oriented languages: *(Static typing, Classes, Interfaces, Generics, Modules, etc.)*. Working with TypeScript has multiple benefits:

- **Static typing** makes code written in TypeScript **more predictable**, and generally **easier to debug**.
- **Modules**, **namespaces** and **strong OOP support** make it **easier to organize the code base** for very large and complicated apps.
- **A compilation step** to JavaScript catches all kinds of errors **before they reach runtime** and cause errors.


The **Angular framework is written in TypeScript** and it’s recommended that developers use the language in their projects as well.

In this lesson, we are going to get started with TypeScript and explore some of these concepts.

Ready?


## Hello World

In this section we'll see how to write a simple _Hello World_ application using TypeScript.


### Let's write some code!

For our first exercise with Typescript, let's print "Hello World!" on the page.

:::info 

:baby_symbol: We don't really need TypeScript to print [Hello World!](https://en.wikipedia.org/wiki/%22Hello,_World!%22_program) but it's common practice to start with Hello World when learning a new technology!

:::

Create an `index.html` file with the following content:

```htmlmixed
<!DOCTYPE html>
<html>
  <head>
      <title> TypeScript Hello World </title>
  </head>
  <body>
    <script src="helloworld.ts"></script>
  </body>
</html>
```

Create a TypeScript file called `helloworld.ts`
:::info
:bulb: TypeScript files have a `.ts` extension
:::


```ts
/* helloworld.ts */
class Greeter {
    constructor(public message: string) { }
    sayHi() {
        return "<h1>" + this.message + "</h1>";
    }
}

let greeter = new Greeter("Hello, world!");

document.body.innerHTML = greeter.sayHi();
```

What's happening here?

**`index.html`**
- We first created a super-basic HTML file with a title and a reference to a TypeScript file called `helloworld.ts`
- Nothing strange so far.

**`helloworld.ts`**
- Then we created a TypeScript file containing a **Greeter class** that just returns a message inside an `<h1>` tag.
- We use the Greeter class with the message `Hello, World!`
- We use the browser's `document` object to add the message to the page.

If you open the `index.html` page now you will see that our page is still blank and we have a console error saying `"Uncaught SyntaxError: Unexpected strict mode reserved word"`.

![A confused Tom Cruise saying "What?"](https://i.imgur.com/63p0KXD.mp4)

**TypeScript is a language that needs to be compiled** and cannot be executed or interpreted by the browser. We need to compile our TypeScript into JavaScript code!

### Code compilation

![TypeScript is a typed superset of JavaScript that compiles the plain JavaScript.](https://i.imgur.com/aMFuvbh.png)

To compile our TypeScript code into valid JavaScript code we now need to install the `tsc` compiler. The compiler is available as a standalone package through `npm`.

![The `typescript` package on npm's website](https://i.imgur.com/UrDzJbZ.png)

Run this terminal command to install it:

```shell
$ npm install -g typescript
```

To confirm that the compiler was installed correctly, we can run `tsc` (**T**ype**S**cript **C**ompiler) from the terminal:

```shell
$ tsc
Version 2.8.1
Syntax:   tsc [options] [file ...]

Examples: tsc hello.ts
          tsc --outFile file.js file.ts
          tsc @args.txt

Options:
 --allowJs                           Allow javascript files to be compiled.
 --allowSyntheticDefaultImports      Allow default imports from modules with no default export.
                                     This does not affect code emit, just typechecking.
 --allowUnreachableCode              Do not report errors on unreachable code.
 --allowUnusedLabels                 Do not report errors on unused labels.
 --baseUrl                           Base directory to resolve non-absolute module names.
 -d, --declaration                   Generates corresponding '.d.ts' file.
 ...
```

Now we are ready to compile our code:

```ts
/* helloworld.ts */
class Greeter {
    constructor(public message: string) { }
    sayHi() {
        return "<h1>" + this.message + "</h1>";
    }
}

let greeter = new Greeter("Hello, world!");

document.body.innerHTML = greeter.sayHi();
```

Run `tsc helloworld.ts` from our app's root directory and `tsc` will create a file with the compiled JavaScript version of our code. The new file will have the same name, except that its file extension will be `.js` instead of `.ts`:

```shell
$ tsc helloworld.ts
```

If we open the compiled JavaScript file we see something like:

```javascript
/* helloworld.js */
var Greeter = /** @class */ (function () {
    function Greeter(message) {
        this.message = message;
    }
    Greeter.prototype.sayHi = function () {
        return "<h1>" + this.message + "</h1>";
    };
    return Greeter;
}());
var greeter = new Greeter("Hello, world!");
document.body.innerHTML = greeter.sayHi();
```

Pretty easy!!

We can now change our source reference in the `<script>` tag inside the `index.html` file so that it references the compiled `.js` file we got from the `tsc` compiler:

```htmlmixed
<!DOCTYPE html>
<html>
  <head>
      <title> TypeScript Hello World </title>
  </head>
  <body>
    <script src="helloworld.js"></script>
  </body>
</html>
```

And opening the HTML file we should see our message
!["Hello, world!" in our page](https://i.imgur.com/Q2U2P79.png)

### TSC compilation

The presence of `tsconfig.json` in a directory indicates that that folder is the root of a TypeScript project.

The `tsconfig.json` file specifies the root files and the compiler options required to compile the project.

A project is compiled in one of the following ways:
1. Running the `tsc` command from the command line, specifying all the options using command flags.
1. Running a plain `tsc` command letting the compiler take all the options from our `tsconfig.json` file.

Typically the first step in any new **TypeScript** project is to create a `tsconfig.json` file. This defines the **TypeScript** project settings such as the compiler options and the files that should be included. To do this, open the folder where you want to store your source and create a file named `tsconfig.json`.

![image](https://user-images.githubusercontent.com/23629340/34101172-2153f2ea-e3e5-11e7-880c-13bccebf3356.png)

A simple `tsconfig.json` looks like this for ES6, CommonJS modules and source maps:

```json
{
    "compilerOptions": {
        "target": "es6",
        "module": "commonjs",
        "sourceMap": true
    }
}
```

Now when you create a `.ts` file as part of the project we will offer up rich editing experiences and syntax validation.

### `typescript.reportStyleChecksAsWarnings``

By default, VS Code TypeScript displays code style issues as warnings rather than errors. Code style issues include the following:

- A variable is declared but never used
- A property is declared but its value is never read
- Unreachable code is detected
- A label is unused
- Falling through a case in a switch
- Not all code paths return a value
- Treating these as warnings is consistent with other tools, such as TSLint. These will still be displayed as errors when you run tsc from the command line.

You can disable this behavior by setting: "typescript.reportStyleChecksAsWarnings": false.

### Transpiling TypeScript into JavaScript
VS Code integrates with `tsc` through our integrated **task runner**. We can use this to transpile `.ts` files into `.js` files. Let's walk through transpiling a simple TypeScript Hello World program.

- Create a simple TS file *(we will use our `HelloWorld.ts` we already created)*

- Run the TypeScript Build
Execute **Run Build Task** from the global Tasks menu. You should present the following picker:

![image](https://user-images.githubusercontent.com/23629340/34103410-e2240710-e3ec-11e7-8564-6be32a73fd2a.png)


Select the entry. This will produce a `HelloWorld.js` and `HelloWorld.js.map` file in the workspace.

Behind the scenes, we run the TypeScript compiler as a task. The command we use is: `tsc -p .`

- Make the TypeScript Build the default

You can also define the **TypeScript** build task as the default build task so that it is executed directly when triggering **Run Build Task**. To do so select **Configure Default Build Task** from the global Tasks menu. This shows you a picker with the available build tasks. Select the TypeScript one which generates the following `tasks.json` file:

{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            "type": "typescript",
            "tsconfig": "tsconfig.json",
            "group": {
                "kind": "build",
                "isDefault": true
            }
        }
    ]
}

The example **TypeScript** file did not have any compile problems, so by running the task all that happened was a corresponding `HelloWorld.js` and `HelloWorld.js.map` file was created.

- Step 4: Reviewing Build Issues

Unfortunately, most builds don't go that smoothly and the result is often some additional information. For instance, if there was a simple error in our TypeScript file, we may get the following output from `tsc`:

```zsh
HelloWorld.ts(3,17): error TS2339: Property 'logg' does not exist on type 'Console'.
```

This would show up in the terminal window (which can be opened using ⌃\) and selecting the terminal **Tasks - build `tsconfig.json` ** in the terminal view drop-down. We parse this output for you and highlight detected problems in the Status Bar.

![image](https://user-images.githubusercontent.com/23629340/34103976-e3e05f70-e3ee-11e7-9d85-eef8aa388c0a.png)

You can click on that icon to get a list of the problems and navigate to them.


![image](https://user-images.githubusercontent.com/23629340/34103981-eac90724-e3ee-11e7-8da0-18a31d24ecf8.png)


You can also use the keyboard to open the list `⇧⌘M`.


## TypeScript Features

Let's dive into some of the major features provided to us by this cool language.

TypeScript implements the ES6 standard completely, as well as all the major features of ES7 and ES8 which are not yet in the standard (*in draft*).

Some of the most prominent features are **Types, Interfaces and Annotations**. Let's dive into those.


### Types

In ES6, when we declare variables, we usually just declare them and assign them values. There is no problem with assigning new values to a variable:

```javascript
let myVarName = "value";
myVarName = 2;
myVarName = new Date();
```

This makes the language very flexible, but at the same time it can create problems. For example:

- We expect a function to receive an object and we pass an array.
- We call a method on a variable thinking that the variable contained a string, but it really contained an integer.

TypeScript tries to address this problem by allowing us to specify the data type of the variable when we declare it:

```typescript
let myVarName: dataType = value;
```

In this example, `dataType` is a placeholder for one of the [TypeScript primitive types](https://www.typescriptlang.org/docs/handbook/basic-types.html). There are many types available to use:

```typescript
let myNumber:   number  = 42;
let myBoolean:  boolean = true;
let myString:   string  = "Hello World";
let unusable:   void    = undefined; // Used for functions which don't return a value.
let myWhatever: any     = { hello: 'World' }; // Can be any other type
```

These type annotations (`number`, `any`, etc) put a specific set of constraints on the variables being created, allowing the compiler and development tools to better assist in the proper use of the variable.

If a variable is created and no type is provided for it, TypeScript will attempt to infer the type from the context in which it is used.

```typescript
let myString = 42;
```

:::info
:bulb: TypeScript: *"I think myString is actually a JavaScript Number"*
:::

Also `null` and `undefined` have their own type:

```typescript
let u: undefined = undefined;
let n: null = null;
```

**Types == Performance Overhead?** (Advanced note)

:::info
:bulb: Despite all of the type annotations and compile-time checking, TypeScript compiles to plain JavaScript and therefore adds absolutely no overhead to the run time speed of our applications.

All of the type annotations are removed from the final code, providing us with both a much richer development experience and a cleaner finished product.
:::


#### Generics

JavaScript arrays are an example of a generic type because they can hold elements of any type:

```javascript
let myArray = [123, "foo", true];
array.length // ==> 3
array.forEach((elem) => { console.log(elem)}); // => 123 foo true
```

Declaring an array variable in JavaScript automatically gives us a whole set of methods and constraints that allow us to work with arrays.


Since TypeScript tries to assign types to all variables, when we create arrays, we should be able to define the value of all the elements inside the array.

TypeScript allows us to define our own generic types. Generics are one way to create classes and functions that can work with multiple types, making them more flexible and reusable.

One built-in example is the `Array` type, which takes an extra type as parameter to fix the type of its elements.

:::info
```typescript
let myArray: Array<T> = [];
```
Generic classes have a **generic type parameter list**, which is represented using angle brackets (`<>`) and including the name of the class `T`
:::

For example, if we need an array of strings we could use:

```typescript
let myArray: Array<string> = ['a','b','c']; // Array of strings
```

If we need numbers, we could use this:

```typescript
let myArray: Array<number> = [1, 2, 3]; // Array of numbers
```

### Interfaces

Interfaces create a contract for developers to use when developing new objects or writing methods to interact with existing ones.

You can think of an interface like an abstract class: you cannot create objects out of an interface, **classes can implement an interface**.

:::info
:bulb: You can read more about [abstract classes](https://en.wikipedia.org/wiki/Abstract_type) in classical Object Oriented Programming
:::

The role of an interface is just to set some rules about how a class that implements it should look. In particular, **an interface sets rules about the methods and properties** objects of that class need to have.


![Interface as it relates to Class in code hierarchy](https://i.imgur.com/N5fW1Qp.png =400x)

Let’s look at an example of an interface in TypeScript:

```typescript
interface Vehicle {
    engines: number;
    wheels:  number;
}
```

We use the interface keyword to start the interface declaration, then we give the interface a name that we can easily reference from our code. This interface is for an object that has two properties of the number type: `engines` and `wheels`.

Remember that we cannot create `Vehicle` objects. We need to write a class that implements the `vehicles` interface.

#### Interfaces and Classes

Let's see how interfaces should interact with classes.

To implement an interface we need to add the `implements` keyword after the class declaration, following with the name of the interface.

```typescript
class Car implements Vehicle {
  constructor() {}
}
```

Here we have our interface `Vehicle` implemented by the the class `Car`.... but the TypeScript compiler gives us an error:

![image](https://user-images.githubusercontent.com/23629340/34104188-c4d19c10-e3ef-11e7-8085-54bdf226b3bb.png)

This error tells us that the class is not respecting the contract defined by the interface. We need to have the two properties: `engines` and `wheels`, as defined by the `Vehicle` interface.

Let's add the two properties and see what happens:

```typescript
class Car implements Vehicle {
  constructor(public engines: number, public wheels: number) {}
}
```

The error is gone! This means `Car` has the 2 properties needed to implement the `Vehicle` interface.

:::info
The `public` keyword in the constructor automatically assigns the parameter to a class property with the same name. The above example of `public` is equivalent to:

```typescript
class Car implements Vehicle {
  engines: number;
  wheels: number;

  constructor(engines: number, wheels: number) {
    this.engines = engines;
    this.wheels  = wheels;
  }
}
```
:::

Let's add an additional property to the class that isn't part of the interface- the `model` of the car:

```typescript
interface Vehicle {
  engines: number;
  wheels: number;
  isMotorbike(): boolean;
}

class Car implements Vehicle {
  model: string;

  constructor(public engines: number, public wheels: number) {}

  isMotorbike():boolean {
    return (this.wheels === 2);
  }

  setModel(model):void {
    this.model = model;
  }
}

let car = new Car(1, 4);
car.setModel("Nissan GTR");
```

For the car's `model`, we add a new string property and a method `setModel` to set this property.

:::info
`:void` is a special data type that represents "nothing". We use it to express that a function doesn't return a value. This doesn't mean that the function doesn't modify our instances, it only refers to the return value.
:::

Finally, we create our first car instance, passing the needed parameters and setting the model using the method.

**Note:** (Advanced)
:::info
:bulb: TypeScript interfaces are analyzed at compile time and then omitted from the compiled JavaScript, they exist only for developer convenience.
:::

### Decorators

A [Decorator](https://en.wikipedia.org/wiki/Decorator_pattern) is an object-oriented pattern that **adds behaviour to classes and methods**. You can think of a decorator like a function that will enhance those classes or methods with extra functionality.

![](https://media.giphy.com/media/yekZxRr4nM1q0/giphy.gif)

Decorators use the form `@expression`, where `expression` must evaluate to a function that will be called at runtime with information about the decorated declaration.

:::info
:bulb: A decorator is just a function that is going to *decorate* (populate) a target.
:::

Let's say we have a class `Animal` and we want to decorate the class with a property `cleaned` set to `true`. The class decorator would be:

```typescript
function clean(target) {
  target.prototype.cleaned = true;
}

@clean
class Animal {}
```

This will cause every runtime instance of the `Animal` class to have a `cleaned` property set to `true`.

You can also add multiple decorators to a class:

```typescript
@clean
@dirty
class Animal {}
```

The concept of decorators is widely used in Angular, don't worry if it isn't super clear now, we'll come back to it later!

#### Dynamic Decorators (Advanced)

You can also build decorator factories, a function which returns another function that serves as the decorator itself.

Let's see how we can make the `@clean` decorator dynamic by turning it into a decorator factory:

```typescript
function clean(value: boolean) {
  return function (target) {
    target.prototype.cleaned = value;
  }
}

@clean(true)
class Animal {
  open() {}
}
```

This lets us create decorators that can change a class dynamically at runtime.

A decorator can target:
- a class
- a property
- a method
- a parameter
- an accessor (getter or setter)


## TypeScript and Angular
Angular uses TypeScript mostly for its strict type-checking and for decorators.

As we will see, all the framework building blocks are classes decorated in a specific way. This will make the code much easier to read while keeping the learning curve low.

Angular applications can be written with plain JavaScript instead of TypeScript, but the framework itself is entirely written in TypeScript.

**Currently, finding good documentation and examples on Angular using plain JavaScript is extremely hard.** TypeScript will be the standard "language" to be used when developing Angular applications.

We definitely recommend you use TypeScript going forward.

## Summary

In this lesson we have learned how to **compile** TypeScript to valid JavaScript code using the TypeScript compiler. We have seen how to define types in TypeScript, and learned what classes, interfaces, and decorators are.

## Extra Resources
* [TypeScript Documentation](https://www.typescriptlang.org/) - The official TypeScript documentation
* [Stackoverflow](http://stackoverflow.com/a/35048303/4544288) -What is TypeScript and why would I use it in place of JavaScript?
