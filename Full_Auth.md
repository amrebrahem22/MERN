# Authentication

## Setup
- setup react `create-react-app frontend` then `cd frontend` and install `npm i axios`
- in `backend` folder `npm init -y` and `npm i dotenv express mongoose morgan nodemon` 
- and created these folders `config` `controllers` `midllewares` `models` `routes` `utils` and `server.js`
- and add our variables in `config.env`
```
PORT=5000

MONGO_URI=mongodb://localhost:27017/node_auth
```
- then create our DB connection in `config/db.js`
```js
const mongoose = require('mongoose');

const connectDB = async () => {
    await mongoose.connect(process.env.MONGO_URI, {
        useNewUrlParser: true,
        useCreateIndex: true,
        useUnifiedTopology: true,
        useFindAndModify: true
    });

    console.log('MongoDB Connected')
}

module.exports = connectDB;
```
- and in `server.js`
```js
require('dotenv').config({ path: './config.env' })
const express = require('express');
const app = express();
const auth = require('./routes/auth');
const fs = require('fs')
const morgan = require('morgan')
const connectDB = require('./config/db');

// Connect to MongoDB
connectDB()

const PORT = process.env.PORT || 5000;

app.use(express.json())
app.use(morgan())

// Route
app.use('/api/auth', auth);

app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

## Registeration
- first i created the user model in `models/User.js`
```js
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');

// create the user schema and set the `match` to our email for regEx
const userSchema = new mongoose.Schema({
    username: {
        type: String,
        required: [true, 'Please Provide a Username.'],
        trim: true
    },
    email: {
        type: String,
        required: [true, 'Please Provide an Email.'],
        trim: true,
        unique: true,
        match: [
            /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/,
            'Please Provide a Vaild Email.'
        ]
    },
    password: {
        type: String,
        required: [true, 'Please Provide a Password.'],
        trim: true,
        minlength: 6,
        select: false
    },
    resetPasswordToken: String,
    resetPasswordExpire: Date,
});

// and here before we save we wll create hash password
// and next() here mean save()
userSchema.pre('save', async function(next)  {
    if(!this.isModified('password')) {
        next()
    }

    const salt = await bcrypt.genSalt(10)
    this.password = await bcrypt.hash(this.password, salt)
    next()
})

const User = mongoose.model('User', userSchema);

module.exports = User;
```
- in `routes/auth.js`
```js
const express = require('express');
const router = express.Router()

const { register } = require('../controllers/auth');

router.post('/register', register)
module.exports = router;
```
- and in `controllers`
```js
const User = require('../models/User');

exports.register = async (req, res, next) => {
    const { username, email, password } = req.body;

    try {

        const user = await User.create({username, email, password});

        res.status(201).json({
            success: true,
            user
        })
    } catch(err) {
        res.status(501).json({
            success: false,
            error: error.message
        })
    }
}
```

## Login
- in `models/User.js` i created a custom method for checking the password
```js
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');

const userSchema = new mongoose.Schema({
    username: {
        type: String,
        required: [true, 'Please Provide a Username.'],
        trim: true
    },
    email: {
        type: String,
        required: [true, 'Please Provide an Email.'],
        trim: true,
        unique: true,
        match: [
            /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/,
            'Please Provide a Vaild Email.'
        ]
    },
    password: {
        type: String,
        required: [true, 'Please Provide a Password.'],
        trim: true,
        minlength: 6,
        select: false
    },
    resetPasswordToken: String,
    resetPasswordExpire: Date,
});

userSchema.pre('save', async function(next)  {
    if(!this.isModified('password')) {
        next()
    }

    const salt = await bcrypt.genSalt(10)
    this.password = await bcrypt.hash(this.password, salt)
    next()
});

// will compare the two passwods and return Boolean
userSchema.methods.matchPasswords = async function(password) {
    return await bcrypt.compare(password, this.password)
}

const User = mongoose.model('User', userSchema);

module.exports = User;
```
- and in `routes/auth.js`
```js
const express = require('express');
const router = express.Router()

const { register, login } = require('../controllers/auth');

router.post('/register', register)
router.post('/login', login)


module.exports = router;
```
- and in `controllers/auth.js`
```js
const User = require('../models/User.js')

