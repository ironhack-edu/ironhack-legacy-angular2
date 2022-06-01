<!-- ![](https://i.imgur.com/1QgrNNw.png)

# Irontrello | Backend Architecture -->

![IronTrello](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_fcdb0427f3f5d65df64de7df182a4e65.png)

## Learning Goals

After this lesson, we will have:

- Designed back-end architecture of IronTrello.
- Structureed the IronTrello API.
- Implemented the API in our project.

## Setup

### Introduction

As you may know, an Angular application is divided into two different parts: back-end and front-end.

The back-end will connect our front-end with the database through an API, so we have to create the database models  and routes as a base.

Before we get started, let's add some packages and project structure to our application.

### Project structure

#### General Structure

First of all, let's analyze how the project is structured. Remember we are going to work on the `server` side, so ignore the `client` folder for now.

The `server` is structured as follows:

```
server/
├── api
│   ├── card
│   |   ├── card.controller.js
│   |   ├── card.model.js
│   |   └── index.js
│   └── list
│   |   ├── list.controller.js
│   |   ├── list.model.js
│   |   └── index.js
├── bin
│   └── www
├── public
│   └── favicon.ico
├── routes
│   └── index.js
├── .gitignore
├── app.js
└── package.json
```

:::info
:bulb: As we've seen during the course there are many ways to organize an express app. Don't be concerned if you have done or seen something different: in a few months, when you have enough experience working as a programer, you will find the best way **for you** to organize your code :)
:::

We are mainly going to be working in 3 folders: api, public, and routes.

#### `api` folder

In this folder we find two different directories: `card` and `list`. Each one contains all the necessary files to integrate our front-end with our database: the controller and the model.

Pay special attention to the `index.js` file. Here you have the content of the `/api/card/index.js` file:

```javascript
const express = require('express');
const controller = require('./card.controller');

const router = express.Router();

router.post('/', controller.createCard);
router.put('/:id', controller.editCard);
router.put('transfer/:id', controller.transferCard);
router.delete('/:id', controller.removeCard);

module.exports = router;
```

First of all, we require the controller of the model we are using, in this case, the `card` model. Then, we export all the routes from the `card` controller. By doing this, we can require the whole functionality (model and controller) by adding a reference to the folder.

#### `routes` folder

In the routes folder we have a file that exposes all API endpoints through folders you can find inside the `api` folder. How? As we said, the `index.js` file allows us to include both files just by adding a route path, and callback function:

```javascript=5
app.use("/api/list", require("../api/list"));
app.use("/api/card", require("../api/card"));
```

#### `public` folder

In the next step, we will store all of our compiled, tree shaken, optimized, minimized, and obfuscated Angular code in the public folder and serve it as a static asset.

More details on this later.

### Packages

We are going to use two different packages, some of which may be familiar:

