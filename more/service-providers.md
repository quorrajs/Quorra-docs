# Service Providers

 - [Introduction](#introduction)
 - [Defining A Service Provider](#defining-a-service-provider)
 - [Registering A Service Provider At Run-Time](#registering-a-service-provider-at-run-time)

## introduction

Service providers are a great way to group related service registrations in a single location. Think of them as a way
 to bootstrap components in your application. Within a service provider, you might register a custom authentication
 driver, or any other service that needs to be regiseterd with the Application object.

In fact, most of the core Quorra components include service providers. All of the registered service providers for your
application are listed in the `providers` array of the `app/config/app.js` configuration file.

By default, service providers contain two methods: `boot` and `register`. The `register` method is called immediately
when the service provider is registered, while the `boot` method is only called right after all services are
registered. So, if actions in your service provider rely on another service provider already being registered, or you
 are overriding services bound by another provider, you should use the boot method.

## Defining A Service Provider

To create a service provider, simply extend the `ServiceProvider` class provided by the `@positron/support` module and
define a register method:

```javascript
var ServiceProvider = require('@positron/support').ServiceProvider;
var FooService = require('./FooService');

var FooServiceProvider = ServiceProvider.extend(function(app) {
    this.__app = app;
});

FooServiceProvider.prototype.register = function (done) {
    this.__app.foo = new FooService();

    done();
}
```

Now you can use foo service anywhere in you application as `App.foo`. Note that in the constructor method, the
application object is available to you as an argument which is saved as
`this.__app` property and used in the register method. Once you have created a provider and are ready to register it
with your application, simply add it to the `providers` array in your `app` configuration file. You may place your
custom service providers inside `project-root/app/providers` directory.

> **Note:** To register your custom service provider, provide absolute path of your service provider
file in the `providers` array. To register a custom npm Quorra service provider module you can just use the
module name in the `providers` array.

## Registering A Service Provider At Run-Time

You may also register a service provider at run-time using the `App.register` method:

```javascript
App.register('FooServiceProvider');
```

