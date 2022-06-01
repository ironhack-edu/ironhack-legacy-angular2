![Ironhack Logo](https://i.imgur.com/1QgrNNw.png)

# Services

## Learning Goals
After this lesson, you will be able to:

* Use services to share information within your application.
* Use Angular Dependency Injection to share services across the application.
* Explain services to a novice.

## What Are Services?

A service is a class that lets us define functionalities independent from our interfaces and components.

We use services to:

- Write application logic.
- Share code among components.
- Wrap external communication such as AJAX, Web socket, local storage, etc.

A service should be:

- Reusable.
- Independent from the views and templates.

A service could:

- Consume other services using dependency injection.
- Be used in components using dependency injection.

:::info
:bell: We will talk about **dependency injection** in the second part of the lesson!
:::

## Using Services

Let's create an example service that is designed to share logic. The service we will create is a `Counter` service that lets us increment a *count* value.

Why create a service for this?  Well, we could just write this logic directly into one of our components, but what if we want multiple components to have access to this information?  This is where services come in.  We will see how in a moment.

Let's keep working in the same project we created in the previous lesson.

To create this new service let's create a folder called `services` inside the `app` folder, and a new file called `counter.service.ts` in it.

Here's the code:

```typescript
import { Injectable } from '@angular/core';

@Injectable()
export class CounterService {
    count: number = 0;
    constructor() { }

    increment() {
        this.count++;
        console.log(`Count is now ${this.count}`);
    }
}
```

We have:

- A class `CounterService` that is our service and is prefixed with the `@Injectable` decorator.
- The `Injectable` decorator is imported from the `@angular/core` library.
- The `CounterService` exposes a method called `increment` that increments and logs the current count.

:::info
:information_source: `@Injectable` doesn't require a metadata argument.
:::

Now that we've got our `CounterService`, lets see how we can use it in our components.

### Service Injection

In most cases, services contain functions, and they expose these functions to components.  Our service has a function (the one we wrote to increment `count` by one) and now we need to make this function available for use in our components.   

Let's create a new component with:

```
$ ng g component my-counter
```

and create a button to increment the `Counter` count:

```typescript
import { Component, OnInit } from '@angular/core';
import { CounterService } from '../services/counter.service';

@Component({
  selector: 'app-my-counter',
  template: `
    <p> My Counter </p>
    <button (click)="incrementCounter()"> increment </button>
  `,
  styles: [],
  providers: [CounterService]
})

export class MyCounterComponent implements OnInit {
  constructor(private theCounter: CounterService) {}

  ngOnInit() {}

  incrementCounter() {
    this.theCounter.increment();
  }
}
```

There are a few things going on here. First, some stuff we've already seen:

- We have a `my-counter` component with a button that calls the `incrementCounter` method of the class.
- We have defined a method called `incrementCounter` in the component class.


Now for the new stuff:

- We imported the `CounterService` from the file (line #2).
- We added the `CounterService` to the new component metadata `providers` (line #11).
- We *injected* the service into the component constructor as a private property (line #15).
- We called `CounterService`'s `increment()` method inside the `incrementCounter()` method (line #20).

It may seem complicated at first, but you can think of it kind of like a small chain-reaction of simple things.  

- We click the button
- The `incrementCounter()` method is executed
- This method calls the `increment()` method in the service
- `count` is incremented by one and logged in the console.


That's it. If we add the component's tag to the `app.component.ts` HTML template metadata, reload, and click the button, we should see some logs in the console:

![](https://i.imgur.com/M5HfmIn.gif)


### Multiple instance vs Singleton

Our `providers` metadata takes an array of services, where we can add as many services we want.

Let's see what happens if we create a second component, a copy of `my-counter`, and interact with the service in the same way:

We add a new component with:
```
ng g component my-second-counter
```
Copy the code from the `my-counter` component (don't overwrite the selector or exported class name) and **make sure to add both components into your app-component template** so we can see them on the same page together.  

We should get something like this:

![](https://i.imgur.com/rJhP1Nm.gif)

It works...sort of.  Each of the two `my-counter` buttons does increment the counter.  The problem is that the two buttons seem to be incrementing two different counters.  

What's happenning here is that each component is receiving a new instance of the `CounterService`.

In many cases, we may want to achieve this effect, as we are achieving *code reusability*. In this case, though, we want to share one instance of our service across the whole application. How can we do that?

We need to register our service in our `NgModule` instead of in our components directly.

So in our `app.module.ts` we can add a `providers` metadata containing the counter service:

```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
...
import { CounterService } from './services/counter.service';

@NgModule({
  declarations: [
      // Declarations
  ],
  imports: [
      // External modules
  ],
  providers: [CounterService],      // <-- Register the CounterService
  bootstrap: [AppComponent]
})
export class AppModule { }
```

And delete `providers: [CounterService]` from `my-counter` and `my-second-counter`.

Running the app now, we can see both components act on the same unique instance of the `CounterService`.

![](https://i.imgur.com/Lmqx35n.gif)

We register services in the container `NgModule` if we need a single instance of the service (singleton), otherwise in every component that requires it (multiple instance).

:::success
:sunglasses: **Best Practice**- If a service needs to keep track of information globally, _provide_ the service in `NgModule`.

If a service's functionality is specific to each individual component, _provide_ the service in each component.
:::

### Dependency Injection (Advanced)

[Dependency Injection (DI)](https://en.wikipedia.org/wiki/Dependency_injection) is a system to make parts of our program accessible to other parts of the program, and we can configure how that happens.

> A **dependency** is an object that can be used. For example: a service.
>
> An **injection** is the passing of a dependency to a dependent object (a client) that would use it.
>
> The service is made part of the client's state. Passing the service to the client, rather than allowing a client to build or find the service, is the fundamental requirement of the pattern.

![](https://i.imgur.com/7FY80W1.png)

Angular comes with its own dependency injection system which allows us to create and inject services throughout our application. It's composed of three elements:

- The **Provider** - maps a token (that can be a string or a class) to a list of dependencies. It tells Angular how to create an object, given a token.
- The **Injector** - holds a set of bindings and is responsible for resolving dependencies and injecting them when creating objects.
- The **Dependency** - is being injected.

Let's see how all this theory makes our life easier when it comes to building applications.

## A Real World Example

In this section we will build a contact service which will hold a list of contacts and expose 3 methods:

- `getList` will return all the contacts.
- `get` will return only the contact having the specified id.
- `edit` will override the specified contact and return the new one.

For this purpose we have the chance to install a 3rd party library that may be familiar to you: [`underscore.js`](http://underscorejs.org/).

Type:

`$ npm install --save underscore`
`$ npm install --save @types/underscore`

To install both the library and its TypeScript definition, which lets us import it as a plain module with this line:

```typescript
import * as _ from 'underscore';
```

We're going to use the [`findWhere`](http://underscorejs.org/#findWhere) method in our service class.


Now run `$ ng g service contact-service` and paste:

```typescript
import { Injectable } from '@angular/core';
import * as _ from 'underscore';

@Injectable()
export class ContactService {

  contacts: Array<Object> = [
    { id: 100, name: 'Andy' },
    { id: 201, name: 'George' },
    { id: 302, name: 'Max' }
  ];

  constructor() { }

  getList(): Array<Object> {
    return this.contacts;
  }

  get(idContact: number): Object {
    return _.findWhere(this.contacts, { id: idContact });
  }

  edit(contact) {
    let idx = _.findIndex(this.contacts, { id: contact.id });
    if (idx >= 0) {
      this.contacts[idx] = contact;
    }
    return this.contacts[idx];
  }
}
```

After creating the service we need to declare it as a provider on our `app.module.ts` file:

```javascript
// app.module.ts

//....code
import { ContactService } from './contact.service'

//....more code
providers: [ ContactService ],
//....more code
```

Now, let's see an example of how we can use the `contact.service`. On our `contact-list` component, instead of having the list of our contacts, we will retrieve that data from the `service`.

First we need to import the `ContactService` on the `contact-list.components.ts`:

```javascript
import { ContactService } from '../contact.service';
```

So we can just call the `getList()` method of the service to get all contacts:

```javascript
export class ContactListComponent implements OnInit {
  contacts: Array<Object> = [];
  constructor(private router: Router, private contactService: ContactService) { }

  ngOnInit() {
    this.contacts = this.contactService.getList();
  }

  viewDetails(id) {
    this.router.navigate(['contact', id]);
  }
}
```

:::info
Notice we remove the info of the contacts from the `contacts` variable, but we kept the `type` declaration. Then, on our `ngOnIniti()` method, we call the `getList()` method to get the info from the `service`.

:bulb: Remember to declare the `contactService` on the constructor!
:::

If everything goes ok, we should see the same we had before, but using our service. On the following lessons, we will use the other methods we have on our `service`!

![image](https://user-images.githubusercontent.com/23629340/39175837-9d75560a-47ab-11e8-9d68-7fd270c3afbf.png)

:::info
For now, we're keeping all the contacts here in the service, while in the future we'll retrieve them from a server using HTTP requests.
:::

## Summary

In this lesson we have learnt how to write Angular services to share code in the application, and how to use Angular's dependency injection to use those services in an organized manner.

## Extra Resources
* [Dependency Injection](https://en.wikipedia.org/wiki/Dependency_injection) - wikipedia
