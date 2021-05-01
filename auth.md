# Authentication and Authorization

## Creating the user model
```js
const Joi = require('joi');
const mongoose = require('mongoose');

const User = mongoose.model('User', new mongoose.Schema({
    name: {
        type: String,
        required: true,
        minlength: 5,
        maxlength: 50
    },
    email: {
        type: String,
        required: true,
        minlength: 5,
        maxlength: 255,
        unique: true
    },
    password: {
        type: String,
        required: true,
        minlength: 5,
        maxlength: 1024
    },
})); 

function validateUser(user) {
    const schema = {
        name: Joi.string().min(5).max(50).required(),
        email: Joi.string().min(5).max(255).required().email(),
        password: Joi.string().min(5).max(255).required(),
    }

    return Joi.validate(user, schema);
}

exports.User = User;
exports.validate = validateUser;
```

## Registering Users

```js
const { User, validate } = require('../models/user');
const mongoose = require('mongoose');
const express = require('express');
const router = express.Router();

router.post('/', async (req, res) => {
    const { error } = validate(req.body);
    if (error) return res.status(400).send(error);

    let user = await User.findOne({ email: req.body.email });
    if(user) return res.status(400).send('User Already Registered.')

    user = new User({ 
        name: req.body.name,
        email: req.body.email,
        password: req.body.password
    });
    await user.save();

    res.send(user);
});

module.exports = router;
```

## Using Lodash

- install it by `npm i lodash`
- i wanna use lodash to return just email and name without the password in the response
```js
// import lodash first
const _ = require('lodash');
const { User, validate } = require('../models/user');
const mongoose = require('mongoose');
const express = require('express');
const router = express.Router();

router.post('/', async (req, res) => {
    const { error } = validate(req.body);
    if (error) return res.status(400).send(error);

    let user = await User.findOne({ email: req.body.email });
    if(user) return res.status(400).send('User Already Registered.')

    user = new User({ 
        name: req.body.name,
        email: req.body.email,
        password: req.body.password
    });
    await user.save();

    // to get just the name and email without the password
    res.send(_.pick(user, ['name', 'email']));
});

module.exports = router;

```

- and for handling password search in google for `joi-password-complexity`

## Hashing Passwords

- install `npm i bcrypt`
- in `hash.js`

```js
const bcrypt = require('bcrypt');

async function run() {
    // first you will need to create the salt
    const salt = await bcrypt.genSalt(10); // generate salt with teen digits
    // then hash it
    const hashed = await bcrypt.hash('1234', salt);
    console.log(salt);
    console.log(hashed);
}

run();
```

- now use it in our route
```js
// import lodash first
const _ = require('lodash');
const { User, validate } = require('../models/user');
const mongoose = require('mongoose');
const express = require('express');
const router = express.Router();

router.post('/', async (req, res) => {
    const { error } = validate(req.body);
    if (error) return res.status(400).send(error);

    let user = await User.findOne({ email: req.body.email });
    if(user) return res.status(400).send('User Already Registered.')

    user = new User({ 
        name: req.body.name,
        email: req.body.email,
        password: req.body.password
    });
    
    // reset the password and hash it
    const salt = await bcrypt.genSalt(10);
    user.password = await bcrypt.hash(user.password, salt);
    await user.save();

    // to get just the name and email without the password
    res.send(_.pick(user, ['name', 'email']));
});

module.exports = router;
```

## Authenticating Users

```js
// import lodash first
const _ = require('lodash');
const { User, validate } = require('../models/user');
const mongoose = require('mongoose');
const express = require('express');
const router = express.Router();

router.post('/', async (req, res) => {
    const { error } = validate(req.body);
    if (error) return res.status(400).send(error);

    let user = await User.findOne({ email: req.body.email });
    if(user) return res.status(400).send('Invalid Email or Password.')

    // we use this method to compare our hashed password with the passed password and this will return true or false
    const validPasswprd = await bcrypt.compare(req.body.password, user.password);
    
    if(!validPassword) return res.status(400).send('Invalid Email or Password.');

    res.send(true);
});

function validate(req) {
    const schema = {
        email: Joi.string().min(3).required(),
        password: Joi.string().min(3).required(),
    }

    return Joi.validate(req, schema);
}

module.exports = router;
```

