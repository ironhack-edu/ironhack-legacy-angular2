![Ironhack Logo](https://i.imgur.com/1QgrNNw.png)

# Deployment

## Learning Goals
After this lesson, you will be able to:

* Understand the difference between development, staging and production.
* Bundle an application with the angular-cli.
* Deploy an Angular application online.

## Workflow

Deployment should be treated as part of the development workflow, not as an afterthought.

When developing a web site or an application, the deployment workflow should be split into 3 different environments: Development, Staging and Production.

- **Development** is where developers test their work locally. Sometimes for large applications, this will be on a remote server such as Heroku. This is a sandbox, something for developers to play around with without doing any serious damage.
- **Staging** - Staging is a test environment that mocks the real production environment. This is on a remote server such as Heroku or AWS. It is meant to mimic the production environment so you can catch any production related bugs before the app is deployed.
- **Production** - This is the real deal, the website that your users will actually see.

![](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_1f71ba7160030f430656f0ad3ced22e7.png)

In this case the workflow might look like this:

- Developers work on bugs and features on their own separate git branches. Tiny updates can be added directly to the stable development branch.
- Once features are implemented, they are merged into the staging branch and deployed to the Staging environment for quality assurance and testing.
- After testing is complete, feature branches are merged into the development branch.
- On the release date, the development branch is merged into production and then deployed to the Production environment.

## Introduction

Depending on the type of application you have and what it does, you can choose different services to host it. For example, a client application such as a portfolio doesn't really need a custom backend. It just needs a server to host the static files.

<div style="margin: 50px auto; min-height: 100px;">

<img style="float: left;" src="https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_f861c1f3073e13eafd52ad1704427e25.png">

</div>

For this scenario [GitHub pages](https://pages.github.com/) is perfect, it lets us push our code directly to a server using git repositories. Angular-cli has a built-in command to deploy to gh-pages.



<div style="margin: 50px auto; min-height: 100px;">


<img style="float: left" width="350" src="https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_788898ea19304c521e348bc2755db363.png">

</div>


On the other hand, our `phone-app` is a little more complex; we have our own node backend exposing our APIs. GitHub pages won't let us change the backend (not the purpose of the platform) so we need a different service, we'll use [Heroku](https://www.heroku.com/).

### Heroku

In this learning unit we are going to deploy our *Phone store* fullstack application. We have the express server, connected to a mongo database, and an Angular client.

#### Prerequisites

We should have:

- A Heroku account created
- The [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli) installed

## Bundle the Angular application

![](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_c6234ee95f4599da6df0b1290ac8769c.png =300x)

Angular-CLI applications are easy to bundle, the CLI comes with a built-in command to pack everything into a few optimized files.

We've already used this command in the previous lessons, but let's take a look at what it actually does:

```
$ ng build --prod --aot
```

With this command:

1. All the files in the Angular application are minified and all spaces and line breaks are removed, making the overall size of the files much smaller.
2. [Tree shaking](https://webpack.js.org/guides/tree-shaking/) occurs.
3. The code in these chunks is "uglifyied" ([obfuscated](https://en.wikipedia.org/wiki/Obfuscation_(software)) and optimized).
4. A new `index.html` is created replacing the old references with the new files.
5. The boostrap process is provided with different parameters to enable production mode (disable runtime debug).
6. The `aot`(short for *ahead of time* compilation) command perform also a [further compilation step](https://en.wikipedia.org/wiki/Ahead-of-time_compilation) which transforms the source into an ultra-optimized, vm friendly format.

:::info
<img src="https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_688ed1fc9b367e0c8328693c82a85e3a.gif" width="100" style="float: left; margin-right: 20px" />

**Tree shaking**:
Normally if you require a module you import the whole thing. ES6 modules allow you to just import only the pieces you need, without mucking around with custom builds. Tree shaking removes unused code reducing the final bundle size.
:::

The build step will produce a new directory containing the optimized code. In the `angular-cli.json` file we can configure the `outDir` property to set the destination directory, by default it will be called `dist` (distribution).

Let's run the command in the `phone-app` project directory and then copy the resulting folder into the server side (express) `/public` directory.

## Heroku App

First we need to create a new Heroku app for our application:

1. From your dashboard, add a new app and give it a meaningful name

![](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_54caeef6131a10f304e6ab7cf30a184e.png =650x)

2. Now that we created our app, we can select the *Deploy* tab and follow the instructions. We just need to log in into Heroku, set up the remote and push the code to deploy as it is a git repository.

Before deploying, let's configure our remote database.

### MLab configuration

We are going to use the [MLab](https://mlab.com/) DaaS (Database-as-a-Service) to persist our data in the cloud.

The first thing to do is install the [MLab add-on](https://elements.heroku.com/addons/mongolab) for Heroku.

From the terminal, inside the express folder:

```bash
$ heroku addons:create mongolab:sandbox
```

In our Heroku dashboard, under the *Resources* tab we should see the addon successfully installed:

![](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_1c5e92183137f22a714b6599e305bcbd.png)

:::warning
:exclamation: If your heroku account doesn't have a valid credit card associated with it you will be prompted to add one. No worries though, they won't actually charge you for anything.
:::

Next, run:

```
$ heroku config:get MONGODB_URI
```

We should be able to see our brand new database's connection string.

We know we **must never publish any sensitive data** (such as database connection, passwords, or other personal information) within our applications. To enable the database connection we should use *Environment variables* instead.

### Backend Environment Variables

The express project folder is almost ready for deploy. We just need to configure the database connection and have it pointing to the [MLab](https://mlab.com/) service we will use with Heroku.

#### Local setup

First let's define an `environment` configuration file containing our database URI.

Create a `.env` file and paste the db connection string:

```text
MONGODB_URI = mongodb://localhost/phone-store
```

Now in the `config/database.js` file we need to read the environment:

```javascript=
require("dotenv").config();
```

And change the current connection to:

```javascript=10
mongoose.connect(process.env.MONGODB_URI);
```

If we start the server with `npm start`, our application should work normally, pointing to the local database.

So, we have defined the variable in our local project... what happens with Heroku now? Let's configure it to manage our environment variables.

#### Heroku setup

Environment variables on Heroku are pretty easy to manage. We can find them in our application dashboard, under the *Settings* tab. In the section called "Config Variables" you can expand our environment variables. There you should find the following:

![](https://s3-eu-west-1.amazonaws.com/ih-materials/uploads/upload_4553ed8d650838a7857da58e7950ac49.png)

Our environment variable is already defined! This is because when we installed the MongoLab add-on, it defined the variable in our project.

### Deploy

We are finally ready to deploy our fullstack application.

From the terminal run

```shell
$ gi add .
$ git commit -m'message'
$ git push heroku master
```

and you should see:

```shell
...
remote:        Installing any new modules (package.json)
remote:
remote: -----> Caching build
remote:        Clearing previous node cache
remote:        Saving 2 cacheDirectories (default):
remote:        - node_modules
remote:        - bower_components (nothing to cache)
remote:
remote: -----> Build succeeded!
remote: -----> Discovering process types
remote:        Procfile declares types     -> (none)
remote:        Default types for buildpack -> web
remote:
remote: -----> Compressing...
remote:        Done: 19.4M
remote: -----> Launching...
remote:        Released v4
remote:        https://phone-store-app.herokuapp.com/ deployed to Heroku
remote:
remote: Verifying deploy... done.
To https://git.heroku.com/phone-store-app.git
   42cb230..5ad9dae  master -> master
```

You can see from the log the URL where the app has been published.

Just open the browser and see the app working!

## Github Pages | Angular 2 Only, No Backend (Extra)

If you want to host an Angular app without hosting your own API in the same location, you can use GitHub pages.

GitHub pages is a GitHub feature that allows you to host a static website or web app for free, and it’s as simple as putting the files in a gh-pages branch of your project’s repository. The Angular CLI makes it even easier to deploy to Github pages.

If your project repository is already hosted on GitHub, a deploy is as simple as:

```
ng github-pages:deploy
```

With this you'll have the app running at `https://<username>.github.io/<repository-name>`

*Super Easy*

## Summary

In this lesson we have learnt how to differentiate between development, staging and production environments. We have seen how to bundle an application with the angular-cli, and finally how to deploy an Angular application online.

## References

- [GitHub pages](https://pages.github.com/)
- [Heroku](https://dashboard.heroku.com/)
- [MLab](https://mlab.com/)
