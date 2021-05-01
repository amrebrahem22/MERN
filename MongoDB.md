# MongoDB

## Installing

- got to [mongodb.com](mongodb.com)
- click on `Download`
- choose the `Community Server` then click `Download msi` Button
- setup the `complete` version and check `install mongo compass` to install the mongo client server or you can download compass separatly
- and to download the compass go to the mongodb site and click on `Download` and choose `compass` tap, choose the platform and hit `download`
- now go to this path on your machine `c > program fiels > MongoDB > Server > 3.6 > bin` and copy this path and add it to `Environment Vaiables` and open it and add it to `path` in the environemt variables
- now open command line and run `mongod` and you will get error you will need to create a folder in the `C://` call `data/db`
- after creating this folder run `mongod` and you will run the server
- then go to the mongo compass app and leave the default settings and connect

## Connecting to MongoDB

- install it first `npm i mongoose`
- then in `index.js`
```js
const mogoose = require('mongoose');

mongose.connect('mongodb://localhost/playground')
.then(() => console.log('Connected to MongoDB...'))
.catch(err => console.error('Could Not connect to MongoDB...', err))
```

## Schemas
*We use schemas ib mongoto define the shape of documents in mongoDB Collection*

```js
const mogoose = require('mongoose');

mongose.connect('mongodb://localhost/playground')
.then(() => console.log('Connected to MongoDB...'))
.catch(err => console.error('Could Not connect to MongoDB...', err))

const CourseSchema = new mongoose.Schema({
    name: String,
    author: String,
    tags: [ String ], // Array of string
    date: { type: Date, default: Date.now }, // to set a default value
    isPublished: Boolean
});
```

## Models
*to create a document you have to compile the schema to model*

```js
const mogoose = require('mongoose');

mongose.connect('mongodb://localhost/playground')
.then(() => console.log('Connected to MongoDB...'))
.catch(err => console.error('Could Not connect to MongoDB...', err))

const CourseSchema = new mongoose.Schema({
    name: String,
    author: String,
    tags: [ String ], // Array of string
    date: { type: Date, default: Date.now }, // to set a default value
    isPublished: Boolean
});

const Course = mongoose.model('Course', CourseSchema);

// to create course
const course = new Course({
    name: 'Node.js Course',
    author: 'Amr',
    tags: ['node', 'backend'],
    isPublished: true
});
```
- with this will have the `Course` class in our application

## Saving a Document

```js
const mogoose = require('mongoose');

mongose.connect('mongodb://localhost/playground')
.then(() => console.log('Connected to MongoDB...'))
.catch(err => console.error('Could Not connect to MongoDB...', err))

const CourseSchema = new mongoose.Schema({
    name: String,
    author: String,
    tags: [ String ], // Array of string
    date: { type: Date, default: Date.now }, // to set a default value
    isPublished: Boolean
});

const Course = mongoose.model('Course', CourseSchema);

async function createCourse() {
    // to create course
    const course = new Course({
        name: 'Node.js Course',
        author: 'Amr',
        tags: ['node', 'backend'],
        isPublished: true
    });

    // this save method will return a promise so we can await it
    // and get the result
    // and the await method should by inside async function
    const result = await course.save();
    console.log(course);
}

createCourse();
```

## Querying Documents

```js
async function getCourses() {
    const courses = await Course.find(); // will return a list of courses
    console.log(courses);
}

getCourses();
```

- for Filtering
```js
const courses = await Course.find({ author: 'Mosh', isPublished: true });
```

- for Sorting => `1` == `ASC` and `0` == `DESC`
```js
const courses = await Course.find({ author: 'Mosh', isPublished: true })
.limit(10)
.sort({ name: 1 });
```

- to select a property that we want to return
```js
const courses = await Course.find({ author: 'Mosh', isPublished: true })
.limit(10)
.sort({ name: 1 })
.select({ name: 1, tags: 1 });
```

## Comparison Query Opertators

- `eq` (equal)
- `ne` (not equal)
- `gt` (greater than)
- `gte` (greater than or equal)
- `lt` (less than)
- `lte` (less than or equal)
- `in`
- `nin` (not in)

- to get the courses greater than or equal to $10
```js
// we use $ with operators
const courses = await Course.find({ price: { $gte: 10 } })
.limit(10)
.sort({ name: 1 })
.select({ name: 1, tags: 1 });
```

- using two operators
```js
const courses = await Course.find({ price: { $gte: 10, $lte: 20 } })
```

- using the `in` operator
```js
const courses = await Course.find({ price: { $in: [10, 15, 20] } })
```

## Logical Operators

- `or`
- `and`

```js
const courses = await Course.find()
.or([ { author: 'Mosh' }, { isPublished: true } ]) // takes an array of objects
```

```js
const courses = await Course.find()
.or([ { author: 'Mosh' }, { isPublished: true } ])
.and([{ price: { $gte: 10, $lte: 20 } }])
```

