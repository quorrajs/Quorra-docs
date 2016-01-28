# Body Parser

[Introduction](#introduction)
[Configuration](#configuration)

## Introduction

A pretty common task for every web application is handling user input. When you submit form data with a POST request,
that form data can be encoded in many ways. The default type for HTML forms is application/x-www-urlencoded, but you
can also submit data as JSON or any other arbitrary format.

Body parser parser parses input data based on the request `Content-Type` and stores the result as an object in `req
.body` and is used by various method's provided by the Quorra's request object. This middleware uses module
[body-parser](https://www.npmjs.com/package/body-parser) internally.

## Configuration

You can enable or disable this module in `app/config/middleware.js`.

Body parser middleware offers to parse four types of request data: `JSON`, `url-encoded`, `text` and `raw`. By
default it's enabled and configured to parse JSON and URL-encoded requests. Checkout [body-parser](https://www.npmjs
.com/package/body-parser#api) to find out all the available options for different parsers.