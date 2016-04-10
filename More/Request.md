# Request

 - [Introduction](#introduction)
 - [Properties](#properties)
 - [Methods](#methods)

## Introduction

The req object represents the HTTP request and has properties for the request query string, parameters, body, HTTP
headers, and so on. In this documentation and by convention, the object is always referred to as req (and the HTTP
response is res) but its actual name is determined by the parameters to the callback function in which you’re working.

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

### req.app

This property holds a reference to the instance of the Quorra application which process the request.

### req.body
Contains key-value pairs of data submitted in the request body. It is populated only when you enable bodyParser
middleware in `app/config/middleware.js`.

### req.cookies
When using enable cookieParser middleware in `app/config/middleware.js`, this property is an object that contains
cookies sent by the request. If the request contains no cookies, it defaults to {}.

```javascript
// Cookie: name=anchu
req.cookies.name
// => "anchu"
```

### req.fresh
Indicates whether the request is “fresh.” It is the opposite of `req.stale`.

It is true if the cache-control request header doesn’t have a no-cache directive and any of the following is true:

 - The `if-modified-since` request header is specified and `last-modified` request header is equal to or earlier than
  the `modified` response header.
 - The `if-none-match` request header is *.
 - The `if-none-match` request header, after being parsed into its directives, does not match the etag response header.

For more information, issues, or concerns, see [fresh](#https://github.com/jshttp/fresh).

### req.host

Contains the hostname derived from the Host HTTP header.

When the [trust proxy setting](/docs/v1/more/miscellaneous.md#quorra-behind-proxies) is set to a non-falsey value, the value of the
X-Forwarded-Host header field will be
used instead. This header can be set by the client or by the proxy.

```javascript
// Host: "example.com:3000"
req.hostname
// => "example.com"
```

### req.ip

The remote IP address of the request.

When the [trust proxy setting](/docs/v1/more/miscellaneous.md#quorra-behind-proxies) is set to a non-falsey value, the
value is
derived from the left-most entry in the `X-Forwarded-For` header. This header can be set by the client or by the proxy.

```javascript
req.ip
// => "127.0.0.1"
```

### req.ips

When the [trust proxy setting](/docs/v1/more/miscellaneous.md#quorra-behind-proxies) is set to a non-falsey value, this property
contains an array of IP addresses specified in the `X-Forwarded-For` request header. Otherwise, it contains an empty
array. This header can be set by the client or by the proxy.

For example, if `X-Forwarded-For` is client, proxy1, proxy2, `req.ips` would be `["client", "proxy1", "proxy2"]`, where
`proxy2` is the furthest downstream.

### req.method

Contains a string corresponding to the HTTP method of the request: GET, POST, PUT, and so on.

### req.routeParameters

An object containing properties mapped to the named route “parameters”. For example, if you have the route
`/user/{name}`, then the “name” property is available as `req.routeParameters.name`. This object defaults to {}.

```javascript
// GET /user/clu
req.routeParameters.name
// => "clu"
```

### req.path

Contains the path part of the request URL.

```javascript
// example.com/users?sort=desc
req.path
// => "/users"
```

### req.protocol

The request protocol string, http or https when requested with TLS.

When the [trust proxy setting](/docs/v1/more/miscellaneous.md#quorra-behind-proxies) is set to a non-falsey value, the value of the
`X-Forwarded-Proto` header field will be trusted and used if present. This header can be set by the client or by the
proxy.

```javascript
req.protocol
// => "http"
```

### req.query

An object contains a property for each query string parameter in the route when `queryParser` middleware is enabled. If
there is no query string, it is the empty object, {}.

```javascript
// GET /search?q=tobi+ferret
req.query.q
// => "tobi ferret"

// GET /shoes?order=desc&shoe[color]=blue&shoe[type]=converse
req.query.order
// => "desc"

req.query.shoe.color
// => "blue"

req.query.shoe.type
// => "converse"
```

### req.route
The currently-matched route, a string. For example:

```javascript
app.get('/user/{id?}', function userIdHandler(req, res) {
  console.log(req.route);
  res.send('GET');
});
```

Example output from the previous snippet:

```javascript
{ __uri: 'user/{id?}',
  __action: { uses: [Function: userIdHandler] },
  __methods: [ 'GET', 'HEAD' ],
  __wheres: {},
  __compiled:
   { staticPrefix: '/user',
     regex: { /^\/user(?:\/([^\/]+))?$/ xregexp: [Object] },
     tokens: [ [Object], [Object] ],
     pathVariables: [ 'id' ],
     hostRegex: null,
     hostTokens: [],
     hostVariables: [],
     variables: [ 'id' ] },
  __filter:
   { __sorted: { 'router.before': [Object] },
     __wildcards: {},
     __filters:
      { 'positron.app.down': [Object],
        'router.before': [Object],
        'router.filter: auth': [Object],
        'router.filter: csrf': [Object] },
     __patternFilters: {},
     __regexFilters: {} },
  __host: '',
  __requirements: [],
  __defaults: [ id: null ],
  __path: '/user/{id}' }
```

### req.secure

A Boolean that is true if a TLS connection is established. Equivalent to:

```javascript
'https' == req.protocol;
```

### req.signedCookies

When `cookieParser` middleware is enabled, this property contains signed cookies sent by the request, unsigned and
ready for use. Signed cookies reside in a different object to show developer intent; otherwise, a malicious attack
could be placed on req.cookie values (which are easy to spoof). Note that signing a cookie does not make it “hidden”
or encrypted; but simply prevents tampering (because the secret used to sign is private). If no signed cookies are
sent, the property defaults to {}.

```javascript
// Cookie: user=tobi.CP7AWaXDfAKIRfH49dQzKJx7sKzzSoPq7/AcBBRVwlI3
req.signedCookies.user
// => "tobi"
```

For more information, issues, or concerns, see [cookie-parser](https://github.com/expressjs/cookie-parser).

### req.stale

Indicates whether the request is “stale,” and is the opposite of req.fresh. For more information, see req.fresh.

```javascript
req.stale
// => true
```

### req.subdomains

An array of subdomains in the domain name of the request.

```javascript
// Host: "tobi.ferrets.example.com"
req.subdomains
// => ["ferrets", "tobi"]
```

### req.xhr

A Boolean value that is true if the request’s “X-Requested-With” header field is “XMLHttpRequest”, indicating that
the request was issued by a client library such as jQuery.

```javascript
req.xhr
// => true
```

## Methods

### req.accepts(types)

Checks if the specified content types are acceptable, based on the request’s Accept HTTP header field. The method
returns the best match, or if none of the specified content types is acceptable, returns false (in which case, the
application should respond with 406 "Not Acceptable").

The type value may be a single MIME type string (such as “application/json”), an extension name such as “json”, a
comma-delimited list, or an array. For a list or array, the method returns the best match (if any).

```javascript
// Accept: text/html
req.accepts('html');
// => "html"

// Accept: text/*, application/json
req.accepts('html');
// => "html"
req.accepts('text/html');
// => "text/html"
req.accepts(['json', 'text']);
// => "json"
req.accepts('application/json');
// => "application/json"

// Accept: text/*, application/json
req.accepts('image/png');
req.accepts('png');
// => undefined

// Accept: text/*;q=.5, application/json
req.accepts(['html', 'json']);
// => "json"
```

For more information, or if you have issues or concerns, see [accepts](https://github.com/jshttp/accepts).

### req.acceptsCharsets(charset [, ...])

Returns the first accepted charset of the specified character sets, based on the request’s `Accept-Charset` HTTP header
 field. If none of the specified charsets is accepted, returns false.

For more information, or if you have issues or concerns, see [accepts](https://github.com/jshttp/accepts).

### req.acceptsEncodings(encoding [, ...])

Returns the first accepted encoding of the specified encodings, based on the request’s `Accept-Encoding` HTTP header
field. If none of the specified encodings is accepted, returns false.

For more information, or if you have issues or concerns, see [accepts](https://github.com/jshttp/accepts).

### req.acceptsLanguages(lang [, ...])
Returns the first accepted language of the specified languages, based on the request’s `Accept-Language` HTTP header
field. If none of the specified languages is accepted, returns false.

For more information, or if you have issues or concerns, see [accepts](https://github.com/jshttp/accepts).

### req.get(field)
Returns the specified HTTP request header field (case-insensitive match). The Referrer and Referer fields are
interchangeable.

```javascript
req.get('Content-Type');
// => "text/plain"

req.get('content-type');
// => "text/plain"

req.get('Something');
// => undefined
Aliased as req.header(field).
```

### req.input.all()

Gets all input values for the request

### req.input.except

Get a subset of the items from the input data.

```javascript
var input = req.input.except('username');

var input = req.input.except('username', 'password');

var input = req.input.except(['username', 'password']);
```

### req.input.flash()

You may need to keep input from one request until the next request. For example, you may need to re-populate a form
after checking it for validation errors. This will work only if you have enabled session middleware in
(`app/config/middleware.js`)

Flashes all input items to the session

```javascript
req.input.flash();
```
> **Note:** You may flash other data across requests using the [session](/docs/v1/middlewares/session.md) middleware methods.

### req.input.flashExcept(name)

You may need to keep input from one request until the next request. For example, you may need to re-populate a form
after checking it for validation errors. This will work only if you have enabled session middleware in
(`app/config/middleware.js`)

Flashes a subset of input items to the session

```javascript
req.input.flashExcept('password');
```

### req.input.flashOnly(name)

You may need to keep input from one request until the next request. For example, you may need to re-populate a form
after checking it for validation errors. This will work only if you have enabled session middleware in
(`app/config/middleware.js`)

Flashes a subset of input items to the session

```javascript
req.input.flashOnly('username', 'email');
```

### req.input.file(name)

Retrieves a file from the request.

> **Note:** The bodyParser middleware included with Quorra only handles JSON and urlencoded form submissions, not multipart. For
this method to work you may need to add additional file parsing middleware to your application.

```javascript
var file = req.input.file('photo');
```

### req.input.get(name, [, defaultValue])

Gets the value of a request input param `name` when present or `defaultValue`.

 - Checks body params, ex: id=12, {"id":12}
 - Checks query string params, ex: ?id=12

Lookup is performed in the following order:

 - req.body
 - req.query

`bodyParser` middleware must be loaded for `req.param()` to work predictably. Refer req.body for details.

### req.input.has()

Determines if the request contains a non-empty value for an input item.

```javascript
if (req.input.has('name'))
{
    //
}
```

### req.input.hasFile([name])

Determines if the uploaded data contains a file.

The bodyParser middleware included with Quorra only handles JSON and urlencoded form submissions, not multipart. For
this method to work you may need to add additional file parsing middleware to your application.

```javascript
var file = req.input.hasFile('photo');

var file = req.input.hasFile('photo', 'fbPic');

var file = req.input.hasFile(['photo', 'fbPic']);
```

### req.input.old()

You may need to keep input from one request until the next request. For example, you may need to re-populate a form
after checking it for validation errors. This will work only if you have enabled session middleware in
(`app/config/middleware.js`)

You can retrieve the old flashed data with this method.

```javascript
var oldUsername = req.input.old('username')
```

### req.input.only([name])

Get a subset of the items from the input data.

```javascript
var input = req.input.only('username');

var input = req.input.only('username', 'password');

var input = req.input.only(['username', 'password']);
```

### req.is(type)

Returns true if the incoming request’s “Content-Type” HTTP header field matches the MIME type specified by the type
parameter. Returns false otherwise.

```javascript
// With Content-Type: text/html; charset=utf-8
req.is('html');
req.is('text/html');
req.is('text/*');
// => true

// When Content-Type is application/json
req.is('json');
req.is('application/json');
req.is('application/*');
// => true

req.is('html');
// => false
```

For more information, or if you have issues or concerns, see [type-is.](https://github.com/jshttp/type-is)

### req.param(name [, defaultValue])

Return the value of param name when present.

```javascript
// ?name=tobi
req.param('name')
// => "tobi"

// POST name=tobi
req.param('name')
// => "tobi"

// /user/tobi for /user/:name
req.param('name')
// => "tobi"
```

Lookup is performed in the following order:

 - req.routeParameters
 - req.body
 - req.query

Optionally, you can specify defaultValue to set a default value if the parameter is not found in any of the request
objects.

Direct access to `req.body`, `req.routeParameters`, and `req.query` should be favoured for clarity - unless you truly
accept input from each object.

`bodyParser` middleware must be loaded for `req.param()` to work predictably. Refer req.body for details.

### req.routeParameter(name, [, defaultValue])

Return the route parameter for the specified key from `req.routeParameters` object. Optionally, you can specify
defaultValue to set a default value if the parameter is not found in any of the request routeParameters object.

```javascript
// /user/tobi for /user/:name
req.routeParameter('name')
// => "tobi"
```
