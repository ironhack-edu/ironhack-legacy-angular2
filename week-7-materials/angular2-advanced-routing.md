![Ironhack Logo](https://i.imgur.com/1QgrNNw.png)

# Advanced Routing

## Learning Goals

After this lesson, you will be able to:

* Pass optional parameters between pages.
* Add child routes.

## Optional Parameters

In the routing lesson, we saw how we can pass route parameters between pages by configuring a route with a `/:parameter` signature. However, this type of parameter is strictly required in order to navigate the route, and if it is not provided, the router does not activate the corresponding component.

Sometimes we want to allow optional parameters to be set, to pass non-critical information between pages. To do this, we need to set the extra parameters in the navigation method.

Let's add an optional parameter to our `contact-list` page:

(This is the full component if you need a fresh starting point. If the links don't work, be sure to add a router outlet tag to the app.component's HTML.)

```typescript
// contact-list.component.ts
import { Router } from '@angular/router';
import { Component, OnInit } from '@angular/core';
import { ContactService } from '../services/contact-service';

@Component({
  selector: 'contact-list',
  template: `
    <table>
      <tr>
        <th> Id</th>
        <th> Name </th>
        <th> Details (btn) </th>
        <th> Details (link) </th>
        <th> Opt. Param </th>
      </tr>
      <tr *ngFor="let contact of contacts">
        <td> {{ contact.id }} </td>
        <td> {{ contact.name }} </td>
        <td>
          <button (click)="viewDetails(contact.id, optionalParam.value)">
              details (btn)
          </button>
        </td>
        <td>
          <a [routerLink]="['/contact', contact.id]"
             [queryParams]="{ foo: optionalParam.value }">
              details (link) </a>
        </td>
        <td> <input #optionalParam (keyup)="0" /> </td>
      </tr>
    </table>
  `
})

export class ContactListComponent implements OnInit {

  contacts: Array<any>;

  constructor(
    private router: Router,
    private contactService: ContactService
  ) {}

  ngOnInit() {
    this.contacts = this.contactService.getList();
  }

  viewDetails(id, param){
    this.router.navigate(['contact', id], { queryParams: { foo: param }});
  }
}
```

This time, instead of hard-coding our array of contacts in the component class, we used the contact service created in the service lesson, which provides two methods to retrieve contacts and another method to edit a contact. Don't forget to import the service and add it as a provider in `app.module.ts`.

```typescript
// contact-service.ts
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

In the component, we can see we have two ways to send optional parameters:
1. Using the `[queryParams]` directive on a link
2. Sending an extra object in the `navigate` method of the router service

Since these parameters are optional, our routes don't need any extra configuration, we just need to change the way we read parameters:

```typescript
// contact.component.ts
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

@Component({
  selector: 'contact',
  template: `
    <h3>
      This is the page for the contact: {{ contactId }}
    </h3>
    <p *ngIf="optionalParameter">
        An optional parameter: {{ optionalParameter }}
    </p>
    <a [routerLink]="['/']"> Back to list </a>
  `
})
export class ContactComponent implements OnInit {
  contactId: number;
  optionalParameter: string;

  constructor(private route: ActivatedRoute) { }

  ngOnInit() {
    this.route.params
      .subscribe((params) => {
        this.contactId = +params['id'];
    });

    this.route.queryParams
      .subscribe((queryParams) => {
        this.optionalParameter = queryParams['foo'];
    });
  }
}
```

Here we added a new subscription, this time to the `queryParams` Observable provided by the `ActivatedRoute` service. Once we receive it, we assign it to our `optionalParameter` class property to display it, the result should look something like:

![](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_067976f7d0fba29b88a847508ae84af6.gif)

## Child Routes

When some routes can only be accessed or viewed within other routes, it may be appropriate to create them as child routes.

For example, if we add edit functionality to our contact list, the routes might look like this:

1. `/contact/100/` will show info about the contact 100
2. `/contact/100/edit` will show an edit form for the contact 100

We don't want the `/edit` be a child of the `/` route because the features are siblings rather than nested. They share the same `/contact/100` path and provide different features.

It's easy to understand the policy we are following - we want a path to be self-explanatory. This approach is also very extendable, as we can just add routes such as `/contact/100/duplicate`.

So let's build our nested routes:

In our `app.module.ts`:

```typescript
const routes: Routes = [
  { path: '', component: ContactListComponent },
  { path: 'contact/:id', component: ContactComponent,
    children: [
      { path: '', component: ContactOverviewComponent },
      { path: 'edit', component: ContactEditComponent }
    ]
  }
];
```

We added a `children` property to our routes, specifiyng the `ContactOverviewComponent` component as default, and the `ContactEditComponent` for the `edit` path.

These components still don't exist. Add them in the command prompt:

```
$ ng g component contact-overview
$ ng g component contact-edit
```

Now we can refactor our main `ContactComponent`:

```typescript
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'contact',
  template: `
    <a [routerLink]="['/']"> Back to list </a>
    <router-outlet></router-outlet>
  `
})
export class ContactComponent implements OnInit {
  contactId: number;
  ngOnInit() { }
}
```

We removed most of the logic and added a nested `router-outlet` directive which defines where child routes are going to be rendered.

:::danger
If you don't specify any outlet, the child routes will raise an error
:::

Our nested components will just show some feature content:

```typescript
// contact-overview.ts
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';
import { ContactService } from '../contact.service';

