# Authorization

- [Introduction](#introduction)
- [Gates](#gates)
    - [Writing Gates](#writing-gates)
    - [Authorizing Actions](#authorizing-actions-via-gates)
- [Creating Policies](#creating-policies)
    - [Generating Policies](#generating-policies)
    - [Registering Policies](#registering-policies)
- [Writing Policies](#writing-policies)
    - [Policy Methods](#policy-methods)
    - [Methods Without Models](#methods-without-models)
    - [Policy Filters](#policy-filters)
- [Authorizing Actions Using Policies](#authorizing-actions-using-policies)
    - [Via The User Model](#via-the-user-model)

## Introduction

In addition to providing [authentication](/docs/{{version}}/middlewares/authentication.md) services out of the box, Quorra also provides a simple way to authorize user actions against a given resource. Like authentication, Quorra's approach to authorization is simple, and there are two primary ways of authorizing actions: gates and policies.

Think of gates and policies like routes and controllers. Gates provide a simple, Closure based approach to authorization while policies, like controllers, group their logic around a particular model or resource. We'll explore gates first and then examine policies.

You do not need to choose between exclusively using gates or exclusively using policies when building an application. Most applications will most likely contain a mixture of gates and policies, and that is perfectly fine! Gates are most applicable to actions which are not related to any model or resource, such as viewing an administrator dashboard. In contrast, policies should be used when you wish to authorize an action for a particular model or resource.

## Gates

### Writing Gates

Gates are Closures that determine if a user is authorized to perform a given action and are typically defined in the `app/providers/AuthServiceProvider` class using the `gate` manager. Gates always receive a user instance as their first argument, and may optionally receive additional arguments such as a relevant Waterline model:

```javascript
    /**
     * Register any application authentication / authorization services.
     *
     * @param {function} done
     */
    AuthServiceProvider.prototype.boot = function (done) {
        this.registerPolicies();

        this.__app.gate.define('update-post', function (user, post, callback) {
            return callback(user.id == post.user_id);
        });

        done();
    };
```

Gates may also be defined using a `Class@method` style callback string, like controllers:

```javascript
    /**
     * Register any application authentication / authorization services.
     *
     * @param {function} done
     */
    AuthServiceProvider.prototype.boot = function (done) {
        this.registerPolicies();

        this.__app.gate.define('update-post', 'policies/PostPolicy@update')

        done();
    };
```

#### Resource Gates

You may also define multiple Gate abilities at once using the `resource` method:

```javascript
    this.__app.gate.resource('posts', 'policies/PostPolicy');
```

This is identical to manually defining the following Gate definitions:

```javascript
    var gate = this.__app.gate;

    gate.define('posts.view', 'policies/PostPolicy@view');
    gate.define('posts.create', 'policies/PostPolicy@create');
    gate.define('posts.update', 'policies/PostPolicy@update');
    gate.define('posts.delete', 'policies/PostPolicy@delete');
```

By default, the `view`, `create`, `update`, and `delete` abilities will be defined. You may override the default abilities by passing an object as a third argument to the `resource` method. The keys of the object define the names of the abilities while the values define the method names. For example, the following code will create two new Gate definitions - `posts.image` and `posts.photo`:

```javascript
    var gate = this.__app.gate;

    gate.resource('posts', 'policies/PostPolicy', {
        'image': 'updateImage',
        'photo': 'updatePhoto',
    });
```

### Authorizing Actions

To authorize an action using gates, you should use the `allows` or `denies` methods. Note that you are not required to pass the currently authenticated user to these methods. Quorra will automatically take care of passing the user into the gate Closure:

```javascript
    var Gate = request.gate;

    Gate.allows('update-post', post, function (allows) {
        if(allows) {
            // The current user can update the post...
        }
    });

    Gate.denies('update-post', post, function (denies) {
        if(denies) {
            // The current user can't update the post...
        }
    });
```

If you would like to determine if a particular user is authorized to perform an action, you may use the `forUser` method on the `Gate` service:

```javascript
    var Gate = request.gate;

    if (Gate.forUser(user)->allows('update-post', function (allows) {
        if(allows) {
          // The current user can update the post...
        }
    });

    Gate.denies('update-post', post, function (denies) {
        if(denies) {
            // The current user can't update the post...
        }
    });
```

## Creating Policies

### Generating Policies

Policies are classes that organize authorization logic around a particular model or resource. For example, if your application is a blog, you may have a `Post` model and a corresponding `PostPolicy` to authorize user actions such as creating or updating posts.

You may generate a policy using the `generate-policy` [quorra command](/docs/{{version}}/quorra-cli/overview.md). The generated policy will be placed in the `app/policies` directory. If this directory does not exist in your application, Quorra will create it for you:

    quorra generate-policy PostPolicy

The `generate-policy` command will generate an empty policy class. If you would like to generate a class with the basic "CRUD" policy methods already included in the class, you may specify a `--model` when executing the command:

    quorra generate-policy --model=Post PostPolicy

### Registering Policies

Once the policy exists, it needs to be registered. The `AuthServiceProvider` included with fresh Quorra applications contains a `policies` property which maps your Waterline models to their corresponding policies. Registering a policy will instruct Quorra which policy to utilize when authorizing actions against a given model:

```javascript
    var ServiceProvider = use('AuthServiceProvider');

    var AuthServiceProvider = ServiceProvider.extend(function(app) {
        this.__policies = {
            "Post": "app/policies/PostPolicy"
        };

    });

    /**
     * Register any application authentication / authorization services.
     *
     * @param {function} done
     */
    AuthServiceProvider.prototype.boot = function (done) {
        this.registerPolicies();

        // define application gates below

        done();
    };

    module.exports = AuthServiceProvider;
```

## Writing Policies

### Policy Methods

Once the policy has been registered, you may add methods for each action it authorizes. For example, let's define an `update` method on our `PostPolicy` which determines if a given `User` can update a given `Post` instance.

The `update` method will receive a `User` and a `Post` instance as its arguments, and should return `true` or `false` indicating whether the user is authorized to update the given `Post`. So, for this example, let's verify that the user's `id` matches the `userId` on the post:

```javascript
    function PostPolicy() {
    }

   /**
    * Determine if the given post can be updated by the user.
    *
    * @param {object} user - waterline User model instance
    * @param {object} post - waterline Post model instance
    * @param {function} callback
    */
    PostPolicy.prototype.update = function (user, post, callback) {
        return callback(user.id === post.userId)
    }

    module.exports = PostPolicy;
```

You may continue to define additional methods on the policy as needed for the various actions it authorizes. For example, you might define `view` or `delete` methods to authorize various `Post` actions, but remember you are free to give your policy methods any name you like.

> **Note:** If you used the `--model` option when generating your policy via the quorra cli, it will already contain methods for the `view`, `create`, `update`, and `delete` actions.

### Methods Without Models

Some policy methods only receive the currently authenticated user, callback method and not an instance of the model they authorize. This situation is most common when authorizing `create` actions. For example, if you are creating a blog, you may wish to check if a user is authorized to create any posts at all.

When defining policy methods that will not receive a model instance, such as a create method, it will not receive a model instance. Instead, you should define the method as only expecting the authenticated user and callback method:

```javascript
    /**
     * Determine if the given user can create posts.
     *
     * @param {object} user - waterline User model instance.
     */
    PostPolicy.prototype.create = function (user, callback) {
        //
    }
```

### Policy Filters

For certain users, you may wish to authorize all actions within a given policy. To accomplish this, define a `before` method on the policy. The `before` method will be executed before any other methods on the policy, giving you an opportunity to authorize the action before the intended policy method is actually called. This feature is most commonly used for authorizing application administrators to perform any action:

```javascript
    PostPolicy.prototype.before(user, ability, policyArguments, callback)
    {
        if (user.isSuperAdmin()) {
            callback(true);
        }
    }
```

If you would like to deny all authorizations for a user you should return `false` from the `before` method. If `null` is returned, the authorization will fall through to the policy method.

## Authorizing Actions Using Policies

### Via The User Model

The `User` model that is included with your Quorra application includes two helpful methods for authorizing actions: `can` and `cant`. The `can` method receives the action you wish to authorize, the relevant model and callback method. For example, let's determine if a user is authorized to update a given `Post` model:

```javascript
    user.can('update', post, function (can) {
        if(can) {
            //
        }
    });
```

If a [policy is registered](#registering-policies) for the given model, the `can` method will automatically call the appropriate policy and return the boolean result. If no policy is registered for the model, the `can` method will attempt to call the Closure based Gate matching the given action name.

#### Actions That Don't Require Models

Remember, some actions like `create` may not require a model instance. In these situations, you may pass a model global id to the `can` method. The model global id will be used to determine which policy to use when authorizing the action:

```javascript
    // Executes the "create" method on the relevant policy...
    user.can('create', 'Post', function (can) {
        if(can)  {
            //
        }
    });
```