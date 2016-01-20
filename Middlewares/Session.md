#Session

[Configuration](#configuration)
[Session Usage](#session-usage)
[Flash Data](#flash-data)
[CSRF Token](#csrf-token)
[Database Sessions](#database-sessions)
[Session Drivers](#session-drivers)

## Configuration

Since HTTP driven applications are stateless, sessions provide a way to store information about the user across
requests. NodeSession ships with a variety of session back-ends available for use through a clean, unified API.
Support for back-ends such as File and databases is included out of the box.

The session configuration is stored in app/config/session.js. Be sure to review the well documented options available
to you in this file. By default, Quorra is configured to use the file session driver, which will work well for the
majority of applications.

### Reserved Keys

The Quorra framework uses the flash session key internally, so you should not add an item to the session by that name.


## Session Usage

The session can be accessed via the HTTP request object's session property.

### toring An Item In The Session

```javascript
req.session.put('key', 'value');
```

### Push A Value Onto An Array Session Value

```javascript
req.session.push('user.teams', 'developers');
```

### Retrieving An Item From The Session

```javascript
var value = req.session.get('key');
```

### Retrieving An Item Or Returning A Default Value

```javascript
var value = req.session.get('key', 'default');
```

### Retrieving An Item And Forgetting It

```javascript
var value = req.session.pull('key', 'default');
```

### Retrieving All Data From The Session

```javascript
var data = req.session.all();
```

### Determining If An Item Exists In The Session

```javascript
if (req.session.has('users'))
{
    //
}
```

### Removing An Item From The Session

```javascript
req.session.forget('key');
```

### Removing All Items From The Session

```javascript
req.session.flush();
```

### Regenerating The Session ID

```javascript
req.session.regenerate();
```

## Flash Data

Sometimes you may wish to store items in the session only for the next request. You may do so using the `req.session
.flash` method:

```javascript
req.session.flash('key', 'value');
```

### Reflashing The Current Flash Data For Another Request

```javascript
req.session.reflash();
```

### Reflashing Only A Subset Of Flash Data

```javascript
req.session.keep('username', 'email');
```

## CSRF Token

By default Quorra generates and keeps CSRF token for your application in session.

### Access CSRF token

```javascript
req.session.getToken()
```

## Regenerate CSRF token

```javascript
req.session.regenerateToken()
```

## Database Sessions

When using the database session driver, you may need to setup a table to contain the session items based on database.
Below is a required schema for the table:

| filed        | type    | index  |
|--------------|---------|--------|
| id           | string  | unique |
| payload      | string  |        |
| lastActivity | integer |        |


## Session Drivers

The session "driver" defines where session data will be stored for each request. Quorra ships with several great
drivers out of the box:

- file - sessions will be stored in files in a specified location.
- database - sessions will be stored in a database.