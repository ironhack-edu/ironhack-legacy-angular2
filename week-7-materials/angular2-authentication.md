![Ironhack Logo](https://i.imgur.com/1QgrNNw.png)

# Authentication

## Learning Goals

After this lesson, you will be able to:

* Authenticate to an Express server application using Angular.
* Perform authenticated requests to obtain private resources.

## Setting up the server

### Express scaffolding

As usual, we will create a new Express application using the generator:

```
$ express --git --view=ejs angular-auth
```

Then we will move into the project folder and add some dependencies:

```
$ cd angular-auth
$ npm install --save mongoose
$ npm install --save-dev nodemon
```

Finally, we will configure the `package.json` to start the app with nodemon:

```javascript
// ...
"scripts": {
  "start": "node ./bin/www",
  "mon": "nodemon ./bin/www"
}
//...
```

### Define the user model

Create a new file in `models/user-model.js` to define the structure of our user resource:

```javascript
const mongoose = require('mongoose');
const Schema   = mongoose.Schema;

const userSchema = new Schema({
  username: String,
  password: String
}, {
  timestamps: {
    createdAt: 'created_at',
    updatedAt: 'updated_at'
  }
});

const User = mongoose.model('User', userSchema);
module.exports = User;
```

Nothing new so far, we use `mongoose` to abstract MongoDB operations. Our users will have `username` and `password` fields, along with some useful timestamps.

### Configure Passport

Passport needs a little bit of configuration to know how to serialize and deserialize users and which strategy to use.

We're going to implement a simple `username`/`password` login mechanism. To do so, Passport provides the `passport-local` module.

Install the module and `bcrypt` with:

```
$ npm install --save passport-local bcrypt
```

Then create a new configuration file in `config/passport.js`:

```javascript
const LocalStrategy = require('passport-local').Strategy;
const User          = require('../models/user-model');
const bcrypt        = require('bcrypt');

module.exports = function (passport) {

  passport.use(new LocalStrategy((username, password, next) => {
    User.findOne({ username })
    .then(foundUser => {
      if (!foundUser) {
        next(null, false, { message: 'Incorrect username' });
        return;
      }

      if (!bcrypt.compareSync(password, foundUser.password)) {
        next(null, false, { message: 'Incorrect password' });
        return;
      }

      next(null, foundUser);
    })
    .catch(error => cb(error))

  }));

  passport.serializeUser((loggedInUser, cb) => {
    cb(null, loggedInUser._id);
  });

  passport.deserializeUser((userIdFromSession, cb) => {
    User.findById(userIdFromSession)
    .then(userDocument => {
      cb(null, userDocument);
    })
    .catch(error => cb(error))
  });

}
```

Here we import and use the `passport-local` strategy and tell passport how to find and save our users when checking the authentication. This code comes from the [Passport documentation](http://passportjs.org/docs/username-password). Refer to it for other implementations.

### Define our routes

We want to provide basic authentication features. Along with login and logout methods, we want to expose a way for the client to know if the user is logged in.

We will define this API:

| Method | URL | Parameters | Description |
|:--------:|:-----:|:------------:|-------------|
|  POST  | `/signup` | username, password | Create a new user if the username doesn't exist in the db|
|  POST  | `/login` | username, password | Create a new session and return the user resource|
|  POST  | `/logout` | - | Delete the current session |
|  GET  | `/loggedin` | - | Check for a session and return the user resource |
|  GET  | `/private` | - | Check for a session and return some private data |


Notice that we don't need routes for the _Sign Up_ and _Log In_ forms. Those will be on the Angular side!

We're going to define these routes in a new routes file called `authRoutes`.
Let's add a new file in `routes/auth-routes.js` folder and import the modules we need:

```javascript
const express    = require('express');
const passport   = require('passport');
const bcrypt     = require('bcrypt');

// Our user model
const User       = require('../models/user-model');

const authRoutes = express.Router();
```

After that we can start implementing our routes.

1. The **Sign Up** POST route checks for a user with the same username. If a user does not exist, it creates a new one:

```javascript
authRoutes.post('/signup', (req, res, next) => {
  const username = req.body.username;
  const password = req.body.password;

  if (!username || !password) {
    res.status(400).json({ message: 'Provide username and password' });
    return;
  }

  User.findOne({ username }, '_id')
  .then(foundUser => {
    if (foundUser) {
      res.status(400).json({ message: 'The username already exists' });
      return;
    }

    const salt     = bcrypt.genSaltSync(10);
    const hashPass = bcrypt.hashSync(password, salt);

    const theUser = new User({
      username,
      password: hashPass
    });

    theUser.save()
    .then(user => {
      req.login(theUser, (err) => {
        if (err) {
          res.status(500).json({ message: 'Something went wrong' });
          return;
        }

        res.status(200).json(req.user);
      });
    })
    .catch(error => next(error))
  })
  .catch(error => next(error))
});
```

2. The **Log In** route uses the passport middleware to authenticate the user. If an error happens or the user is not found it returns an error:

```javascript
authRoutes.post('/login', (req, res, next) => {
  passport.authenticate('local', (err, theUser, failureDetails) => {
    if (err) {
      res.status(500).json({ message: 'Something went wrong' });
      return;
    }

    if (!theUser) {
      res.status(401).json(failureDetails);
      return;
    }

    req.login(theUser, (err) => {
      if (err) {
        res.status(500).json({ message: 'Something went wrong' });
        return;
      }

      // We are now logged in (notice req.user)
      res.status(200).json(req.user);
    });
  })(req, res, next);
});
```

3. The **Log Out** route logs the user out and returns a success message:

```javascript
authRoutes.post('/logout', (req, res, next) => {
  req.logout();
  res.status(200).json({ message: 'Success' });
});
```

4. The **Logged In** route's purpose is to let the user know if they are logged in or not. The Passport function `isAuthenticated` comes into play here. If the result is `true` the API returns the user, otherwise it returns an `Unauthorized` message:

```javascript
authRoutes.get('/loggedin', (req, res, next) => {
  if (req.isAuthenticated()) {
    res.status(200).json(req.user);
    return;
  }

  res.status(403).json({ message: 'Unauthorized' });
});
```

5. The **Private** route looks a lot like the `loggedin` routes. The difference lies in the response content, in this case, a secret message:

```javascript
authRoutes.get('/private', (req, res, next) => {
  if (req.isAuthenticated()) {
    res.json({ message: 'This is a private message' });
    return;
  }

  res.status(403).json({ message: 'Unauthorized' });
});
```

Don't forget to export the router so you can use it in the `app.js` file.

```javascript
module.exports = authRoutes;
```

### Express configuration

We're almost done with the backend application. We just need to make sure Passport is initialized and the session is stored.

For the latter to happen we use the `express-session` module. Install it with:

```
$ npm install --save express-session
```

Then in `app.js` import the following:

```javascript
...
const authRoutes = require('./routes/auth-routes');
const session    = require('express-session');
const passport   = require('passport');
...
```

After that we need to configure `mongoose`:

```javascript
const mongoose = require('mongoose');
mongoose.connect('mongodb://localhost/angular-auth');
```

Require passport's configuration:

```javascript
const passportSetup = require('./config/passport');
passportSetup(passport);
```

Configure the session:

```javascript
app.use(session({
  secret: 'angular auth passport secret shh',
  resave: true,
  saveUninitialized: true,
  cookie : { httpOnly: true, maxAge: 2419200000 }
}));
```

Initialize auth modules:
```javascript
app.use(passport.initialize());
app.use(passport.session());
```

Import the authRoutes:

```javascript
app.use('/', authRoutes);
```

Finally, since we know we're going to use a client application, we want to configure the client public `index.html` as a fallback page:

```javascript
app.use((req, res, next) => {
  res.sendfile(__dirname + '/public/index.html');
});
```

## Angular app

The client will be much more straightforward. Most of the logic was implemented in the backend.

Let's create a new app using the `angular-cli`:

```
$ ng new auth-app
```

:::info
Since we're using the `angular-cli` our client app will be served by a different Express server (the one running on port 4200). This means we won't be able to perform requests to a different domain (our express server running on port 3000) because of [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS) restrictions.

For the purpose of this lesson we'll build the client app as usual, then bundle it and move it in our Express server's public folder. Only at that point will we be able to see it working.
:::

### Authentication service

We already know that in Angular all AJAX requests should be placed in a service. We can start from here to create our functionalities.

Run `$ ng g service session` to create a new service.

```typescript
import { Injectable } from '@angular/core';
import { Http, Response } from '@angular/http';
import 'rxjs/add/operator/map';
import 'rxjs/add/operator/catch';
import { Observable } from 'rxjs/Rx';

@Injectable()
export class SessionService {

  constructor(private http: Http) { }

  handleError(e) {
    return Observable.throw(e.json().message);
  }

  signup(user) {
    return this.http.post(`/signup`, user)
      .map(res => res.json())
      .catch(this.handleError);
  }

  login(user) {
    return this.http.post(`/login`, user)
      .map(res => res.json())
      .catch(this.handleError);
  }

  logout() {
    return this.http.post(`/logout`, {})
      .map(res => res.json())
      .catch(this.handleError);
  }

  isLoggedIn() {
    return this.http.get(`/loggedin`)
      .map(res => res.json())
      .catch(this.handleError);
  }

  getPrivateData() {
    return this.http.get(`/private`)
      .map(res => res.json())
      .catch(this.handleError);
  }
}
```

Here we injected the `Http` service to perform AJAX request.

We've defined all our outgoing requests, catching errors and delegating them to an `handleError` method.

Don't forget to register the `SessionService` in the `AppModule` in order to use it:

```typescript=6
import { SessionService } from "./session.service";
```

```typescript=17
providers: [ SessionService ],
```

### Login form

To keep the app simple we're going to implement the authentication logic in our `AppComponent`.

Let's start from our template in `app.component.html`:

```htmlmixed
<h1> Authentication Sample </h1>
<form>
  <h2> Login or Signup </h2>
  <label> Username </label>
  <input type="text" [(ngModel)]="formInfo.username" name="username"/>
  <br>
  <label> Password </label>
  <input type="password" [(ngModel)]="formInfo.password" name="password"/>

  <button (click)="login()"> login </button>
  <button (click)="signup()"> signup </button>
</form>
<p class="error"> {{ error }} </p>
```

We added a simple form to login or signup. We have two event bindings on the buttons, so let's implement those methods. In `app.component.ts`:

```typescript
...
export class AppComponent {

  formInfo = {
    username: '',
    password: ''
  };

  user: any;
  error: string;

  constructor(private session: SessionService) { }

  login() {
    this.session.login(this.formInfo)
      .subscribe(
        (user) => this.user = user,
        (err) => this.error = err
      );
  }

  signup() {
    this.session.signup(this.formInfo)
      .subscribe(
        (user) => this.user = user,
        (err) => this.error = err
      );
  }
}
```

We use the `formInfo` class property to hold the form data (inserted by the user). We also have `user` and `error` properties to hold the results of the requests.

The `login` and `signup` methods delegate the `SessionService` and subscribe to the *Observable* response coming back.


If you try to click on the buttons you should get the common CORS error:

:::danger
*XMLHttpRequest cannot load http://localhost.com:3000/login. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://localhost.com:4200' is therefore not allowed access. The response had HTTP status code 403.*
:::

### Authenticated requests

Let's say the user is able to login, our `user` class property is then assigned. Once we have it we can display it and show a different content:

```htmlmixed=15
<!-- app.component.html -->
<div *ngIf="user">
  <h2> You are now logged in as {{ user.username }}!! </h2>
  <p> Here's the user object </p>
  <pre> {{ user | json }} </pre>

  <p *ngIf="privateData"> Here's some data fetched from a protected API </p>
  <pre> {{ privateData | json }} </pre>

  <button (click)="logout()"> logout </button>
  <button (click)="getPrivateData()"> Get private data </button>
</div>
```

Once logged in we show the user object (as a JSON for debug purpose) and the `logout` / `getPrivateData` buttons.

The 2 new methods are pretty much similar to the login one:

```typescript
// app.component.ts

logout() {
  this.session.logout()
    .subscribe(
      () => this.user = null,
      (err) => this.error = err
    );
}

getPrivateData() {
  this.session.getPrivateData()
    .subscribe(
      (data) => this.privateData = data,
      (err) => this.error = err
    );
}
```

### Wrap up the client

Here's the final `app.component.html`:

```htmlmixed
<h1> Authentication Sample </h1>
<form *ngIf="!user">
  <h2> Login or Signup </h2>
  <label> Username </label>
  <input type="text" [(ngModel)]="formInfo.username" name="username"/>
  <br>
  <label> Password </label>
  <input type="password" [(ngModel)]="formInfo.password" name="password"/>

  <button (click)="login()"> login </button>
  <button (click)="signup()"> signup </button>
</form>

<div *ngIf="user">
  <h2> You are now logged in as {{ user.username }}!! </h2>
  <p> Here's the user object </p>
  <pre> {{ user | json }} </pre>

  <p *ngIf="privateData"> Here's some data fetched from a protected API </p>
  <pre> {{ privateData | json }} </pre>
  <button (click)="logout()"> logout </button>
  <button (click)="getPrivateData()"> Get private data </button>
</div>

<p class="error"> {{ error }} </p>
```

and `app.component.ts`:

```typescript
import { Component, OnInit } from '@angular/core';
import { SessionService } from './session.service';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {
  user: any;
  formInfo = {
    username: '',
    password: ''
  };
  error: string;
  privateData: any = '';

  constructor(private session: SessionService) { }

  ngOnInit() {
    this.session.isLoggedIn()
      .subscribe(
        (user) => this.successCb(user)
      );
  }

  login() {
    this.session.login(this.formInfo)
      .subscribe(
        (user) => this.successCb(user),
        (err) => this.errorCb(err)
      );
  }

  signup() {
    this.session.signup(this.formInfo)
      .subscribe(
        (user) => this.successCb(user),
        (err) => this.errorCb(err)
      );
  }

  logout() {
    this.session.logout()
      .subscribe(
        () => this.successCb(null),
        (err) => this.errorCb(err)
      );
  }

  getPrivateData() {
    this.session.getPrivateData()
      .subscribe(
        (data) => this.privateData = data,
        (err) => this.error = err
      );
  }

  errorCb(err) {
    this.error = err;
    this.user = null;
  }

  successCb(user) {
    this.user = user;
    this.error = null;
  }
}
```

We  refactored the code a little bit to make it cleaner and implemented the `ngOnInit` lifecycle hook to automatically check if the user is already logged in at startup.

## Final result

Now we're ready to test our whole application. Let's bundle the client. From the `auth-app` folder run:

```
$ ng build --prod
```

This will concatenate and optimize our source code into some small chunks. Don't forget to import `FormsModule` and `HttpModule` in the `app.module.ts` file if you haven't yet.
After the process is complete, we can move all the files in the `/dist` folder into the `public` folder of our Express server application's folder. If necessary, start the `mongod` service from your terminal so the database is functional.

Now we can `$ npm start` from inside the Express application.

If everything is fine, the result is visible at `localhost:3000` and should look like this:

![](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_4b49ae176e5921488d177d915f75ce51.gif)

## Summary

In this lesson we have learned how to authenticate to an Express application using Angular, how to perform authenticated requests to obtain private resources, and how to maintain state through page refreshes.