exports.login = async (req, res, next) => {
    const { email, password } = req.body;

    // check if there's password and email
    if (!email || !password) {
        res.status(400).json({ success: false, error: 'Please Provide Email and Password.' })
    }

    try {
        // get the user and add the password field with the return result
        // because we set password to { selcet: false } in the model
        const user = await User.findOne({email}).select('+password');

        // check if the user exist
        if(!user) {
            res.status(404).json({ success: false, error: 'Invalid Credentials.' })
        }

        // check if the password match our record
        const isMatch = await user.matchPasswords(password);

        if(!isMatch) {
            res.status(404).json({ success: false, error: 'Invalid Credentials.' })
        }

        res.status(200).json({
            success: true,
            token: 'jhjjs12snjks,ab'
        })

    } catch(err) {
        res.status(501).json({
            success: false,
            error: error.message
        })
    }
}
```


## Custom Error Handler Middleware

- first i created a cutom error class in `backend/utils/errorResonse.js`
```js
// will extend the Error
class ErrorResponse extends Error {
    // and expect to get the message and status code
    constructor(message, statusCode) {
        super(message);
        this.statusCode = statusCode
    }
}

module.exports = ErrorResponse;
```
- then in our middleware folder will create the error middleware `backend/middlewares/error.js`
```js
const ErrorResponse = require('../utils/errorResponse');

const handleError = (err, req, res, next) => {
    // clone the orignal error
    let error = { ...err };

    // set the error message
    error.message = err.message;

    // to see the errors
    console.log(err);

    // check the mongo error => and create our custom error
    if (err.code === 11000) {
        const message = 'Duplicate Field Value Enter';
        error = new ErrorResponse(message, 400);
    }

    // if we have validation error
    if (err.name === 'ValidationError') {
        const message = Object.values(err.errors).map(val => val.message)
        error = new ErrorResponse(message, 400);
    }

    // resturn a response
    res.status(error.statusCode || 5000).json({
        success: false,
        error: error.message || 'Server Error'
    })
}

module.exports = handleError;
```
- and in `server.js` add this middlewarea s the last piece of middleware at the end
```js
require('dotenv').config({ path: './config.env' })
const express = require('express');
const app = express();
const auth = require('./routes/auth');
const fs = require('fs')
const morgan = require('morgan')
const connectDB = require('./config/db');
const handleError = require('./middlewares/error');

// Connect to MongoDB
connectDB()

const PORT = process.env.PORT || 5000;

app.use(express.json())
app.use(morgan())

// Route
app.use('/api/auth', auth);

// Error Handler ( Should be the last piece of middleware )
app.use(handleError)

app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```
- and in `controllers/auth.js` use this error handler
```js
const User = require('../models/User');
const ErrorResponse = require('../utils/errorResponse');

exports.register = async (req, res, next) => {
    const { username, email, password } = req.body;
    console.log('reach to register controller ======')

    try {
        console.log('before create the user ======')

        const user = await User.create({username, email, password});

        console.log('after create the user ======')


        res.status(201).json({
            success: true,
            user
        })
    } catch(err) {
        next(err)
    }
}

exports.login = async (req, res, next) => {
    const { email, password } = req.body;

    if (!email || !password) {
        return next(new ErrorResponse('Please Provide Email and Password.', 400))
    }

    try {
        const user = await User.findOne({email}).select('+password');

        if(!user) {
            return next(new ErrorResponse('Invalid Credentials', 404))
        }

        const isMatch = await user.matchPasswords(password);

        if(!isMatch) {
            return next(new ErrorResponse('Invalid Credentials', 404))
        }

        res.status(200).json({
            success: true,
            token: 'jhjjs12snjks,ab'
        })

    } catch(err) {
        res.status(400).json({ success: false, error: err.message })
    }
}
```

## Generate token
- first i will install `npm i jsonwebtoken`
- and will need to create `JWT_SECRET` and `JWT_EXPIRE` so i open the terminal and run
```bash
node
```
- this will open the node termial, run this code and this will generate a random long text with 35 characters we will use it in `JWT_SECRET`
```js
require('crypto').randomBytes(35).toString('hex')
```
- now in our config `config.env`
```
PORT=5000

MONGO_URI=mongodb://localhost:27017/node_auth

JWT_SECRET=36452bc1d93cafcc47a6ce17e45cc7cadc5cc93ac8a71fe91f212edc82c446f79c88de

JWT_EXPIRE=10min
```
- then will create a method that will generate the token in `models/User.js`
```js
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');

const userSchema = new mongoose.Schema({
    username: {
        type: String,
        required: [true, 'Please Provide a Username.'],
        trim: true
    },
    email: {
        type: String,
        required: [true, 'Please Provide an Email.'],
        trim: true,
        unique: true,
        match: [
            /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/,
            'Please Provide a Vaild Email.'
        ]
    },
    password: {
        type: String,
        required: [true, 'Please Provide a Password.'],
        trim: true,
        minlength: 6,
        select: false
    },
    resetPasswordToken: String,
    resetPasswordExpire: Date,
});

userSchema.pre('save', async function(next)  {
    if(!this.isModified('password')) {
        next()
    }

    const salt = await bcrypt.genSalt(10)
    this.password = await bcrypt.hash(this.password, salt)
    next()
});

