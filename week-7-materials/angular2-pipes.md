![Ironhack Logo](https://i.imgur.com/1QgrNNw.png)

# Angular Pipes

## Learning Goals
After this lesson, you will be able to:

* Use Angular's built-in pipes to customize data being displayed in the view.
* Create your own custom pipe.

## What are pipes?

Pipes are functions that format the value of an expression to optimize it for display to the user. They can be used in view templates or classes. Angular comes with a collection of built-in pipes, but it is easy to define our own as well.

## Using pipes

### Syntax

We can use pipes in any interpolation expression we make in one of our templates.  All we need to do is add a pipe symbol `|` followed by the pipe's name:

```htmlmixed
{{ expression | pipe }}
```

### Example of Syntax

Here is a hypothetical example of what it might look like to use a pipe within an application.

```htmlmixed
{{ contact.name | uppercase }}
```

### Example of Usage

Let's get started with a new project so we can get a little experience with Angular's pipe feature.  

```
$ ng new angular-pipes
```

Now, let's create a new component where we can play with pipes.

From the terminal, run:

``` shell
$ ng g component built-in-pipes
```

We'll try the `uppercase` pipe first, which will transform any string input to Uppercase.

Open the `built-in-pipes.component.ts`, and edit the component as follows:

```typescript
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-built-in-pipes',
  template: `
    <p> My name is {{ 'Jack' | uppercase }} </p>
  `,
  styles: []
})
export class BuiltInPipesComponent implements OnInit {

  constructor() { }

  ngOnInit() { }

}
```

We just added an interpolation expression with a pipe applied.

Remember, you need to add this component's selector tag into the `app.component` template in order to see it. Delete the contents of the default app.component.html file, then add the following:


```
  <h1>
    app works!
  </h1>

  <app-built-in-pipes></app-built-in-pipes>

```



The result should look like:

![](https://i.imgur.com/wQV0YjK.png)

:::warning
:bulb: In the previous code you may have noticed a new keyword `implements OnInit`, and the `ngOnInit()` function. You don't really need to declare them explicitly, but they will be added by default by the Angular-CLI and we will be learning more about them in the coming lessons.

You can read more about the [ngOnInit lifecycle hook](https://angular.io/guide/lifecycle-hooks) in the documentation.
:::
### Built-in pipes

Angular comes with many built-in pipes that provide enough functionality that they can cover most use cases that we will come across. Occasionally, though, we will need to build our own pipe to handle special cases.  

We've already seen the `UpperCasePipe`, here's a list of some other important pipes:

|Name|Description|
|--------|---------|
|DatePipe|Formats a date object with the specified format|
|UpperCasePipe|Transforms the input to uppercase|
|LowerCasePipe|Transforms the input to lowercase|
|CurrencyPipe|Adds the specified currency symbol to the input|
|DecimalPipe|Formats a decimal with the specified format|
|PercentPipe|Converts the number to a percentage and adds a `%` |
|JSONPipe| Converts a JSON object into a string (used for debugging) |
|SlicePipe| Works like `Array.prototype.slice()`|
|AsyncPipe| Takes an input as an asynchronous object and waits for its value |

To read about by these pipes in more detail, check out the [official documentation](https://angular.io/guide/pipes).

### Parameters

Usually our pipes are customizable and need one or more configuration parameter to work properly.

To pass parameters to a pipe, we simply list them, after the name of the pipe, separated by a colon `:`.
We can pass as many parameters as we want, as long as the pipe supports them.

```htmlmixed
{{ expression | pipe : argument1 : argument2: ... }}
```

One example of a configurable pipe is the `DatePipe`. Let's say we want to display a date in the format `25/03/2017`, we can use the `DatePipe` and specify the format with a parameter. Reuse the built-in-pipes component for this example.


```typescript
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-built-in-pipes',
  template: `
      <p> Date: {{ today }} </p>
      <p> Date: {{ today | date:'dd/MM/yyyy' }} </p>
  `,
  styles: []
})
export class BuiltInPipesComponent implements OnInit {

  today = new Date();

  constructor() { }

  ngOnInit() {}

}
```

We created a `today` class property and set it equal to `new Date()`, and then we added two interpolation expressions that show the date with and without pipe transformation.

This is the result:

![](https://i.imgur.com/PeJx9Di.png)

You can see that the piped version follows our format, and it will be updated with every change that happens to the `today` property.

Unfortunately, as of now, we don't have any way to change our `today` property from the view.

How might we go about implementing this functionality?
Maybe an `increment` method in our class that increments the date by one day? Perhaps.

Let's take a look at how we might accomplish that:

```typescript
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-built-in-pipes',
  template: `
      <p> Date: {{ today }} </p>
      <p> Date: {{ today | date:'dd/MM/yyyy' }} </p>

      <button (click)="increment()"> Add one day! </button>
  `,
  styles: []
})
export class BuiltInPipesComponent implements OnInit {
                                     
  today = new Date();                 
                                      
  constructor() { }                   
                                      
  ngOnInit() {}                       
                                      
  increment() {                       
    // copy the date                  
    const changedDate = new Date(this.today);

    // increase the day of the month
    changedDate.setDate(this.today.getDate() + 1);

    // update our date
    this.today = changedDate;
  }
}
```
![](https://i.imgur.com/dbnBYXp.gif)

When we click the button, the value of the `today` class property is updated by Angular data binding, and the pipe gets re-applied to the resulting value.

:::info
The DatePipe supports all the formats defined by the W3C standard [ISO 8601](https://www.w3.org/TR/1998/NOTE-datetime-19980827)
:::

### Chaining pipes

We know that JavaScript and TypeScript give us the freedom to run a chain of methods on an object.  For example, if we are given a number, and we want to see if the first digit of the number is 1, we could do this:

```typescript
const theNumber = 28
return theNumber.toString().startsWith("1")
```


Similarly, we can chain pipes as well.  To chain multiple pipes, we just need to append a pipe symbol `|` followed by the name of the pipe.  

This is the syntax:

```htmlmixed
{{ expression | filter1 | filter2 }}
```

Keep in mind that the order _does_ matter. If we apply both the `uppercase` and `lowercase` pipe, the result will depend on how we ordered them.

The first pipe listed is always the first applied.

## Creating pipes

We said before that a pipe is a simple function that takes in data as an input and transforms it to a desired output. Let's create our own pipe that can take a string and return its capitalized version.
To create pipe, simply type: 
```
$ ng g pipe <path/to/pipe-name> 
```
So if you'll have folder "pipes" (which is highly recommended), you should type:
```
$ ng g pipe pipes/capitalize
```

As we can see, there is already built-in decorator `@Pipe` which passes, as metadata, the pipe name that we are going to use in our templates.

```typescript
import { Pipe } from '@angular/core';

@Pipe({
    name: 'capitalize'
})
```

:::info
When choosing a name for a new pipe, make sure to prefix it with an app prefix or use a conflict-proof name.
:::

Also, there is `PipeTransform`, built-in interface provided by Angular, and it's imported and implemented already by default.

```typescript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
    name: 'capitalize'
})
export class CapitalizePipe implements PipeTransform {

}
```

Besides this, the `PipeTransform` interface requires a method called `transform` that accepts an input value followed by optional parameters and returns the transformed value.

So we see something like this:
```typescript
export class CapitalizePipe implements PipeTransform {
  transform(value: any, args?: any): any {
    return null;
  }
} 
```

And we'll define our `transform` like this:

```typescript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
    name: 'capitalize'
})
export class CapitalizePipe implements PipeTransform {
    transform(input: string, args: any[]): any {
        input = input.toLowerCase();
        return input.substring(0, 1).toUpperCase() + input.substring(1);
    }
}
```

This is one solution, but you can find other ways to capitalize a string. Remember that pipes are applied on-the-fly in our templates and therefore they get executed every time the "piped" value changes. 

This means that performance could be a factor here, and trying to perform this pipe on an empty value would return an error.  Let's tighten up our code a bit.  

Now the final version is:

```typescript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
    name: 'capitalize'
})
export class CapitalizePipe implements PipeTransform {
    transform(input: string, args: any[]): any {
        if (!input) {
            return '';
        }

        input = input.toLowerCase();
        return input.substring(0, 1).toUpperCase() + input.substring(1);
    }
}
```

Let's create a new component to use our new pipe by running:
```shell
$ ng g component custom-pipes
```

Then add an input field bound to a paragraph through `ngModel`.

Here we can type in our name and it shows up in the DOM in real time as we type. Since it is a name, we should capitalie it.

Let's make use of our `capitalize` pipe:

```typescript
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'custom-pipes',
  template: `
        <input [(ngModel)]="name">
        <p> Hello {{ name | capitalize }}! </p>
    `
})
export class CustomPipesComponent {}
```

:::info
Remember to add the `<custom-pipes></custom-pipes>` selector into the `app.component` template, to enable our component. Also, don't forget to import FormsModule and add it to the imports array in the `app.module.ts` file.
:::

If we run the app with `ng serve`, we notice that it says "loading..." an error is shown in the console:

![](https://i.imgur.com/yL4pyXn.png)

The error is pretty clear, our `capitalize` pipe is not recognized.

This happens because Angular recognizes only declarations listed in the current NgModule. Our `AppModule` doesn't have any knowledge about our new pipe.

So let's add a new element `CapitalizePipe` to the declarations array of our `AppModule`:

```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { FormsModule } from '@angular/forms';        // <----- Make sure you have FormsModule here to enable [(ngModel)]

import { AppComponent } from './app.component';
import { CustomPipesComponent } from './custom-pipes/custom-pipes.component;

// Other imports here

import { CapitalizePipe } from './pipes/capitalize.pipe';

@NgModule({
  declarations: [
    AppComponent,
    CustomPipesComponent,
    // ...
    CapitalizePipe                // <-- Pipe is imported here
  ],
  imports: [
    BrowserModule,
    FormsModule,
    HttpModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

The app should now work normally and the pipe should be active - when we type in a name, it should be capitalized, like so:

![](https://i.imgur.com/xtpvD0t.png)

:::success
:thumbsup: Although we created a pipe manually during this lesson in an effort to point out the necessary steps to allow the files within our Angular projects to communicate effectively, in a real-world scenario, it is probably more convenient to run the cli command to generate a pipe `ng g pipe <path/to/pipe-name>`. This command takes care of creating the right interface and adding the new pipe to the necessary places in the app module file for you.
:::

### Pure vs Impure (advanced)

Let's try to build a `filter` pipe, that takes an array of elements and two query parameters as input, and returns a filtered list.

Let's use the Angular CLI command this time:

```
$ ng g pipe pipes/filter
```

And we can implement our pipe into the `pipes/filter.pipe.ts` file.

```typescript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'filter'
})
export class FilterPipe implements PipeTransform {

  transform(items: any[], field: string, value: string): any[] {
    if (!items) {
      return [];
    }

    if (!value) {
      return items;
    }

    const myPattern = new RegExp(value, 'i');
    return items.filter(it => it[field].match(myPattern));
  }
}

```

We don't want to reinvent the wheel so we simply use the native [filter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) method of JavaScript, after checking our input values.

To see it in action we want to build a simple form. The form will allow us to build an array of stuff, adding items and filtering by name.

We can use the `custom-pipes.component.ts` we created in the previous section, and add the logic we need:

```typescript=
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'custom-pipes',
  templateUrl: `custom-pipes.component.html`
})

export class CustomPipesComponent implements OnInit {

  today = new Date();
  stuff: Array<Object> = [];
  pattern: string;

  constructor() {}

  ngOnInit() {}

  addItem(item) {
    this.stuff.push({name: item});
  }
}
```

Because this is a longer HTML template, we should use the provided HTML file rather than a string. Add the following to `custom-pipes.component.html`:

```htmlmixed
  <div>
    <label for="thing_name"> Thing Name </label>
    <input placeholder="new thing" type="text" #thingName>
    <button (click)="addItem(thingName.value); thingName.value=''">
        add
    </button>
  </div>

  <div>
    <label for="things_name"> Search thing: </label>
    <input id="things_name" placeholder="thing name"
            type="text" [(ngModel)]="pattern">
  <div>

  <h4> Stuff list </h4>
  <ul>
    <li *ngFor="let thing of stuff | filter:'name':pattern; let i = index">
      {{ i+1 }}) {{ thing.name }}
    </li>
  </ul>
```

![](https://i.imgur.com/i1a4oD7.gif)

Everything is working perfectly, right?

Well, actually, if you pay close attention, we have a pretty major bug. If we add items while we are already searching for something, our items don't show up in the list until after we exit the search feature.

This is because angular executes pipes only when it detects a pure change to the input value.

:::info
:question: A pure change is either a change to a primitive input value (String, Number, Boolean, Symbol) or a changed object reference/reassignment (Date, Array, Function, Object).
:::

In our case, we are changing a non-primitive item: we are pushing a new value into an array and Angular doesn't catch that change by default.

We can change this behavior by modifying our pipe such that Angular re-evaluates it every time any change happens. To do this, we just need to specify our pipe as "impure".

:::info
Impure pipes will be called a lot, as often as every keystroke or mouse-move, so we need to keep the pipe logic code as efficient as possible
:::

Let's add, through the `pure` metadata, this feature to our pipe:

```typescript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'filter',
  pure: false     // <!-- We mark the pipe as "impure"
})
export class FilterPipe implements PipeTransform {

  transform(items: any[], field: string, value: string): any[] {
    if (!items) {
      return [];
    }

    if (!value) {
      return items;
    }

    const myPattern = new RegExp(value, 'i');
    return items.filter(it => it[field].match(myPattern));
  }
}
```

That should take care of our bug, and illustrate the difference between pure and impure pipes.

## Summary

In this lesson we learned what pipes are and how to use them to customize data visualization. We also created our own pipe and used it in an application.

## Extra Resources
* [Angular Documentation](https://angular.io/docs/ts/latest/guide/pipes.html) - The official Angular documentation for pipes.
