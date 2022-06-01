![Ironhack Logo](https://i.imgur.com/1QgrNNw.png)

# File Upload

## Learning Goals
After this lesson, you will be able to:

* Create an API that can accept files uploaded by the user with Multer.
* Upload a file from an Angular app.

## Phone store

Remember the phone store application we developed the other day?
Now it's time to add a feature so that users can add a phone and upload an image along with it.

Let's make our changes to the server application first, later we'll move to the client.

### Multer upload module

In this lesson, we'll use the node `multer` module:

> Multer is a node.js middleware for handling multipart/form-data, which is primarily used for uploading files. It is written on top of busboy for maximum efficiency.

Install it with:

```
$ npm install --save multer
```

#### Configuration

`multer` is pretty easy to configure. We just need to tell it where to save the files and what to call them.

Multer by default doesn't keep the file extension, so we'll use the `path` module to find it and add it to the filename.

To use `path` you need to install it with
```
$ npm install --save path
```

Let's add a new file in `configs/multer.js`:

```javascript
const multer = require('multer');
const path = require('path');

const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, '../public/uploads/')
  },
  filename: (req, file, cb) => {
    cb(null, `${Date.now()}${path.extname(file.originalname)}`)
  }
});

const upload   = multer({ storage });
module.exports = upload;
```

We chose the `public/uploads` as destination folder, and we are naming the uploaded images using the current timestamp to avoid name conflicts.

The multer configuration is done, let's move to the API to add a new phone.

### Adding the upload API

We already have a POST on the `/phones` route defined, so we're going to modify it to allow file uploads.

In `routes/phone.js` first import `multer`:

```javascript=4
const upload = require('../configs/multer');
```

and then add it to the POST route as follows:

```javascript=17
/* CREATE a new Phone. */
router.post('/phones', upload.single('phoneImage'),(req, res, next) => {
  const thePhone = new Phone({
    brand: req.body.brand,
    name: req.body.name,
    specs: JSON.parse(req.body.specs) || []
  });

  if(req.file){
    thePhone.image = `/uploads/${req.file.filename}`,
  }

  thePhone.save()
  .then(thePhone => {
    res.json(thePhone);
  })
  .catch(error => next(error))
});
```

In the `req` variable we'll have access to all the values submitted in the form.

:::info
Since the `specs` field is an array we prefer to send it as a string from the client, to avoid possible errors. Here we use `JSON.parse()` to deserialize the field.
:::

## Angular upload module

Now let's move to client side of the application. The project was called `phone-app`, let's go to that folder.

### Library overview

We'll use the awesome [`ng2-file-upload`](http://valor-software.com/ng2-file-upload/) module to create the upload form.

![](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_dd5568dc135c24c8ac75edf80e5ebeb1.png)

You can install it with:

```
$ npm install --save ng2-file-upload
```

Once it's installed in the project, we have to declare the variable in the `app.module.ts` file:

```typescript=14
import { FileUploadModule } from "ng2-file-upload";
```

```typescript=18
imports: [
  // ...
  FileUploadModule
]
```

### Adding the route

Our "add phone" page will be separated from the list. For this reason we'll crate a new component and assign it to a new route.

From the terminal run:

```
$ ng g component add-phone
```

Here we will add the form to create a new phone.

Now we need to add a new route, in `app.routing.ts` our routes should look like:

```typescript
export const routes: Routes = [
  { path: '', component: PhoneListComponent },
  { path: 'add', component: AddPhoneComponent },
  { path: 'phone/:id', component: PhoneDetailsComponent },
  { path: '**', redirectTo: '' }
];
```

The 2nd route is `add` and will show the form.

### Adding the form

Our form should contain the following fields:

- brand
- name
- image
- specs (opt)

#### Template

```htmlmixed
<h1> Add a new phone </h1>
<a [routerLink]="['']"> Back to list </a>

<form>
  <fieldset>
    <legend> Phone info </legend>
    <label> Brand* </label>
    <input type="text" [(ngModel)]="newPhone.brand" name="brand" required />

    <label> Name* </label>
    <input type="text" [(ngModel)]="newPhone.name" name="name" required />

    <label> Specs </label>
    <input type="text" #spec />
    <button (click)="addSpec(spec.value); spec.value = ''"> add </button>

    <ul>
      <li *ngFor="let spec of newPhone.specs"> {{ spec }} </li>
    </ul>
    <small *ngIf="!newPhone.specs.length"> This phone has no specs :( </small>

    <input type="file" name="phoneImage" ng2FileSelect [uploader]="uploader" />

  </fieldset>
  <button (click)="submit()"> submit </button>
</form>

<p> {{ feedback }} </p>
```

Let's break it down:

1. On the very top we have a link to navigate back to the list of phones
2. The form has 3 text input for `brand`, `name` and `specs`
3. The `add` button adds the current *spec* to the `specs` array field. The current spec is captured using the `#spec` template reference value
4. We have a file-input, which allows the user to select an image, and we attached to it the `ng2FileSelect` attribute and we bound an uploader class property which we'll see in a bit
5. The submit button sends the request to create the new phone


Also, let's update our PhoneService with the method that is connecting our client side with corresponding route in the backend:

```typescript

import { Injectable } from '@angular/core';
import { Http } from '@angular/http';
import 'rxjs/add/operator/map';
import { environment } from '../environments/environment'

@Injectable()
export class PhoneService {

  ...

  createNewPhone(newPhone){
    return this.http.post(`${environment.BASE_URL}/api/phones`, newPhone)
      .toPromise()
      .then(res => res.json());
  }
}

```

#### Component

```typescript

import { PhoneService } from '../phone.service';

import { Router }    from "@angular/router";

export class AddPhoneComponent implements OnInit {
  uploader = new FileUploader({
    url: "http://localhost:3000/api/phones",

    itemAlias: "phoneImage"
  });

  newPhone = {
    name: '',
    brand: '',
    specs: []
  };

  feedback: string;

  constructor(private myPhoneService: PhoneService, private myRouter: Router ) { }

  ngOnInit() { }

  addSpec(spec) {
    this.newPhone.specs.push(spec);
  }

  submit() {
    if (this.uploader.getNotUploadedItems().length === 0) {
      this.savePhoneNoImage();
    } else {
      this.savePhoneWithImage();
    }
  }

  private savePhoneNoImage(){
      this.myPhoneService.createNewPhone(this.newPhone)
        .then(newPhone => {
          this.myRouter.navigate(["/phones"]);
        })
        .catch(err => {
          this.feedback = "There was an error while saving the new phone.";
        });
    } 

  private savePhoneWithImage(){

    this.uploader.onBuildItemForm = (item, form) => {
      form.append('name', this.newPhone.name);
      form.append('brand', this.newPhone.brand);
      form.append('specs', JSON.stringify(this.newPhone.specs));
    };
    this.uploader.onSuccessItem = (item, response) => {
      this.feedback = JSON.parse(response).message;
    };

    this.uploader.onErrorItem = (item, response, status, headers) => {
      this.feedback = JSON.parse(response).message;
    };

    this.uploader.uploadAll();
  }
}
```

Here we have an `uploader` property of type `FileUploader`. This comes from the `ng2-file-upload` module and lets us handle file upload.

A string `feedback` property and a `newPhone` complete the class properties for this component.

In the `ngOnInit` life cycle hook we register 2 callbacks to get notified in case of upload success or error.

:::info
`onSuccessItem`, `onErrorItem` and `onBuildItemForm` are methods defined by the library we're using. Most library prefer to expose Observables to register to events, but this one instead want us to define callbacks.
:::

In the `submit` method we use the `uploader` service to add form data to the request, and `uploadAll()` will trigger the outgoing request.

As said before, we prefer to `JSON.stringify()` the specs array and send it as a string.

### Wrap up

Now we're all set and it's time to test the full stack.

Open the terminal and start, in 2 different windows, both the server (the express app) and client (the angular app) :

```
// terminal 1
$ npm run dev

// terminal 2
$ ng serve
```

If everything is fine, you should be able to browse [http://localhost:4200](http://localhost:4200) and see the app:

![](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_48f3cfc24069644f653e718f5dfea8ba.gif)

**NOTE**: we added some basic styling to make the UI prettier, it's not required for this lesson.
> Focus on features, beauty is nothing without them.

## Summary

In this lesson we have learnt how to create an Express API using multer to upload files. We have also learnt how to upload a file from an Angular app.
