# bookshelf-bcrypt.js
[![Version](https://badge.fury.io/js/bookshelf-bcryptjs.svg)](http://badge.fury.io/js/bookshelf-bcryptjs)
[![Downloads](http://img.shields.io/npm/dm/bookshelf-bcryptjs.svg)](https://www.npmjs.com/package/bookshelf-bcryptjs)

Automatic password hashing for your bookshelf models. Uses bcrypt.js instead of regular bcrypt.  
Everything else is a direct fork from [bookshelf-bcrypt](https://www.npmjs.com/package/bookshelf-bcrypt).

### Installation

After installing `bookshelf-bcrypt.js` with `npm i --save bookshelf-bcryptjs`,
all you need to do is add it as a bookshelf plugin and enable it on your models.

```javascript
let knex = require('knex')(require('./knexfile.js').development)
let bookshelf = require('bookshelf')(knex)

// Add the plugin
bookshelf.plugin(require('bookshelf-bcryptjs'))

// Enable it on your models
let User = bookshelf.Model.extend({ tableName: 'users', bcrypt: { field: 'password' } })

// By default, an error will be thrown if a null/undefined password is detected. Use the following to allow null/undefined passwords
let User = bookshelf.Model.extend({ tableName: 'users', bcrypt: { field: 'password', allowEmptyPassword: true } })
```

### Usage

Nothing fancy here, just keep using bookshelf as usual.

```javascript
// Wow such h4x0r, much password
let user = yield User.forge({ password: 'h4x0r' }).save()
console.log(user.get('password')) // $2a$12$K2CtDP7zSGOKgjXjxD9SYey9mSZ9Udio9C95K6wCKZewSP9oBWyPO
```

This plugin will also hash the password again if it detects that the field
changed, so you're good to do this:

```javascript
let user = yield User.forge({ id: 1000 }).fetch()

// Update the user
user.set('password', 'another_pwd')
yield user.save() // Password automatically hashed with the new value

// You can also avoid hashing by using an options
yield user.save({ bcrypt: false })
```

### Settings

`bookshelf-bcrypt` uses 12 salt rounds by default. By default we don't try and detect
a rehash because a user may use a password that looks like a bcrypt hash. If you
add a detectBcrypt function value and it returns a truthy value, an error will be thrown.
You can also override the onRehash function in settings.

```javascript
bookshelf.plugin(require('bookshelf-bcrypt'), {
  rounds: 10 // >= 12 recommended though,
  detectBcrypt: password => password.length > 50,
  onRehash: function () {
    // This will avoid throwing error but be aware that you can loose
    // user's password if you don't know what you're doing.
    // The function is also binded to the model instance that raised the event
    // so you can use any method to better handle it
    console.warn(`Rehash detected for ${this.tableName}`)
    this.set('need_password_change', true)
  }
})
```

### Testing

```bash
git clone git@github.com:estate/bookshelf-bcrypt.git
cd bookshelf-bcrypt && npm install && npm test
```
