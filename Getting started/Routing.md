# Routing

 - [Basic Routing](#basic-routing)
 - [Route Parameters](#route-parameters)
 - [Route Filters](#route-filters)
 - [Named Routes](#named-routes)
 - [Route Groups](#route-groups)
 - [Sub-Domain Routing](#sub-domain-routing)
 - [Route Prefixing](#route-prefixing)
 - [Route Model Binding](#route-model-binding)
 - [Throwing 404 Errors](#throwing-404-errors)
 - [Routing To Controllers](#routing-to-controllers)

## Basic Routing

Most of the routes for your application will be defined in the `app/routes.js` file. The simplest Quorra routes
consist of a URI and a Closure callback.

### Accessing application router instance

```javascript
var Route = require('positron').router;
```

### Basic GET Route

```javascript
Route.get('/', function(req, res)
{
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end('Hello World');
});
```

### Basic POST Route

```javascript
Route.post('foo/bar', function(req, res)
{
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end('Hello World');
});
```

### Registering A Route For Multiple Verbs

```javascript
Route.match(['GET', 'POST'], '/', function(req, res)
{
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end('Hello World');
});
```

### Registering A Route Responding To Any HTTP Verb

```javascript
Route.any('foo', function()
{
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end('Hello World');
});
```


### Forcing A Route To Be Served Over HTTPS

```javascript
Route.get('foo', {'https': true, uses: function(req, res)
{
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end('Must be over HTTPS');
}});
```

Often, you will need to generate URLs to your routes, you may do so using the URL.to method:

```javascript
var URL = require('positron').url;
var url = URL.to('foo');
```

## Route Parameters

```javascript
Route.get('user/{id}', function(req, res, id)
{
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end('User '+id);
});
```

### Optional Route Parameters

```javascript
Route.get('user/{name?}', function(req, res, name)
{
    if(name === undefined) {
        name = 'no name';
    }

    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end(name);
});
```

### Regular Expression Route Constraints

```javascript
Route.get('user/{name}', function(req, res, name)
{
    //
})
.where('name', '[A-Za-z]+');

Route.get('user/{id}', function(req, res, id)
{
    //
})
.where('id', '[0-9]+');
```

### Passing An Object Of Wheres

Of course, you may pass an object of constraints when necessary:

```javascript
Route.get('user/{id}/{name}', function(req, res, id, name)
{
    //
})
.where({'id': '[0-9]+', 'name': '[a-z]+'});
```

### Defining Global Patterns

If you would like a route parameter to always be constrained by a given regular expression, you may use the pattern method:

```javascript
Route.pattern('id', '[0-9]+');

Route.get('user/{id}', function(req, res, id)
{
    // Only called if {id} is numeric.
});
```

### Accessing A Route Parameter Value

If you need to access a route parameter value outside of a route, you may use the req.parameter method:

```javascript
Route.filter('foo', function(req, res, next)
{
    if (req.parameter('id') == 1)
    {
        //
    }
});
```

## Route Filters

Route filters provide a convenient way of limiting access to a given route, which is useful for creating areas of
your site which require authentication. You can register your application filters in the `app/filters.js` file.

Defining A Route Filter

```javascript
var URL = App.url;

Route.filter('old', function(req, res, next)
{
    if (req.get('age') < 200)
    {
        res.redirect(URL.to('home'));
    }
});
```

### Attaching A Filter To A Route

```javascript
Route.get('user', {'before': 'old', uses: function(req, res)
{
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end('You are over 200 years old!');
}});
```

### Attaching A Filter To A Controller Action

```javascript
Route.get('user', {'before': 'old', 'uses': 'UserController@showProfile'});
```

### Attaching Multiple Filters To A Route

```javascript
Route.get('user', {'before': 'auth|old', uses: function(req, res)
{
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end('You are authenticated and over 200 years old!');
}});
```

### Attaching Multiple Filters Via Array

```javascript
Route.get('user', {'before': ['auth', 'old'], uses: function(req, res)
{
     res.writeHead(200, {'Content-Type': 'text/plain'});
     res.end('You are authenticated and over 200 years old!');
}});
```

### Specifying Filter Parameters

```javascript
Route.filter('age', function(req, res, next, value)
{
    //
});

Route.get('user', {'before': 'age:200', uses: function(req, res)
{
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end('Hello World');
}});
```

### Pattern Based Filters

You may also specify that a filter applies to an entire set of routes based on their URI.

```javascript
Route.filter('admin', function(req, res, next)
{
    //
});

Route.when('admin/*', 'admin');
```

In the example above, the admin filter would be applied to all routes beginning with `admin/`. The asterisk is used as
 a wildcard, and will match any combination of characters.

You may also constrain pattern filters by HTTP verbs:

```javascript
Route.when('admin/*', 'admin', ['post']);
```

## Named Routes

Named routes make referring to routes when generating redirects or URLs more convenient. You may specify a name for a route like so:

```javascript
Route.get('user/profile', {'as': 'profile', uses: function(req, res)
{
    //
}});
```

You may also specify route names for controller actions:

```javascript
Route.get('user/profile', {'as': 'profile', 'uses': 'UserController@showProfile'});
```

Now, you may use the route's name when generating URLs:

```javascript
var URL = App.url;
var profileUrl = URL.route('profile');
```

## Route Groups

Sometimes you may need to apply filters to a group of routes. Instead of specifying the filter on each route, you may use a route group:

```javascript
Route.group({'before': 'auth'}, function()
{
    Route.get('/', function(req, res)
    {
        // Has Auth Filter
    });

    Route.get('user/profile', function(req, res)
    {
        // Has Auth Filter
    });
});
```

You may also use the namespace parameter within your group array to specify all controllers within that group as being in a given namespace:

```javascript
Route.group({'namespace': 'controllers/admin'}, function()
{
    //
});
```

## Sub-Domain Routing

Quorra routes supports sub-domain routing. Quorra routes are also able to handle wildcard sub-domains, and will pass
your wildcard parameters from the domain:

### Registering Sub-Domain Routes

```javascript
Route.get('users', {domain: 'users.myapp.com', 'uses': function(req, res) {
    //
}});

Route.group({'domain': '{account}.myapp.com'}, function() {

    Route.get('user/{id}', function(req, res, account, id) {
        //
    });

});
```

## Route Prefixing

A group of routes may be prefixed by using the prefix option in the attributes object of a group:

```javascript
Route.group({'prefix': 'admin'}, function()
{

    Route.get('user', function()
    {
        //
    });

});
```

## Route Model Binding

Model binding provides a convenient way to inject model instances into your routes. For example, instead of injecting
a user's ID, you can inject the entire User model instance that matches the given ID. First, use the `Route.model`
method to specify the model that should be used for a given parameter:

### Binding A Parameter To A Model

```javascript
var Route = App.router;

Route.model('user', 'User');
```

Next, define a route that contains a `{user}` parameter:

```javascript
Route.get('profile/{user}', function(req, res, user)
{
    //
});
```
Since we have bound the `{user}` parameter to the User model, a User instance will be injected into the route. So, for
example, a request to `profile/1` will inject the User instance which has an ID of 1.

> **Note:** If a matching model instance is not found in the database, a 404 error will be thrown.

If you wish to specify your own "not found" behavior, you may pass a callback as the third argument to the model method:

```javascript
Route.model('user', 'User', function(req, res, next)
{
    throw new Error(404);
});
```

Sometimes you may wish to use your own resolver for route parameters. Simply use the `Route::bind` method:

```javascript
Route.bind('user', function(value, route, req, callback)
{
    User.findOne(id, function(err, model){
        callback(model);
    });
});
```

## Throwing 404 Errors

There are two ways to manually trigger a 404 error from a route. First, you may use the App.abort method:

```javascript
req.abort(404);
```
Second, you may throw an instance of Error with status property equal to 404.

More information on handling 404 exceptions and using custom responses for these errors may be found in the errors section of the documentation.


## Routing To Controllers

Quorra allows you to not only route to Closures, but also to controller classes.

See the documentation on [Controllers](/docs/1/Getting%20started/Controllers.md) for more details.