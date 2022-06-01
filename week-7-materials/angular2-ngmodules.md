![Ironhack Logo](https://i.imgur.com/1QgrNNw.png)

# Angular Modules and NgModule

## Learning Goals
After this lesson, you will be able to:

* Define Angular modules and understand them.
* Use Angular's built-in NgModule.
* Bootstrap Angular applications.

## Introduction

In JavaScript there's always a problem of [namespaces](https://en.wikipedia.org/wiki/Namespace), and if we are not careful we can easily end up with functions or variables in the global namespace.

In addition, JavaScript does not provide any features for code organization.

Modules are here to help us to solve these issues.

## ES6 Modules

**A module is a cohesive block of code that wraps a given functionality.**

We write modules to organize our code and keep our applications readable and maintainable.

In ES6 every file that imports or exports something is a module.

TypeScript implements the ES6 specification for modules, and Angular uses it as well, so as we create code files and import or export data, we create a module for the application.

That's the theory, let's see how it works in practice.

First create a file called `math.ts` with the following content:

```typescript
// ---- math.ts ----
export function powerOf2(x = 0): number {
  return x * x;
}
```

and then a file called `calc.ts`:

```typescript
// ---- calc.ts ----
import { powerOf2 } from './math';

const r1 = powerOf2(42); // r1 = 1764
```

Both of these files are modules, since the first *exports* a function and the second *imports* a function.

:::warning
:exclamation: Remember to compile the modules using `tsc`.
:::

Importing the `powerOf2` definition in the `calc.ts` will make it available to use, like we do with `@Component` or any other Angular declaration.

Let's add some more features to our calculator class to experiment with different uses of modules.

### Multiple imports

First, let's create a power of two functions:

```typescript
// ---- math.ts ----
export function powerOf2(x = 0): number {
  return x * x;
}

export function sum(x, y): number {
  return x + y;
}
```

and then we can import each function simply listing all the declarations in curly braces:

```typescript
// ---- calc.ts ----
import { powerOf2, sum } from './math';

const r1 = sum(42, 1);
const r2 = powerOf2(42);

console.log(`Result of sum(42, 1) is ${r1}`);
console.log(`Result of powerOf2(42) is ${r2}`);
```

To see the result we can compile our TypeScript and run the code in node.

- First run `tsc calc.ts math.ts`
- Run the script in node with `node calc.js`.

The expected result is:

```bash
Result of sum(42, 1) is 43
Result of powerOf2(42) is 1764
```


### * wildcard and aliasing

We could also alias the imported functions, such as:

```typescript
// ---- calc.ts ----
import * as OurMath from './math';

const r1 = OurMath.sum(42, 1);
const r2 = OurMath.powerOf2(42);
```

This makes the code easier to read while keeping the import statement the same for future changes.

If we decide to add a `squareRoot` function to our `math.ts` library, we can define it, export it and start using it in our `calc.ts` file. That would work because the `*` wildcard imports everything that is exported by the `math.ts` module.

### Default Keyword

One last important thing to know about module usage is the `default` keyword.

An ES6 module can pick a default export, or **the most important / *default* exported value.** Default exports are especially easy to import.

If we mark something as the default member to be exported, it will be the one imported if someone does not specify which part of the module they'd like to import.

Let's see how it works:

```typescript
// ---- math.ts ----
export function powerOf2(x = 0): number {
  return x * x;
}

  // make sum() ----------
  // the default export  |
  //      |              |
export default function sum(x, y): number {
  return x + y;
}
```

and then:

```typescript
// ---- calc.ts ----
import mathSum from './math';

const r1 = mathSum(42, 1);
// here root is not available anymore
```

Usually the default keyword is used on single-exported member file or library, where the member exported often does not have a name. Instead, it is to be identified via its module’s (file's) name.


## NgModule

At a high level we write Angular applications by composing templates and component classes, adding application logic in services, and boxing components and services in modules.

:::info
:bulb: We will be learning what these other concepts are shortly. Don't panic!
:::

JavaScript modules are a great way to organize our application and extend it with capabilities from external libraries. However, they are limited since they don't have any knowledge about Angular architecture.

To fill this hole in knowledge, Angular added an extra layer of modularity through the `NgModule` decorator.

`NgModule` consolidates Angular's application parts into blocks of functionality. All the modules we define will compose our application.

As an example we can take a look at the `app.module.ts` located in the `src/app` folder of our helloworld application.

```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';


import { AppComponent } from './app.component';
import { MyNestedComponent } from './my-nested/my-nested.component';


@NgModule({
  declarations: [
    AppComponent,
    MyNestedComponent
  ],
  imports: [
    BrowserModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

As we can see, the `@NgModule` decorator takes a **metadata configuration object** for the module.

We have:
- `declarations` a list that identifies the application's only components, pipes, and directives.
- `imports` a list of modules that the module itself needs in order to provide its own functionalities. In this case we imported the `BrowserModule` which contains common utilities such as `ngFor` and `ngIf`.
- `providers` a list of providers.
- `bootstrap` identifies the `AppComponent` as the bootstrap component, the one we want the app launched on.

In this example we imported names such as `BrowserModule` and `NgModule` through the TypeScript module system, while we created a new module called `AppModule` using the Angular module system.

NgModules and TypeScript modules are two different and complementary module systems. We use both to write and organize our apps.

At first glance, having two different module systems might not sound like the ideal solution, but in the end it will give us all the flexibility we need to keep our code better organized.

Remember that every Angular app has one root module class. By convention it's a class called `AppModule` in a file named `app.module.ts`.

:::info
:bulb: We still don't know what **pipes**, **directives** and **providers** are but don't worry, we'll fill in details as we go.
:::

## Angular Bootstrap

Modules not only provide a better way to organize code and features, they also allow Angular to *bootstrap* our applications.

In the file called `main.ts` in the `src` folder we can see that `AppModule` is the entry point for the application.

The lines:

```typescript
import { platformBrowserDynamic } from
        '@angular/platform-browser-dynamic';
import { AppModule } from './app.module';

[...]

platformBrowserDynamic().bootstrapModule(AppModule)
  .catch(err => console.log(err));
```

Here we import the needed declarations and ask the framework to launch our application from the `AppModule` module.

This way the `AppModule` becomes the root module of our application, and it will be the only module specifying a `bootstrap` metadata with the root component.

In this case `AppComponent` **will be the first component loaded for the application**.

## Summary

In this lesson we have learned what ES6 modules are, and how we define and use them. We have also seen how Angular provides an extra module layer with the NgModule decorator, and how we bootstrap Angular applications.