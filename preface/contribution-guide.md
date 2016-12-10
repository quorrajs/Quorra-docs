# Contribution Guide

- [Introduction](#introduction)
- [New Features](#new-features)
- [Bugs](#bugs)
- [Which Branch?](#which-branch)
- [Security Vulnerabilities](#security-vulnerabilities)

## Introduction

Quorra is an open-source project and anyone may contribute to Quorra for its improvement. We welcome contributors,
regardless of skill level, gender, race, religion, or nationality. Having a diverse,
vibrant community is one of the core values of the framework!

The Quorra source code is managed on Github, and there are repositories for each of the Quorra projects:

- [Quorra Framework](https://github.com/quorrajs/Positron)
- [Quorra Application](https://github.com/quorrajs/Quorra)
- [Quorra CLI tool](https://github.com/quorrajs/Quorra-cli)
- [Quorra Documentation](https://github.com/quorrajs/Quorra-docs)
- [Quorra Website](https://github.com/quorrajs/quorrajs.github.io)

## New Features

Before sending pull requests for new features, please contact Harish Anchu(@harishanchu) via the [quorrajs Gitter chat room](https://gitter.im/quorrajs/quorrajs). If the feature is found to be a good fit for the
framework, you are free to make a pull request. If the feature is rejected, don't give up! You are still free to turn
your feature into a node module which can be released to the world via [NPM](https://npmjs.com/).

When adding new features, don't forget to add unit tests! Unit tests help ensure the stability and reliability of the
framework as new features are added.

## Bugs

### Via Unit Test

Pull requests for bugs may be sent without prior discussion with the Quorra development team. When submitting a bug
fix, try to include a unit test that ensures the bug never appears again!

If you believe you have found a bug in the framework, but are unsure how to fix it,
please send a pull request containing a failing unit test. A failing unit test provides the development team "proof"
that the bug exists, and, after the development team addresses the bug, serves as a reliable indicator that the bug
remains fixed.

If are unsure how to write a failing unit test for a bug, review the other unit tests included with the framework. If
you're still lost, you may ask for help in the QuorraJS gitter chat room.

## Which Branch?

**All** bug fixes should be sent to the latest stable branch. Bug fixes should **never** be sent to the `master`
branch unless they fix features that exist only in the upcoming release.

**Minor** features that are **fully backwards compatible** with the current Quorra release may be sent to the latest
stable branch.

**Major** new features should always be sent to the `master` branch, which contains the upcoming Quorra release.

If you are unsure if your feature qualifies as a major or minor, please ask Harish Anchu(@harishanchu) in the
[QuorraJS gitter chat room](https://gitter.im/quorrajs/quorrajs).

## Security Vulnerabilities

If you discover a security vulnerability within Quorra, please send an e-mail to Harish Anchu at [harishanchu@gmail
.com](mailto:harishanchu@gmail.com). All security vulnerabilities will be promptly addressed.