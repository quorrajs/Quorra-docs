# Authentication

 - [Configuration](#configuration)
 - [Storing Passwords](#storing-passwords)
 - [Authenticating Users](#authenticating-users)
 - [Manually Logging In Users](#manually)
 - [Protecting Routes](#protecting-routes)
 - [HTTP Basic Authentication](#http-basic-authentication)

## Configuration

Quorra aims to make implementing authentication very simple. In fact, almost everything is configured for you out of the box. The authentication configuration file is located at `app/config/auth.js`, which contains well documented auth related options for tweaking the behavior of the authentication facilities.

By default, Quorra includes a `User` model in your `app/models` directory. Please remember when building the Schema for this Model to ensure that the password field is a minimum of 60 characters.

> **Note:** Don't forget to set auth and session middlewares to true in the `app/config/middleware.js` file to enable authentication support in your application.

## Storing Passwords

The Quorra `Hash` service provides secure Bcrypt hashing which wraps the popular [bcryptjs](https://www.npmjs.com/package/bcryptjs) npm module:

### Hashing A Password Using Bcrypt

```javascript
	App.hash.make('password', function (err, hash) {
	    // Use hashed password
	});
```

### Verifying A Password Against A Hash

```javascript
	App.hash.check('password', hashedPassword, function (err, result) {
	    if(result) {
            // The passwords match...
        }
	});
```

## Authenticating Users

To log a user into your application, you may use the `auth.attempt` method.

```javascript
	req.auth.attempt({email: 'email', password: 'password'}, function (result) {
	    if(result) {
	        // direct user to appropriate page
	    }
	});
```

Take note that `email` is not a required option, it is merely used for example. You should use whatever column name corresponds to a "username" in your database.

### Determining If A User Is Authenticated

To determine if the user is already logged into your application, you may use the `check` method:

```javascript
	req.auth.check(function (result) {
	    if(result) {
	        // The user is logged in...
	    }
	})
```

### Authenticating A User And "Remembering" Them

If you would like to provide "remember me" functionality in your application, you may pass `true` as the third argument to the `attempt` method, which will keep the user authenticated indefinitely (or until they manually logout). Of course, your `users` table must include the string `remember_token` column, which will be used to store the "remember me" token.

```javascript
	req.auth.attempt({email: 'email', password: 'password'}, function (result) {
	    if(result) {
            // The user is being remembered...
        }
    }, true);
```

> **Note:** If the `attempt` method returns callback with `true`, the user is considered logged into the application.

### Determining If User Authenticated Via Remember
If you are "remembering" user logins, you may use the `viaRemember` method to determine if the user was authenticated using the "remember me" cookie:

```javascript
	if (req.auth.viaRemember())
	{
		//
	}
```

### Authenticating A User With Conditions

You also may add extra conditions to the authenticating query:

```javascript
    req.auth.attempt({email: 'email', password: 'password', active: true}, function (result) {
        if(result) {
            // The user is active, not suspended, and exists.
        }
    });
```

> **Note:** For added protection against session fixation, the user's session ID will automatically be regenerated after authenticating.

### Accessing The Logged In User

Once a user is authenticated, you may access the User record:

```javascript
	req.auth.user(function (user) {
	    var email = user.email;
	});
```

To retrieve the authenticated user's ID, you may use the `id` method:

```javascript
	var id = req.auth.id();
```

To simply log a user into the application by their ID, use the `loginUsingId` method:

```javascript
	req.auth.loginUsingId(1, function (user) {
	    // login success
	});
```

### Validating User Credentials Without Login

The `validate` method allows you to validate a user's credentials without actually logging them into the application:

```javascript
	req.auth.validate(credentials, function (result) {
	    if(result) {
	        //
	    }
	})
```

### Logging A User In For A Single Request

You may also use the `once` method to log a user into the application for a single request. No sessions or cookies will be utilized.

```javascript
	req.auth.once(credentials, function (result) {
	    if(result) {
	        //
	    }
	});
```

### Logging A User Out Of The Application

```javascript
	req.auth.logout(function() {
	    //
	});
```

## Manually Logging In Users

If you need to log an existing user object into your application, you may simply call the `login` method with the object:

```javascript
	User.findOne(1).exec(function (err, user) {
        req.auth.login(user, function () {
            // user logged in
        });
    });
```

This is equivalent to logging in a user via credentials using the `attempt` method.

## Protecting Routes

Route filters may be used to allow only authenticated users to access a given route. Quorra provides the `auth` filter by default, and it is defined in `app/filters.js`.

#### Protecting A Route

```javascript
	Route.get('profile', {before: 'auth', uses: function()
	{
		// Only authenticated users may enter...
	}});
```

### CSRF Protection

Quorra provides an easy method of protecting your application from cross-site request forgeries.

#### Inserting CSRF Token Into Form

Pass csrfToken to view after generating it with method `request.csrfToken` from a route.

For example

```javascript
	Route.get('/register', function(req, res) {
	    res.view('form', { csrfToken: req.csrfToken() });
	});
```

Use csrf token in your view

```javascript
    <input type="hidden" name="_token" value="{{ csrfToken }}">
```

#### Validate The Submitted CSRF Token

```javascript
    Route.post('register', {before: 'csrf', uses: function(req, res) {
        //'You gave a valid CSRF token!';
    }});
```

## HTTP Basic Authentication

HTTP Basic Authentication provides a quick way to authenticate users of your application without setting up a dedicated "login" page. To get started, attach the `auth.basic` filter to your route:

#### Protecting A Route With HTTP Basic

```javascript
	Route.get('profile', {before: 'auth.basic', uses: function(req, res) {
		// Only authenticated users may enter...
	}});
```

By default, the `basic` filter will use the `email` column on the user record when authenticating. If you wish to use another column you may pass the column name as the second parameter to the `basic` method in your `app/filters.js` file:

```javascript
	Route.filter('auth.basic', function(request, response, next) {
		req.auth.basic(function (result) {
            if (result) {
                next();
            } else {
                var error = new Error('Invalid credentials');

                error.status = 401;
                response.setHeader('WWW-Authenticate', 'Basic');
                response.abort(error);
            }
        }, 'username');
	});
```

#### Setting Up A Stateless HTTP Basic Filter

You may also use HTTP Basic Authentication without setting a user identifier cookie in the session, which is particularly useful for API authentication. To do so, define a filter that executes the `onceBasic` method:

```javascript
	Route.filter('basic.once', function(request, response, next) {
		req.auth.onceBasic(function (result) {
            if (result) {
                next();
            } else {
                var error = new Error('Invalid credentials');

                error.status = 401;
                response.setHeader('WWW-Authenticate', 'Basic');
                response.abort(error);
            }
        });
	});
```