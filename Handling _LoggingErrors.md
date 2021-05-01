# Handling and Logging Errors

## Express Error Middleware

```js
// in index.js
const express = require('express');
const app = express();

// after all middleware
app.use(function(err, req, res, next) {
    // log the exception
    res.status(500).send('An Error Accure.');
});
```

- and in our route
```js
router.get('/', async (req, res, next) => {
    try {
        const genres = await Genre.find();
        res.send(genre);
    } catch (ex) {
        // pass the error to the next middleware
        next(ex);
    }
});
```
- but now in middleware folder i created a new file call `error.js`
```js
module.exports = function(err, req, res, next) {
    // log the exception
    res.status(500).send('An Error Accure.');
}
```

- and in index.js
```js
const error = require('./middleware/error.js');

app.use(error)
```
