userSchema.methods.matchPasswords = async function(password) {
    return await bcrypt.compare(password, this.password)
}

// we have to send the id, based on the secrect key we set, because will use it to verify after that
userSchema.methods.getSignedToken = async function() {
    return jwt.sign({ id: this._id }, process.env.JWT_SECRET, { expiresIn: process.env.JWT_EXPIRE })
}

const User = mongoose.model('User', userSchema);

module.exports = User;
```
- and in `controllers/auth.js` i will generate the token after login and register by creating a method to handle that
```js
const User = require('../models/User');
const ErrorResponse = require('../utils/errorResponse');

exports.register = async (req, res, next) => {
    const { username, email, password } = req.body;
    console.log('reach to register controller ======')

    try {
        console.log('before create the user ======')

        const user = await User.create({username, email, password});

        console.log('after create the user ======')

        // 2.use our method to generate token and send it back to the response
        sendToken(user, 201, res);
    } catch(err) {
        next(err)
    }
}

exports.login = async (req, res, next) => {
    const { email, password } = req.body;

    if (!email || !password) {
        return next(new ErrorResponse('Please Provide Email and Password.', 400))
    }

    try {
        const user = await User.findOne({email}).select('+password');

        if(!user) {
            return next(new ErrorResponse('Invalid Credentials', 404))
        }

        const isMatch = await user.matchPasswords(password);

        if(!isMatch) {
            return next(new ErrorResponse('Invalid Credentials', 404))
        }

        // 3. use it in the login
        sendToken(user, 200, res);

    } catch(err) {
        res.status(400).json({ success: false, error: err.message })
    }
}

exports.forgotPassword = async (req, res, next) => {
    res.send('Forgot Password route')
}

exports.resetPassword = async (req, res, next) => {
    res.send('Reset Password route')
}

// 1. this method will create the token and send it back
const sendToken = async (user, statusCode, res) => {
    const token = await user.getSignedToken();

    res.status(statusCode).json({ success: true, token })
}
```

## Protected routes
- now i will create a middleware that will check the authorization in `middlewares/auth.js`
```js
const jwt = require('jsonwebtoken');
const User = require('../models/User');
const ErrorResponse = require('../utils/errorResponse');

exports.protect = async (req, res, next) => {
    let token;

    // 1.check if the authorization exist and startus with bearer
    // because i will send the token like that 
    // Bearer 4255shgjakg51ss2a24d51f22den2 
    if (req.headers.authorization && req.headers.authorization.startsWith('Bearer')) {
        // get the token
        token = req.headers.authorization.split(' ')[1];
    }

    if (!token) {
        return next(new ErrorResponse('Not Authorized to Access to this route', 401))
    }

    try {
        // 2.verify the token
        const decoded = jwt.verify(token, process.env.JWT_SECRET);

        // 3.get the user by the id that comes from decoded
        const user = await User.findById(decoded.id);

        if (!user) {
            return next(new ErrorResponse('No user found with this id', 404))
        }

        // 4.set the user to request
        req.user = user;

        next();
    } catch(err) {
        return next(new ErrorResponse('Not Authorized to Access to this route', 401))
    }
}
```
- then i created a test route in `routes/private.js` that will use our middleware to check the token
```js
const express = require('express');
const router = express.Router();
const { getPrivateData } = require('../controllers/private');
const {protect} = require('../middlewares/auth');

router.get('/', protect, getPrivateData);

module.exports = router;
```
- and in `controllers/private.js`
```js
exports.getPrivateData = (req, res, next) => {
    res.status(200).json({
        success: true,
        data: 'you got access to the private data in this route'
    })
}
```
- and in `server.js`
```js
const private = require('./routes/private');

app.use('/api/private', private);
```

## Password Reset
- first signup in `SendGrid`
- get the api key and all needed data and set it in `config.env` file
```
EMAIL_SERVICE=SendGrid
EMAIL_USERNAME=apikey
EMAIL_PASSWORD=SG.wnhmxjhb5ha12an5xsx2x12a5a3z5zxb5hb2ljs

EMAIL_FROM=mynewemail@gmail.com
```
- and in `models/User.js` create a method that will reset password
```js
const crypto = require('crypto');
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');

const userSchema = new mongoose.Schema({
    username: {
        type: String,
        required: [true, 'Please Provide a Username.'],
        trim: true
    },
    email: {
        type: String,
        required: [true, 'Please Provide an Email.'],
        trim: true,
        unique: true,
        match: [
            /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/,
            'Please Provide a Vaild Email.'
        ]
    },
    password: {
        type: String,
        required: [true, 'Please Provide a Password.'],
        trim: true,
        minlength: 6,
        select: false
    },
    resetPasswordToken: String,
    resetPasswordExpire: Date,
});

