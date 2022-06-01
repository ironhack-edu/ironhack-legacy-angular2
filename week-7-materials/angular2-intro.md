![Ironhack Logo](https://i.imgur.com/1QgrNNw.png)

# Hello Angular 2

## Learning Goals

After this lesson, you will:

* Understand what JavaScript frameworks are and why we need them.
* Understand what a Single-Page Application (SPA) is.
* Have a basic Angular "Hello World" application for future reference.
* Be able to display data using string interpolation.
* Know how to iterate through collections with `ngFor`.
* Be able to add conditional logic with `ngIf`.

## Client-Server Architecture

So far we have built applications where HTML pages were provided by the server. In other words, for every URL request made by the browser, the server responded back with an HTML document.

This approach is good when creating basic websites, but every time the user clicks a link or needs to view or send data, the whole page is refreshed, resulting in a slow user interaction.

 ![](https://i.imgur.com/dVoa0bb.png)

Recently, a different, better approach has gained popularity.

Applications now have two parts:

* **Back-End** that doesn't render HTML anymore but instead provides APIs for a client application.
* **Client-Side** that provides a user interface, handles user input and communicates with the back-end through APIs.

  ![](https://i.imgur.com/6ggb1Sx.jpg)

**Front-End:**
* Client-Side
* Usually visible to users as an interactive interface

The main challenge a front-end developer faces is the *continuous transformation* of the site. They ensure that the same site is displayed correctly in different browsers, operating systems and devices.

 ![](https://i.imgur.com/7GVkQ7z.png)

**Back-End:**
* Not visible
* Can interact directly with front-end or it can be mediated by another program
* Comprised of server(s) and database(s)

A "back-end" application or program indirectly serves and supports front-end services, usually by being closer to the required resource or having the capability to communicate with the required resource.  Typically, the back-end of the application will handle the retrieval of information from the database (or other storage location), and will also handle the transmission of that information to the front-end so that the front-end interface may display that information in some way (although sometimes the transmission of this info is handled by yet another intermediate program).

### Single Page Applications (SPA)

This client-server architecture is often how single-page applications are designed. **A [SPA](https://en.wikipedia.org/wiki/Single-page_application)(single page application) is a web application where most of a user's interactions occur on a single page.**

This page will contain all the views that we need for our entire application, loading data from the back-end (Node API).

![](https://i.imgur.com/trxhjVF.png)

The most notable difference between a regular website and an SPA is the reduced number of page refreshes, as we can load multiple views into it.

Some examples of SPAs include Facebook, Gmail, and Twitter.

Our Angular application will have just one HTML page, usually called index.html. The framework will then dynamically load pieces of HTML code (templates), in response to user navigation.

## What is a Framework?

**A software framework is:**

> A universal, reusable software environment to facilitate development of software applications, products and solutions.

It may include:
- Support programs.
- Compilers.
- Code libraries.
- Toolsets.
- APIs (Application Programming Interfaces).

A **JavaScript framework** is a web application framework written in JavaScript.

Why should we care about a JavaScript framework?

* It describes the structure of the application.
* It gives the developer a way to organize the code to make an app more flexible and scalable.

You may know some other frameworks, like [Express](expressjs.com), a Node.js framework. In this module we will learn a framework for the front-end: Angular.

## Angular Introduction

![](https://i.imgur.com/nRGfgxR.png)

Angular is an open-source front-end framework created by Google in 2010 and has been widely adopted by the web development community.

Angular is written in TypeScript, a superset of JavaScript built by Microsoft in 2012.

**Angular (formerly referred to as AngularJS) is a complete JavaScript-based open-source front-end web application framework.** It is primarily maintained by Google, along with a community of individuals and corporations to address many of the challenges encountered in developing single-page applications.

The Angular framework works by first reading the HTML page, which has custom tag attributes embedded.

Angular interprets those attributes as **directives** to bind input or output parts of the page to a **model** that is represented by standard JavaScript variables. The values of those JavaScript variables can be manually set within the code, or retrieved from static or dynamic JSON resources.

Angular is used on the websites for Wolfram Alpha, NBC, Walgreens, Intel, Sprint, ABC News, and approximately 12,000 other sites out of 1 million tested in October 2016.

Angular is the frontend component of the MEAN stack, consisting of **M**ongoDB database, **E**xpress web application server framework, **A**ngular, and the **N**ode.js runtime environment.

### AngularJS vs Angular 2+

![](https://i.imgur.com/p1KNeu6.png)

It's worth noting that even though AngularJS started in 2008, the project was completely rewritten in TypeScript and has kept the same name: Angular.


Therefore Angular.io & Angular, refer to the newest version and AngularJS, refers to the previous version **which is incompatible with the newest Angular**.


:::warning

:exclamation: **For all angular documentation use https://angular.io instead of ~~https://angularjs.org/~~**.

:::

## Hello World 

To Start playing with Angular, go to [this plnkr starter code](http://plnkr.co/edit/CNcQ0yD1aAd5REZIbJp3):

![](https://i.imgur.com/9NMDrPq.png)

Here you will have the following files:

```
├── app
│   ├── app.component.ts
│   ├── app.module.ts
├── index.html
├── styles.css
```

:::info

:bulb: If it's not enabled, click on the **live preview** icon (an eyeball) at the top of the right sidebar to see the code running!

:::

### Loading Angular

In order to use Angular in our websites, we will have to add a few JavaScript files to our `index.html` `<head>` section. This will add the angular framework libraries, loaders and everything we need to get started with angular:

```htmlmixed=9
<!-- 1. Load libraries -->
<!-- Polyfill for older browsers -->
<script src="https://unpkg.com/core-js/client/shim.min.js"></script>
<script src="https://unpkg.com/zone.js@0.6.25?main=browser"></script>
<script src="https://unpkg.com/reflect-metadata@0.1.8"></script>
<script src="https://unpkg.com/systemjs@0.19.39/dist/system.src.js"></script>

<!-- 2. Configure SystemJS -->
<script src="https://cdn.rawgit.com/angular/angular.io/.../systemjs.config.web.js"></script>
```

### Components

Our application will be mounted in the HTML document using a special new HTML tag: `app-root`.

```htmlmixed=22
<body>
  <app-root>Loading...</app-root>
</body>
```

This will load our app component onto the page.
**A component is a way we write custom html** (like app-root). It contains views(HTML) and behaviour (TypeScript classes).
**In other words,** we are going to write an entire HTML document in a different file later, and that HTML will show up wherever we use the `<app-root>` tag.  We can do this for many components.

We will look at this in detail later on, but for now, just notice how we have an `app` directory with a `component.ts` file:

```
├── app
│   ├── app.component.ts
│   ├── app.module.ts
├── index.html
├── styles.css
```

```typescript=3
// app.component.ts
@Component({
  selector: 'app-root',
  template: '<h1>Hello Angular2!</h1>'
})
```

Now we are going to practice with a few basic Angular features. Ready?

### Interpolation

Let's display some simple data defined in our component class.

To use **interpolation** we need to put a class property name enclosed in double curly braces: `{{ myProperty }}` in the view template. The framework will then **bind** the property with the value displayed.

```typescript
// app.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
                      // Class Variable
  template: '<h1>Hello {{ message }}</h1>'
})

export class AppComponent {
  message: string = "World"
}
```
Interpolation also supports Angular expressions like:

- `1+2`
- `a+b`
- `user.name`
- `items[index]`

and the framework will update the result of the expression (value displayed) every time any of the properties contained changes its value.

### Iterate through collections of data with [`ngFor`](https://angular.io/docs/ts/latest/guide/displaying-data.html#!#ngFor)
Like most of the programming languages, Angular provides its own set of conditional statements (if-statements) and iterative statements (loops).

This section is about iteration over a collection of data.

Let's say we have an Array full of `animals` in our component class:

```ts
// app.component.ts
export class AppComponent {
  animals: Array<string> = ['Dog', 'Duck', 'Elephant', 'Zebra'];
}
```

We can use the `ngFor` angular directive in our **view template (app/app.component.ts)** to display all the `animals` as an HTML list (`<ul></ul>`).

:::danger

:bomb: Replace the single quotes(') of the `template` to *backticks* (\`) when using multi-line strings.

:::


:::info

```htmlmixed
<!-- app.component.ts -->
@Component({
  selector: 'app-root',
  template: `
  <h1>Hello Angular2!</h1>
  <ul>
    <li *ngFor="let animal of animals">
        {{ animal }}
    </li>
  </ul>`
})
```


at this point you should see:

| HTML Equivalent | HTML Render
|-----------------|----------
`<ul>`<br/>`<li>Dog</li>`<br/>`<li>Duck</li>`<br/>`<li>Elephant</li>`<br/>`<li>Zebra</li>`<br/>`</ul>` | ![](https://i.imgur.com/S4FVkwu.png)

:::

Most of the time we'll be using more complex data and want to display only certain properties. For example let's make our animals Array a bit more advanced:

```ts
export class AppComponent {
  animals: Array<Object> = [
    {
      id: 1,
      category: 'mammal',
      name: 'Dog'
    }, {
      id: 2,
      category: 'oviparous',
      name: 'Duck'
    }, {
      id: 3,
      category: 'mammal',
      name: 'Elephant'
    }, {
      id: 4,
      category: 'reptile',
      name: 'Snake'
    }
  ];
}
```

Now we can decide to show only specific properties while iterating:

:::info

```htmlmixed
<h1>Hello Angular2!</h1>
<ul>
    <li *ngFor="let animal of animals">
        <span> {{ animal.name }} </span>
        <small> ({{ animal.category }}) </small>
    </li>
</ul>
```

Result:

![](https://i.imgur.com/2o6uIeC.png)

:::

:::warning

:exclamation: Remember to put the leading asterisk (*) in `*ngFor`. It is an essential part of the syntax. To learn more about the asterisk, see [the structural directives page](https://angular.io/docs/ts/latest/guide/structural-directives.html#!#asterisk) in the official documentation and [this stackoverflow answer](http://stackoverflow.com/a/35498727/4624718).

If you are interested in learning all of the options for `*ngFor`, see the [official documentation](https://angular.io/docs/ts/latest/guide/template-syntax.html#ngFor).

:::

### Display content conditionally with [`ngIf`](https://angular.io/docs/ts/latest/guide/displaying-data.html#ngIf)

Sometimes an app needs to display a view or a portion of a view only under specific circumstances.

For example, if there are more than three animals, we want to display a message.

The Angular **ngIf directive** inserts or removes an element based on a truthy/falsey condition. Add the following paragraph to the bottom of the template and notice how the element dissappears or reappears based on the number of animals in the array.  

```htmlmixed
<p *ngIf="animals.length > 3"> There are many animals! </p>
```

![](https://i.imgur.com/LHykVLG.jpg)

:::warning

:exclamation: Remember to put the leading asterisk (*) in `*ngIf`. It is an essential part of the syntax.

If you are interested in learning all the different options for `*ngIf`, see the [official documentation](https://angular.io/docs/ts/latest/guide/template-syntax.html#ngIf).

:::

We will see more on this powerful directive in the coming lessons.

## Summary

In this lesson we have learned about single-page applications, and how frameworks help us when writing SPAs. We have created our first "hello world" application with Angular, and we have also used **Interpolation** with double curly braces to display a class property.

Finally, we have been introduced to **ngFor** and **ngIf**, which will be covered in more depth in future lessons.

## Extra Resources
* [JS Framework wikipedia](https://en.wikipedia.org/wiki/JavaScript_framework)
* [Angular Documentation](https://angular.io/docs/ts/latest/) - The official Angular 2 documentation
