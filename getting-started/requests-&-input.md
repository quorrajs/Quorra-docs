# Requests & Input

 - [Basic Input](#basic-input)
 - [Cookies](#cookies)
 - [Old Input](#old-input)
 - [Files](#files)
 - [Request Information](#request-information)

Read more about Quorra request object [here](/docs/{{version}}/more/request.md).

## Basic Input

You may access all user input with a few simple methods. You do not need to worry about the HTTP verb used for the request, as input is accessed in the same way for all verbs.

### Retrieving An Input Value

```javascript
var name = req.input.get('name');
```

### Retrieving A Default Value If The Input Value Is Absent

```javascript
var name = req.input.get('name', 'Sally');
```

### Determining If An Input Value Is Present

```javascript
if (req.input.has('name'))
{
    //
}
```

### Getting All Input For The Request

```javascript
var input = req.input.all();
```

### Getting Only Some Of The Request Input

```javascript
var input = req.input.only('username', 'password');


var input = req.input.except('credit_card');
```

## Cookies

All cookies created by the Quorra framework are encrypted and signed with an authentication code, meaning they will be
considered invalid if they have been changed by the client. See [Cookie Parser](/docs/{{version}}/middlewares/cookie-parser
.html) middleware for more.

### Retrieving A Cookie Value

```javascript
var value = req.cookies.name;
```

### Attaching A New Cookie To A Response

```javascript
res.cookie('name', 'tobi', { domain: '.example.com', path: '/admin', secure: true });
res.cookie('rememberme', '1', { expires: new Date(Date.now() + 900000), httpOnly: true });
```

## Old Input

You may need to keep input from one request until the next request. For example, you may need to re-populate a form
after checking it for validation errors.

### Flashing Input To The Session

```javascript
req.input.flash();
```

### Flashing Only Some Input To The Session

```javascript
req.input.flashOnly('username', 'email');

req.input.flashExcept('password');
```

### Retrieving Old Data

```javascript
req.input.old('username');
```

## Files

> **Note:** The bodyParser middleware included with Quorra only handles JSON and urlencoded form submissions, not multipart. For
file methods to work you may need to add additional file parsing middleware to your application.

### Retrieving An Uploaded File

```javascript
var file = req.input.file('photo');
```
For example if a file field was named “image”, and a file was uploaded, `req.input.file('image')` would return
following File object:

```javascript
{ size: 74643,
  path: '/tmp/8ef9c52abe857867fd0a4e9a819d1876',
  name: 'edge.png',
  type: 'image/png',
  hash: false,
  lastModifiedDate: Thu Aug 09 2012 20:07:51 GMT-0700 (PDT),
  _writeStream:
   { path: '/tmp/8ef9c52abe857867fd0a4e9a819d1876',
     fd: 13,
     writable: false,
     flags: 'w',
     encoding: 'binary',
     mode: 438,
     bytesWritten: 74643,
     busy: false,
     _queue: [],
     _open: [Function],
     drainable: true },
  length: [Getter],
  filename: [Getter],
  mime: [Getter] }
```

### Determining If A File Was Uploaded

```javascript
if (req.input.hasFile('photo'))
{
    //
}
```

## Request Information

The Request object provides many methods for examining the HTTP request for your application and extends the
Node http request object. Here are some of the highlights.

### Retrieving A Request Header

```javascript
var header = req.header('referer');
```

### Check If The Given type(s) Is Acceptable

```javascript
 // Accept: text/html
 req.accepts('html');
 // => "html"
```

### Determining If The Request Is Over HTTPS

```javascript
if(req.secure) {
    //
}
```

### Determine If The Request Is Using AJAX

```javascript
if(req.xhr) {
    //
}
```
