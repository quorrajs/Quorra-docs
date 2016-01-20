# Localization

 - [Introduction](#introduction)
 - [Language Files](#language-files)
 - [Basic Usage](#basic-usage)
 - [Object notation](#object-notation)
 - [Pluralization](#pluralization)

## Introduction

Quorra makes use of the popular [i18n](https://github.com/mashpie/i18n-node/tree/1b0c86d14c22bc85257735e30bcbb154a4e8b91b) module for localization which provides a
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

Language files will be generated automatically on-the-fly when first used in your app. You can turn this off if  you
don't want to write new locale information to disk in the app/config/lang.js file.

### Example Language File

Language files are simply json file with keyed strings. For example:

```
{
    "welcome": "Welcome to our application"
}
```

### Changing The Default Language At Runtime

The default language for your application is stored in the app/config/lang.js configuration file. You may change the
active language at any time using the `app.setLocale` method:

```javascript
app.setLocale('es');
```

You may change the active language for a particular request by using the `req.setLocale` method:

```javascript
req.setLocale('es');
```

### Setting The Fallback Language

You may also configure "fallbacks" for each language that you use in your application, which will be used when the
active language does not contain a given language line. Like the default language, the fallback language is also
configured in the app/config/lang.js configuration file:

```javascript
    // fall back from Dutch to German
    fallbacks:{'nl': 'de'},
```

## Basic Usage

### In a view

```
<h1> <%= __('Hello') %> </h1>

// passing specific locale
<h1> <%= __({phrase: 'Hello', locale: 'fr'}) %> </h1>
```

### In a route:

```
// (req.locale == 'de')
req.__('Hello'); // => Hola

// passing specific locale
req.__({phrase: 'Hello', locale: 'fr'}); // Salut

```

### Making Replacements In Lines

You may also define place-holders in your language lines:

```javascript
__('Hello %s', 'Marcus'); // Hallo Marcus
__('Hello {{name}}', { name: 'Marcus' }); // Hallo Marcus

// passing specific locale
__({phrase: 'Hello %s', locale: 'fr'}, 'Marcus'); // Salut Marcus
```

## Object notation

In addition to the traditional, linear translation lists, i18n also supports hierarchical translation catalogs.

To enable this feature, be sure to set objectNotation to true in language configuration file. Note: If you can't or
don't want to use . as a delimiter, set objectNotation to any other delimiter you like.

Instead of calling `__("Hello")` you might call `__("greeting.formal")` to retrieve a formal greeting from a translation
 document like this one:

```
"greeting": {
    "formal": "Hello",
    "informal": "Hi",
    "placeholder": {
        "formal": "Hello %s",
        "informal": "Hi %s"
    }
}
```

In the document, the translation terms, which include placeholders, are nested inside the "greeting" translation.
They can be accessed and used in the same way, like so `__('greeting.placeholder.informal', 'Marcus')`.

## Pluralization

Pluralization is a complex problem, as different languages have a variety of complex rules for pluralization. You may
easily manage this with i18n module. By using a "coma" character, you may separate the singular and plural forms of a
string:

```javascript
__n("%s cat", "%s cats", 1); // 1 Katze
__n("%s cat", "%s cats", 3); // 3 Katzen

// passing specific locale
__n({singular: "%s cat", plural: "%s cats", locale: "fr"}, 1); // 1 chat
__n({singular: "%s cat", plural: "%s cats", locale: "fr"}, 3); // 3 chat

__n({singular: "%s cat", plural: "%s cats", locale: "fr", count: 1}); // 1 chat
__n({singular: "%s cat", plural: "%s cats", locale: "fr", count: 3}); // 3 chat
```