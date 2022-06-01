![Ironhack Logo](https://i.imgur.com/1QgrNNw.png)

# Angular Databinding

## Learning Goals

After this lesson, you will be able to:

* Use interpolation to show a component's properties and make use of property binding.
* Respond to user interaction with event binding.
* Define and use template variables.

## Binding Syntax

Thanks to data binding, we can control what users see by using class property values in our Angular component classes.  

We simply declare bindings between binding sources and target HTML elements and let the framework do the work.

**In other words**, we bind something in the front-end logic, to something in the HTML.  For example, we might bind a property's value to an `<input>` tag to save user input, or we might bind a function to a button so the function runs when you click the button.  

These are all tasks you can accomplish with regular Javascript, as well as many other frameworks, but Angular has its own way of doing this.

Angular provides 3 kinds of data binding:

- Interpolation
- Property binding
- Event binding

First we'll take a high-level view of Angular data binding and its syntax.

|Binding|Syntax|Use|
|-------|------|---|
|Interpolation| `{{ myProperty }}` | Bind to a class property to display its value
|Property| `<img [src]="myUrl" />` | Bind a class property to an element property
|Event| `<button (click)="sayHello()"></button>` | Bind a class method to an element target event

Let's dive in a little deeper.

## Property Binding

Property binding allows you to bind values to properties of an element to modify its behavior or appearance.

We specify one-way bindings to DOM properties using square brackets and template expressions.

We're going to build a `PropertyBinding` component to play with this awesome feature. We'll go deeper into components in the next lessons, so don't worry if this code doesn't look familiar to you yet.

**Let's open the same project we were working on in the previous lesson** and add some property binding in a new component.

From the CLI run the following to generate a new component:
```
$ ng g component my-property-binding
```

The most common thing we use property for is to bind an element's property value to a component's property value. A common example is binding the `src` property of an image element to a component’s property.

To bind to a DOM event, surround the DOM property name in square brackets and assign a component property name as its value.

We can now open our `myPropertyBinding` component and configure it like this:

```typescript
@Component({
  selector: 'app-my-property-binding',
  template: `<img [src]="myImageSource" />`
})
export class MyPropertyBindingComponent {
    myImageSource: string = 'http://placekitten.com/g/300/300';
}
```

Now we need to add the `app-my-property-binding` selector to our `app.component` template, to make sure this new component is loaded. The app component template should look something like:

```htmlmixed
<div style="text-align:center">
  <h1>
    Welcome to {{ title }}!
  </h1>
  <h2>
    {{ content }}
  </h2>
  <app-my-nested></app-my-nested>
  <app-my-property-binding></app-my-property-binding>
  ...
```

Once we save and refresh the page, we should now see our image loaded:

![](https://i.imgur.com/XQ7Lz2e.png?1)


Property binding works with all the native DOM properties, such as `class`, `disabled`, `href`, or `textContent` (see the [full list](https://developer.mozilla.org/en-US/docs/Web/API/Element)). We can use all of these bindable properties **to keep our UI in sync with our data**.

We will play with some other examples once we introduce event binding.

## Event Binding

Angular event bindings let us respond to any DOM event. Most DOM events are triggered by the user. Binding to these events provides a way to get input from the user.

To bind to a DOM event, surround the DOM event name in parentheses and assign a quoted template statement to it.

The following example shows an event binding that implements a click handler:

```html
<button (click)="buttonClickMethod()"> Click me! </button>
```

The `(click)` to the left of the equals sign identifies the button's `click` event as the target of the binding.

The text in quotes to the right of the equals sign is the template statement, which responds to the click event by calling the component's `buttonClickMethod` method.

Pretty easy, right?

Let's see an example.

### Passing Data

An event binding can carry information about the event, including data values, through an event object called `$event`.

Let's take a look at how we can pass information with Angular's event binding feature.

Let's say we want to capture a password inserted in an input box and print it to Chrome's developer console.  We can use Angular's event binding to accomplish this.

First let's create a new component in our ongoing project, just as we did in the previous section (just to keep examples separated and more readable).

So from the CLI run:
```
$ ng g component my-event-binding
```

**Remember, we need to add the selector to our `app.component` template so that our new component will show up as part of our `AppComponent` HTML.**

To be notified after each letter the user types, we can bind a function to the (keyup) event, like this:

```htmlmixed
<input (keyup)="recordAllTheKeyStrokes($event)">
```

Now we need a function to call whenever our event is raised, we referred to it as `recordAllTheKeyStrokes` in our HTML, so we'll need a corresponding function in our Typescript class, which receives the `$event` object as a parameter, extracts the relevant information, and prints it to the console.

```typescript
recordAllTheKeyStrokes(event) {
        console.log(`Key inserted: ${event.key}`);
        //console.log(`Input value: ${event.currentTarget.value}`);
    }
```

We're ready, put everything together and the full code should look something like this:

```typescript
import { Component, OnInit } from '@angular/core';

@Component({
    selector: 'app-my-event-binding',
    template: `
        <label>Input: </label>
        <input (keyup)="recordAllTheKeyStrokes($event)">
    `
})
export class MyEventBindingComponent {
    recordAllTheKeyStrokes(event) {
        console.log(`Key inserted: ${event.key}`);
        //console.log(`Input value: ${event.currentTarget.value}`);
    }
}

```

Now as we type letters into the input box, we should see them appear in the console in real time, like this:

![](https://i.imgur.com/sLIwc75.png?1)

The `$event` parameter is a [DOM event object](https://developer.mozilla.org/en-US/docs/Web/Events) that holds all the event information such as `currentTarget` and the keycode associated with the event (if the event involved a key being pressed on the keyboard.)


### Combining Event and Property Binding

Let's look at another example to see how event binding and property binding can work together.

Let's create another component.  Remember to add it into the `app.component` template after you do.

```
$ ng g component my-mixed-binding
```
Let's say we have an input field and we want a way to enable/disable it with a button.

We'll need to bind to the property `disabled` of the input field:

```typescript
import { Component, OnInit } from '@angular/core';

@Component({
    selector: 'app-my-mixed-binding',
    template: `
        <input type="text" [disabled]="isInputDisabled" />
        <button (click)="toggleInput()"> Toggle input </button>
    `
})
export class MyMixedBindingComponent {
    isInputDisabled: boolean = false;

    toggleInput() {
        this.isInputDisabled = !this.isInputDisabled;
    }
}
```

## Two-way data binding
The “Wow” feature in Angular is probably its two-way data binding system. Changes in the application state will be automatically reflected into the view and vice-versa.

We have a directive that implements this feature: `ngModel`

Let's create another component, as usual, called MyTwoWayBinding, like this:

```shell
$ ng g component my-two-way-binding
```

Remember to add the selector to the `app.component` template.

And finally we can add a two-way expression using `ngModel`:

```typescript
@Component({
    selector: 'app-my-two-way-binding',
    template: `
        <input [(ngModel)]="username">
        <p> Hello {{username}}! </p>
    `
})
export class MyTwoWayBindingComponent {
    username: String = "";      // <-----this line is optional
}
```

**Note**: Right now, this is broken. The `ngModel` directive is part of the `FormsModule` module. We haven't imported `FormsModule` in `app.module.ts`, and we should do that now:

```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { FormsModule } from '@angular/forms';
...
  imports: [
    BrowserModule,
    FormsModule // <-- This is what adds ngModel
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

This looks pretty magical, right?

When typing into the input, the input’s value is written into the username model and then reflected back into the view, resulting in a nice greeting.

If we type the word `World` we should see something like:

![](https://i.imgur.com/Ks8n8x2.png?1)


## Template Variables
Another way to get user data is using Angular template variables.

A template variable is just a reference of the DOM Object the variable is attached on.

To declare a template variable we just need to add an identifier prefixed with a hash character (#) on any HTML Object.

Let's see an example:
```typescript
@Component({
    template: `
        <input #myInput (keyup)="0">
        <p> {{ myInput.value }} </p>
    `
})
```
We create a `myInput` template variable and print its value using interpolation.

Here we bind to the `keyup` event (the shortest template statement possible) because Angular updates the UI only in response to asynchronous events, such as keystrokes.
So even though this event binding doesn't do anything useful, it tells the framework to update the UI.

Template variables, as the name suggests, only live in our templates (HTML) so we don't have access to their value in our component. However, we can use them as a parameter.

To see this we can update the event binding example:

```typescript
@Component({
    template: `
        <input #myInput (keyup)="onKey(myInput.value)">
    `
})
export class MyEventBindingComponent() {
    onKey(value) {
        console.log(`Input value: ${value}`);
    }
}
```

and we should see the value of the field:

![](https://i.imgur.com/gkaYOvF.png?1)


## Summary

In this lesson we have learned how to bind to a DOM property using property binding. We have also seen how to respond to user input with event binding, and finally, how to define template variables.  

These are the primary means by which we will move data back and forth between the UI and the front-end logic with the Angular framework.  Keep the code you generated during this lesson and refer back to it later.
