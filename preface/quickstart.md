# Quorra Quickstart

 - [Installation](#installation)
 - [Routing](#routing)
 - [Creating A View](#creating-a-view)
 - [Waterline ORM](#waterline-ormodm)
 - [Displaying Data](#displaying-data)

## Installation

First, download the Quorra cli using npm.

```
npm install -g quorra-cli
```

Once installed, the simple `quorra new` command will create a fresh Quorra installation in the directory you specify.
For instance, `quorra new blog` would create a directory named `blog` containing a fresh Quorra installation with all
dependencies installed.

### Permissions

Quorra may require one set of permissions to be configured: folders within `app/storage` require write access by the web
server.

### Serving Quorra

You can use the `ride` Quorra-cli command to serve Quorra applicaton:

```
quorra ride
```
By default the HTTP-server will listen to port 3000. However if that port is already in use or you wish to serve
multiple applications this way, you might want to specify what port to use. Just add the --port argument:

```
quorra ride --port 8080
```

You can configure your default Quorra port in `app/config/app.js`

### Directory Structure

After installing the framework, take a glance around the project to familiarize yourself with the directory structure.
The app directory contains folders such as controllers, and models. Most of your application's code will reside
somewhere in this directory. You may also wish to explore the `app/config` directory and the configuration options that
are available to you.

## Routing

To get started, let's create our first route. In Quorra, the simplest route is a route to a Closure. Pop open the
`app/routes.js` file and add the following route to the bottom of the file:

```javascript
Route.get('users', function(req, res)
{
     res.send('Users!');
});
```

Now, if you hit the `/users` route in your web browser, you should see `Users!` displayed as the response. Great! You've
 just created your first route.

Routes can also be attached to controllers. For example:

```javascript
Route.get('users', 'UserController@getIndex');
```

This route informs the framework that requests to the `/users` route should call the `getIndex` method on the
`UserController`. For more information on controller routing, check out the [controller documentation](/docs/{{version}}/getting-started/controllers.md).

## Creating A View

Next, we'll create a simple view to display our user data. Views live in the `resources/views` directory and contain
the HTML of your application. We're going to place two new views in this directory: `layout.pug` and `users.pug`
First, let's create our layout.pug file:

```
doctype html
html
  head
    title='Quorra Quickstart'
  body
    block content
```

Next, we'll create our `users.pug` view:

```
extends layout

block content
  p Users!
```

Here we use [Pug](https://pugjs.org/) templating system, Quorra comes pre-installed with Pug
template engine. You can configure template engines for your application in `app/config/view.js`. Quorra uses module
[consolidate](https://github.com/tj/consolidate.js), a template engine consolidation library and supports all the
template engines [supported by the consolidate library](https://github.com/tj/consolidate
.js#supported-template-engines).

Now that we have our views, let's send it from our /users route as the response. Instead of sending `Users!` from the
route, return the view instead:

```javascript
Route.get('users', function(req, res)
{
    res.view('users');
});
```

Wonderful! Now you have setup a simple view that extends a layout. Next, let's start working on our database layer.

## Waterline ORM/ODM

Quorra makes connecting with databases and running queries extremely simple. Quorra comes installed with a powerful
ORM/ODM called [Waterline](https://github.com/balderdashy/waterline), a datastore-agnostic tool that dramatically
simplifies interaction with one or more databases.

First, let's define a model. An waterline model can be used to query an associated database table. Don't worry, it
will all make sense soon! Models are typically stored in the `app/models` directory. Let's define a `User.js` model in
that directory like so:

```javascript
// path: app/models/user.js
var User = {
    attributes: {
       ...
    }
};

module.exports = User;
```

Note that we do not have to tell Waterline which table to use.

Using your preferred database administration tool, insert a few rows into your user table, and we'll use Waterline to
retrieve them and pass them to our view.

Now let's modify our `/users` route to look like this:

```javascript
Route.get('users', function(req, res)
{
    User.find(function (err, users) {
        res.view('users', {users: users});
    });
});
```

Let's walk through this route. First, the all method on the User model will retrieve all of the rows in the users
table. Next, we're passing these records to the view via view method along with the view name. Now the `users` data
is available to the view.

Awesome. Now we're ready to display the users in our view!

## Displaying Data

Now that we have made the users available to our view, we can display them like so:

```
extends layout

block content
 each val in users
    p= val.name
```

Now, you should be able to hit the `/users` route and see the names of your users displayed in the response.

This is just the beginning. In this tutorial, you've seen the very basics of Quorra, but there are so many more
exciting things to learn. Keep reading through the documentation and dig deeper into the powerful features of Quorra
framework.
