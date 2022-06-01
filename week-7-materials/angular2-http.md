![Ironhack Logo](https://i.imgur.com/1QgrNNw.png)

# HTTP

## Learning Goals
After this lesson, you will be able to:

* Use Angular's library to perform HTTP requests.
* Communicate with external services to retrieve data.
* Take advantage of observables to power up the requests.

## Introduction

When we make calls to an external server, we want our user to continue to be able to interact with the page. That is, we don’t want our page to freeze until the HTTP request returns from the external server. To achieve this effect, our HTTP requests are asynchronous.

Dealing with asynchronous code is, historically, more tricky than dealing with synchronous code. In JavaScript, there are generally three approaches to dealing with async code:

- Callbacks
- Promises
- Observables

## Angular HTTP
Angular comes with its own HTTP library which we can use to perform requests to external APIs. To use it we need to import the `HttpModule` from `@angular/http` in our `app.module.ts`.

First create a new project with `ng new angular-http`, then add the following:

```typescript
...
import { HttpModule } from '@angular/http';

@NgModule({
  declarations: [
    ...
  ],
  imports: [
    ...
    HttpModule           // <!-- Angular's HTTP module
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

## Usage

To start playing with HTTP, we'll use the free public [ICNDB API](http://www.icndb.com/api/) which will return us some [Chuck Norris](https://en.wikipedia.org/wiki/Chuck_Norris) jokes.

Let's create a service to send the request and a component to display the result. We'll place every related file in a dedicated folder named `jokes`.

Run `mkdir src/app/jokes` to create the directory, then `ng g component jokes/jokes` and `ng g service jokes/jokes` to generate the new files.

Our service should expose a method to fetch a random joke from the external API. The component then requests it through a button and displays the result.

```typescript
// jokes.service.ts
import { Injectable } from '@angular/core';
import { Http, Response } from '@angular/http';
import { Observable } from 'rxjs/Observable';

@Injectable()
export class JokesService {

  constructor(private http: Http) { }

  getRandom() {
    return this.http.get('http://api.icndb.com/jokes/random')
      .map((res) => res.json());
  }
}
```

In the service on line 12 we take the response and use the `map` method to parse the JSON string from the API into a JavaScript object.

Now for the jokes component:

```typescript
// jokes.component.ts
import { Component, OnInit } from '@angular/core';
import { JokesService } from '../jokes.service';

@Component({
  selector: 'jokes',
  template: `
    <h3> Jokes </h3>
    <button (click)="getRandomJoke()"> Get Random Joke </button>
    <pre>{{ joke | json }} </pre>
  `,
  providers: [JokesService]
})
export class JokesComponent implements OnInit {

  joke: any;
  constructor(private jokes: JokesService) { }

  ngOnInit() {}

  getRandomJoke() {
    this.jokes.getRandom()
      .subscribe((joke) => this.joke = joke);
  }
}
```

In the component we have a button (line 9) that invokes the component's `getRandomJoke()` method, which in turn invokes the `getRandom()` method of the service (line 22). Then we subscribe to the Observable response (line 23), assigning the resulting `joke` to the instance property `this.joke`. The `this.joke` property is then printed in a `<pre>` element in the template (line #10). We use the built-in `json` pipe to display the object as a JSON string.

If we try to run the app, we can see that it is broken :disappointed:
From the console that our `jokes.service.ts` we can spot an error:

:::danger
`Property 'map' does not exist on type 'Observable<Response>'`
:::

To fix this we need to zoom out a little and understand the origin.
The `Http` service provided by Angular returns an [Observable](https://github.com/tc39/proposal-observable) of [`Response`](https://angular.io/docs/ts/latest/api/http/index/Response-class.html) objects, which represent a server response.

We can think an observable as an array of asynchronous values, each of which is the response to the "current" request.

The point of this is that the `map` function we are invoking after the `GET` request is not the native [`Array.protorype.map()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map) as we might think, but something else related to the `Observable` type, specifically, an operator.

TypeScript simply doesn't know what `map` on an `Observable<Response>` is, and raises an error. To fix this, we just need to import the `map` operator from the `rxjs` library by adding the following to the top of the service:

```typescript
import 'rxjs/add/operator/map';
```

Now that we have no errors, let's press the button!

