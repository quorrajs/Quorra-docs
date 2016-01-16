# HTTP Middleware

[Introduction](#introduction)
[Custom Middleware](#custom-middleware)
[Registering Middleware](#registering-middleware)

## Introduction

Each time your app receives an HTTP request, the configured HTTP middleware stack runs in order. HTTP middleware
provide a convenient mechanism for filtering/modifying HTTP requests entering your application. For
example, Quorra includes a middleware that attaches the session object to the request object before it reaches the
route. Like that one can create a custom auth middleware which will allow the request to pass to the route only if
user is authenticated. A CORS middleware might be responsible for adding the proper headers to all responses leaving
your application. A logging middleware might log all incoming requests to your application.

There are several middleware included in the Quorra framework by default, including middleware for session handling,
query parsing, enabling http method override feature, and more. All of these middlewares can be turned off or on from
`app/config/middleware.js` configuration file.

## Custom Middleware

To create a new middleware, use the make:middleware Quorra command:
@todo:depends on quorra make:middleware command
```
quorra make:middleware AgeMiddleware
```

This command will place a new AgeMiddleware constructor within your app/middleware directory. In this middleware, we
will only allow access to the route if the supplied age is greater than 200. Otherwise, we will issue a request not
acceptable error.

```javascript

function AgeMiddleware(app, next){
    this.__app = app;
    this.__next = next;
}

AgeMiddleware.prototype.handle = function(request, response) {
    if (request.input.get('age') <= 200) {
       response.abort(406);
    } else {
         this.__next.handle(request, response);
    }
};


module.exports = AgeMiddleware;
```

As you can see, if the given age is less than or equal to 200, the middleware will throw an HTTP error; otherwise, the
request will be passed further into the application. To pass the request deeper into the application (allowing the
middleware to "pass"), simply call the next callback with the request and response object.

It's best to envision middleware as a series of "layers" HTTP requests must pass through before they hit your
application. Each layer can examine the request and even reject it entirely.

## Registering Middleware

You can register your custom middleware with the application in `app/routes.js' file outside all routes, or you can
create a middleware.js file in 'app' folder and include it in the `app/start/global.js` file like `require(path.join
(App.path.app, 'middleware'));`

Syntax for registering a custom middleware is

```javascript
app.middleware(require('<middleware constructor>'));
```