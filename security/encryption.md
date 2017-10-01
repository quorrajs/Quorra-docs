# Encryption

 - [Introduction](#introduction)
 - [Configuration](#configuration)
 - [Encrypting A Value](#encrypting-a-value)
 - [Decrypting A Value](#decrypting-a-value)

Quorra provides facilities for strong AES encryption through encrypt service. All of Quorra's encrypted values are signed using a message authentication code (MAC) so that their underlying value can not be modified once encrypted.

## Configuration

Before using Quorra's encrypter, you must set a key option in your `config/app.js` configuration file. You should use the quorra-cli `generate-key` command to generate this key since this quorra-cli command will use Node's secure random bytes generator to build your key. If this value is not properly set, all values encrypted by Quorra will be insecure.

## Encrypting A Value

```javascript
	var encrypted = App.encrypter.encrypt('secret');
```

> **Note:** Be sure to set a random string in the `key` option of the `app/config/app.js` file. Otherwise, encrypted values will not be secure.

## Decrypting A Value

```javascript
	var  decrypted = App.encrypter.decrypt(encrypted);
```