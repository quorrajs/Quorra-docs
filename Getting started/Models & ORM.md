# Models & ORM

[Configuration](#configuration)

## Configuration

Quorra makes connecting with databases and running queries extremely simple. Quorra comes installed with a powerful
ORM/ODM called [Waterline](https://github.com/balderdashy/waterline), a datastore-agnostic tool that dramatically
simplifies interaction with one or more databases. It provides an abstraction layer on top of the underlying
database, allowing you to easily query and manipulate your data without writing vendor-specific integration code.
The database configuration file is app/config/database.js. In this file you may define all of your database connections,
as well as specify which connection should be used by default. Examples for some of the supported database systems
are provided in this file, for more refer to waterline [docs](https://github
.com/balderdashy/waterline#community-adapters). You can also define waterline model configurations in this file. To
know more about Waterline and Waterline model configurations please refer to the Waterline [docs](https://github
.com/balderdashy/waterline-docs).