---
layout: post
title: "ES7 Async/Await in Node.js with Babel.js"
tags: [nodejs]
---

Our production [Worktile Pro](https://pro.worktile.com) was built based on Node.js 0.12 when we firstly developed and at that moment we tried our best to prevent our code from the style below.

![001.png]({{site.baseurl}}/img/2016-06-14-es7-async-await-in-node-with-bebel/001.jpg)

Which is usually called [Callback Hell](http://callbackhell.com/).

But things have been got changed since we started to refactor our Messaging Service based on the latest version of Node.js and [`Bebel.js`](https://babeljs.io/). With Node.js v6 we can use iterator and [tj](https://github.com/tj)'s [`co`](https://github.com/tj/co) to flatten the callback hell code.

```javascript
co (function *() {
    const db = yield MongoClient.connect();
    const tasks = yield db.collections('tasks').findMany({
        project: new ObjectID(req.params.pid)
    }).toArray();
    yield db.disconnect();

    // following operations against `tasks`
});
```

The code above looks great so far, but we wanted to step forward a bit more. In ES7 proposal it introduced a new feature regarding asynchronous programming, formally called [Asynchronous Iterators](https://github.com/tc39/proposal-async-iteration), which we always called **Async/Await**. With the power of [`Bebel.js`](https://babeljs.io/) we can upgrade the above code like this.

```javascript
const doSomethingAgainstTasks = async function () {
    const db = await MongoClient.connect();
    const tasks = await db.collections('tasks').findMany({
        project: new ObjectID(req.params.pid)
    }).toArray();
    await db.disconnect();

    // following operations against `tasks`
};
```

This code works similar as the previous one. When Node.js invokes a function with `async` marked, it means the result might be returned in the future. And when it runs the code with `await` marks, it will pause current workflow and waiting for the function result after `await` to be fulfilled. Once the result is ready, the workflow will be resumed and executes until it arrive another `await`, or end of the `async` function.

It works like a magic. The code runs very smart to pause and wait, and then resumed. But what happened underlying?

### Async

When I added `async` keyword in front of a function, it will be wrapped by a [Promise](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise). When this function was returned, it will be tweaked to something like `return resolve(result);`. And when we through an error, it will be `return reject(error);`. For example, I created an async function, invoked and print the value.

```javascript
const myAsyncFn = async function (x, y) {
    return x + y;
};
const result = myAsyncFn(3, 5);
console.log(result);
```

When we executed with Node.js v6 and `Babel.js`, the result is **NOT** 8, but an instance of a Promise.

```
> async-await-es7@1.0.0 run /Users/shaun/Documents/Playground/async-await-es7
> babel-node app.js --presets es2015-node5,stage-3

Promise { _c: [], _a: undefined, _s: 1, _d: true, _v: 8, _h: 0, _n: true }
8
```

In this case we need to handler `Promise.then` to retrieve the result.

```javascript
const myAsyncFn = async function (x, y) {
    return x + y;
};
const result = myAsyncFn(3, 5);
result.then((value) => {
    console.log(value);
});
```

### Await

Similarly, `await` keyword will assume its following expression should return a Promise. It will handle `Promise.then`, assign the value if it has, handle `Promise.catch`, invokes current `Promise`'s `return reject(error);` to terminate workflow. For example the code below just `await` a number.

```javascript
(async function () {
    const value = await 123;
    console.log(value);
})().then(() => {
    console.log('Done');
});
```

It works like `const value = await Promise.resolve(123);`.

```
> async-await-es7@1.0.0 run /Users/shaun/Documents/Playground/async-await-es7
> babel-node app.js --presets es2015-node5,stage-3

123
Done
```

The code below might make more sense.

```javascript
(async function () {
    const value = await new Promise((resolve, reject) => {
        setTimeout(() => {
            return resolve(123);
        }, 1000);
    });
    console.log(value);
})().then(() => {
    console.log('Done');
});
```

It will display `123` 1 second later.

```
> async-await-es7@1.0.0 run /Users/shaun/Documents/Playground/async-await-es7
> babel-node app.js --presets es2015-node5,stage-3

123
Done
```

### Async / Await

As we talked `async` will always return a Promise while `await` always assume following a Promise. This makes our coding more easily by using both of them.

```javascript
const myAsyncFn = async function (x, y) {
    return new Promise((resolve) => {
        setTimeout(() => {
            return resolve(x + y);
        });
    });
};

(async function () {
    const value = await myAsyncFn(3, 5);
    console.log(value);
})().then(() => {
    console.log('Done');
});
```

The code below can be rewritten without `async` and `await`.

```javascript
const myAsyncFn = function (x, y) {
    return new Promise((resolve) => {
        setTimeout(() => {
            return resolve(x + y);
        });
    });
};

myAsyncFn(3, 5).then((value) => {
    console.log(value);
}).then(() => {
    console.log('Done');
});
```

Hope this helps,
Shaun
