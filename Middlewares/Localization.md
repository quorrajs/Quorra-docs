# Localization

 - [Introduction](#introduction)

## Introduction

Quorra makes use of the popular [i18n](https://www.npmjs.com/package/i18n) module for localization which provides a
convenient way of retrieving strings in various languages, allowing you to easily support multiple languages within your
 application.

## Language Files

Language strings are stored in files within the app/lang directory. Within this directory there should be a json file
for each language supported by the application with language short code as file name.

```
/app
    /lang
        en.json
        es.json
```

Language file for your default language will be generated automatically. You can turn this off if you don't want to
write new locale information to disk in the app/config/lang.js file.

### Example Language File

Language files are simply json file with keyed strings. For example:

```
{
    "welcome": "Welcome to our application"
}
```

