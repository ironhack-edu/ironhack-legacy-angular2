![Ironhack Logo](https://i.imgur.com/1QgrNNw.png)

# Angular Directives

## Learning Goals

After this lesson, you will be able to:

* Use Angular's built-in directives.
* Explain Angular directives to a novice.

## What Are Directives?

Angular lets us extend HTML with new attributes called **directives**. We use directives to augment the behavior of other (built-in or custom) HTML elements.

A directive is just a component without a template.

![](https://i.imgur.com/wBt0nOo.png)

Angular provides us with some built-in directives that provide functionality to our applications, such as `ngFor` and `ngIf`, which we've already seen.

In this lesson we're going to play with some more built-in directives.

### NgIf

The `ngIf` directive is used when you want to display or hide an element based on a condition. Visibility is determined by the result of an expression that you pass into the directive.

If the result of the expression returns a false value, the element will be removed from the DOM.

Let's take a look at an example:

```htmlmixed
<div *ngIf="true"> Always displayed </div>

<div *ngIf="false"> Never displayed </div>

<div *ngIf="answer === 42">
    Displayed only if the "answer" property is equal to 42
</div>
```

Here we see that `ngIf` evaluates both static values such as `true` or `false` and expressions like `answer === 42`.

### NgFor

In the "helloworld" lesson, we saw how `ngFor` allows us to iterate over arrays and generate HTML code with the data.

Let's use the `ngFor` directive to create a table with a list of animals.

In the angular-helloworld project, create a new component:
- from the terminal run:`ng g component ng-for-example`

```ts
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'ng-for-example',
  template: `
    <p> ng-for-example Works! </p>
  `,
  styles: []
})
export class NgForExampleComponent {
  animals: Array<Object> = [{
      id: 1,
      category: 'mammal',
      name: 'Dog'
  },{
      id: 2,
      category: 'oviparous',
      name: 'Duck'
  },{
      id: 3,
      category: 'mammal',
      name: 'Elephant'
  },{
      id: 4,
      category: 'reptile',
      name: 'Snake'
  }];
}
```

#### Useful Variables

The `NgFor` directive provides us with some useful variables within its scope:
| Name | Value
|---------|--------
| index | Current iteration index. Starts from 0.
| first | Boolean value that indicates if the current iteration is the first
| last | Boolean value that indicates if the current iteration is the last
| even | Boolean value that indicates if the iteration index is even
| odd | Boolean value that indicates if the iteration index is odd

To take advantage of these values, we need to save them to local variables. Add the following to the template string in the `ng-for-example.component.ts` file:

:::info

```htmlmixed
<ul>
    <li *ngFor="let animal of animals, let i = index">
        {{ i }} - {{ animal.name }}
    </li>
</ul>
```

And change the app.component.html file so that it looks like this:
```htmlmixed
<h1>Hello Angular 2!</h1>
<ng-for-example></ng-for-example>
```
result:

![](https://i.imgur.com/Jz9SYzU.png)
:::

You can initialize more than one local variable at a time:

:::info
```htmlmixed
<ul>
    <li *ngFor="let animal of animals, let i = index, let x = odd">
        {{ i }} - {{ x }} - {{animal.name}}
    </li>
</ul>
```
result:

![](https://i.imgur.com/g7M2Lqu.jpg)
:::

#### Tips (Advanced)

We might initially think that every time our array changes, the `ngFor` directive tears down the old list, discards those DOM elements, and re-builds a new list with new DOM elements. But actually, Angular **tries to avoid doing that** as much as possible.

Internally, Angular **keeps track of each object in the array** and which DOM element belongs to each one so Angular knows which DOM elements to add, and which to remove.

If our list is really long or we are constantly recreating the array, this could negatively impact the performance of our app. In order to avoid this, we can **help Angular keep track of each object** in the array by giving the objects a unique property to identify them (like a database ID).

To help Angular in that way, we can provide `ngFor` a function that tells it how it should keep track of array items. In this case, we will tell it to use the' `id` property. Here is a function implementing this:


```ts
animalTrackerFunction(index: number, animal: any) {
    return animal.id;
}
```

Now we need to set the ngFor `trackBy` directive to that tracking function.
```htmlmixed
<ul>
    <li *ngFor="let animal of animals; trackBy:animalTrackerFunction">
        <span>  {{ animal.name }} </span>
        <small> {{ animal.category }} </small>
    </li>
</ul>
```

:::info
In this example, `trackBy` is a keyword that informs the `ngFor` directive how to track the objects in the array that is being iterated through.  `trackBy` wants to be given a function as an input, in this case, we have created a function called `animalTrackerFunction` and we supply this function to the `trackBy` directive.
:::

### NgSwitch

The `NgSwitch` directive mimics the logic of the [JavaScript switch statement](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Statements/switch), it evaluates a series of expressions and executes the statement associated with the one that returns a truthy value.  If no values return a truthy value, the (required) default block of code is executed. 

`NgSwitch` allows us to achieve this functionality from our templates: it works much like `ngIf` but is better for use cases with many conditions.

For example, sometimes you need to show different elements depending on a given condition. You could write some code like:

```htmlmixed
<div class="container">
  <div *ngIf="value === 1"> Value is A </div>
  <div *ngIf="value === 2"> Value is B </div>
  <div *ngIf="value !== 1 && value !== 2"> Value is something else </div>
</div>
```

This code works fine, but every time we want to add a new condition we need to add a new line and edit the last `ngIf` block. Angular gives us a cleaner solution to define our logic, the `ngSwitch` statement. The code above becomes:

```htmlmixed
<div class="container" [ngSwitch]="value">
  <div *ngSwitchCase="1"> Value is A </div>
  <div *ngSwitchCase="2"> Value is B </div>
  <div *ngSwitchDefault> Value is something else </div>
</div>
```

The `ngSwitchCase` directive informs the parent `NgSwitch` of which view to display when the expression is evaluated. When no matching expression is found on a ngSwitchCase view, the `ngSwitchDefault` view is displayed.

### NgClass

The `NgClass` directive allows us to dynamically apply CSS classes to a given HTML element.

The directive takes either a string, an object or an array of strings as a parameter. Depending on which it receives, it will perform different actions:

|Parameter type| Action |
|--------------|--------|
| String | the CSS classes listed in the string (space delimited) are added |
| Object | keys are CSS classes that get added when the expression given in the value returns true |
| Array | the CSS classes declared as Array elements are added |


So, if we have some CSS classes to apply stored in our Component's class properties, we could do something like:

```htmlmixed
<div [ngClass]="class1 class2">
</div>

<div [ngClass]="['class1','class2']">
</div>

<div [ngClass]="myArray">
</div>
```

In this case, `class1` and `class2` are class properties that we have defined in our Typescript Class, and `myArray` is an Array of strings that correspond with CSS classes (like 'red', 'shaded', 'centered', etc.)


The most common case for `NgClass` is to conditionally apply classes, like:

```htmlmixed
<!-- CSS style -->
<style>
    .bordered {
        border: 1px solid blue;
        background-color: gray;
    }
</style>

<!-- HTML markup -->
<div [ngClass]="{ bordered: false }">
    This is never bordered
</div>

<div [ngClass]="{ bordered: true }">
    This is always bordered
</div>

<div [ngClass]="{ transparent: isTransparent }">
    This is trasparent if "isTransparent" class property evaluates to "true"
</div>
```

Here we created a simple class `bordered` which sets the border style and the background color, and applied this class using the `NgClass` directive.

In this case the directive takes an object parameter where:

- The key represents the CSS class
- The value expression, that should evaluate to a boolean, tells Angular whether or not to apply the class


### NgStyle

The `NgStyle` directive allows us to set CSS properties on an element using Angular expressions.

Say we want to dynamically set the background color of one `div` element, we already know that we could do something like this:

```htmlmixed
<div [style.background-color]="'blue'"]>
    Blue background here
</div>
```

`NgStyle` allows us to do the same thing in a more clear way:

```htmlmixed
<div [ngStyle]="{ 'background-color': 'blue' }">
    Blue background here
</div>
```

Since we are passing an object, we can specify multiple values, like this:

```htmlmixed
<div [ngStyle]="{ 'background-color': 'blue', 'border': 2px solid orange }">
    Blue background here
</div>
```

## Summary

In this lesson we have learned how to use ngFor and ngIf directives in our applications. We have also seen other directives like ngSwitch, ngClass, and ngStyle.

These directives are core to Angular and can help you craft interactive interfaces and experiences for users as well as enable you to more easily handle data within your components on the front-end.