- We will use [dotenv](https://www.npmjs.com/package/dotenv) to use environment variables in our code, so that we can avoid pushing private data (AWS keys, MongoDB URIs, other API keys) to our repos.
- [Lodash](https://www.npmjs.com/package/lodash), for array and object utilities.

We can install these packages by running the following commands in the terminal:

```bash
$ npm install dotenv lodash
```

Once you have added the libraries, you have to:

- Require the `dotenv` configuration in the `app.js` file.
- Change the MongoDB URL in the Mongoose configuration to use an environment variable called `MONGODB_URI`.

## API design

Before coding our API, we need to know what we have to do. We are going to clone a couple of Trello features: cards and lists. It's a good practice to create a  routes table with the different endpoints we have, including the method, action, and a description.

### Card

| Method | Action | Description |
|--------|--------|-------------|
| *POST* | `createCard` | It will create a card new inside |
| *PUT* | `editCard` | It will edit the indicated card |
| *PUT* | `transferCard` | It will move a card from one list to another |
| *DELETE* | `removeCard` | It will remove a card from the database |

The `editCard` method will be a bit special in our application, as it can be called in two different situations:

- When we edit a card's information.
- When we change the card's position inside the same list.

### List

| Action | Method | Description |
|--------|--------|-------------|
| *GET* | `getLists` | It will load all the lists from the database |
| *POST* | `createList` | It will create a new list |
| *PUT* | `editList` | It will edit a list  |
| *DELETE* | `removeList` | It will remove a list from the database |

As in the `card`, the `editList` method will be called in two different situations:

- When we edit a list's information.
- When we change the list's position in our board.

Once we have defined the methods we are going to use, it's time to write the code.

## Card Resource

### Introduction

As we have already said, a card is an object that will contain the information of a feature that we have to implement in our project. It's time to decide which information we are going to have in a card.

We are going to create cards with the following information:

- `title` of the feature we will implement.
- `description` of how we are going to implement a feature.
- `dueDate` will indicate the date it has to be done.
- `position` of the card inside a list.
- `list` will be a reference to the List `ObjectId` where we have the card.
- As usual, we will also have `timestamp` fields.

:::success
:question: **Do you think is it enough? Discuss with the class if these are all the properties we should implement or if we should to add/remove some other fields.**
:::

### Card Files

We have already seen in the project folder structure which files are we going to use to implement the Card API features:

```
├── card
│   ├── card.controller.js
│   ├── card.model.js
│   └── index.js
```

We will first go through the model and then the controller to analyze all of the methods to see how they work.

### Model

In our models, the first thing we have to do is to require the `mongoose` Schema to be able to create a new model schema. Once we have required the different files we need, we can create our model. In this case, the Card model in the `api/card/card.model.js` file is the following:

```javascript=6
const CardSchema = new mongoose.Schema({
  title: {
    type: String,
    required: true
  },
  description: String,
  dueDate: Date,
  position: Number,
  list: {
    type: Schema.Types.ObjectId,
    ref: 'List',
    required: true
  }
}, {
  timestamps: {
    createdAt: "created_at",
    updatedAt: "updated_at"
  }
});
```

Nothing new in here, so let's move on to the controller.

### Card Controller

The controller is the result from combining two different files: `card.controller.js` and `index.js`, both inside the `api/card/` folder.

We are going to export route methods from the `card.controller.js` file, and define the route's path in the `index.js` file.

#### Create card

**`card.controller.js`**

```javascript=6
exports.createCard = (req, res, next) => {
  const newCard = new cardModel({
		title: req.body.title,
		description: req.body.description,
		dueDate: req.body.dueDate,
		list: req.body.list,
    position: req.body.position
	});

	newCard.save()
	.then(card => {
		listModel.findByIdAndUpdate(card.list, { $push: {cards: card}}, {new: true})
		.then(list => {
			res.status(200).json(card)
		})
	})
	.catch(error => next(error))
}
```


Once we have defined the method that will implement the creation of a new card, we have to define the route in the `index.js` file:

```javascript=3
var express = require("express");
var controller = require("./card.controller");

var router = express.Router();

router.post('/', controller.createCard);
```

:::success
With the class, analyze the other API methods we have defined for our Trello `card`'s:

- `editCard`.
- `transferCard`.
- `removeCard`.
:::

## List Resource

As we have already seen, a list contains several cards under the same title or *stage*. This stage can represent the status of a card: On Going, To Do, Next Up, etc.

Before coding, we have to decide which fields we are going to store in our database. The full list is the following:

- `title` that will be an string, and will be mandatory.
- `position` will be a number with a default value of 0, to indicate the position of the list in our board.
- `cards` will contain an array of cards. It will have an empty array as a default value, and it will be required.
- As usual, we will define `timestamps` to store the date when a list was created and updated.

### List Files

We have already seen in the project folder structure which files we are going to use to implement the List API features:

```
├── list
│   ├── list.controller.js
│   ├── list.model.js
│   └── index.js
```

We are going to create the database model, then we will go through the controller to implement some of the methods we will need:

### Model

Open the `api/list/list.model.js` file:

```javascript
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

const ListSchema = new mongoose.Schema({
  title: {
    type: String,
    require: true
  },
  position: {
    type: Number,
    default: 0
  },
  cards: [
    { 
      type: Schema.Types.ObjectId, 
      ref: 'Card', 
      require: true
    }
  ],
}, {
  timestamps: {
    createdAt: "created_at",
    updatedAt: "updated_at"
  }
});

module.exports = mongoose.model('List', ListSchema);
```

### Controller

Once we have created our model, it's time to go to our controller. As with the card controller, we have different methods that we will use form the `index.js` to link our routes to the different methods we are going to use in our API.

With the help of your lead instructor, analyze the different methods we have in this controller. Note how we are using promises to load the different information from the database to interact with the API.

Now that we have the model and the controller already implemented, it's time to go back to the `card` controller and finish the code we have there. We have to implement the update method of the cards.

:::success
**Update the card list by pushing the card id in `cards` array field defined in the `list` model.**
:::

Once you have implemented this, we are almost done with the server side of our Trello clone.

## Exposing our routes

To finish up this section, we have to expose the routes we have defined in the different controllers of our API. We will use the `routes/index.js` file to do that:

```javascript=
const path = require('path');

module.exports = (app) => {
  app.use('/api/list', require('../api/list'));
  app.use('/api/card', require('../api/card'));

  // catch 404 and forward to Angular
  app.all('/*', (req, res) => {
    res.sendfile(__dirname + '/public/index.html');
  });
};
```

We are doing two different things here:

- Exposing our routes at their respective paths on lines 4 and 5.
- Configuring the server to serve all the Angular files that we will add in the `public` folder.

## Summary

In this lesson we have learnt how to create the server side of our application. We have created a folder structure that will help us keep all the files in our project organized.

Then, we analyzed which trello features we had to implement, and we have implemented both `card` and `list` endpoints and models.

We have also learnt how an `index.js` file helps us require the files in a folder without having to provide the whole route to the file.

## Extra Resources

- [Trello](https://www.trello.com/)
