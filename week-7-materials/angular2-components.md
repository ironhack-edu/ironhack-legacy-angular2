![Ironhack Logo](https://i.imgur.com/1QgrNNw.png)

# Angular Components

## Learning Goals

After this lesson, you will be able to:

* Create and serve an angular-cli application.
* Use multiple Angular components and understand what they are.
* Build your own Angular components defining TypeScript classes and metadata.
* Import and export components.

## Angular2 CLI

![](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_9e2f7e54e56da428625c6a0d769f5b15.png)

When working with Angular, we have access to the [Angular CLI](https://cli.angular.io/) (Command Line Interface), which allows us to easily create applications in a consistent way, and also (if used correctly) can take care of importing/exporting files from certain locations.  The Angular CLI is a very efficient tool for building Angular apps.


The CLI provides several commands to manage the whole application life cycle, going from the scaffolding of the initial structure to the deployment of the application to production.

You can install the CLI easily with NPM:

```bash
$ npm install -g @angular/cli
```

You can test that everything works by calling `ng --help`:

```
$ ng --help
ng build <options...>
  Builds your app and places it into the output path (dist/ by default).
  aliases: b
  --target (String) (Default: development) Defines the build target.
    aliases: -t <value>, -dev (--target=development), -prod (--target=production), --target <value>
  --environment (String) Defines the build environment.
    aliases: -e <value>, --environment <value>
  --output-path (Path) Path where output will be placed.
  ...
```


A few useful commands packaged with the CLI include:

| Command | Description
|---------|--------
| ng new <project-name> | Creates a new, functional Angular application.
| ng generate | Generates component, directive, route, service and pipe with a simple command
| ng serve | Run your application on a local server for testing purposes

Ready to start playing with Angular?

## Angular HelloWorld

To start playing with Angular, we need to create a new Angular project. We can do that easily with the CLI:

```
$ ng new angular-helloworld
  create angular-helloworld/README.md (1033 bytes)
  create angular-helloworld/.angular-cli.json (1253 bytes)
  create angular-helloworld/.editorconfig (245 bytes)
  create angular-helloworld/.gitignore (544 bytes)
  ...
You can `ng set --global packageManager=yarn`.
Project 'angular-helloworld' successfully created.
```

Then, run:
```
$ cd angular-helloworld
```

This will navigate into the folder we just created, called `angular-helloworld` with the starter code provided by the Angular CLI. The folder comes with its own git repository with one commit:

```
$ git log
commit c3b102c56aeffec9d956cd0f3bf9fbad1ea2f86e (HEAD -> master)
Author: Angular CLI <angular-cli@angular.io>
Date:   Wed Apr 11 15:38:39 2018 -0400

    chore: initial commit from @angular/cli

        _                      _                 ____ _     ___
       / \   _ __   __ _ _   _| | __ _ _ __     / ___| |   |_ _|
      / △ \ | '_ \ / _` | | | | |/ _` | '__|   | |   | |    | |
     / ___ \| | | | (_| | |_| | | (_| | |      | |___| |___ | |
    /_/   \_\_| |_|\__, |\__,_|_|\__,_|_|       \____|_____|___|
                   |___/
(END)
```

The CLI comes with a built-in Express server that allows us to play with the front-end code. To start this development server, all we need to do is run the `ng serve` and go to [http://localhost:4200](http://localhost:4200).

```
$ ng serve

** NG Live Development Server is listening on localhost:4200, open your browser on http://localhost:4200/ **
Date: 2018-04-11T19:43:04.662Z                                                 Hash: fecea84e956af33e09ce
Time: 5419ms
chunk {inline} inline.bundle.js (inline) 3.85 kB [entry] [rendered]
chunk {main} main.bundle.js (main) 17.9 kB [initial] [rendered]
chunk {polyfills} polyfills.bundle.js (polyfills) 554 kB [initial] [rendered]
chunk {styles} styles.bundle.js (styles) 41.5 kB [initial] [rendered]
chunk {vendor} vendor.bundle.js (vendor) 7.42 MB [initial] [rendered]

webpack: Compiled successfully.
```

If everything went fine, you should see this in your browser:

![](https://i.imgur.com/4gKogrq.png?1)

### Set default CLI configuration

Angular-CLI comes with a default configuration file in `angular-cli.json` where we can see the options defined for our project.

We want to change just a few to simplify our workflow throughout the course.

Open the terminal window (if ng serve is still running, either open a new tab or push control+c) and run:

- set inline style and templates for our components by running:
```
$ ng set defaults.inline.style true
$ ng set defaults.inline.template true
```

- Remove auto-generated test files by running:
```
$ ng set defaults.spec.component false
$ ng set defaults.spec.pipe false
$ ng set defaults.spec.service false
```



## Project Structure

The project folder contains many different files to start with, but our Angular application lives inside the  `src` folder, which looks like this:

```
src/
├── app    <-- APP is where we will write our code!
│   ├── app.component.spec.ts
│   ├── app.component.ts
│   ├── app.module.ts
│   └── index.ts
├── assets
├── environments
│   ├── environment.prod.ts
│   └── environment.ts
├── favicon.ico
├── index.html    <-- The only HTML file!
├── main.ts
├── polyfills.ts
├── styles.css
├── test.ts
├── tsconfig.json
└── typings.d.ts
```

We should notice that Angular usually follows the [Single Page Application (SPA)](https://en.wikipedia.org/wiki/Single-page_application) architecture. Therefore we will only have one HTML document that will be dynamically modified by the user.

In this course, we are going to do amazing things with Angular and all of them will require us to only venture into the `app` directory!

## Components

### Introduction

We can think of an Angular application as a set of components in which we design and build each component individually and stack them together to create our application.

![](https://i.imgur.com/AeFtc1U.png)

We also design each component to work with the others and ultimately to provide an excellent user experience.

### Structure

**Angular components are made by a template, a class and some metadata.**

The ++template++:

- Provides the View Layout.
- Is created with HTML.
- Includes Bindings and Directives.

The ++class++:

- Defines code to support the view.
- Is created with TypeScript.
- Defines properties and methods.

We attach a specific decorator to our `@Component` class defining some metadata to tell Angular the nature of the class: how to build it and how to configure it.

Looking at the `app.component.ts` file, we can see that the `@Component` decorator takes a configuration object with the metadata.

```typescript
@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'app';
}
```

Before, we included all the HTML as a string within the component, but we can also break these out into separate files. The `templateUrl` and `styleUrls` strings give file paths to HTML and CSS files for this specific component. We can see the current HTML for this component in the `app.component.html` file.

```htmlmixed
<!--The content below is only a placeholder and can be replaced.-->
<div style="text-align:center">
  <h1>
    Welcome to {{ title }}!
  </h1>
  <img width="300" alt="Angular Logo" src="data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNTAgMjUwIj4KICAgIDxwYXRoIGZpbGw9IiNERDAwMzEiIGQ9Ik0xMjUgMzBMMzEuOSA2My4ybDE0LjIgMTIzLjFMMTI1IDIzMGw3OC45LTQzLjcgMTQuMi0xMjMuMXoiIC8+CiAgICA8cGF0aCBmaWxsPSIjQzMwMDJGIiBkPSJNMTI1IDMwdjIyLjItLjFWMjMwbDc4LjktNDMuNyAxNC4yLTEyMy4xTDEyNSAzMHoiIC8+CiAgICA8cGF0aCAgZmlsbD0iI0ZGRkZGRiIgZD0iTTEyNSA1Mi4xTDY2LjggMTgyLjZoMjEuN2wxMS43LTI5LjJoNDkuNGwxMS43IDI5LjJIMTgzTDEyNSA1Mi4xem0xNyA4My4zaC0zNGwxNy00MC45IDE3IDQwLjl6IiAvPgogIDwvc3ZnPg==">
</div>
<h2>Here are some links to help you start: </h2>
<ul>
  <li>
    <h2><a target="_blank" rel="noopener" href="https://angular.io/tutorial">Tour of Heroes</a></h2>
  </li>
  <li>
    <h2><a target="_blank" rel="noopener" href="https://github.com/angular/angular-cli/wiki">CLI Documentation</a></h2>
  </li>
  <li>
    <h2><a target="_blank" rel="noopener" href="https://blog.angular.io/">Angular blog</a></h2>
  </li>
</ul>
```

Let's add an additional element to the page. First add a new string variable to the `app.component.ts` file: 

```typescript
export class AppComponent {
  title = 'app';
  content = 'A new world is waiting for you!'
}
```

Then add that property name to the `app.component.html` file

```htmlmixed
...
  <h1>
    Welcome to {{ title }}!
  </h1>
  <h2>
    {{ content }}
  </h2>
...
```

:::info
:bulb: We name our classes in [PascalCase](https://en.wikipedia.org/wiki/PascalCase), adding a `Component` suffix to promote readability throughout the application.
:::

:::info
:bulb: As Angular convention, we also name the root component of every Angular application `AppComponent`.
:::

### Multiple components

We know that Angular applications are made up of many components.

Now that we know how a simple component is made, we need to learn how to make two or more components work together.

Let's create another component. From the terminal, run:

:::info
:bulb: It would help to open another tab in your terminal in the same directory, so you can continue to run your angular-cli server and run ng-cli commands.
:::
```
$ ng g component my-nested
```

:::info
:bulb: When creating components we should add a custom prefix, in this case "my", to avoid any possible collisions with current (or future) native HTML elements.

Also, angular-cli enforces [`kebab-case`](http://wiki.c2.com/?KebabCase) when generating components.
:::

Inspecting the new file at `app/my-nested/my-nested.component.ts` we can see the component selector is `app-my-nested`, and its template comes with a pre-defined paragraph saying "my-nested Works!".

Now we can use the `app-my-nested` selector in any other HTML template of our `NgModule` to load this component.

Let's add the `app-my-nested` selector into our `app.component.html` template file:

```htmlmixed
...
  <h1>
    Welcome to {{ title }}!
  </h1>
  <h2>
    {{ content }}
  </h2>
  <app-my-nested></app-my-nested>
...
```

Doing so we expect out `app-my-nested` component to be loaded right after the `<h2>` tag. This is the result:

![](https://i.imgur.com/lpNO60W.png?1)


### Components style

Angular applications are styled with regular CSS. That means we can apply everything we know about CSS stylesheets to our Angular applications directly.

For every Angular component we write, we may define not only an HTML template, but also the CSS styles that go with that template, specifying any selectors, rules, and media queries that we need.

The Angular CLI creates a CSS file for a given component when creating it, you can add styles there as expected. Add the following style rule to the `app.component.css` file.

```css
h1, h2, p {
  color: blue;
}
```

The cool thing here is that the CSS rules written in this file will only apply to elements of **the current component's template**. They are *localized* styles. 

If we refresh the page we can see that the paragraph in the `my-nested.component.ts` is still shown in black color, even though we added a rule to make paragraph tags blue.

![](https://i.imgur.com/zDARS66.png?1)


### Tying a component together

Before we can use an external function or class we need to define where the code is located.

To do that we use the TypeScript `import` statement. The `import` statement allows us to use exported members from external modules.

When defining a component we need to tell the module loader where to find the `@Component` decorator.

We import it from the `angular2/core`  library.

```typescript
import { Component } from 'angular2/core';
```

We also use the export keyword in the class definition to make the component available for use by other components in the application (actually, in the same module as we will see later).

```typescript
export class AppComponent {}
```

Angular CLI's commands such as `ng g component cmp-name` comes with an auto-import feature that makes this step automatic to us, but when adding files manually we need to make sure everything we reference is imported (or exported) at the beginning of the file.

### What is Metadata? (Advanced)

In TypeScript we add *metadata* to a class through decorators. The component decorator is called `@Component`. A class becomes an Angular component when we apply the component decorator to it with its metadata. Angular needs this metadata to:

- Know how to build the component.
- Construct its view.
- Interact with the component.

A decorator is applied by positioning it immediately in front of the feature to be decorated. When decorating a class *we place the decorator immediately above the class*.

We know from the lesson 1 that a decorator is a function, that's why we use parenthesis after its definition.

In most cases we'll also specify an object as a parameter containing all the properties to configure the target.

In this case we specify 3:
- `selector` is specified when we need to reference our component in the HTML.
- `templateUrl` is the HTML rendered by Angular every time we use the tag `<app-root></app-root>` in our application
- `styleUrls` is an array of file paths to CSS files used to style the HTML in the `templateUrl` file.

There a lot more available and we'll use them all down the road the full list is available in the [component documentation](https://angular.io/docs/ts/latest/api/core/index/Component-decorator.html).

## Summary

In this lesson we have learnt how to create an ng application using the Angular CLI. We have seen what Angular components are, and how to use them in our application using import and export statements.