# JSON Web Tokens

## Generating Authentication Tokens

- install it first `npm i jsonwebtoken`

```js
// import jwt first
const jwt = require('jsonwebtoken');
const _ = require('lodash');
const { User, validate } = require('../models/user');
const mongoose = require('mongoose');
const express = require('express');
const router = express.Router();

router.post('/', async (req, res) => {
    const { error } = validate(req.body);
    if (error) return res.status(400).send(error);

    let user = await User.findOne({ email: req.body.email });
    if(user) return res.status(400).send('Invalid Email or Password.')

    // we use this method to compare our hashed password with the passed password and this will return true or false
    const validPasswprd = await bcrypt.compare(req.body.password, user.password);
    
    if(!validPassword) return res.status(400).send('Invalid Email or Password.');

    // this will return the token
    const token = jwt.sign({ _id: user._id }, 'jwtPrivateKey');

    res.send(token);
});

function validate(req) {
    const schema = {
        email: Joi.string().min(3).required(),
        password: Joi.string().min(3).required(),
    }

    return Joi.validate(req, schema);
}

module.exports = router;
```

## Storing Secrets in Environment Variables

- first install `npm i config`
- then in root folder create folder call `config`
- then create a file for the default configuration call `default.js`
```json
{
    "jwtPrivateKey": ""
}
```
- then create another file in this folder call `custom-environment-variables.json`
```json
{
    "jwtPrivateKey": "vidly_jwtPrivateKey"
}
```
- then in our `route`
```js
// import config
const config = require('config');

router.post('/', async (req, res) => {
    const { error } = validate(req.body);
    if (error) return res.status(400).send(error);

    let user = await User.findOne({ email: req.body.email });
    if(user) return res.status(400).send('Invalid Email or Password.')

    // we use this method to compare our hashed password with the passed password and this will return true or false
    const validPasswprd = await bcrypt.compare(req.body.password, user.password);
    
    if(!validPassword) return res.status(400).send('Invalid Email or Password.');

    // this will get the config setting
    const token = jwt.sign({ _id: user._id }, config.get('jwtPrivateKey'));

    res.send(token);
});
```
- and in `index.js` we need to make sure its defined
```js
const config = require('config');

if (!config.get('jwtPrivateKey')) {
    console.error('FATAL ERROR: jwtPrivetKey is not defined');
    // to exit a process with error
    // but zero mean success
    process.exit(1);
}
```

## Setting Response Headers

- after the user signup we need to log him in so we will need to set the auth token in the response

```js
const _ = require('lodash');
const { User, validate } = require('../models/user');
const mongoose = require('mongoose');
const express = require('express');
const router = express.Router();

router.post('/', async (req, res) => {
    const { error } = validate(req.body);
    if (error) return res.status(400).send(error);

    let user = await User.findOne({ email: req.body.email });
    if(user) return res.status(400).send('User Already Registered.')

    user = new User({ 
        name: req.body.name,
        email: req.body.email,
        password: req.body.password
    });
    
    const salt = await bcrypt.genSalt(10);
    user.password = await bcrypt.hash(user.password, salt);
    await user.save();

    const token = jwt.sign({ _id: user._id }, config.get('jwtPrivateKey'));
    // to set the token to the response
    // and it should start with `x-`
    res.header('x-auth-token', token).send(_.pick(user, ['name', 'email']));
});

module.exports = router;
```

## Encapsulating Logic in Mongoose Models

- we need to create a method in the user model to create a token
```js
const Joi = require('joi');
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
    name: {
        type: String,
        required: true,
        minlength: 5,
        maxlength: 50
    },
    email: {
        type: String,
        required: true,
        minlength: 5,
        maxlength: 255,
        unique: true
    },
    password: {
        type: String,
        required: true,
        minlength: 5,
        maxlength: 1024
    },
});

userSchema.methods.generateAuthToken = function() {
    // to generate a token to this user
    // so we need to user regular function not arrow function to use `this` keyword
    const token = jwt.sign({ _id: this._id }, config.get('jwtPrivateKey'));
    return token;
}

const User = mongoose.model('User', userSchema); 

function validateUser(user) {
    const schema = {
        name: Joi.string().min(5).max(50).required(),
        email: Joi.string().min(5).max(255).required().email(),
        password: Joi.string().min(5).max(255).required(),
    }

    return Joi.validate(user, schema);
}

exports.User = User;
exports.validate = validateUser;
```
- then in `user.js` use it

