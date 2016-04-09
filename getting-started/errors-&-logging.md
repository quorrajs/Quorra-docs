#Errors & Logging

 - [Configuration](#configuration)
 - [Handling Errors](#handling-errors)
 - [Handling 404 Errors](#handling-404-errors)
 - [Logging](#logging)

## Configuration

The logging handler for your application is registered in the `app/start/global.js` start file. By default, the logger
is configured to use a single log file; however, you may customize this behavior as needed. Since Quorra uses the
popular [Winston](https://github.com/winstonjs/winston) logging library, you can take advantage of the variety of
handlers that Winston offers.

Quorra provides some additional utility methods over winston logger so that you can easily configure log handler in
your application.

 - useFiles(FileTransportOptions)
 - useDailyFiles(DailyRotateFileTransportOptions)
 - useMongoDB(MongodbTransportOptions)
 - useMail(MailTransportOptions)
 - useConsole(ConsoleTransportOptions)

For example, if you wish to use daily log files instead of a single, large file, you can make the following change to
 your start file:

```javascript
var Log = App.log;
Log.useDailyFiles({filename: path.join(App.path.storage, 'logs/quorra.log'), level: 'silly'});
```

Or you can manually add transport mechanism provided by winston library just like following.

```javascript
var Log = App.log;
var winston = require('winston'); // For this to work you have to install winston with command `npm install winston`
Log.add(winston.transports.File, {filename: path.join(App.path.storage, 'logs/quorra.log'), level: 'silly'});
```


### Error Detail

By default, error detail is enabled for your application. This means that when an error occurs you will be shown an
error page with a detailed stack trace and error message. You may turn off error details by setting the debug option
in your app/config/app.js file to false.

> **Note:** It is strongly recommended that you turn off error detail in a production environment.

## Handling Errors

By default, the app/start/global.js file contains an error handler for all exceptions:

```javascript
App.error(function(error, code, request, response, next)
{
    var extra = {};

    if(error.stack) {
        extra.stackTrace = error.stack;
    }

    Log.error(error.message || error, extra);
    next();
});
```

This is the most basic error handler. However, you may specify more handlers if needed.

If an exception handler returns a response, that response will be sent to the browser. A handler can stop the
execution of the other registered handlers by choose to not call the next callback.

### Where To Place Error Handlers

There is no default "home" for error handler registrations. Quorra offers you freedom in this area. One option is to
define the handlers in your start/global.js file. In general, this is a convenient location to place any
"bootstrapping" code. If that file is getting crowded, you could create an `app/errors.js` file, and require that file
from your start/global.js script.

## HTTP Exceptions

Some exceptions describe HTTP error codes from the server. For example, this may be a "page not found" error (404),
an "unauthorized error" (401) or even a developer generated 500 error. In order to return such a response, use the
following:

```javascript
res.abort(404);
```

Optionally, you may provide a response:

```javascript
res.abort(403, 'Unauthorized action.');
```

This method may be used at any time during the request's lifecycle.


## Handling 404 Errors

You may register an error handler that handles all "404 Not Found" errors in your application, allowing you to easily
 return custom 404 error pages:

```javascript
App.missing(function(error, req, res, next)
{
    res.status(400).view('errors.missing');
});
```

## Logging

The Quorra logging facilities provide an instance of powerful winston library with some additional methods. By
default,  Quorra is configured to create a single log file for your application, and this file is stored in
app/storage/logs/quorra.log.
You may write information to the log like so:

```javascript
var Log = App.log;

Log.info('This is some useful information.');

Log.warning('Something could be going wrong.');

Log.error('Something is really going wrong.');
```
The logger provides the six logging levels: silly, debug, verbose, info, warn and error.

A JSON object of contextual data may also be passed to the log methods:

```javascript
Log.info('Log message', {'context': 'Other helpful information'});
```

Winston has a variety of additional handlers you may use for logging.