## Regular Expressions

- Starts with
```js
const courses = await Course.find({ author: /^Mosh/ }) // Start with Mosh
```

- Ends with
```js
const courses = await Course.find({ author: /Hamedani$/ }) // ends with Hamedani
```

- case insensitive `i` at the end
```js
const courses = await Course.find({ author: /Hamedani$/i }) // ends with Hamedani
```

- Contains
```js
const courses = await Course.find({ author: /.*Mosh.*/ }) // contains mosh
```

## Count

- for counting the results
```js
const courses = await Course.find({ author: /Hamedani$/i }).count();
```

## Pagination

```js
async function getCourses() {
    const pageNumber = 2;
    const pageSize = 10;

    const courses = await Course.find({author: 'Mosh'})
    .skip( (pageNumber - 1) * pageSize )
    .limit(pageSize)
    .sort({ name: 1 });
    console.log(courses);
}

getCourses();
```

## Updating a Document

```js
async function updateCourse(id) {
    const course = await Course.findById(id);
    if (!course) retur;

    course.set({
        isPublished: true,
        author: 'Another Author'
    });

    const result = await course.save();
    console.log(result);
}

updateCourse('6032bdd71dfea40bbc2c7655');
```

- another way
```js
async function updateCourse(id) {
    const result = await Course.update({_id: id}, {
        $set: {
            author: 'Mosh',
            isPublished: false
        }
    });
    console.log(result);
}

updateCourse('6032bdd71dfea40bbc2c7655');
```
- you can search for `MongoDB Update Operators`

- another way
```js
async function updateCourse(id) {
    const result = await Course.findByIdAndUpdate(id, {
        $set: {
            author: 'Mosh',
            isPublished: false
        }
    }, { new: true }); // to update it
    console.log(result);
}

updateCourse('6032bdd71dfea40bbc2c7655');
```

## Removing Document

```js
async function removeCourse(id) {
    // const result = await Course.deleteMany({_id: id});
    // const result = await Course.findByIdAndRemove(id);
    const result = await Course.deleteOne({_id: id});
    console.log(result);
}

updateCourse('6032bdd71dfea40bbc2c7655');
```

# Mongo - Data Validation

```js
const courseSchema = new mongoose.Schema({
    name: { type: String, required: true }, // to make it required
});
```

## Built-in Validators

- to make the price required id isPublished is true
```js
// DO NOT USE ARROW FUNCTION
const courseSchema = new mongoose.Schema({
    name: { type: String },
    isPublished: Boolean,
    price: {
        type: Number,
        required: function() { return this.isPublished; }
    }
});
```

```js
const courseSchema = new mongoose.Schema({
    name: { 
        type: String,
        required: true,
        minlength: 5,
        maxlength: 55,
        match: /pattern/
    },
    isPublished: Boolean,
    price: {
        type: Number,
        required: function() { return this.isPublished; }
    }
});
```

```js
const courseSchema = new mongoose.Schema({
    name: { 
        type: String,
        required: true,
        minlength: 5,
        maxlength: 55,
        match: /pattern/
    },
    category: {
        type: String,
        required: true,
        enum: ['web', 'mobile'] // to define a predefinedd categories to choose from
    },
    isPublished: Boolean,
    price: {
        type: Number,
        required: function() { return this.isPublished; },
        min: 10,
        max: 2000
    }
});
```

## Custom Validators

```js
const courseSchema = new mongoose.Schema({
    name: { 
        type: String,
        required: true,
        minlength: 5,
        maxlength: 55,
        match: /pattern/
    },
    isPublished: Boolean,
    tags: {
        type: Array,
        validate: {
            validator: function(value) {
                return v && v.length > 3;
            },
            message: " A course should at least has a tag."
        }
    },
    price: {
        type: Number,
        required: function() { return this.isPublished; }
    }
});
```

## Async Validators

```js
const courseSchema = new mongoose.Schema({
    name: { 
        type: String,
        required: true,
        minlength: 5,
        maxlength: 55,
        match: /pattern/
    },
    isPublished: Boolean,
    tags: {
        type: Array,
        validate: {
            isAsync: true, // set it to true
            validator: function(value, callback) {
                setTimeout(function() {
                    // do some async code
                    const result = v && v.length > 3;
                    callback(result);
                }, 1000);
            },
            message: " A course should at least has a tag."
        }
    },
    price: {
        type: Number,
        required: function() { return this.isPublished; }
    }
});
```

## Validation Erorrs

```js
try {
    const result = await course.save();
    console.log(result);
} catch(ex) {
    for (field in ex.errors) 
        console.log(ex.errors[field].message);
}
```

## Schema Type Options

