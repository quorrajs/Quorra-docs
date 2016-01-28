# Query Parser

[Introduction](#introduction)
[Configuration](#configuration)

## Introduction

Query parser middleware deserialize a request url query string and stores in `req.query' as an object and is used by
various method's provided by the Quorra's request object.

## Configuration

You can enable or disable this module in `app/config/middleware.js`.

Possible options for `queryParser` are following:

- `simple`: query parser will parse query string with Node's `querystring.parse` function.
- `extended`: query string will be parsed using [qs module](https://www.npmjs.com/package/qs) which provides some
which is helpful when parsing arrays from query string.
- function: a custom query string parsing function. The function will receive the complete query string, and must
return an object of query keys and their values.