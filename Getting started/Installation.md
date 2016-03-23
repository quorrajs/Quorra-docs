# Installation

 - [Quorra Installation](#quorra-installation)
 - [System requirements](#system-requirements)
 - [Configuration](#configuration)

## Quorra Installation

First, download the Quorra cli using npm.

```
npm install -g quorra-cli
```

Once installed, the simple `quorra new` command will create a fresh Quorra installation in the directory you specify.
For instance, `quorra new blog` would create a directory named `blog` containing a fresh Quorra installation with all
dependencies installed.

## System requirements

Quorra has a few system requirements.
@todo: update

- NodeJS >= xx
- npm >= xx

## Configuration

The first thing you should do after installing Quorra is set your application key to a random string. If you installed
Quorra via quorra-cli, this key has probably already been set for you by the `generate-key` command. Typically, this
string should be 32 characters long. The key can be set in the app.js configuration file. If the application key is
not set, your user sessions and other encrypted data will not be secure.

Quorra needs almost no other configuration out of the box. You are free to get started developing! However, you may
wish to review the app/config/app.js file and its documentation. It contains several options such as timezone and
locale that you may wish to change according to your application.

Once Quorra is installed, you should also configure your local environment. This will allow you to receive detailed
error messages when developing on your local machine. By default, detailed error reporting is disabled in your
production configuration file.

Note: You should never have `app.debug` set to true for a production application. Never, ever do it.

### Permissions

Quorra may require one set of permissions to be configured: folders within app/storage require write access by the web
server.

### Paths

Several of the framework directory paths are configurable. To change the location of these directories, check out the
 bootstrap/paths.js file.