![](https://i.imgur.com/7NJOW1k.gif)

:::success
External communication logic such as AJAX requests should be encapsulated in services. We don't want our components to deal with HTTP.
:::

We're now performing HTTP requests from the client!

## Taking advantage of observables

Observables are an interesting technology introduced by the Angular team. They give us a lot of power when it comes to asynchronous programming.

### The async pipe

In the pipe lesson we learned about the principal pipes the framework gives us. The `Async` pipe is one of them. It takes an observable as input and waits for its value to render, so it does subscription for us.

First, we'll update our `JokesService` to only return the required data: the joke value.

```typescript
...

@Injectable()
export class JokesService {

  constructor(private http: Http) { }

  getRandom(): Observable<string> {
    return this.http.get('http://api.icndb.com/jokes/random')
      .map((res: Response) => res.json())
      .map((res) => res.value.joke);
  }
}
```

We also set a return type of `Observable<string>` to our method since we're only returning the joke value, which is a string.

Now let's see how our component changes:

```typescript
import { Component, OnInit } from '@angular/core';
import { JokesService } from '../jokes.service';
import { Observable } from 'rxjs/Observable';

@Component({
  selector: 'jokes',
  template: `
    <h3> Jokes </h3>
    <button (click)="getRandomJoke()"> Get Random Joke </button>
    <p>{{ joke$ | async }} </p>
  `,
  providers: [JokesService]
})
export class JokesComponent implements OnInit {

  joke$: Observable<string>;
  constructor(private jokes: JokesService) {}

  ngOnInit() {}

  getRandomJoke() {
    this.joke$ = this.jokes.getRandom();
  }
}
```

Instead of subscribing directly in our `getRandomJoke` method, we let the async pipe do that for us, just by giving it the observable as input.

:::info
To make observable variables easy to recognize we postfix the names with a `$` symbol, like `joke$`.
:::

![](https://i.imgur.com/eOIXtzW.gif)

While this looks a lot clearer, it does not allow us to catch errors, if any. We should only use this technique if we don't plan to handle errors in the component.

### Chain operators (advanced)

`RxJS`'s power lives in operators, which let us handle a lot of different event scenarios. The `map` operator is just one provided by the library, in this section we'll see some more.

:::info
Take a look at the full list of [operators](http://reactivex.io/documentation/operators.html).
:::

To showcase some operators we'll create an observable from the click event on the button and perform, at max, one request per second.

```typescript
...
import { Observable } from 'rxjs/Observable';
import 'rxjs/add/observable/fromEvent';
import 'rxjs/add/operator/switchMap';
import 'rxjs/add/operator/throttleTime';

@Component({
  selector: 'jokes',
  template: `
    <h3> Jokes </h3>
    <button id="joke-btn"> Get Random Joke </button>
    <p> {{ joke$ | async }} </p>
  `,
  providers: [JokesService]
})
export class JokesComponent implements OnInit {

  joke$: Observable<string>;
  constructor(private jokes: JokesService) {}

  ngOnInit() {
    this.joke$ = Observable
      .fromEvent<MouseEvent>(document.getElementById('joke-btn'), 'click')
      .throttleTime(1000)
      .switchMap(
        (e: MouseEvent) => this.jokes.getRandom()
      );
  }
}
```

We create an observable from the click event, then we chain the `throttleTime` and `switchMap` operators. These let us control the number of requests per second and continue the chain.

![](https://i.imgur.com/LfxzM4h.gif)

### Fallback to promise (Advanced)

Observables are a powerful way to deal with asynchronous events, however the library is not straightforward and might be confusing at first.

If you feel more comfortable using promises, we get you covered, just use the `toPromise` method of observables:

```typescript
// jokes.service.ts
import 'rxjs/add/operator/toPromise';

getRandom(): Promise<string> {
  return this.http.get('http://api.icndb.com/jokes/random')
    .toPromise()
    .then((res: Response) => res.json())
    .then((res) => res.value.joke);
}
```

And then we can just apply the `async` pipe to the promise returned:

```typescript
import { Component, OnInit } from '@angular/core';
import { JokesService } from '../jokes.service';

@Component({
  selector: 'jokes',
  template: `
    <h3> Jokes </h3>
    <button (click)="getRandomJoke()"> Get Random Joke </button>
    <p>{{ joke | async }} </p>
  `,
  providers: [JokesService]
})
export class JokesComponent implements OnInit {

  joke: Promise<string>;

  constructor(private jokes: JokesService) {}

  ngOnInit() {}

  getRandomJoke() {
    this.joke = this.jokes.getRandom();
  }
}
```

This solution looks much easier to implement, but reactive programming using observables is a good pattern to learn.

## Summary

In this lesson, we learned how to use Angular's library to perform HTTP requests that will help us communicate with external services to retrieve data. We also learned how to take advantage of observables to power up the requests.

## Extra Resources
* [Angular HTTP](https://angular.io/docs/ts/latest/guide/server-communication.html) - The official Angular documentation
