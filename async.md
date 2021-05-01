# Asynchronous JavaScript

## Callbacks

```js
console.log('Before');

getUser(1, function(user) {
    console.log('User: ' + user);
});
console.log('After')

function getUser(id, callback) {
    setTimeout(() => {
        console.log('Doing Something...');
        callback({id: 1, message: "Hello"});
    }, 2000);
}
```

## Promises

```js
const p = new Promise((resolve, reject) => {
    // Kick off some async work
    // ...
    resolve(1);
    // reject(new Error('message'))
});

p.then(result => console.log('Result: ' + result)); // should be => Reult: 1
```

- to display the result after two seconds
```js
const p = new Promise((resolve, reject) => {
    // Kick off some async work
    // ...
    setTimeout(() => {
        resolve(1);
    }, 2000);
    // reject(new Error('message'))
});

p.then(result => console.log('Result: ' + result)); // should be => Reult: 1
```

- to display error after two seconds
```js
const p = new Promise((resolve, reject) => {
    // Kick off some async work
    // ...
    setTimeout(() => {
        reject(new Error('message'));
    }, 2000);
    
});

p.then(result => console.log('Result: ' + result))
.catch(err => console.log(err.message));

```