@Component({
  selector: 'contact-overview',
  template: `
    <div>
      <h4> This is the contact overview page </h4>
      <label> Id: {{ contact.id }} </label>
      <label> Name: {{ contact.name }} </label>
      <p *ngIf="optionalParameter"> An optional parameter: {{ optionalParameter }} </p>
    </div>
  `
})
export class ContactOverviewComponent implements OnInit {
  contact: any;
  optionalParameter: string;

  constructor(
    private route: ActivatedRoute,
    private contactService: ContactService
  ) { }

  ngOnInit() {
    this.route.params
      .subscribe((params) => {
        this.contact = this.contactService.get(+params['id']);
    });

    this.route.queryParams
      .subscribe((queryParams) => {
        this.optionalParameter = queryParams['foo'];
    });
  }
}
```

The `overview` component will retrieve both parameters (route and optional) and fetch the specific contact from the `ContactService` using the `get` method.

The `edit` component will instead show an input to edit the contact name:

```typescript
// contact-edit.component.ts

import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';
import { ContactService } from '../contact.service';

@Component({
  selector: 'contact-edit',
  template: `
    <div>
      <h4> This is the edit contact page </h4>
      <label> id: {{ editContact.id }} </label>
      <br>
      <label> name: </label>
      <input type="text" [(ngModel)]="editContact.name" />
      <br>
      <button (click)="save()"> Save </button>
    </div>
  `
})
export class ContactEditComponent implements OnInit {
  originalContact: any;
  editContact: any;

  constructor(
    private route: ActivatedRoute,
    private contactService: ContactService
  ) { }

  ngOnInit() {
    let paramId = +this.route.snapshot.parent.params['id'];
    this.originalContact = this.contactService.get(paramId);
    this.editContact = {
      id: this.originalContact.id,
      name: this.originalContact.name
    };
  }

  save() {
    this.originalContact = this.contactService.edit(this.editContact);
  }
}
```

We have two instance properties:
- `originalContact` is the contact stored in our service
- `editContact` is the contact we show to the user to edit

Notice that even if the `overview` and the `edit` route are siblings (children of the same route), they have different behavior when reading URL parameters. The `overview` route can read the params as usual, because it doesn't add any segment to the URL (empty path). The `edit` route needs to access the parent snapshot to read the same params with:

```typescript
this.route.snapshot.parent.params['id'];
```

which is not an observable to subscribe to but a plain string variable.

Once retrieved the parameter, it can load the contact using the `contactService` and display the form.
The save method just replace the original contact with the edited one.

If everything is working, we can add an edit link to the `contact-list` component and edit the entries:

```typescript
  template: `
    <table>
      <tr>
        <th> Id</th>
        <th> Name </th>
        <th> Details (btn) </th>
        <th> Details (link) </th>
        <th> Edit </th>
        <th> Opt. Param </th>
      </tr>
      <tr *ngFor="let contact of contacts">
        <td> {{ contact.id }} </td>
        <td> {{ contact.name }} </td>
        <td>
          <button (click)="viewDetails(contact.id, optionalParam.value)">
              details (btn)
          </button>
        </td>
        <td>
          <a [routerLink]="['/contact', contact.id]"
             [queryParams]="{ foo: optionalParam.value }">
              details (link) </a>
        </td>
        <td>
          <a [routerLink]="['/contact', contact.id, 'edit']"
             [queryParams]="{ foo: optionalParam.value }">
              Edit</a>
        </td>
        <td> <input #optionalParam (keyup)="0" /> </td>
      </tr>
    </table>
  `
```

![](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_b87c4760159c6ccbadc7d29ae750de5d.gif)

## Summary

In this lesson, we have studied how to set child routes for related features, and how to define optional parameters for our routes.

## Extra Resources
* [Angular Router](https://angular.io/docs/ts/latest/guide/router.html#!#optional-route-parameters) - The official Angular documentation
