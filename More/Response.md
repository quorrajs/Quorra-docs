# Response

 - [Introduction](#introduction)
 - [Properties](#properties)
 - [Methods](#methods)

## Introduction

The res object represents the HTTP response that an Quorra app sends when it gets an HTTP request.

In this documentation and by convention, the object is always referred to as res (and the HTTP request is req) but
its actual name is determined by the parameters to the callback function in which you’re working.

For example:

```javascript
Route.get('/user/{id}', function(req, res) {
  res.send('user ' + req.params.id);
});
```

But you could just as well have:

```javascript
Route.get('/user/{id}', function(request, response) {
  response.send('user ' + request.params.id);
});
```

## Properties

### res.locals

An object that contains response local variables scoped to the request, and therefore available only to the view(s)
rendered during that request / response cycle (if any). Otherwise, this property is identical to app.locals
(miscellaneous#application-locals).

This property is useful for exposing request-level information such as the request path name, authenticated user,
user settings, and so on.

```javascript
function myMiddleware(app, next){
    this.__app = app;
    this.__next = next;
}

myMiddleware.prototype.handle = function(request, response){
    response.locals.user = req.user;
    response.locals.authenticated = ! req.user.anonymous;
    this.__next();
}

App.middleware(myMiddleware);
```

## Methods

### res.abort(statusCode [, message])

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

### res.append(field [, value])

Appends the specified value to the HTTP response header field. If the header is not already set, it creates the
header with the specified value. The value parameter can be a string or an array.

Note: calling res.set() after res.append() will reset the previously-set header value.

```javascript
res.append('Link', ['<http://localhost/>', '<http://localhost:3000/>']);
res.append('Set-Cookie', 'foo=bar; Path=/; HttpOnly');
res.append('Warning', '199 Miscellaneous warning');
```

### res.attachment([filename])

Sets the HTTP response Content-Disposition header field to “attachment”. If a filename is given, then it sets the
`Content-Type` based on the extension name via `res.type()`, and sets the `Content-Disposition` “filename=” parameter.

```javascript
res.attachment();
// Content-Disposition: attachment

res.attachment('path/to/logo.png');
// Content-Disposition: attachment; filename="logo.png"
// Content-Type: image/png
```

### res.cookie(name, value [, options])

Sets cookie name to value. The value parameter may be a string or object converted to JSON.

The options parameter is an object that can have the following properties.

| Property |   Type   |                                         Description                                         |
|----------|----------|---------------------------------------------------------------------------------------------|
| domain   | String   | Domain name for the cookie. Defaults to the domain name of the app.                         |
| encode   | Function | A synchronous function used for cookie value encoding. Defaults to encodeURIComponent.      |
| expires  | Date     | Expiry date of the cookie in GMT. If not specified or set to 0, creates a session cookie.   |
| httpOnly | Boolean  | Flags the cookie to be accessible only by the web server.                                   |
| maxAge   | String   | Convenient option for setting the expiry time relative to the current time in milliseconds. |
| path     | String   | Path for the cookie. Defaults to “/”.                                                       |
| secure   | Boolean  | Marks the cookie to be used with HTTPS only.                                                |
| signed   | Boolean  | Indicates if the cookie should be signed.                                                   |


All res.cookie() does is set the HTTP Set-Cookie header with the options provided. Any option not specified defaults
to the value stated in RFC 6265.
For example:

```javascript
res.cookie('name', 'tobi', { domain: '.example.com', path: '/admin', secure: true });
res.cookie('rememberme', '1', { expires: new Date(Date.now() + 900000), httpOnly: true });
```

The encode option allows you to choose the function used for cookie value encoding. Does not support asynchronous
functions.

Example use case: You need to set a domain wide cookie for another site in your organization to use. This other site
(not under your administrative control) does not use URI encoded cookie values.

```javascript
//Default encoding
res.cookie('some_cross_domain_cookie', 'http://mysubdomain.example.com',{domain:'example.com'});
// Result: 'some_cross_domain_cookie=http%3A%2F%2Fmysubdomain.example.com; Domain=example.com; Path=/'

//Custom encoding
res.cookie('some_cross_domain_cookie', 'http://mysubdomain.example.com',{domain:'example.com', encode: String});
// Result: 'some_cross_domain_cookie=http://mysubdomain.example.com; Domain=example.com; Path=/;'
```
The maxAge option is a convenience option for setting “expires” relative to the current time in milliseconds. The
following is equivalent to the second example above.

```javascript
res.cookie('rememberme', '1', { maxAge: 900000, httpOnly: true });
```

You can pass an object as the value parameter; it is then serialized as JSON and parsed by `bodyParser` middleware.

```javascript
res.cookie('cart', { items: [1,2,3] });
res.cookie('cart', { items: [1,2,3] }, { maxAge: 900000 });
```

When cookieParser middleware is enabled, this method also supports signed cookies. Simply include the signed option
set to true. Then res.cookie() will use the application encryption key in `app/config/app.js` to sign the value.

```javascript
res.cookie('name', 'tobi', { signed: true });
```

Later you may access this value through the req.signedCookie object.

### res.clearCookie(name [, options])

Clears the cookie specified by name. For details about the options object, see res.cookie().

```javascript
res.cookie('name', 'tobi', { path: '/admin' });
res.clearCookie('name', { path: '/admin' });
```

### res.download(path [, filename] [, fn])

Transfers the file at path as an “attachment”. Typically, browsers will prompt the user for download. By default,
the Content-Disposition header “filename=” parameter is path (this typically appears in the browser dialog). Override
this default with the filename parameter.

When an error occurs or transfer is complete, the method calls the optional callback function fn. This method uses
`res.sendFile()` to transfer the file.

```javascript
res.download('/report-12345.pdf');

res.download('/report-12345.pdf', 'report.pdf');

res.download('/report-12345.pdf', 'report.pdf', function(err){
  if (err) {
    // Handle error, but keep in mind the response may be partially-sent
    // so check res.headersSent
  } else {
    // decrement a download credit, etc.
  }
});
```

### res.end([data] [, encoding])

Ends the response process. This method actually comes from Node core, specifically the response.end() method of http
.ServerResponse.

Use to quickly end the response without any data. If you need to respond with data, instead use methods such as `res
.send()` and `res.json()`.

```javascript
res.end();
res.status(404).end();
```

### res.format(object)

Performs content-negotiation on the Accept HTTP header on the request object, when present. It uses req.accepts() to
select a handler for the request, based on the acceptable types ordered by their quality values. If the header is not
specified, the first callback is invoked. When no match is found, the server responds with 406 “Not Acceptable”, or
invokes the default callback.

The Content-Type response header is set when a callback is selected. However, you may alter this within the callback
using methods such as `res.set()` or `res.type()`.

The following example would respond with `{ "message": "hey" }` when the Accept header field is set to
`application/json` or `*/json` (however if it is `*/*`, then the response will be `hey`).

```javascript
res.format({
  'text/plain': function(){
    res.send('hey');
  },

  'text/html': function(){
    res.send('<p>hey</p>');
  },

  'application/json': function(){
    res.send({ message: 'hey' });
  },

  'default': function() {
    // log the request and respond with 406
    res.status(406).send('Not Acceptable');
  }
});
```

In addition to canonicalized MIME types, you may also use extension names mapped to these types for a slightly less
verbose implementation:

```javascript
res.format({
  text: function(){
    res.send('hey');
  },

  html: function(){
    res.send('<p>hey</p>');
  },

  json: function(){
    res.send({ message: 'hey' });
  }
});
```

### res.get(field)

Returns the HTTP response header specified by field. The match is case-insensitive.

```javascript
res.get('Content-Type');
// => "text/plain"
```

### res.json([body])

Sends a JSON response. This method is identical to res.send() with an object or array as the parameter. However, you
can use it to convert other values to JSON, such as null, and undefined. (although these are technically not valid
JSON).

```javascript
res.json(null);
res.json({ user: 'tobi' });
res.status(500).json({ error: 'message' });
```

### res.jsonp([body])

Sends a JSON response with JSONP support. This method is identical to res.json(), except that it opts-in to JSONP
callback support.

```javascript
res.jsonp(null);
// => null

res.jsonp({ user: 'tobi' });
// => { "user": "tobi" }

res.status(500).jsonp({ error: 'message' });
// => { "error": "message" }
```

By default, the JSONP callback name is simply `callback`. Override this with the `jsonpCallbackName` configuration in
`app/config/response.js`.

The following are some examples of JSONP responses using the same code:

```javascript
// ?callback=foo
res.jsonp({ user: 'tobi' });
// => foo({ "user": "tobi" })

App.config.set('response.jsonpCallbackName', 'cb');

// ?cb=foo
res.status(500).jsonp({ error: 'message' });
// => foo({ "error": "message" })
```

### res.links(links)

Joins the links provided as properties of the parameter to populate the response’s Link HTTP header field.

```javascript
res.links({
  next: 'http://api.example.com/users?page=2',
  last: 'http://api.example.com/users?page=5'
});

yields:

```javascript
Link: <http://api.example.com/users?page=2>; rel="next",
      <http://api.example.com/users?page=5>; rel="last"
```

### res.location(path)

Sets the response Location HTTP header to the specified path parameter.

```javascript
res.location('/foo/bar');
res.location('http://example.com');
res.location('back');
```

A path value of `back` has a special meaning, it refers to the URL specified in the `Referer` header of the request. If
the `Referer` header was not specified, it refers to “/”.

Quorra passes the specified URL string as-is to the browser in the Location header, without any validation or
manipulation, except in case of back.

Browsers take the responsibility of deriving the intended URL from the current URL or the referring URL, and the URL
specified in the Location header; and redirect the user accordingly.

### res.redirect([status,] path)

Redirects to the URL derived from the specified path, with specified HTTP status code status. If you don’t specify
status, the status code defaults to “302 “Found”.

```javascript
res.redirect('/foo/bar');
res.redirect('http://example.com');
res.redirect(301, 'http://example.com');
res.redirect('../login');
```

Redirects can be a fully-qualified URL for redirecting to a different site:

```javascript
res.redirect('http://google.com');
```

Redirects can be relative to the root of the host name. For example, if the application is on "http://example
.com/admin/post/new", the following would redirect to the URL "http://example.com/admin":

```javascript
res.redirect('/admin');
```

Redirects can be relative to the current URL. For example, from "http://example.com/blog/admin/" (notice the trailing
slash), the following would redirect to the URL "http://example.com/blog/admin/post/new".

```javascript
res.redirect('post/new');
```

Redirecting to "post/new" from "http://example.com/blog/admin" (no trailing slash), will redirect to "http://example
.com/blog/post/new".

If you found the above behavior confusing, think of path segments as directories (with trailing slashes) and files,
it will start to make sense.

Path-relative redirects are also possible. If you were on "http://example.com/admin/post/new", the following would
redirect to "http//example.com/admin/post":

```javascript
res.redirect('..');
```

A back redirection redirects the request back to the referer, defaulting to / when the referer is missing.

```javascript
res.redirect('back');
```

### res.send([body])

Sends the HTTP response.

The body parameter can be a Buffer object, a String, an object, or an Array. For example:

```javascript
res.send(new Buffer('whoop'));
res.send({ some: 'json' });
res.send('<p>some html</p>');
res.status(404).send('Sorry, we cannot find that!');
res.status(500).send({ error: 'something blew up' });
```

This method performs many useful tasks for simple non-streaming responses: For example, it automatically assigns the
Content-Length HTTP response header field (unless previously defined) and provides automatic HEAD and HTTP cache
freshness support.

When the parameter is a Buffer object, the method sets the Content-Type response header field to
`application/octet-stream`, unless previously defined as shown below:

```javascript
res.set('Content-Type', 'text/html');
res.send(new Buffer('<p>some html</p>'));
```

When the parameter is a String, the method sets the Content-Type to “text/html”:

```javascript
res.send('<p>some html</p>');
```

When the parameter is an Array or Object, Quorra responds with the JSON representation:

```javascript
res.send({ user: 'tobi' });
res.send([1,2,3]);
```

### res.sendFile(path [, options] [, fn])

Transfers the file at the given path. Sets the Content-Type response HTTP header field based on the filename’s
extension. Unless the root option is set in the options object, path must be an absolute path to the file.

The following table provides details on the options parameter.

|   Property   |                                               Description                                               | Default  |
|--------------|---------------------------------------------------------------------------------------------------------|----------|
| maxAge       | Sets the max-age property of the Cache-Control header in milliseconds or a string in ms format          | 0        |
| root         | Root directory for relative filenames.                                                                  |          |
| lastModified | Sets the Last-Modified header to the last modified date of the file on the OS. Set false to disable it. |          |
| headers      | Object containing HTTP headers to serve with the file.                                                  |          |
| dotfiles     | Option for serving dotfiles. Possible values are “allow”, “deny”, “ignore”.                             | “ignore” |


The method invokes the callback function fn(err) when the transfer is complete or when an error occurs. If the
callback function is specified and an error occurs, the callback function must explicitly handle the response process
either by ending the request-response cycle, or by passing control to the next route.

Here is an example of using res.sendFile with all its arguments.

```javascript
Route.get('/file/{name}', function (req, res, next) {

  var options = {
    root: __dirname + '/../public/',
    dotfiles: 'deny',
    headers: {
        'x-timestamp': Date.now(),
        'x-sent': true
    }
  };

  var fileName = req.routeParameters.name;

  res.sendFile(fileName, options, function (err) {
    if (err) {
      console.log(err);
      res.status(err.status).end();
    }
    else {
      console.log('Sent:', fileName);
    }
  });

});
```

res.sendFile provides fine-grained support for file serving as illustrated in the following example:

```javascript
Route.get('/user/{uid}/photos/{file}', function(req, res){
  var uid = req.routeParameters.uid
    , file = req.routeParameters.file;

  user.mayViewFilesFrom(uid, function(yes){
    if (yes) {
      res.sendFile('/uploads/' + uid + '/' + file);
    } else {
      res.status(403).send('Sorry! you cant see that.');
    }
  });
});
```
For more information, or if you have issues or concerns, see res.send().

### res.sendStatus(statusCode)

Set the response HTTP status code to statusCode and send its string representation as the response body.

```javascript
res.sendStatus(200); // equivalent to res.status(200).send('OK')
res.sendStatus(403); // equivalent to res.status(403).send('Forbidden')
res.sendStatus(404); // equivalent to res.status(404).send('Not Found')
res.sendStatus(500); // equivalent to res.status(500).send('Internal Server Error')
```

If an unsupported status code is specified, the HTTP status is still set to statusCode and the string version of the
code is sent as the response body.

```javascript
res.sendStatus(2000); // equivalent to res.status(2000).send('2000')
```

[More about HTTP Status Codes](http://en.wikipedia.org/wiki/List_of_HTTP_status_codes)

### res.set(field [, value])

Sets the response’s HTTP header field to value. To set multiple fields at once, pass an object as the parameter.

```javascript
res.set('Content-Type', 'text/plain');

res.set({
  'Content-Type': 'text/plain',
  'Content-Length': '123',
  'ETag': '12345'
});
```

Aliased as `res.header(field [, value])`.

### res.status(code)

Use this method to set the HTTP status for the response. It is a chainable alias of Node’s response.statusCode.

```javascript
res.status(403).end();
res.status(400).send('Bad Request');
res.status(404).sendFile('/absolute/path/to/404.png');
```

### res.type(type)

Sets the Content-Type HTTP header to the MIME type as determined by `mime.lookup()` for the specified type. If type
contains the `/` character, then it sets the `Content-Type` to `type`.

```javascript
res.type('.html');              // => 'text/html'
res.type('html');               // => 'text/html'
res.type('json');               // => 'application/json'
res.type('application/json');   // => 'application/json'
res.type('png');                // => image/png:
```

### res.vary(field)

Adds the field to the Vary response header, if it is not there already.

```javascript
res.vary('User-Agent').render('docs');
```

### res.view(view [, locals] [, callback])

Renders a view and sends the rendered HTML string to the client. Optional parameters:

 - view, view file relative to path `applicationRoot/resources/views`
 - locals, an object whose properties define local variables for the view.
 - callback, a callback function. If provided, the method returns both the possible error and rendered string, but does
 not perform an automated response. When an error occurs, the method invokes next(err) internally.

Note: The local variable `cache` enables view caching. Set it to true, to cache the view or caching will be
determined by the configuration `cache` in `app/config/view.js`.

```javascript
// send the rendered view to the client
res.view('index');

// if a callback is specified, the rendered HTML string has to be sent explicitly
res.view('index', function(err, html) {
  res.send(html);
});

// pass a local variable to the view
res.view('user', { name: 'Tobi' }, function(err, html) {
  // ...
});
```