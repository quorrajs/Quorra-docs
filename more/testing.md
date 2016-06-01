# Testing

- [Introduction](#introduction)
- [Defining & Running Tests](#defining-&-running-tests)
- [Test Environment](#test-environment)
- [Calling Routes From Tests](#calling-routes-from-tests)

## Introduction

Quorra is pre-configured with [mocha](https://mochajs.org) and [should.js](https://github.com/shouldjs/should.js) for application testing.

Some example test files are provided in the `app/tests/integration` directory. After creating a new Quorra application, simply run `npm test` on the command line to run your tests.

> **Note:** Install dev dependencies by running `npm install` before running unit tests.

## Defining & Running Tests

To create a test case, simply create a new test file in the `app/tests/unit` directory.

You may organise your test files like below.

```
    ./myApp
    ├── app
    │    ├── config/
    │    ├── controllers/
    │    ├── ...
    │    └── tests/
    │       ├── integration/
    │       │  ├── controllers/
    │       │  │  └── userController.spec.js
    │       │  ├── models/
    │       │  │  └── user.spec.js
    │       │  └── ...
    │       ├── unit/
    │       ├── fixtures/
    │       ├── ...
    │       ├── bootstrap.js
    │       └── mocha.opts
    ├── bootstrap/
    ├── resources/
    └── ...

```

You may run all of the tests for your application by executing the `npm test` command from your terminal.

## Test Environment

When running unit tests, Quorra will automatically set the configuration environment to `testing`. Also, Quorra includes configuration file for `mail` service in the test environment. Mail driver is set to memory based while in the test environment, meaning no actual mail will be send while testing. You are free to create other testing environment configurations as necessary.

## Calling Routes From Tests

To test application route responses you can use [Supertest](https://github.com/visionmedia/supertest) library which provides several useful methods for testing HTTP requests.