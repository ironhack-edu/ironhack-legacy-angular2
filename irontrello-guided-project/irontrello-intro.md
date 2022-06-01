<!-- ![](https://i.imgur.com/1QgrNNw.png)

# Irontrello | Project Introduction -->

## Learning Goals

After this lesson you will be able to:

- Understand the scope of the guided project we're going to build.
- Understand the process of building and planning a MEAN app.

## Introduction

### Trello

![](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_fcdb0427f3f5d65df64de7df182a4e65.png)

[Trello](https://trello.com/) is one of the most popular tools used nowadays in Project Management. Its goal is to help teams organize all the tasks and features they have to implement during project execution.

In this exercise, we will build a MEAN application clone of Trello! We will not implement all the features here. Our goal is to create a basic MVP clone of Trello with some of its core features. We will call it **IronTrello**.

:::info
:bulb: Trello is a *huge* application. We've divided this codealong into 4 lessons, and provided significant starter code in which we will fill in the gaps.

The main objective of this codealong is to demonstrate how a real world, large app is put together while reviewing all of the topics we've covered over the 3 modules.
:::

As with previous codealongs and projects, we've discovered that it's a bad idea to jump straight into the code. Before we start coding, let's talk about some core **business logic** and data modeling to see what's important.

### Pseudo Data Modeling

You can check out the official [Trello Guide](https://trello.com/guide/board_basics.html) to cover the different features that the application implements. We are going to cover the lists and the cards in depth, and the boards at a surface level.

#### Boards

![](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_d4192ebb5483a1829d9db426f04602f9.png)


In Trello, we can create different boards to organize different projects. **We won't create different boards, instead we will have just one board where we will organize our lists and cards.**

#### List

![](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_7a5096eaf1c5f3aa8a9646086ab720cd.png)

Lists contain the different cards you have organized. Each list usually represents a stage of your project:

- **Backlog** - List of proposed features and bugs. Anyone can add to this.
- **Next up** - Tickets prioritized to be worked on next
- **Ongoing** - Tickets being worked on by someone
- **Done** - Tickets that have been completed, but not looked over
- **Reviewed** - Tickets that have been completed, and reviewed by another team member.

These are just a sample. People organize their boards in different ways, but for the purpose of this example, this will do.

#### Card

![](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_d238f97e9e18d2c54270d076b880984f.png)



Last, but not least, we have the cards. Each card represents a task, and it has different details we fill out, such as:

- Title
- Description
- Due Date
- List of tasks

![](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_1c82d6e5da36d524445f6e4cafbe64ca.png)

These fields will be ones that we eventually store in the database.

## Strategy

### Introduction

Before we start coding, we should come up with a strategy of how to develop our application. It wouldn't make much sense to start writing the logic for cards without having creating lists first. We need to formalize the process so we don't develop features we don't need.

### Optimistic User Driven Design

A good way to start is by envisioning how a User would use your application, optimistically. This is luckily pretty easy since we already have a [working application](https://trello.com/) to play with.

:::info
:bulb: This process is often times done by a *UX Developer*
:::

#### Example User Flow | Create List

```flow
st=>operation: User Visits Board
op=>operation: User Clicks "Add List"
op1=>operation: User fills a form with list title
op2=>operation: User submits form, and the list is created
op3=>operation: User Clicks "Add Card" on previously created list
op4=>operation: User fills a form for the new card
op5=>operation: User submits form, and the card is created, and displayed on list

st->op->op1->op2->op3->op4->op5
```

We're going to be completing this basic flow for IronTrello.

### Components

There are 3 high level components to build our application:

```
app
├── board
│   ├── board.component.html
│   ├── board.component.scss
│   ├── board.component.ts
├── card
│   ├── card.component.html
│   ├── card.component.scss
│   ├── card.component.ts
├── list
│   ├── list.component.html
│   ├── list.component.scss
│   ├── list.component.ts
```

- **Board** provides the basic layout and fetches the lists.
- **List** will provide the single list template for viewing/editing a list.
- **Card** will provide the single card template for viewing/editing a card.

## Project Setup

We are ready to go! Let's start our project by creating the server side of our project. This will be an express app, with a set of API endpoints.

### Express Project

**Let's start the app!**

We are going to provide you an incomplete repository, that we will add to during the lessons. Please, clone the [IronTrello](https://github.com/ironhack-labs/lab-trello-clone) from our Github.

Once we have cloned the repo, we will need to add mongoose to be able to insert data in our database:

```
$ npm install mongoose
```

After installing, require and configure mongoose's connection to a database named `irontrello`.


Once you are done with that, we are ready to go!

## Summary

In this lesson, we have learned what Trello is and a bit about its main features. We have also scoped our project to know which features we will implement in our project.

Finally, we have created the project we will use to implement our solution, and we have added `mongoose` to be able to use our database, that we have also configured in the project.

In the next lesson, we'll start implementing features on both the front and backend!

## Extra Resources

- [Trello](https://www.trello.com/)
