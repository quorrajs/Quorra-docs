# HTTP Method Override

 - [Introduction](#introduction)
 - [Usage](#usage)

## Introduction

This middleware lets you use HTTP verbs such as PUT or DELETE in places where the client doesn't support it. You can
enable or disable this middleware in `app/config/middleware.js` by setting attribute `httpMethodOverride` to boolean
value.

## Usage

Do a POST request with any of the following option:

 - `X-HTTP-Method-Override` - set this header with value as your real indented method.
 - `_method` - send this request parameter with value as your real intended method along with other data.

 Now Quorra will identify the requests method as any value of any of these option as request method instead of `POST`.


