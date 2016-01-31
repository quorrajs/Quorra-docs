# Mail

 - [Configuration](#configuration)
 - [Basic Usage](#basic-usage)


## Configuration

Quorra provides a clean, simple API over the popular [nodemailer](https://github.com/nodemailer/nodemailer) library.
The mail configuration file is app/config/mail.js, and contains options allowing you choose nodemailer transport
diver, driver configuration as well as set a global from address for all messages delivered by the library. You may
use any nodemailer trasnport you wish.

For test purpose you may install and use [node-mailer-stub-transport](https://github
.com/andris9/nodemailer-stub-transport) which just returns message via callback method.

## Basic Usage

The `Mail.send` method may be used to send an e-mail message:

```javascript
var Mail = App.mail;

Mail.send('emails/welcome', {'key': 'value'}, {'to': 'email'}, function(err, info)
{
    if(err){
        return console.log(err);
    }
    console.log('Message sent: ' + info.response);
});
```
The first argument passed to the send method is the name of the view that should be used as the e-mail body. The
second is the data to be passed to the view, often as an object where the data items are available to the view by key.
The third is the nodemailer mail options like to, cc, attachments etc. The list of available options can be found [here]
(https://github.com/nodemailer/nodemailer#e-mail-message-fields). The fourth is a callback method allowing you to
receive the response from nodemailer after email is sent.

You may also specify a plain text view to use in addition to an HTML view:

```javascript
Mail::send({'html': 'view', 'text': 'view'), viewOptions, mailOptions, callback);
```

Or, you may specify only one type of view using the html or text keys:

```javascript
Mail.send({'text': 'view'}, viewOptions, mailOptions, callback);
```