![Ironhack Logo](https://i.imgur.com/1QgrNNw.png)

# Component To Component Communication

## Learning Goals
After this lesson, you will be able to:

* Pass information between two components.
* Use Input and Output decorators to make nested components communicate.
* Use components' lifecycle hooks.


## Introduction

Our UX design can include features that are complex enough to be broken into separate components. And those components may or may not be used across other views within our application.

In this section we'll concentrate on building components that are designed to be nested within other components. We'll also look at how communication between child and parent components is done.

## Component Interaction

We saw in the components lesson that we can nest components within other components, eventually building a tree of components that comprises our application.

When working with Angular, we can use inputs (as property bindings) and outputs (as event bindings) to allow our components to communicate with their parents/children. That may sound abstract - that's because it is!  Don't worry, we will examine some concrete examples.  

![](https://i.imgur.com/r1v2GlZ.png)

In Angular, data flows from top to bottom, while events flow from bottom to top.

In this section, we'll build a nested component, then we'll learn how to use that nested component as a directive in a container component.

We'll discuss how to pass data to the nested component as a property using the `@Input` decorator, and how to pass data out of the nested component by raising an event defined with the `@Output` decorator.

### Input

Any component can receive inputs through the `@Input` decorator.

**The `@Input` decorator is used to pass data from a parent component to a child component which needs it to function**.

As an example we'll build a quotes list. We'll display a list of quotes and provide a button to remove quotes.

Thinking about it at a high level, we'll have something like this:

:::danger
:bomb: **Pseudocode Warning**
:::

```htmlmixed
<!-- PSEUDO-CODE -->
<quote-list>
  <ul>
    <li *ngFor="let quote of quotes">
      <quote-item [quote]="theQuote"></quote-item>
    <li>
  </ul>
</quote-list>
```

A `quote-list` component holds a list of quotes in the `quotes` model and iterates through it.

For each quote it generates a nested quote item component, and passes that component data about the quote.

This is not really far from what we'll build. Thinking about reusable components is a good starting point when modeling our applications.

Let's go through the full process together.  You can continue working in the project we began in the previous lesson (angular-forms).  

#### Quote Item

Let's start from the bottom, our `quote-item` component.

Type `ng g component quote-item` to generate the component, and paste this code:

```typescript
import { Component, OnInit, Input } from '@angular/core';

@Component({
  selector: 'quote-item',
  template: `
    <span> {{ theQuote.text }} </span>
    <small> {{ theQuote.author }} </small>
  `,
  styles: [`span { display: block; }`]
})
export class QuoteItemComponent implements OnInit {
  @Input() theQuote: any;

  constructor() { }

  ngOnInit() {}

}
```

We have the new `Input` keyword here. This decorator lets us define an input property that we have created called `theQuote`.  This property is just like any other class property we define inside our component, except that instead of defining it right there, the **parent component will have to pass data into `QuoteItemComponent`** to set the value of `theQuote`.

We print `theQuote`'s `name` and `author` properties in our template, so that we have a small reusable component ready to display a quote.

To see this in action let's build our `quote-list` component.

#### Quote List

Use `ng g component quote-list` to generate the component, then:

```typescript
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'quote-list',
  template: `
    <h1> Welcome to Angular </h1>
    <h3> An awesome quote list! </h3>
    <ul>
      <li *ngFor="let quote of quotes">
        <quote-item [theQuote]="quote"></quote-item>
      </li>
    </ul>
  `
})
export class QuoteListComponent implements OnInit {
  quotes: Array<any> = [{
    id: 1,
    text: 'Walk as if you are kissing the Earth with your feet.',
    author: 'Thich Nhat Hanh'
  }, {
    id: 2,
    text: 'Life is a journey. Time is a river. The door is ajar.',
    author: 'Jim Butcher'
  }, {
    id: 3,
    text: 'Simplicity is the ultimate sophistication.',
    author: 'Leonardo da Vinci'
  }];

  constructor() {}

  ngOnInit() {}
}
```

Here we have an array of quote objects composed of `id`, `text` and `author` properties. 

Inside the `<li></li>` tag we added our custom `quote-item` component and passed it an input, setting its `theQuote` property equal to `quote`, which is the current quote we are iterating through with the `ngFor` directive.


![](https://i.imgur.com/VU7fa92.png)

### Output

We just saw how a parent component can pass data to its child component through property binding. We demonstrated that data flow is from top to bottom, from parent to child, as we mentioned.

We now want the child component to communicate with its parent.

This is done using the `@Output` decorator. In short, the `@Output` decorator **is used to create custom events in our components**.

#### Custom Events?

Consider for a moment the `(click)` event we've already discussed. `(click)` is an **event** that is attached to every element by default.

We typically use the `(click)` event to execute some code when a particular element receives a click. Usually, we accomplish this by attaching a function to our click event, so that the function runs when the element is clicked. From there, we implement the function.

With the `@Output` decorator, we can define our own events like `(click)` in our components.

#### Quote Item

Let's say we want the `quote-item` component to execute some function when the user deletes the quote (from within the quote-item template).  

Well, since there is no `delete` event by default, we can create our own `onDelete` event.  It's good idea to get into the habit of naming our events something like `onDelete` rather than `delete`, or other words that are common in computer programming, in order to avoid confusion and make it clear that this is a custom event.  (You could even name it something like `onMyCustomDeleteEvent` if it helps make it more clear.)

We will use the `@Output` decorator to do this:

```typescript
import {
  Component,
  OnInit,
  Input,
  Output,
  EventEmitter
} from '@angular/core';

@Component({
  selector: 'quote-item',
  template: `
    <span> {{ theQuote.text }} </span>
    <small> {{ theQuote.author }} </small>
    <button (click)="onQuoteDelete()"> Delete </button>
  `
})
export class QuoteItemComponent implements OnInit {
  @Input() theQuote: any;
  @Output() onDelete = new EventEmitter<string>();

  constructor() { }

  ngOnInit() {}

  onQuoteDelete () {
    this.onDelete.emit(this.theQuote.id);
  }
}
```

All `Output`s we define are instances of the `EventEmitter` type.

The `EventEmitter` emits an event through the `emit` method. When the `EventEmitter` sends its message, we can have other components listen to it.

Here we just added a `remove` button and bound its `click` event to a class method the we called `onQuoteDelete`. (You can call this method whatever you want.) 

Here, in our `onQuoteDelete` method, we have one line of code.  All we are doing is emitting our event called `onDelete` and we are passing in `this.theQuote.id`.  

So, now that we have set up our custom event, how do we use it to communicate with the parent component?

To do that, we will need to make some adjustments to the parent component, the `quote-list` component, so that it can catch our event when it fires.   

#### Quote List

```typescript
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'quote-list',
  template: `
    <h3> An awesome quote list! </h3>
    <ul>
      <li *ngFor="let quote of quotes">
        <quote-item [theQuote]="quote" (onDelete)="removeQuote($event)">
        </quote-item>
      </li>
    </ul>
  `
})
export class QuoteListComponent implements OnInit {

  // The quotes object

  removeQuote (id) {
    this.quotes = this.quotes.filter(
      (quote) => quote.id !== id
    );
  }
}
```

Once you have created a custom event, it is relatively easy to catch it in the parent component.  

Now that we have defined an `onDelete` event for the `quote-item` component, any `<quote-item>` tag we have in our HTML will have an `onDelete` event that we can reference as if it were a regular default event like `click`.  

`$event` is an Angular keyword, and **we cannot use a different name**. This is used to pass the event parameters into our component's method. In this case it will be the quote id. (Why is the quote id avilable here? Because we passed it in when we emitted the `onDelete` event in the `quote-item` component.)

![](https://i.imgur.com/bKdrHFM.gif)

Yeah! We now have two nested components communicating well and coexisting in perfect harmony.  

Of course, we can add as many inputs and outputs as the situation calls for.  

## Lifecycle hooks (advanced)

We might have noticed that our CLI-generated components have a special interface implemented called `OnInit`. The only rule for this interface is that the component must have a method called `ngOnInit`.

**This is a special kind of method that the framework calls for us at a specific moment in the component lifecycle.**

Examples of **lifecycle** events would include:

- When a component is initialized
- When a component is mounted on the DOM
- When a component's properties change
- When a component is removed from the page

`ngOnInit` is one of many `lifecycle hooks` that fires at certain, predictable times.  Since we know when these `lifecycle hooks` will fire, we can use them to get information about the current state of the component, or to update the DOM, and we can choose the exact moment we want it to happen.  

Some of the most frequently used are:

|Name | description |
|---|---|
|ngOnChanges| Respond when Angular (re)sets data-bound input properties. |
| ngOnInit | Initialize the directive/component after Angular first displays the data-bound properties and sets the directive/component's input properties. |
| ngOnDestroy | Cleanup just before Angular destroys the directive/component. Used to detach event handlers to avoid memory leaks. |

You can find the full list in the [official documentation](https://angular.io/docs/ts/latest/guide/lifecycle-hooks.html#!#hooks-purpose-timing).

Let's take a look at how to actually use some of these custom bindings:

```typescript
// quote-list.component.ts
import { Component, OnInit, OnDestroy } from '@angular/core';
// ...

export class QuoteListComponent implements OnInit, OnDestroy {
  // ...

  ngOnInit() {
    console.log('ngOnInit: quote-list component');
  }

  ngOnDestroy() {
    console.log('ngOnDestroy: quote-list component');
  }
}

// quote-item.component.
import {
  Component,
  OnInit,
  OnDestroy,
  OnChanges,
  Input,
  Output,
  EventEmitter
} from '@angular/core';

// ...

export class QuoteItemComponent implements OnInit, OnDestroy, OnChanges {
  // ...

  ngOnInit() {
    console.log('ngOnInit: quote-item component');
  }

  ngOnDestroy() {
    console.log('ngOnDestroy: quote-item component');
  }

  ngOnChanges(change) {
    console.log('ngOnChanges: quote-item component', change);
  }
}
```

And voilà!

![](https://i.imgur.com/U0YIAM0.gif)

## Extra

### Decorators
Both the `@Input` and `@Output` decorators are imported from the `@angular/core` library. Also, all other new declarations we used in this lesson, such as `EventEmitter`, `OnInit`, `OnDestroy`, and `OnChanges`, are a part of the core.

### Third Party Code

As is the case with almost any programming language or framework, there are many 3rd party packages that we can use with Angular.  Some of these packages give us additional lifecycle hooks, events, inputs, outputs, pipes, etc. that we can use in our projects as if they were built into the core.  

You might wonder how we're supposed to know if some 3rd party code will be compatible with ours.  Well, the only way to figure this out is to put in the time and read the docs for that package.  

The inputs and outputs are part of the "public API" of a component.

For an example, check out [angular 2 maps component](https://angular-maps.com/docs/getting-started.html#setup-the-template). Note the `latitude` and `longitude` inputs.

## Summary

In this lesson we covered how to pass information between nested components.

We also used Input and Output decorators to make nested components communicate.

Finally we have learnt the basics about component lifecycle hooks and how to use them.

## Extra Resources
* [Advanced components interaction](https://angular.io/docs/ts/latest/cookbook/component-communication.html#!#sts=Pass%20data%20from%20parent%20to%20child%20with%20input%20binding) - The official Angular documentation
* [Components lifecyle hooks](https://angular.io/docs/ts/latest/guide/lifecycle-hooks.html) - The official Angular documentation
