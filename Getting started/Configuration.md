# Configuration

 - [Introduction](#introduction)
 - [Environment Configuration](#environment-configuration)
 - [Protecting Sensitive Configuration](#protecting-sensitive-configuration)
 - [Maintenance Mode](#maintenance-mode)

## Introduction

All of the configuration files for the Quorra framework are stored in the app/config directory. Each option in every
file is documented, so feel free to look through the files and get familiar with the options available to you.

Sometimes you may need to access configuration values at run-time. You may do so using the config object:

### Accessing A Configuration Value

```javascript
var Config = require('positron').config;
Config.get('app.timezone');
```

You may also specify a default value to return if the configuration option does not exist:

```javascript
var timezone = Config.get('app.timezone', 'UTC');
```

### Setting A Configuration Value

Notice that "dot" style syntax may be used to access values in the various files. You may also set configuration values at run-time:

```javascript
Config.set('database.model.connection', 'mongo');
```

Configuration values that are set at run-time are only set for the current quorra application process, and will not be
carried over to a new quorra application process.


## Environment Configuration

It is often helpful to have different configuration values based on the environment the application is running in.
For example, you may wish to use a different session driver on your local development machine than on the production
server. It is easy to accomplish this using environment based configuration.

Simply create a folder within the config directory that matches your environment name, such as local. Next, create
the configuration files you wish to override and specify the options for that environment. For example, to override
the session driver for the local environment, you would create a session.js file in app/config/local with the following
content:

```javascript
var config = {

    'driver': 'file',
}

module.exports = config;
```
Note: Do not use 'testing' as an environment name. This is reserved for unit testing.
Notice that you do not have to specify every option that is in the base configuration file, but only the options you
wish to override. The environment configuration files will "cascade" over the base files.

Next, we need to instruct the framework how to determine which environment it is running in. The default environment
is always production. However, you may setup other environments within the `bootstrap/start.js` file at the root of
your installation. In this file you will find an `app.detectEnvironment` call. The object passed to this method is
used to determine the current environment. You may add other environments and machine names to the object as needed.

```javascript

var env = app.detectEnvironment({

    'local' => ['your-machine-name'],

});
```
In this example, 'local' is the name of the environment and 'your-machine-name' is the hostname of your server. On
Linux and Mac, you may determine your hostname using the hostname terminal command.

If you need more flexible environment detection, you may pass a Closure to the detectEnvironment method, allowing you
 to implement environment detection however you wish:

```javascript
var env = app.detectEnvironment(function()
{
    return process.env.NODE_ENV;
});
```

Note: If you don't wish to use Quorra's machine based environment detection at all you can just set NODE_ENV to
whatever you wish from your commandline also, or pass commandline argument `--env development` when you run Quorra app.
This will override whatever you configure for the environment detection in the `bootstrap/start.js` file.


### Accessing The Current Application Environment

You may access the current application environment via the environment method:

```javascript
var App = require('positron');

var environment = App.environment();
```

You may also pass arguments to the environment method to check if the environment matches a given value:

```javascript
if (App.environment('local')) {
    // The environment is local
}

if (App.environment('local', 'staging')) {
    // The environment is either local OR staging...
}
```

## Protecting Sensitive Configuration

For "real" applications, it is advisable to keep all of your sensitive configuration out of your configuration files.
 Things such as database passwords, Stripe API keys, and encryption keys should be kept out of your configuration
 files whenever possible. So, where should we place them? Thankfully, Quorra provides a very simple solution to
 protecting these types of configuration items using "dot" files.

First, configure your application to recognize your machine as being in the local environment. Next, create a .env
.local.js file within the root of your project, which is usually the same directory that contains your package.json
file. The .env.local.php should return an array of key-value pairs, much like a typical Quorra configuration file:

```javascript
var config = {

    TEST_STRIPE_KEY: 'super-secret-sauce',

};

module.exports = config;
```

All of the key-value pairs returned by this file will automatically be available via the `process.env` NodeJS global
variable. You may now reference these globals from within your configuration files:

```javascript
key: process.env.TEST_STRIPE_KEY
```

Be sure to add the .env.local.js file to your .gitignore file. This will allow other developers on your team to create
 their own local environment configuration, as well as hide your sensitive configuration items from source control.

Now, on your production server, create a .env.js file in your project root that contains the corresponding values for
your production environment. Like the .env.local.js file, the production .env.js file should never be included in
source control.

Note: You may create a file for each environment supported by your application. For example, the development
environment will load the .env.development.js file if it exists. However, the production environment always uses the
.env.js file.

## Maintenance Mode

When your application is in maintenance mode, a custom view will be displayed for all routes into your application.
This makes it easy to "disable" your application while it is updating or when you are performing maintenance. A call
to the `App.down` method is already present in your app/start/global.js file. The response from this method will be
sent to users when your application is in maintenance mode.

To enable maintenance mode, simply execute the `down` Quorra-cli command:

```
quorra down
```

To disable maintenance mode, use the `up` command:

```
quorra up
```

To show a custom view when your application is in maintenance mode, you may add something like the following to your
 application's app/start/global.js file:

```javascript
App.down(function(req, res, next)
{
     res.status(503).view('maintenance');
});
```

If the Closure passed to the down method calls next method, maintenance mode will be ignored for that request.