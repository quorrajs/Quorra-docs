# Controllers

 - [Basic Controllers](#basic-controllers)
 - [Controller Filters](#controller-filters)
 - [Implicit Controllers](#implicit-controllers)
 - [RESTful Resource Controllers](#restful-resource-controllers)
 - [Handling Missing Methods](#handling-missing-methods)

## Basic Controllers

Instead of defining all of your route-level logic in a single routes.js file, you may wish to organize this behavior
 using Controllers. Controllers can group related route logic into a single file.

Controllers are typically stored in the app/controllers directory and loaded to the application based on the routes
declared in the routes.js. However, controllers can technically live in any directory or any sub-directory. Route
declarations are dependent on the location of the controller folder on disk. Default namespace for controllers is
`controllers`. So if you wish to add add controller file in any other path you have to provide namespace
relative to `app` directory.

Here is an example of a basic controller file:

```javascript
var BaseController = require('../BaseController');

var UserController = BaseController.extend(function(){

    /**
     * Show the profile for the given user.
     */
    this.ShowProfile = function(req, res, id)
    {
        var user = User.findOne(id);

        res.view('profile/user', {user: user});
    };
});
```

All controllers should extend the BaseController. The BaseController is also stored in the app/controllers directory,
 and may be used as a place to put shared controller logic. The BaseController extends the framework's Controller.
 Now, we can route to this controller action like so:

```javascript
Route.get('user/{id}', 'UserController@showProfile');
```

If you choose to nest or organize your controller using sub-directories, simply use the fully qualified path to the
controller relative to your `app` directory when defining the route:

```javascript
Route.get('foo', 'controllers/Namespace/FooController@method');
```

You may also specify names on controller routes:

```javascript
Route.get('foo', {'uses': 'FooController@method', 'as': 'name'});
```

To generate a URL to a controller action, you may use the URL.action method:

```javascript
var URL = App.url;
$url = URL.action('FooController@method');
```

## Controller Filters

Filters may be specified on controller routes similar to "regular" routes:

```javascript
Route.get('profile', {before': 'auth', 'uses': 'UserController@showProfile'});
```

However, you may also specify filters from within your controller:

```javascript
var BaseController = require('../BaseController');

var UserController = BaseController.extend(function(){

    this.beforeFilter('auth', {'except': 'getLogin'});

    this.beforeFilter('csrf', {'on': 'post'});

    this.beforeFilter('log', {'only':  ['fooAction', 'barAction']));
});
```

You may also specify controller filters inline using a Closure:

```javascript
var BaseController = require('../BaseController');

var UserController = BaseController.extend(function(){

    this.beforeFilter(function(req, res, next){
        //
    });
});
```

If you would like to use another method on the controller as a filter, you may use @ syntax to define the filter:

```javascript
var BaseController = require('../BaseController');

var UserController = BaseController.extend(function(){

        this.FilterRequests = function(req, res, next){
            //
        };

        this.beforeFilter('@filterRequests');
});
```

## Implicit Controllers

Quorra allows you to easily define a single route to handle every action in a controller. First, define the route using
the Route.controller method:

Route.controller('users', 'UserController');
The controller method accepts two arguments. The first is the base URI the controller handles, while the second is the class name of the controller. Next, just add methods to your controller, prefixed with the HTTP verb they respond to:

```javascript
var BaseController = require('../BaseController');

var UserController = BaseController.extend(function(){

        this.getIndex = function(req, res, next) {
            //
        }

        this.postProfile = function(req, res, next) {
            //
        }

        this.anyLogin = function(req, res, next) {
            //
        }
});
```

The index methods will respond to the root URI handled by the controller, which, in this case, is users.

If your controller action contains multiple words, you may access the action using "dash" syntax in the URI. For example, the following controller action on our UserController would respond to the users/admin-profile URI:

```javascript
    this.getAdminProfile = function(req, res, next) {}
```

## RESTful Resource Controllers

Resource controllers make it easier to build RESTful controllers around resources. For example, you may wish to create a controller that manages "photos" stored by your application.

To register a resourceful route to the controller:

Route.resource('photo', 'PhotoController');
This single route declaration creates multiple routes to handle a variety of RESTful actions on the photo resource.

Actions Handled By Resource Controller

| Verb      | Path                      | Action  | Route Name       |
|-----------|---------------------------|---------|------------------|
| GET       | /resource                 | index   | resource.index   |
| GET       | /resource/create          | create  | resource.create  |
| POST      | /resource                 | store   | resource.store   |
| GET       | /resource/{resource}      | show    | resource.show    |
| GET       | /resource/{resource}/edit | edit    | resource.edit    |
| PUT/PATCH | /resource/{resource}      | update  | resource.update  |
| DELETE    | /resource/{resource}      | destroy | resource.destroy |

Sometimes you may only need to handle a subset of the resource actions, then you may the subset of actions to handle
on the route:

```javascript
Route.resource('photo', 'PhotoController', {'only': ['index', 'show']});

Route.resource('photo', 'PhotoController', {'except': ['create', 'store', 'update', 'destroy']});
```
By default, all resource controller actions have a route name; however, you can override these names by passing a
names object with your options:

```javascript
Route::resource('photo', 'PhotoController', {'names': {'create': 'photo.build'}});
```

You can prefix all resource route names with your own prefix by using `as` attribute:

```javascript
Route::resource('photo', 'PhotoController', {'as': 'prefix'});
```

Handling Nested Resource Controllers

To "nest" resource controllers, use "dot" notation in your route declaration:

```javascript
Route.resource('photos.comments', 'PhotoCommentController');
```

This route will register a "nested" resource that may be accessed with URLs like the following:

photos/{photoResource}/comments/{commentResource}.

```javascript
var BaseController = require('../BaseController');

var PhotoCommentController = BaseController.extend(function(){

        this.show = function(req, res, photoId, commentId) {
            //
        }
});
```

Adding Additional Routes To Resource Controllers

If it becomes necessary for you to add additional routes to a resource controller beyond the default resource routes,
 you should define those routes before your call to Route.resource:

```javascript
Route.get('photos/popular', 'PhotoController@method');
Route.resource('photos', 'PhotoController');
```

## Handling Missing Methods

When using Route.controller, a catch-all method may be defined which will be called when no other matching method is
found on a given controller. The method should be named missingMethod, and receives the method and parameter array for the request:

Defining A Catch-All Method

```javascript
this.missingMethod = function(parameters)
{
    //
}
```