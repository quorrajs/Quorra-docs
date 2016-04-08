# Request Lifecycle

- [Overview](#overview)
- [Application start](#application-start-amp-request-lifecycle)
- [Start Files](#start-files)
- [Application Events](#application-events)

## Overview

When using any tool in the "real world", you feel more confident if you understand how that tool works. Application
development is no different. When you understand how your development tools function, you feel more comfortable and
confident using them. The goal of this document is to give you a good, high-level overview of how the Quorra
framework "works". By getting to know the overall framework better, everything feels less "magical" and you will be
more confident building your applications. In addition to a high-level overview of the request lifecycle, we'll cover
"start" files and application events.

If you don't understand all of the terms right away, don't lose heart! Just try to get a basic grasp of what is going
on, and your knowledge will grow as you explore other sections of the documentation.


## Application start & Request lifecycle

Whenever a Qurra application is started index.js script in the application root will be executed. From here, Quorra
begins the process of starting the application and listening requests. Getting a general idea for the Quorra
bootstrap process will be useful, so we'll cover that now!

index.js file executes the `bootstrap/start.js` script with a callback argument. This file creates the new Positron
Application object(Positron is the core module that powers Quorra.), which serves as the backbone of Quorra application.

After creating the Application object, a few project paths will be set and environment detection will be performed.
Then, an internal Positron bootstrap script will be called with the callback from index.js. This file lives deep
within the Positron source, and sets a few more settings based on your configuration files, such as error reporting,
session init etc and initializes application core components like router, view, filter, log, encrypter etc.

After initialization of all core components, your app/start files will be loaded. Lastly, your app/routes.js file
will be loaded. Once your routes.js file has been loaded, it returns the callback from the index.js file with the
newly created positron application object. Callback function executes the listen method of application object, which
creates a NodeJS server and starts to listen to Request.

So, let's summarize:

 - `index.js` file is executed.
 - `bootstrap/start.js` file creates Application and detects environment.
 - Internal `framework/start.js` file configures settings and loads core components.
 - Application `app/start` files are loaded.
 - Application `app/routes.js` file is loaded.
 - Starts to listen to Request.

Now that you have a good idea of how a quorra application lift a NodeJS server, let's take a closer look at request
life cycle. Whenever Quorra server receives a http request it creates a stacked http kernel object. Stacked http
kernel is nothing but stack of application middleware instances which is programmed to execute in sequence. Then pass
request and response object to the created middleware stack for execution. After all middleware execution global
filters and route level filters are executed from which request reaches to the route.

## Start Files

Your application's start files are stored at `app/start`. By default, two are included with your application: `global
.js` and `local.js`.

The global.js start file contains a few basic items by default, such as configuration of the Logger and the inclusion
of your `app/filters.js` file. However, you are free to add anything to this file that you wish. It will be
automatically included on application initialization, regardless of environment. The `local.js` file, on the other
hand, is only called when the application is executing in the local environment. For more information on
environments, check out the configuration documentation.

Of course, if you have other environments in addition to local, you may create start files for those environments as
well. They will be automatically included when your application is running in that environment. So, for example, if
you have a `development` environment configured in your `bootstrap/start.js` file, you may create a
`app/start/development.js` file, which will be included when application is lifted in that environment.

### What To Place In Start Files

Start files serve as a simple place to place any "bootstrapping" code. For example, you could  configure your logging
preferences, set some js settings, etc. It's totally up to you. Of course, throwing all of your bootstrapping code
into your start files can get messy.


## Application Events

### Registering Application Events

You may do pre request processing by registering before application event:

```javascript
App.before(function(request, response, next) {
	
	//

    next();
});
```

Listeners to this event will be run before request to your application. This event can be helpful for global
filtering or global modification of requests and responses. You may register them in one of your start files.