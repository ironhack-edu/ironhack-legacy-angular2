![Ironhack Logo](https://i.imgur.com/1QgrNNw.png)

# Forms

## Learning Goals

After this lesson, you will:

* Understand how to use forms within the Angular framework.
* Know how to submit a form with Angular.

## Introduction

Forms are an integral part of any web application. Although there are many ways users can interact with web pages, forms are the primary means by which users submit information to a web application.  

On the surface, forms seem pretty straightforward: you create input tags, then the user fills them out and submits the form. How hard could it be?

From the developer's perspective, forms can sometimes end up being very challenging. Here are a few reasons why:

- Form inputs are meant to modify data, both on the page and on the server.
- Input values need to be validated to guarantee security and consistency.
- Dependent fields can have complex logic. For instance, if you wanted to create an input field that only becomes active after another particular input is filled out, you will need to do a little extra work.
- The UI needs to guide the user through the process of filling out the form, clearly alerting the user to any errors they have made in the process.  

In this lesson we’re going to learn how to build forms with Angular, step by step.

### Our first Angular form

For this lesson, we want to create a sign up form that requires a username and password.

In order to do this, let's create a new project using the CLI.  

```
$ ng new angular-forms
```

After that, make sure to import the `FormsModule` as demostrated above. 

Once that's finished, let's create a new component for our form. 

```
$ ng g component my-signup-form
```

## Forms module

Angular helps us build forms through the `FormsModule` in the `@angular/forms` library. This module exposes all the directives and services we need to create dynamic, beautiful forms by providing solutions for many of the problems we discussed in the introduction.

### Setup

To use Angular `FormsModule` we need to import it in our `app.module.ts` file:


```typescript
...
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { FormsModule }   from '@angular/forms'; //Import declaration

import { AppComponent } from './app.component';


@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    FormsModule //Import FormsModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }

```

Using `FormsModule` in Angular applications isn't mandatory. If our application doesn't need forms, we can skip importing or using the library.

Now let's open our `my-signup-form.component.html` files and build our first Angular form (don't forget to add the component's selector to `app.component.html`):

```htmlmixed
<h1>Welcome to Angular</h1>
<form>
  <p> Username
    <input type="text" name="username" required />
  </p>
  <p> Password
    <input type="password" name="password" required />
  </p>
  <button type="submit"> signup </button>
</form>
```

We added an HTML `form` element and some inputs - a pretty basic form. No validation, no logic at all.

To *Angularize* our form we need to add some directives, let's take a look:

```htmlmixed
<h1>Welcome to Angular</h1>
<form (ngSubmit)="submitForm(myForm)" #myForm="ngForm">
  <p> Username
    <input type="text" name="username" [(ngModel)]="username" required />
  </p>
  <p> Password
    <input type="password" name="password" [(ngModel)]="password" required />
  </p>
  <button type="submit"> signup </button>
</form>
```

```typescript
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-my-signup-form',
  templateUrl: './my-signup-form.component.html',
  styleUrls: ['./my-signup-form.component.css']
})
export class MySignupFormComponent implements OnInit {

  constructor() { }

  ngOnInit() {
  }

  submitForm(myForm) {
    console.log(myForm);
  }
}
```

Let's review what we've just done:

-  We're using `ngForm`, which is a directive that keeps track of the state of the form. It tells us if the form is valid, dirty, etc.  
- We have also created a local template variable called `myForm` and we have exported our `ngForm` directive into this local template variable so that we can reference this variable and have access to all the information that `ngForm` keeps track of. 
- We have applied `ngModel` to every input element to enable two-way data-binding to our component class.
- We have an `(ngSubmit)` event binding to catch the form submit event and run a method called `submitForm`, passing in our `myForm` variable as a parameter.  
- We added a button with a type equal to "submit". 
- We added a name attribute for each input element.

The result should look something like this:

![](https://i.imgur.com/QGRnCum.png)

The `submitForm` method, which runs when we submit the form, simply logs the parameter being passed into it.  (It can be a useful strategy to create methods like this at first, just to make sure they are firing at the correct time, and then building out their functionality afterward.) 

In this case, the parameter we are passing in is the entire HTML form, but we are using the `ngForm` directive to create a big, complex object filled with lots of information about the form's current state.  That object is what is actually being passed into the function and it is what we should see when we check the console after submitting our form.  

It should look something like this:


![](https://i.imgur.com/BgpZQt2.png)


This `ngForm` object contains a lot of useful information about our form, some of the important keys in this object you should be familiar with include:

- **controls** holds information about the current state of the form
- **errors** holds information about any invalid elements
- **value** holds the values of all the inputs
- **submitted** and **valid** are boolean values indicating whether the form has been submitted and whether it is valid (all its inputs need to be valid for the form to be valid)

:::info
:bulb: We don't need to explicitly use the `ngForm` directive because Angular creates and attaches it to every `<form>` tag automatically when we import the `FormsModule`.
:::

### Input validation

//Why doesn't this work?
One of the biggest challenges when working with forms is handling input validation.
Many languages and frameworks have different ways of validating inputs, but most follow the same general principle:  
We define a set of rules that each input's value must conform to. `HTML5` comes with some built-in validation attributes, such as:

- required
- minlength / maxlength
- pattern

Let's use HTML's default features to require a password and to require the username to be at least 6 characters long.

:::warning
As of Angular 4, Angular automatically adds the 'novalidate' attribute to all forms, preventing this behavior. If you want to use HTML5 form validation attributes, you need to add the ngNativeValidate directive to form tags.
:::

```htmlmixed
<form (ngSubmit)="submitForm(myForm)" #myForm="ngForm" ngNativeValidate>
  <p> Username
    <input type="text" name="username" [(ngModel)]="username" minlength="6" />
  </p>
  <p> Password
    <input type="password" name="password" [(ngModel)]="password" required />
  </p>
  <button type="submit"> signup </button>
</form>
```

Now we have basic form validation. If the user submits the form without a long enough username or without a password, the browser's built-in validation will kick in and prevent the user from submitting data:

![](https://i.imgur.com/VEDax5r.png)

These are very useful features that you should use routinely.  However, that's about as much as we can do with just HTML. We can't show helpful messages for errors or change colors of elements to direct the user's attention.

This is where Angular comes in. First, we're going to get rid of all the HTML validation we had because we want to allow Angular to take care of everything. We will need to add some extra markup to allow Angular to do its job.  


```htmlmixed
<form (ngSubmit)="submitForm(myForm)" #myForm="ngForm">
  <p> Username
    <input type="text" name="username" [(ngModel)]="username"
          #myUsername="ngModel" minlength="6" />
  </p>
  <div *ngIf="myUsername.errors && (myUsername.dirty || myUsername.touched)">
    <p class="error" [hidden]="!myUsername.errors.minlength">
      Username must be at least 6 characters long.
    </p>
  </div>
  <p> Password
    <input type="password" name="password" [(ngModel)]="password"
          #myPassword="ngModel" required />
  </p>
  <div *ngIf="myPassword.errors && (myPassword.dirty || myPassword.touched)">
    <p class="error" [hidden]="!myPassword.errors.required">
      Password is required
    </p>
  </div>
  <button type="submit"> signup </button>
</form>
```

You might notice the code is much more complicated now. Let's break it down and everything will be clear:

- Angular automatically adds a `novalidate` attribute on the form element, this turns off the browser's built-in validation and lets us handle it.
- We added local template reference variables to the two input fields: `myUsername` and `myPassword`. We assigned these names to the `ngModel` directive, which holds onto the value entered in the input field.  This way, we have two variables we can use to easily reference the value of each input.  
- We have added a couple of divs and attached the `ngIf` directive to show/hide them depending on whether or not they are currently filling out the form correctly.  

:::warning
:heavy_exclamation_mark: Remember that client-side validation does not guarantee any security for your applications. All the validation logic must be performed also on the server, the client validation is just a guide for the user.
:::

### Angular CSS classes

In the data-binding lesson we learned how `ngModel` lets us define a powerful two-way binding in our templates. It also updates the element it's attached to with some special Angular CSS classes that reflect the current state of the form. We can leverage these class names to change the appearance of the form and guide the user through it.

| State | Class if true | Class if false |
|-------|---------------|----------------|
| Element has been visited | `ng-touched` | `ng-untouched` |
| Element's value has changed | `ng-dirty` | `ng-pristine` |
| Element's value is valid | `ng-valid` | `ng-invalid` |

This feature is really useful when it comes to guiding the user through a form. We can easily grab the user's attention and notify them if they have missed a field or have filled one out incorrectly.  

Let's style some of these classes and see what happen. Add the following CSS to the `my-signup-form.component.css` file. It will define validation style while Angular will update elements' classes based on the form state.

```css
input:focus {
  outline: none;
}

.ng-touched {
  border-color: blue;
}

.ng-touched.ng-invalid {
  border-color: red;
}

.ng-touched.ng-valid {
  border-color: green;
}
```

The result is really cool:

![](https://i.imgur.com/G1GQJ6G.gif)


## Summary

In this lesson we have learned how to create forms with Angular. We have seen how to add validation messages to help the user fill out fields with guidance, and how to style Angular classes to give the user visual cues.

## Extra Resources

* [Angular Forms](https://angular.io/docs/ts/latest/guide/forms.html) - The official Angular documentation