userSchema.pre('save', async function(next)  {
    if(!this.isModified('password')) {
        next()
    }

    const salt = await bcrypt.genSalt(10)
    this.password = await bcrypt.hash(this.password, salt)
    next()
});

userSchema.methods.matchPasswords = async function(password) {
    return await bcrypt.compare(password, this.password)
}

userSchema.methods.getSignedToken = async function() {
    return jwt.sign({ id: this._id }, process.env.JWT_SECRET, { expiresIn: process.env.JWT_EXPIRE })
}

// 1. generate password token
userSchema.methods.getResetPasswordToken = function () {
  const resetToken = crypto.randomBytes(20).toString("hex");

  // 2.Hash token (private key) and save to database
  this.resetPasswordToken = crypto
    .createHash("sha256")
    .update(resetToken)
    .digest("hex");

  // 3.Set token expire date
  this.resetPasswordExpire = Date.now() + 10 * (60 * 1000); // Ten Minutes

  return resetToken;
};

const User = mongoose.model('User', userSchema);

module.exports = User;
```
- setup `npm i nodemailer`
- and create file to send email in `backend/utils/sendEmail.js`
```js
const nodemailer = require("nodemailer");

// 1. the options will be like => {to: 'amrebrahem226@gmail.com', subject: 'Password Reset', text: `<h2>Custom Message</h2>`}
const sendEmail = (options) => {
    // 2. create transporter
  const transporter = nodemailer.createTransport({
    service: process.env.EMAIL_SERVICE,
    auth: {
      user: process.env.EMAIL_USERNAME,
      pass: process.env.EMAIL_PASSWORD,
    },
  });

  // 3. collect options
  const mailOptions = {
    from: process.env.EMAIL_FROM,
    to: options.to,
    subject: options.subject,
    html: options.text,
  };

  // 4.send email
  transporter.sendMail(mailOptions, function (err, info) {
    if (err) {
      console.log(err);
    } else {
      console.log(info);
    }
  });
};

module.exports = sendEmail;
```
- then in `routes/auth.js`
```js
const express = require('express');
const router = express.Router()

const { register, login, forgotPassword, resetPassword } = require('../controllers/auth');

router.post('/register', register)
router.post('/login', login)
router.post('/forgot-password', forgotPassword)
router.put('/reset-password/:resetToken', resetPassword)


module.exports = router;
```

- and in `controllers/auth.js`
```js
const crypto = require("crypto");
const ErrorResponse = require("../utils/errorResponse");
const User = require("../models/User");
const sendEmail = require("../utils/sendEmail");

// 1.Forgot Password Initialization
exports.forgotPassword = async (req, res, next) => {
  // 2.Send Email to email provided but first check if user exists
  const { email } = req.body;

  try {
    const user = await User.findOne({ email });

    if (!user) {
      return next(new ErrorResponse("No email could not be sent", 404));
    }

    // 3.Reset Token Gen and add to database hashed (private) version of token
    const resetToken = user.getResetPasswordToken();

    await user.save();

    // 4.Create reset url to email to provided email
    const resetUrl = `http://localhost:3000/passwordreset/${resetToken}`;

    // 5.HTML Message
    const message = `
      <h1>You have requested a password reset</h1>
      <p>Please make a put request to the following link:</p>
      <a href=${resetUrl} clicktracking=off>${resetUrl}</a>
    `;

    try {
        // 6. call our method to send email
      await sendEmail({
        to: user.email,
        subject: "Password Reset Request",
        text: message,
      });

      res.status(200).json({ success: true, data: "Email Sent" });
    } catch (err) {
      console.log(err);
        // 7. if fail set these to undefined 
      user.resetPasswordToken = undefined;
      user.resetPasswordExpire = undefined;

      await user.save();

      return next(new ErrorResponse("Email could not be sent", 500));
    }
  } catch (err) {
    next(err);
  }
};

// 8.Reset User Password
exports.resetPassword = async (req, res, next) => {
  // 9.Compare token in URL params to hashed token
  const resetPasswordToken = crypto
    .createHash("sha256")
    .update(req.params.resetToken)
    .digest("hex");

  try {
    // 10.get the user
    const user = await User.findOne({
      resetPasswordToken,
      resetPasswordExpire: { $gt: Date.now() },
    });

    if (!user) {
      return next(new ErrorResponse("Invalid Token", 400));
    }

    // 11. reset the password, will be hashed automatically
    user.password = req.body.password;
    user.resetPasswordToken = undefined;
    user.resetPasswordExpire = undefined;

    await user.save();

    res.status(201).json({
      success: true,
      data: "Password Updated Success",
      token: user.getSignedJwtToken(),
    });
  } catch (err) {
    next(err);
  }
};

```
