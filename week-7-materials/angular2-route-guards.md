![Ironhack Logo](https://i.imgur.com/1QgrNNw.png)

# Route Guards

## Learning Goals

After this lesson, you will be able to:

* Control the access of your routes.
* Confirm or cancel navigation with guards.

## Introduction

We use route guards to control whether a user can enter or leave a specific route. We need guards to protect our routes.

To build our guard we just need to implement the specific interface provided by the framework: `CanActivate` and/or `CanDeactivate`.

Both guards must return either a `boolean` value or an asynchronous variable which will resolve to a `boolean` value, typically the second case.

In this unit we are going to add some route guards to our `phone-app` application.

## CanActivate
We may want some routes to only be accessible once the user has logged in or accepted Terms & Conditions. We can use route guards to check these conditions and control access to routes.

When `CanActivate` returns `true`, the user can activate the route. When `CanActivate` returns `false`, the user can't access the route. In the following example, we allow access to the user after a second.

:::info
Yes, this guard doesn't make much sense. But a 1s delay is a visible effect, the important thing here is to notice the **asynchronous** nature of **guards**.
:::

### Implementation

First let's create a new service called `EnterDetailsGuardService` with:

```
$ ng g service enter-details-guard
```
If you have services folder, thank follow this:

```
$ ng g service services/enter-details-guard
```
Here we created a simple service which *implements* the [canActivate](https://angular.io/docs/ts/latest/api/router/index/CanActivate-interface.html) interface.

Since our service depends on other services (ie. canActivate), we will use the [@injectable](https://angular.io/docs/ts/latest/cookbook/dependency-injection.html#!#nested-dependencies) decorator to allow Angular's dependency injection mechanism to make sure we have the libraries loaded for us.

[Observable](https://angular.io/docs/ts/latest/glossary.html#!#O): An array whose items arrive asynchronously over time. Observables help you manage asynchronous data, such as data coming from a backend service.

:::info
To use observables, Angular uses a third-party library called Reactive Extensions (RxJS). Observables are a feature for ES2016 version of JavaScript.
:::

This service that implements `CanActivate` will always return true:

```typescript
import { Injectable }  from '@angular/core';
import { CanActivate } from '@angular/router';
import { Observable }  from 'rxjs/Rx';

@Injectable()
export class EnterDetailsGuardService implements CanActivate {
  canActivate(): {
    return true;
  }
}
```

A better version will return true asynchronously one second later. Actually, it will return a [Promise](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise), and the promise will be resolved to true 1 second later asynchronously:

```typescript
import { CanActivate } from '@angular/router';
import { Injectable }  from '@angular/core';
import { Observable }  from 'rxjs/Rx';

@Injectable()
export class EnterDetailsGuardService implements CanActivate {
  canActivate(): Observable<boolean> | Promise<boolean> | boolean {
    console.log('canActivate guard has been called!');
    return new Promise((resolve, reject) => {
      setTimeout(() => resolve(true), 1000);
    });
  }
}
```

This interface has only one method, the `canActivate` method, which returns either an **asynchronous** or **synchronous value** that resolves to a boolean. That's why in the signature we added 3 possible return types.

:::info
:bulb: It's a good practice to declare your functions' return types when writing TypeScript. Not only does it makes clear to everyone reading what the function should do, but it also assists our text editors in catching errors before we run the code.
:::

In the guard we added a debug log to print a message in console, just to have a visual hook on what is going on. Finally we have a 1s (1000ms) timeout before resolving to true, this lets the route activate.

### Usage

**Before using the activation guard we need to register our service as a provider in the `app.module.ts` file.**

Now we just need to configure the specific route to load this guard, we'll apply the guard to the `phone/:id` path:

```typescript
export const routes: Routes = [
    { path: '', component: PhoneListComponent },
    { path: 'add', component: AddPhoneComponent },
    {
        path: 'phone/:id',
        component: PhoneDetailsComponent,
        canActivate: [ EnterDetailsGuardService ]    <!-- Add enter guard
    },
    { path: '**', redirectTo: '' }
];
```

Finally we're all set, just refresh the page and you should experience a 1 second delay when browsing the overview route, something like this:

![](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_a6e068596d884b88a772fd5ff982a559.gif =550x)

Here you can see in the console that the route guard gets called when the navigation (mouse click) happens, then after 1s the page changes.


## CanDeactivate

Route guards can also control whether a user can leave a certain route. For example, say the user has typed information into a form on the page, but has not submitted the form. If they were to leave the page, they would lose the information. We may want to prompt the user if the user attempts to leave the route without submitting or saving the information.


`CanDeactivate` works in a similar way to `CanActivate`.

### Implementation

We need to implement the "leave" logic in the `AddPhoneComponent` so we'll create a method called `isFormClean`, which we will call the guard:

```typescript
// add-phone.component.ts

...

isFormClean(): boolean {
  if (this.newPhone.name !== '' || this.newPhone.brand !== '') {
    return window.confirm(`
        Unsaved changes.
        Are you sure you want to leave?
    `);
  }

  return true;
}
```

The logic is pretty simple, we want to prompt the user with an alert if they attempt to leave the page while the form is dirty.  The user will need to click 'ok' in order to leave the page.

The `canDeactivate` function receives the instance of the component that is being deactivated as its argument, so it can call it's methods.

Let's create a new service called `LeaveEditContactGuardService` with:

```
$ ng g service leave-add-phone-guard
```
or: 

```
$ ng g service services/leave-add-phone-guard
```

```typescript
import { Injectable } from '@angular/core';
import { CanDeactivate } from '@angular/router';
import { AddPhoneComponent } from '../add-phone/add-phone.component';
import { Observable } from 'rxjs/Rx';

@Injectable()
export class LeaveAddPhoneGuardService
       implements CanDeactivate<AddPhoneComponent> {

  canDeactivate(component: AddPhoneComponent)
    : Observable<boolean> | Promise<boolean> | boolean {

    return component.isFormClean();
  }

}
```

### Usage

As always, we need to register the provider in the `app.module.ts` file. Add `LeaveAddPhoneGuardService` to the providers array.

Then remember to configure the specific route to load this guard, we'll apply this guard to the `add` path:

```typescript
export const routes: Routes = [
    { path: '', component: PhoneListComponent },
    {
        path: 'add',
        component: AddPhoneComponent,
        canDeactivate: [ LeaveAddPhoneGuardService ]  <!-- Add leave guard
    },
    {
        path: 'phone/:id',
        component: PhoneDetailsComponent,
        canActivate: [ EnterDetailsGuardService ]
    },
    { path: '**', redirectTo: '' }
];
```

The result should be something like:

![](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_5a8fa8c96ff9ce712b362bc525d1604c.gif)

Now if the form has unsubmitted changes and the user attempts leave the page, they will be presented with an alert and will need to click 'ok' in order to proceed with their navigation.

## Resolve (advanced)

`CanActivate` and `CanDeactivate` are two of the most commont guards, but there are many others for more specialized cases. In this unit we'll examine the `resolve` guard, which allows us to fetch some data before navigating to a particular route.

This guard is commonly used when the destination route doesn't make sense without some sensitive data. Instead of letting the user browse and eventually see an empty page, we want to make sure there's something to see there.

One place it would make sense to use this guard in our application would be in the details route. The phone's details route doesn't make any sense if we somehow navigate to a route with an id that doesn't exist in our database. The backend API `/phone/:id` would return an error, and we would probably display an error message to the user.

Let's try to optimize our app's user experience and check if the phone exists before navigating to the details route, and cancel the navigation if no matching phone is found. 

### Implementation

As usual let's create a new service called `ResolveDetailsGuardService` with:

```
ng g service resolve-details-guard
```
or:
```
ng g service services/resolve-details-guard
```

```typescript
import { Observable } from 'rxjs/Rx';
import { Injectable } from '@angular/core';
import { Resolve, ActivatedRouteSnapshot, Router } from '@angular/router';
import { PhoneService } from '../phone.service';

@Injectable()
export class ResolveDetailsGuardService implements Resolve<any> {

  constructor(
    private phoneService: PhoneService,
    private router: Router
  ) {}

  resolve(route: ActivatedRouteSnapshot): Observable<any> {
      return this.phoneService.get(route.params['id'])
        .catch((err) => {
          this.router.navigate(['/']);
          return Observable.of(err);
        });
  }
}
```

Here we implement the `Resolve` interface which needs a `resolve` method.

In the constructor we inject 2 dependencies:

- `PhoneService` to try to fetch the requested phone
- `Router` to navigate back if the phone doesn't exists

The `resolve` method automatically receives the`ActivatedRouteSnapshot` parameter, which we can use to read the `id` parameter.

Now:

- If the `phoneService.get()` call does not raise an error (i.e. the phone is retrieved), the navigation will continue
- Otherwise we have the `rxjs`'s `catch` operator, which will catch the errors and redirect the user to the `/` route.

:::info
We didn't define a `Phone` model to represent a phone object because it was a bit overkill for our application. That would have been helpful in this case, because the `Resolve` interface is a generic that lets us define the type that is going to be resolved by the guard itself, that's why we put `any` in the class signature.
:::

Conceptually this guard does the exact same thing that the `getPhoneDetails` method of our `PhoneDetailsComponent` is doing. The real difference is just in the timing that the two are happening. The guard is triggered **before** the navigation, while the `getPhoneDetails` methods runs after the navigation has completed and the url param has been retrieved.

### Usage

Different guard, same flow.

First let's register our provider in the `app.module.ts` file, so add `ResolveDetailsGuardService` in the providers array next to the other guards.

Then we need to configure our route to use this guard, in `app.routing.ts` our routes become:

```typescript
export const routes: Routes = [
    { path: '', component: PhoneListComponent },
    {
        path: 'add',
        component: AddPhoneComponent,
        canDeactivate: [ LeaveAddPhoneGuardService ]
    },
    {
        path: 'phone/:id',
        component: PhoneDetailsComponent,
        canActivate: [ EnterDetailsGuardService ],
        resolve: {
           phone: ResolveDetailsGuardService           <!-- Add resolve guard
        }
    },
    { path: '**', redirectTo: '' }
];
```

We're almost done, we just need to adapt our `PhoneDetailsComponent` to the new flow. Let's change the `ngOnInit` method:

```typescript
...


ngOnInit() {
  this.route.data.subscribe((resolved) => {
    this.phone = resolved['phone'];
  });
}

...
```

The guard will fetch the phone for us, so there's no need to fetch it again here. We can take the result from the `data` Observable of the active route.



:::info
Now the `PhoneDetailsComponent` will only be activated activated if the phone exists.
:::

Feel free to remove the now unused `getPhoneDetails` method.

If everything went right, the result should be:

![](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_83a8805fd9d7392f163a7661194ba2b1.gif =600x)

## Summary

In this lesson we have learnt how we can control the access of our routes, and how to confirm or cancel navigation with Route Guards.

### Extra

- [Resolve guard](https://angular.io/docs/ts/latest/guide/router.html#!#resolve-guard) - Angular official documentation
