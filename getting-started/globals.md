# Globals

 - [Overview](#overview)
 - [The app object](#the-app-objectapp)
 - [Models](#models)

## Overview

For convenience, Quorra exposes some global variables. By default, your app's models and the global Quorra app object
are available on the global scope; meaning you can refer to them by name anywhere in your backend code (as long as
Quorra has been loaded).

Nothing in Quorra core relies on these global variables - each and every global exposed in
Quorra may be disabled in `app/config/globals.js`.

## The app object(App)

In most cases, you will want to keep the Quorra `App` object globally accessible- it makes your app code much cleaner.
However, if you do need to disable all globals, including `App` object, you can get access to quorra `App` object on the
by requiring the Positron singleton like `var App = require('positron')`(Positron is the core module that powers
Quorra.)

## Models

Your app's models are exposed as global variables using their globalId. For instance, the model defined in the file
app/models/Foo.js will be globally accessible as `Foo`. If this is disabled, you can still access your models via App
.models.*.