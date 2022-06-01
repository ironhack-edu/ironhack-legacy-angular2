![Ironhack Logo](https://i.imgur.com/1QgrNNw.png)

# Backend / Frontend integration

## Learning Goals
After this lesson, you will be able to:

* Have a full-stack application running.
* Consume a REST API using Angular.
* Build an application for release.

## Introduction

In this lesson we will build a small Angular application to consume our REST API. We will have a page showing all the phones in the database. Selecting one of them will show the phone's details and allow you to edit or delete the phone entry. 
## Client app

First scaffold the application using the angular-cli running:

```
$ ng new phone-app
```

this will create the project structure and install the required dependencies.

Remember to change the inline style setting if necessary. From the terminal window, move into the project folder and run:

```
$ ng set defaults.inline.style true
```

:::info
While this last step is not required, it will help us writing components in a single file and keep the lesson code easier to read.
:::

## Basic structure

We want to have a list of phones in the default route, and a detail view at the `phone/<id>` url.

Let's create our 2 components first:

```
$ ng g component phone-list
$ ng g component phone-details
```

then create an `app.routing.ts` file in the `src/app` folder to define the app routes:

```typescript
import { Routes } from '@angular/router';

import { PhoneDetailsComponent } from './phone-details/phone-details.component';
import { PhoneListComponent } from './phone-list/phone-list.component';

export const routes: Routes = [
    { path: '', component: PhoneListComponent },
    { path: 'phone/:id', component: PhoneDetailsComponent },
    { path: '**', redirectTo: '' }
];
```

Now we need to set the routes to the top level module in the `app.module.ts` file:

```typescript
...

import { RouterModule } from "@angular/router";
import { routes } from './app.routing';
import { HttpModule } from '@angular/http';

@NgModule({
  declarations: [
    AppComponent,
    PhoneListComponent,
    PhoneDetailsComponent
  ],
  imports: [
    BrowserModule,
    RouterModule.forRoot(routes)
    HttpModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

We now just want to make sure everything works fine, so let's add a link in both `phone-list.component.ts` and `phone-details.component.ts` to browse the pages.:

```html
// template of phone-list component

<p> phone-list Works! </p>
<a [routerLink]="['phone', 42]"> View Details </a>


// template of phone-details component

<p> phone-item Works! </p>
<a [routerLink]="['']"> Back to list </a>

// template of main component
<router-outlet></router-outlet>
```

If everything is fine, the result should look like this:

![](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_d7e6a8bf2cc6a30ded74174121eef521.gif)

## Use HTTP Module

Now that our pages are working fine, we need to add data to them. Luckily we have a brand new REST API to consume :muscle:

Let's start building a service to wrap all HTTP requests going to our API, from the terminal:

```
$ ng g service phone
```

```typescript
import { Injectable } from '@angular/core';
import { Http } from '@angular/http';
import 'rxjs/add/operator/map';
import { environment } from '../environments/environment'

@Injectable()
export class PhoneService {
  constructor(private http: Http) {}

  getList() {
    return this.http.get(`${environment.BASE_URL}/api/phones`)
      .map((res) => res.json());
  }

  get(id) {
    return this.http.get(`${environment.BASE_URL}/api/phones/${id}`)
      .map((res) => res.json());
  }

  edit(phone) {
    return this.http.put(`${environment.BASE_URL}/api/phones/${phone.id}`, phone)
      .map((res) => res.json());
  }

  remove(id) {
    return this.http.delete(`${environment.BASE_URL}/api/phones/${id}`)
      .map((res) => res.json());
  }
}
```

You'll notice that you get errors related to a new concept - environment variables. Environment variables let us use different variable values for development and production. To add an environment variable, go into the `environments` folder, and add the following to `environments.ts`:

```typescript
export const environment = {
  production: false,
  BASE_URL: 'http://localhost:3000'
};
```

We can now import this file, and use the `BASE_URL` variable with `environment.BASE_URL`. Later, we'll add a `BASE_URL` value to environment.prod.ts for the production version of this app on Heroku.

:::info
:bulb: Remember to register the provider in the `app.module.ts` by adding the `PhoneService` service to the `providers` array.
:::

### Phone list

Now that our service is set up, we can move to the`phone-list.component.ts` and fetch the phone list.

We need to inject the `PhoneService` service in the component constructor and invoke the `getList` method:

```typescript
import { Component, OnInit } from '@angular/core';
import { PhoneService } from './../phone.service';

@Component({
  selector: 'app-phone-list',
  template: `
    <h1> Phone list </h1>
    <div>
      <div *ngFor="let phone of phones">
        <img [src]="phone.image" />
        <h3> {{ phone.name }} </h3>
        <p> {{ phone.brand }} </p>
      </div>
    </div>
  `,
  providers: [PhoneService]
})
export class PhoneListComponent implements OnInit {
  phones;

  constructor(private phone: PhoneService) { }

  ngOnInit() {
    this.phone.getList()
      .subscribe((phones) => {
        this.phones = phones;
      });
  }
}
```

We use the `ngOnInit` lifecycle hook to call the API through the phone service. Then we assign the result to the component `phones` property to iterate over it using the `*ngFor` directive.

![](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_b2e674ef3d960f708eff967569063f13.png =600x)

Now we want to implement the details view. So let's add a link to navigate the single phone and implement that component:

```htmlmixed
// Template in phone-list.component.ts
...

<h1> Phone list </h1>
<div>
  <div *ngFor="let phone of phones">
    <img [src]="phone.image" />
    <h3> {{ phone.name }} </h3>
    <p> {{ phone.brand }} </p>
    <a [routerLink]="['phone', phone._id]"> View Details </a>
  </div>
</div>

...
```

### Phone details

Now that we have a working link we need to capture the URL parameters in the `detail` route and fetch the phone details.

In the `phone-details.component.ts`:

```typescript
import { Component, OnInit, Input } from '@angular/core';
import { ActivatedRoute } from '@angular/router';
import { PhoneService } from '../phone.service';

@Component({
  selector: 'app-phone-details',
  template: `
    <h1> Phone details </h1>
    <a [routerLink]="['']"> Back to list </a>

    <div *ngIf="phone">
      <div class="phone-thumbnail">
        <img height="300" [src]="phone.image" />
      </div>

      <div class="phone-info">
        <h2> {{ phone.name }} </h2>
        <h3> {{ phone.brand }} </h3>

        <div *ngIf="phone.specs.length">
          <h4> Features </h4>
          <ul>
            <li *ngFor="let spec of phone.specs"> {{ spec }} </li>
          </ul>
        </div>
      </div>
    </div>
  `,
  providers: [PhoneService]
})
export class PhoneDetailsComponent implements OnInit {
  phone: any;

  constructor(
    private route: ActivatedRoute,
    private phoneService: PhoneService
  ) { }

  ngOnInit() {
    this.route.params.subscribe(params => {
      this.getPhoneDetails(params['id']);
    });
  }

  getPhoneDetails(id) {
    this.phoneService.get(id)
      .subscribe((phone) => {
        this.phone = phone;
      });
  }
}
```

There seems to be a lot of code in this component, let's see it step by step.

1. In the component constructor, we inject the `ActivatedRoute` and the `ActivatedRoute` services
2. In the `ngOnInit` lifecycle hook, we read the URL parameter usign the active route
3. Once we have the phone id, we fetch its details using the `PhoneService` `get` method
4. We assign the result of the service call to the instance property `phone` to display the content

If everything is fine, you should see something like this:

![](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_f396d1b09b86d2ab61412ccfa4fe088c.gif =600x)

:::info
We used some under-the-hood styling to make it look prettier. Don't worry if your result is a bit out of layout for now.
:::

### Delete a phone

We now want to implement the delete feature. Let's add a button in out details view to delete a phone:

```htmlmixed
// Template in phone-details.component.ts

<!-- Place the button where you like most in the template -->
<button (click)="deletePhone()"> Delete </button>
```

and implement the `deletePhone` method in the `phone-details` component:

```typescript
import { Router, ActivatedRoute }    from "@angular/router";

//

constructor(private route: ActivatedRoute, private router: Router, private phoneService: PhoneService) { }

//

deletePhone() {
  if (window.confirm('Are you sure?')) {
    this.phoneService.remove(this.phone._id)
      .subscribe(() => {
        this.router.navigate(['']);
      });
  }
}
```

Here we ask the user for confirmation and then, in case of positive answer, we perform the request. Once the phone is deleted we redirect to the list page.

The result is:

![](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_3b0773c860edf8e74780ac1f74e8fac0.gif =600x)

## Package the app

Now that our simple app is done we want to optimize it for deployment. We need to do two things - add any environment variables necessary for production, then package the app.

Adding production environment variables is simple: we add them to the file `environment.prod.ts` like we did `environment.ts`, like so:

```typescript
export const environment = {
  production: false,
  BASE_URL: 'http://localhost:3000'
};
```

Next, we build the app. The command `ng build --prod --aot` concatenates and minifies the source code (html + js + css) into few small files and fixes all the links. The result of this step will be placed in the `/dist` folder of the project.

```
Hash: 1bede32199a9d8b16c9f
Time: 16436ms
chunk    {0} main.381668b78a9b1244316b.bundle.js, main.381668b78a9b1244316b.bundle.map (main) 102 kB {2} [initial] [rendered]
chunk    {1} styles.946e5628c7fa4666016a.bundle.css, styles.946e5628c7fa4666016a.bundle.map, styles.946e5628c7fa4666016a.bundle.map (styles) 1.69 kB {3} [initial] [rendered]
chunk    {2} vendor.18738db70414012a2df4.bundle.js, vendor.18738db70414012a2df4.bundle.map (vendor) 1.71 MB [initial] [rendered]
chunk    {3} inline.e6178b546e9108ebfbbf.bundle.js, inline.e6178b546e9108ebfbbf.bundle.map (inline) 0 bytes [entry] [rendered]
```

The console says the process created four chunks (bundles) which contains our optimized application.

:::info
The `aot` flag set the *Ahead-of-time* compilation which transforms the source into vm-friendly code. The resulting code is ready-to-run in the browser so the whole client process will be faster.
:::

## Configure the Express server

Now that we have our client application bundled in the `/dist` folder we can move it into a static folder of our back-end app. We want our Express server to serve the entire client application.

Let's copy all the files in the client `dist` folder to the server's  `public` folder.

If we start the server with `npm start` we can see that the client application is served. There's only one issue, if we browse an uncatched routes like `/crazyurl` the server returns a 404 error because it doesn't know any response for that request.

![](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_84407bb483ec2e278e3bceefd5afe928.png)

We don't want this to happen. Instead we want to render our client application and let the Angular router handle the error.

Let's add a default behavior in our server router: in the server `app.js` file:

```javascript
...

app.use('/api', phonesApi);

// This will be the default route is nothing else is caught
app.use(function(req, res) {
  res.sendfile(__dirname + '/public/index.html');
});

...
```

Now we can start the server with `npm start` and see if the router works fine:

![](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_be4aebdd2ee0a7056082fb664f6b0c1d.gif =500x)

## Summary

In this lesson we have learned how to run a full-stack application, how to consume a REST API using Angular, and how to build an application for release.