```js
const _ = require('lodash');
const { User, validate } = require('../models/user');
const mongoose = require('mongoose');
const express = require('express');
const router = express.Router();

router.post('/', async (req, res) => {
    const { error } = validate(req.body);
    if (error) return res.status(400).send(error);

    let user = await User.findOne({ email: req.body.email });
    if(user) return res.status(400).send('User Already Registered.')

    user = new User({ 
        name: req.body.name,
        email: req.body.email,
        password: req.body.password
    });
    
    const salt = await bcrypt.genSalt(10);
    user.password = await bcrypt.hash(user.password, salt);
    await user.save();

    // to craete token from the user model
    const token = user.generateAuthToken();
    res.header('x-auth-token', token).send(_.pick(user, ['name', 'email']));
});

module.exports = router;
```

## Authorization Middleware

- first i create a folder call `middleware` in the root
- then created a file call `auth.js` inside this folder
```js
const jwt = require('jsonwebtoken');
const config = require('config');

// create the auth middleware
// middleware should pass to the next middleware or raise an error
function auth(req, res, next) {
    // get the token from the header
    const token = req.header('x-auth-token');
    if (!token) res.status(401).send('Access Denied. no Token Provided');

    try {
        // verify the token with our salt and store it in the request then pass to the next middleware
        const decoded = jwt.verify(token, config.get('jwtPrivateKey'));
        req.user = decoded;
        next();
    }
    catch (ex) {
        res.status(400).send('Invalid Token.')
    }
}

module.exports = auth;
```

## Protecting Routes

- use this middleware in some routes
```js
const auth = require('./middleware/auth');

// the second argument is middleware
router.post('/', auth, async (req, res) => {
    const { error } = validate(req.body);
    if (error) return res.status(400).send(error);

    let user = await User.findOne({ email: req.body.email });
    if(user) return res.status(400).send('Invalid Email or Password.')

    // we use this method to compare our hashed password with the passed password and this will return true or false
    const validPasswprd = await bcrypt.compare(req.body.password, user.password);
    
    if(!validPassword) return res.status(400).send('Invalid Email or Password.');

    // this will get the config setting
    const token = jwt.sign({ _id: user._id }, config.get('jwtPrivateKey'));

    res.send(token);
});
```

## Getting the Current User

```js
const auth = require('../middleware/auth');

route.get('/me', auth, async (req, res) => {
    // exclude the password
    const user = await User.findById(req.user._id).select('-password'); 
    res.send(user);
});
```

## Role-Based Authorization

```js
const Joi = require('joi');
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
    name: {
        type: String,
        required: true,
        minlength: 5,
        maxlength: 50
    },
    email: {
        type: String,
        required: true,
        minlength: 5,
        maxlength: 255,
        unique: true
    },
    password: {
        type: String,
        required: true,
        minlength: 5,
        maxlength: 1024
    },
    isAdmin: Boolean
});

userSchema.methods.generateAuthToken = function() {
    const token = jwt.sign({ _id: this._id, isAdmin: this.isAdmin }, config.get('jwtPrivateKey'));
    return token;
}

const User = mongoose.model('User', userSchema); 

function validateUser(user) {
    const schema = {
        name: Joi.string().min(5).max(50).required(),
        email: Joi.string().min(5).max(255).required().email(),
        password: Joi.string().min(5).max(255).required(),
    }

    return Joi.validate(user, schema);
}

exports.User = User;
exports.validate = validateUser;
```

- then i created another middleware for the admin in `./middleware/admin.js`
```js
module.exports = function (req, res, next) {
    // the other middleware will store the req.user in the request
    if (!req.user.isAdmin) return res.status(403).send('Access Denied.')

    next();
}
```

- then add it to our delete route
```js
router.delete('/:id', [auth, admin], async (req, res) => {});
```

