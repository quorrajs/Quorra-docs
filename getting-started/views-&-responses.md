# Views & Responses

 - [Basic Responses](#basic-responses)
 - [Redirects](#redirects)
 - [Views](#views)
 - [Special Responses](#special-responses)

Read more about Quorra request object [here](/docs/v1/more/response.md).

## Basic Responses

### Returning Strings From Routes

```javascript
var Route = App.router;
Route.get('/', function(req, res)
{
    res.send('Hello World');
});
```

### Creating Custom Responses

A Response object provides a variety of methods for building HTTP responses.

```javascript
res.set('Content-Type', 'text/plain');

res.status(200).send('Hello world!');
```

### Attaching Cookies To Responses

```javascript
res.cookie('name', 'value').send('Hello world!')
```

## Redirects

### Returning A Redirect

```javascript
res.redirect('/foo/bar');
res.redirect('http://example.com');
res.redirect(301, 'http://example.com');
res.redirect('../login');
```

## Views

Views typically contain the HTML of your application and provide a convenient way of separating your controller and
domain logic from your presentation logic. Views are stored in the `resources/views` directory.

Quorra uses Jade as its default template engine. Quorra's template implementation relies on Node's
[consolidate](https://www.npmjs.org/package/consolidate) library. Hence Quorra supports all the template engines that
 consolidate library supports. You can configure your template engine settings in `app/config/view.js` configuration
 file.

A simple view could look something like this:

```
<!-- View stored in applicationRoot/resources/views/index.jade -->

extends layout

block content
  h1= title
  p Welcome to #{title}
```

This view may be returned to the browser like so:

```javascript
Route.get('/', function(req, res)
{
    res.view('index', { title: 'Quorra' })
})
```

The second argument passed to `res.view` is an object of data that should be made available to the view.

## Special Responses

### Creating A JSON Response

```javascript
res.json({'name': 'Steve', 'state': 'CA'});
```

### Creating A JSONP Response

```javascript
res.jsonp({'name': 'Steve', 'state': 'CA'})
```
By default the JSONP callback name is simply callback, however you may alter this with the `jsonp` callback name
setting in `app/config/response.js`.

### Creating A File Download Response

Transfer the file at path as an “attachment”, typically browsers will prompt the user for download. The
Content-Disposition “filename=” parameter, aka the one that will appear in the browser dialog is set to path by
default, however you may provide an override filename.

When an error has occurred or transfer is complete the optional callback fn is invoked. This method uses [res.sendfile
()](/docs/v1/more/response.md#ressendfilepath--options--fn) to transfer the file.

```javascript
res.download(pathToFile, fileName, callback);

res.download('/report-12345.pdf');

res.download('/report-12345.pdf', 'report.pdf');

res.download('/report-12345.pdf', 'report.pdf', function(err){
  if (err) {
    // handle error, keep in mind the response may be partially-sent
    // so check res.headerSent
  } else {
    // decrement a download credit etc
  }
});
```
