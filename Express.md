# Express

## Installing Express
```bash
npm i express
```

## Building your first app
*in `index.js`*
```js
const express = require('express');
const app = express();

app.get('/', (req, res) => {
    res.send('Hello World...')
})

app.listen(3000, function() {
    console.log('app listing on port 3000')
})
```

## Nodemon
- install it by `npm i -g nodemon`
- then run the file itself by `nodemon index.js` insteadof `node index.js`

## Environment Variables
```js
const port = process.env.PORT || 3000;
```

## Route Params
```js
app.get('api/courses/:id', (req, res) => {
    res.send(req.params.id);
    // or rquest.query to get the query params
})
```
- another example
```js
app.get('api/courses/:id', (req, res) => {
    const course = courses.find(c => c.id === parseInt(req.params.id));
    if(!course) res.status(404).send('The course with the given ID was not found')
    res.send(course);
})
```

## Handling HTTP POST Requests
```js
// in order to use the body parsing and get acces to the request body
app.use(express.json());

app.post('/api/course', (req, res) => {
    const course = {
        id: courses.length + 1,
        name: req.body.name
    };

    courses.push(course);
    res.send(course);
});
```

## Input Validation

```js
app.post('/api/course', (req, res) => {

    if (!req.body.name || req.body.name.length < 3) {
        // 400 Bad Request
        res.status(400).send("Name is required and should be more than 3 characters.");
        return; // because we don't want the rest of this funtion to be exexuted
    }

    const course = {
        id: courses.length + 1,
        name: req.body.name
    };

    courses.push(course);
    res.send(course);
});
```

- but its better to use `npm joi`
- install it from `npm i joi`
```js
// first ad joi
const Joi = require('joi');

app.post('/api/course', (req, res) => {

    // then define the schema
    const schema = {
        name: Joi.string().min(3).required()
    };

    const result = Joi.validate(req.body, schema);
    console.log(result)

    if (result.error) {
        // 400 Bad Request
        res.status(400).send(result.error.details[0].message);
        return;
    }

    const course = {
        id: courses.length + 1,
        name: req.body.name
    };

    courses.push(course);
    res.send(course);
});
```

## Handling HTTP PUT Requests

```js
// first ad joi
const Joi = require('joi');

app.put('/api/course/:id', (req, res) => {
    const course = courses.find(c => c.id === parseInt(req.params.id));
    if(!course) res.status(404).send('The course with the given ID was not found')

    // then define the schema
    const schema = {
        name: Joi.string().min(3).required()
    };

    const result = Joi.validate(req.body, schema);
    console.log(result)

    if (result.error) {
        // 400 Bad Request
        res.status(400).send(result.error.details[0].message);
        return;
    }

    course.name = req.body.name;
    res.send(course);
});
```

- or we can make it simple

```js
// first ad joi
const Joi = require('joi');

app.put('/api/course/:id', (req, res) => {
    const course = courses.find(c => c.id === parseInt(req.params.id));
    if(!course) res.status(404).send('The course with the given ID was not found')

    const result = validateCourse(req.body);

    if (result.error) {
        // 400 Bad Request
        res.status(400).send(result.error.details[0].message);
        return;
    }

    course.name = req.body.name;
    res.send(course);
});

function validateCourse(course) {
    const schema = {
        name: Joi.string().min(3).required()
    };

    return Joi.validate(course, schema);
}
```

## Handling HTTP Delete Requests

```js
app.put('/api/course/:id', (req, res) => {
    const course = courses.find(c => c.id === parseInt(req.params.id));
    if(!course) res.status(404).send('The course with the given ID was not found')

    const index = courses.indexOf(course);
    courses.splice(index, 1);

    res.send(course);
});
```

## Creating custom Middleware

```js
// to use a middleware
app.use(function(req, res, next) {
    console.log('Logging...');
    // then you will need to call the next()
    // to continue to the next middlware
    next();
});
```

## Built-in Middleware

```js
app.use(express.urlencoded({ extended: true })) // to use the query string in the url
app.use(express.static('public')) // to use the static folder
```

## Third party middleware

- run `npm i helmet`

```js
const helemt = require('helmet');
// then use it
app.use(helmet())
```

## Environments

```js
// to get the node environment, and you can set it your self
// by default it will be undefined
console.log(`NODE_ENV: ${process.env.NODE_ENV}`)
// or you can get it through express
// by default will be "development"
console.log(`app: ${app.get('env')}`)

// and wecan use it like that
if (app.get('env') === "development") {
    app.use(morgan("tiny"))
    console.log('morgan is running...')
}
```

## Configuration

- there's two packages we use for configuration settings `npm rc` and `npm confilg`
- but it will use `npm i config`
- then i created folder call `config` and within this folder i created a file that will have the default configuration `default.json`

```json
{
    "name": "MyExpress App"
}
```
- then i created a file call `development.json`
```json
{
    "name": "MyExpress App - Developmemt",
    "mail": {
        "host": "dev-mail-server"
    }
}
```

- then i created a file call `production.json`
```json
{
    "name": "MyExpress App - Production",
    "mail": {
        "host": "prod-mail-server"
    }
}
```

- then use it in `index.js`
```js
const config = require('config');

console.log(config.get("name"))
console.log(config.get("mail.host"))
```

## Debugging

- install the debug package `npm i debug`

```js
// to call the debug module and it will return a fuction, so we give it a namespace
const startupDebugger = require('debug')('app:startup');
// and again we can use the debug module again for db
const dbDebugger = require('debug')('app:db');

if (app.get('env') === "development") {
    app.use(morgan("tiny"))
    startupDebugger('morgan is running...')
}

dbDebugger('Connected to DB...')

const port = process.env.PORT || 3000;
app.listen(port, () => console.log('app running...'))
```
- and to test it set the enviroment variable to `DEBUG=app:startup` to show the startup messages
- or we can set the enviroment variable to `DEBUG=app:startup,app:db` to show all messages
- or set it `DEBUG=app:*` for all

## Templating Engines

- we have three template engines `pug` `mustache` `ejs`
- so i will use bug `npm i pug`
- and in `index.js`

```js
app.set('view engine', 'pug');
// then set the folder name that will hold our views
app.set('views', './views');
```

- then i created a folder call `views/index.pug`
```pug
html
    head
        title= title
    body
        h1= message
```
- then in `index.js`
```js
app.get('/', (req, res) => {
    // to send variables to the views
    res.render('index', {title: "My Express App", message: "Hello"})
})
```

## Structuring Express Applications

- will have `./public` that will store our assets
- and `./views` to hold our views
- and `./routes` folder to hold our routes and that's will have the CRUD routes and inside this folder will create `courses.js` and use `express.Router()`
```js
const express = require('express');
const router = express.Router();

router.get('/api/courses', (req, res) => {
    res.send(courses);
});

// and at the end of the file
// export the router
module.exports = router;
```

- and in `index.js` require it
```js
const courses = require('./routes/courses');

app.use('/api/courses', courses);
```