```js
const courseSchema = new mongoose.Schema({
    name: { 
        type: String,
        required: true,
        minlength: 5,
        maxlength: 55,
        match: /pattern/
    },
    category: {
        type: String,
        required: true,
        enum: ['web', 'mobile'],
        lowercase: true, // to convert the value to true
        // uppercase: true,
        trim: true, // if we have padding
    },
    isPublished: Boolean,
    price: {
        type: Number,
        required: function() { return this.isPublished; },
        get: v => Math.round(v),
        set: v => Math.round(v),
    }
});
```

## Referencing Documents

```js
const Author = mongoose.model('Author', new mongoose.Schema({
    name: String,
    address: String,
    website: String
}));

const Course = mongoose.model('Course', new mongoose.Schema({
    name: String,
    author: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'Author' // then reference the Author collection
    }
}));
```

## Population

```js
// to get the instance of the author itself not just the id
async function listCourses() {
    const courses = await Course.find()
        .populate('author')
        .select('name author');
}
```

```js
// to just get the name from the author object
async function listCourses() {
    const courses = await Course.find()
        .populate('author', 'name')
        .select('name author');
}
```

```js
// to exclude a property from the author object - with minus '-'
async function listCourses() {
    const courses = await Course.find()
        .populate('author', 'name -_id')
        .select('name author');
}
```

## Embedding Documents

```js
const authorSchema = new mongoose.Schema({
    name: String,
    address: String,
    website: String
});

const Author = mongoose.model('Author', authorSchema);

const Course = mongoose.model('Course', new mongoose.Schema({
    name: String,
    author: authorSchema
}));
```

```js
// to make it required
const Course = mongoose.model('Course', new mongoose.Schema({
    name: String,
    author: {
        type: authorSchema,
        required: true
    }
}));
```

## Using an Array of Sub-documents

```js
const authorSchema = new mongoose.Schema({
    name: String,
    address: String,
    website: String
});

const Author = mongoose.model('Author', authorSchema);

const Course = mongoose.model('Course', new mongoose.Schema({
    name: String,
    author: [authorSchema]
}));

async function createCourse(name, authors) {
    const course = new Course({
        name,
        author
    });

    const result = await course.save();
    console.log(result);
}

createCourse('Node Course', [
    new Author({ name: 'Mosh' }),
    new Author({ name: 'John' }),
]);
```
- to add author to the course's authors

```js
const authorSchema = new mongoose.Schema({
    name: String,
    address: String,
    website: String
});

const Author = mongoose.model('Author', authorSchema);

const Course = mongoose.model('Course', new mongoose.Schema({
    name: String,
    author: [authorSchema]
}));

async function createCourse(name, authors) {
    const course = new Course({
        name,
        author
    });

    const result = await course.save();
    console.log(result);
}

async function addAuthor(courseId, author) {
    const course = await Course.findById(courseId);
    course.authors.push(author); // because its array so we can use push
    course.save()
}

addAuthor('5a7051452sg522466225s', new Author({name:'Amy'}))
```

- to remove Author

```js
async function removeAuthor(courseId, author) {
    const course = await Course.findById(courseId);
    const author = course.authors.id(authorId); // to get author with id
    author.remove(); // to remove it 
    course.save()
}

removeAuthor('5a7051452sg522466225s', '15151520521xhbh5215151c')
```

## Transactions

*Means some operations will run together and done successfully of fail together*

- install `npm i fawn`

- the initialize Fawn
```js
const mongoose = require('mongoose');
const Fawn = require('fawn');

Fawn.init(mongoose);
```

- and when we create a new rental
```js
let rental = new Rental({
    customer: {
        _id: customer._id,
        name: customer.name,
        phone: customer.name
    },
    movie: {
        _id: movie._id,
        title: movie.title,
        dailyRentalRate: movie.dailyRentalRate
    }
});

// to perform some tasks that will run together as a unit
try {
    new Fawn.Task()
    .save('rental', rental) // pass the name of the collection then the new document
    .update('movies', { _id: movie._id }, {
        $inc: { numberInStock: -1 }
    }) // to update the movie stuck
    .run(); // finally you need to call run
    // these will run together

    res.send(rental);
}
.catch(ex) {
    res.status(500).send('Something failed.');
}
```

## ObjectID

```js
const mongoose = require('mongoose');

const id = new mongoose.Types.ObjectId();
console.log(id.getTimestamp()) // to get the timestamp

// to check if that's a valid id
mongoose.Types.ObjectId.isValid('1234');
```

## Validating ObjetIDs

- that's not the perfect idea
```js
if(!mongoose.Types.ObjectId.isValid(req.body.customerID)) // if not valid
    return res.status(400).send('Invalid Customer Id');
``` 

- but the best way is using joi

- and to validate with `joi`
- install `npm i joi-objectid` 
```js
const Joi = require('joi');
Joi.objectid = require('joi-objectid')(Joi);

function validateRental(rental) {
    const schema = {
        customerId: Joi.objectId().required(),
        movieId: Joi.objectId().required()
    };

    return Joi.validate(rental, schema);
}
```