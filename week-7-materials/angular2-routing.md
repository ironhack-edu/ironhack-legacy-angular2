![Ironhack Logo](https://i.imgur.com/1QgrNNw.png)

# Routing

## Learning Goals
After this lesson, you will be able to:

* Use the Angular routing library.
* Add multiple pages to your single page application.
* Make pages interact with parameters and services.

## Introduction

In web development, routing means splitting the application into different areas, usually based on rules that are derived from the current URL in the browser.

For instance, if we visit the `/` path of a website, we may be visiting the home route of that website. Alternatively, if we visit `/about`, we might want to render the “about" page.

Defining routes in our application is useful because we can:
- Separate different areas of the app.
- Maintain the state of the app.
- Protect areas of the app based on certain rules (authorization).

So, if we're writing a single page app, why do we ever have to change the URL in the browser?

If we didn't, we wouldn't be able to:

- Refresh the page and keep your location within the app.
- Bookmark a page and come back to it later.
- Share the URL of that page with others.

## The Angular Router

**Client-side routing means using JavaScript to switch the pages in our application without actually making a request to the server.**

Advantages of using a client-side router include:

- Sending less data: The browser doesn't need to load a brand new page sent from the server on each request.
- (Ideally) speedier load times
- Being able to display loading notifications

**With the introduction of HTML5, browsers acquired the ability to programmatically create browser history entries with unique URLs without the need for a server request to accompany each entry.** Without this change, client-side routing wouldn't have been possible.

While we could directly take advantage of HTML5 features to enabling client-side routing, there are a lot of ready-to-go libraries for us.

Angular provides its own, the **Angular Router**, which we will use in this lesson.

:::warning

:exclamation: There are two things you need to be aware of when using the HTML5 mode routing feature:

1. Not all browsers support HTML5 mode routing, so if you need to support older browsers you might be stuck with hash-based routing for a while.

2. The same thing is true for servers!  Not all servers support HTML5 based routing so you will need to check your server as well.  
:::

## Installation

In order to use the Angular router, we need to import all the needed components from the Angular framework and bind them to our component’s injector.

:::info
:bulb: We scaffolded our application using the angular-CLI, so the router library comes pre-installed in our node-modules, although it is up to us to import it and use it. 
:::

Without the CLI, we would need to run
```
npm install --save @angular/router
```



## Usage

When using routing in Angular we rely on three main components:

- **Routes** describes the routes our application supports.
- **RouterOutlet** is a “placeholder” component that shows Angular where to put the content of each route.
- **RouterLink** is a directive used to link to client-side routes.

In this section we'll look at them more closely while building a simple application with 2 pages: *home* and *about*.

**If you haven't already, go ahead and generate a new project using the Angular CLI.**  

### Setting Up Routes

Routes are simple objects that map a string token to a component. Every time the token is hit by the browser navigation, the router activates the corresponding component.

To define the routes, we have to add the following in the `app.module.ts` file:

```typescript
// app.module.ts
// ...import statements
import { Routes } from "@angular/router";

const routes: Routes = [
  { path: '', redirectTo: 'home', pathMatch: 'full' },
  { path: 'home',  component: MyHomeComponent },
  { path: 'about', component: MyAboutComponent }
];
// ...rest of app.module.ts
```

We are configuring the router so that it:
- Redirects to the `home` path every time the path is empty.
- Activates the `MyHomeComponent` every time the path `home` is visited.
- Activates the `MyAboutComponent` every time the path `about` is visited.

We haven't created the two components we are referencing, so run:
`ng g component my-home` and `ng g component my-about` to generate them.

:::info
:bulb: The Angular CLI team is working on a command to create multiple components at once, but for now we need to run multiple commands.
:::

As of now the components are generated with a placeholder text that is enough for us. Let's move on before editing these components.

:::info
<!-- Don't remove the ``` backticks around this snippet. The base href tag breaks HackMD -->
```
Remember to add a `<base href>` element tag in the <head> of the index.html.
We define `<base href="/">` because our application is loaded in the root
server directory.
```
:::

### Add routes to our app

The array of routes needs to be provided in the container module, the `AppModule`.

In this module, we need to use the `forRoot` method provided by the `RouterModule` service in the imports metadata:

```typescript
// app.module.ts

import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [
  { path: '', redirectTo: 'home', pathMatch: 'full' },
  { path: 'home',  component: MyHomeComponent },
  { path: 'about', component: MyAboutComponent }
];

imports: [
  BrowserModule,
  RouterModule.forRoot(routes)  //  <!-- "routes" is the array defined above
];
...
```

We imported `RouterModule` and `Routes` definitions and added them to our top level module. The router is now configured.

### Routing outlets

If we run the app now with `ng serve`, it loads the default HTML in AppComponent and redirects to home, but doesn't show the HTML for the my-home component. How do we fix this?

We introduced the `RouterOutlet` directive in the previous section. It is a placeholder that tells the router where to put the active component.

Because we are using our `AppComponent` as the parent of all other components (as is usually the case),  we can put our router outlet tag here.  Since this component is always being shown, activating different components within this one will create the same effect as having multiple pages in our application.  

**In other words, depending on which route you visit, the `<router-outlet>` tag will be replaced by whichever component is associated with the current route.**

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <h1> {{ title }} </h1>
    <router-outlet></router-outlet>
  `
})
export class AppComponent {
  title:   string = 'Welcome to Angular';
}
```

Now we see the home component loaded, and if you notice the router redirected us to the `home` URL as specified.

![](https://i.imgur.com/3tgvdUd.png)

If we manually type `about` in the URL we see the browser refreshes the page and then the Router loads the `MyAboutComponent`.

It works, but this is not the user experience we want. First of all, we want links, and we also don't want the page to be refreshed every time.

### Page navigation

Let's go ahead and add a couple links to the top of our page.

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <nav>
      <ul>
        <li> <a href="/home"> Home </a> </li>
        <li> <a href="/about"> About </a> </li>
      </ul>
    </nav>
    <h1> {{ title }} </h1>
    <router-outlet></router-outlet>
  `
})
export class AppComponent {
  title:   string = 'Welcome to Angular';
}
```

The result should look something like this:

![](https://i.imgur.com/kRegfnN.gif)

The routing works - every time we browse to a page, the right component is loaded. But we don't want the app to be refreshed every time.

#### Navigation with links

The `RouterLink` directive replaces the `href` attribute of links, enabling the client-side routing we are looking for:

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <nav>
      <ul>
        <li> <a [routerLink]="['home']"> Home </a> </li>
        <li> <a [routerLink]="['about']"> About </a> </li>
      </ul>
    </nav>
    <h1> {{ title }} </h1>
    <router-outlet></router-outlet>
  `
})
export class AppComponent {
  title:   string = 'Welcome to Angular';
}
```

Every page is now dynamically loaded on-the-fly by Angular. This creates a much smoother experience for the user.

![](https://i.imgur.com/kZi5cLf.gif)

#### Imperative

This is good, but sometimes we need to navigate through our app by other means, without links. Sometimes, we need to navigate after a form is submitted, or we might want to redirect users based on their current permissions.

To do this, we need to inject our component with the `Router` service provided by the `@angular/router` library, and use its `navigate` method.

Let's add a button in our `my-about.component.html` to navigate back to the home page, and add a navigation method to `my-about.component.ts`:

```htmlmixed
<p>my-about works!</p>
<button (click)="goToHome()"> home </button>
```

```typescript
import { Component, OnInit } from '@angular/core';
import { Router } from '@angular/router';

@Component({
  selector: 'app-my-about',
  templateUrl: './my-about.component.html',
  styleUrls: ['./my-about.component.css']
})
export class MyAboutComponent implements OnInit {

  constructor(private router: Router) { }

  ngOnInit() {}

  goToHome() {
    this.router.navigate(['/home']);  // <!-- Programmatically navigate to home
  }
}
```

And the result:

![](https://i.imgur.com/vKV37gC.gif)

### Route parameters

In our apps we often want to navigate to a specific resource. For instance, let's say we have a contact list application.

Each contact may have an ID, and if we had a contact with ID 3 then we might navigate to that contact by visiting the URL `/contact/3`, while if we had a contact with an ID of 4 we would access it at `/contact/4` and so on.

Obviously we don't want to write a route for each contact, but we want to use a variable, or route parameter. We can specify that a route takes a parameter by putting a colon `:` in front of the path segment.

We might end up with something like this. Go ahead and change our routes variable in `app.module.ts` to the following:

```typescript
const routes: Routes = [
  { path: '', component: ContactListComponent },
  // { path: 'contact/:id', component: ContactComponent }
];
```

We have the default route that shows a list of contacts, and the route that reads the parameter in the URL and shows the specific contact. The latter route is commented out currently, as it will let us test to make sure the first route works without requiring the second - when we make a component for individual contacts, we'll uncomment this.

First, let's build our contact-list component. it will be a simple table displaying the contacts, with a button to view the contact details:

#### Contact List

Run `ng g component contact-list` and paste the following code segments into the HTML, TypeScript, and CSS files for the component, respectively:

```htmlmixed
  <table>
    <tr>
      <th> Id</th>
      <th> Name </th>
      <th colspan="2"> Details </th>
    </tr>
    <tr *ngFor="let contact of contacts">
      <td> {{ contact.id }} </td>
      <td> {{ contact.name }} </td>
      <td>
        <button (click)="viewDetails(contact.id)"> details (btn)  </button>
      </td>
      <td>
        <a [routerLink]="['contact',contact.id]"> details (link) </a>
      </td>
    </tr>
  </table>
```

```css
td { 
  min-width: 50px; 
  padding: 10px; 
  text-align: center; 
  }
```

```typescript
import { Component, OnInit } from '@angular/core';
import { Router } from '@angular/router';

@Component({
  selector: 'app-contact-list',
  templateUrl: './contact-list.component.html',
  styleUrls: ['./contact-list.component.css']
})
export class ContactListComponent implements OnInit {
  contacts: Array<Object> = [
    { id: 100, name: 'Andy' },
    { id: 201, name: 'George' },
    { id: 302, name: 'Max' }
  ];

  constructor(private router: Router) { }

  ngOnInit() {}

  viewDetails(id) {
    this.router.navigate(['contact', id]);
  }
}
```

So far we have a table that displays three contacts, and two ways to access the details of each contact (button and link), both of which navigate to the `contact` route passing in the `id` of the contact as a second parameter. Don't forget to remove the old nav links from `app.component.ts`:

![](https://i.imgur.com/yI1k8Fa.png)

#### Single Contact

Now we need to build the `contact` component and read the parameter.

Run `ng g component contact`

and then uncomment the route in `app.module.ts`.

The Router will need to extract the route parameter (id) from the URL and supply it via the `ActivatedRoute` service.

```typescript
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

@Component({
  selector: 'contact',
  template: `
    <h3>
      This is the page for the contact: {{ contactId }}
    </h3>
    <a [routerLink]="['/']"> Back to list </a>
  `
})
export class ContactComponent implements OnInit {
  contactId: Number;

  constructor(private route: ActivatedRoute) { }

  ngOnInit() {
    this.route.params
      .subscribe((params) => this.contactId = Number(params['id']));
  }
}
```

The `ActivatedRoute` service allows us to access the route parameters through an observable. In order to retrieve it we need to subscribe to it.

:::info
:bulb: Calling `Number()` with `params['id']` just converts the parameter from a string to a number.
:::

Final result:

![](https://i.imgur.com/ifNadFh.gif)

## Summary

In this lesson we have set up the routes of our application to be able to navigate between pages using the RouterLink directive.

We have also created a way to navigate between pages using the Router Service in our components' class.

Finally, we have added parameters to the URL and used them to load data dynamically.

## Extra Resources
    * [Router Documentation](https://angular.io/docs/ts/latest/guide/router.html#!#route-parameters) - The official Angular documentation
