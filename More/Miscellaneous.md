# Miscellaneous

[CSRF Protection](#csrf-protection)
[Quorra Behind Proxies](#quorra-behind-proxies)
[Application Locals](#application-locals)

## CSRF Protection

### Adding The CSRF Token To A Form

Quorra provides an easy method of protecting your application from cross-site request forgeries. First, a random token
is placed in your user's session. You can use following methods to access and regenerate tokens. Include the token in
 your html form as a hidden field with `_token` as field name.

### Access CSRF token

```javascript
req.session.getToken()
```

### Regenerate CSRF token

```javascript
req.session.regenerateToken()
```

### Attaching The CSRF Filter To A Route

```javascript
Route.post('profile', {'before': 'csrf', 'uses': function() {
    //
});
```
## Quorra Behind Proxies

When running an Quorra app behind a proxy, set the configuration value `trustProxy`(app/config/request.js) to one of the
values listed in the following table.

Note: Although the app will not fail to run if the configuration value `trustProxy` is not set, it will incorrectly
register the proxy’s IP address as the client IP address unless `trustProxy` is configured.

|     Type     |                                                                                                                                                                                                                                                                                                                                                                                                                                                                   Value                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
|--------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Boolean      | If true, the client’s IP address is understood as the left-most entry in the X-Forwarded-* header.If false, the app is understood as directly facing the Internet and the client’s IP address is derived from req.connection.remoteAddress. This is the default setting.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| IP addresses | An IP address, subnet, or an array of IP addresses and subnets to trust. The following list shows the pre-configured subnet names: <ul><li>loopback - 127.0.0.1/8, ::1/128</li><li>linklocal - 169.254.0.0/16, fe80::/10</li><li>uniquelocal - 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16, fc00::/7</li></li></ul>You can set IP addresses in any of the following ways: <pre><code>'trustProxy': 'loopback' // specify a single subnet<br>'trustProxy': 'loopback, 123.123.123.123' // specify a subnet and an address <br>'trustProxy': 'loopback, linklocal, uniquelocal' // specify multiple subnets as CSV<br>'trustProxy': ['loopback', 'linklocal', 'uniquelocal'] // specify multiple subnets as an array</code></pre>When specified, the IP addresses or the subnets are excluded from the address determination process, and the untrusted IP address nearest to the application server is determined as the client’s IP address. |
| Number       | Trust the nth hop from the front-facing proxy server as the client.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| Function     | Custom trust implementation. Use this only if you know what you are doing. <pre><code>'trustProxy' :function (ip) {,if (ip === '127.0.0.1' \|\| ip === '123.123.123.123') return true; // trusted IPs,else return false; });</code></pre>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |

Setting a non-false trust proxy value results in three important changes:

The value of req.hostname is derived from the value set in the X-Forwarded-Host header, which can be set by the client or by the proxy.

X-Forwarded-Proto can be set by the reverse proxy to tell the app whether it is https or http or even an invalid name. This value is reflected by req.protocol.

The req.ip and req.ips values are populated with the list of addresses from X-Forwarded-For.

The trust proxy setting is implemented using the [proxy-addr](https://www.npmjs.com/package/proxy-addr) package. For more information, see its documentation.

## Application Locals

In Quorra application local variables within the application can be saved to locals attribute of Quorra application
instance, ie `App.locals`.

```javascript
App.locals.title
// => 'My App'

App.locals.email
// => 'me@myapp.com'

App.locals.strftime = require('strftime');
```

Once set, the value of `App.locals` properties persist throughout the life of the application, in contrast with res
.locals properties that are valid only for the lifetime of the request.

You can access local variables in templates rendered within the application. This is useful for providing helper functions to templates, as well as app-level data. Locals are available in middleware via req.app.locals (see req.app)