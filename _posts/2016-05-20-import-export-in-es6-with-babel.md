---
layout: post
title: "Import & Export in ES6 with Node.js & Babel.js"
tags: [nodejs]
---

ES6 introduced a new way to organize modules. It's different from `CommonJS` and `AMD` we are currently using, which through the new keywords `import` and `export`. It still **NOT** being supported by the latest version of Node.js. But fortunately we can leverage `Babel.js` to play it right now.

When we are rewriting the next version of Instant Message module of our production - [Worktile Pro](https://pro.worktile.com), we used this feature under Node.js v6 and `Babel.js`. So in this post I would like to introduce what it is and how I was using.

### Development Environment with `Babel.js`
[`Bebel.js`](https://babeljs.io/) is an open source JavaScript compiler which allows us to use latest features, especially something still in draft running in lower level runtime. In our case, we need `Babel.js` to translate and build our backend source code written with ES6 and ES7 features, to the code in ES6 that compatible with Node.js v6.

Below is part of the `package.json` file we are using.

```json
{
  "name": "Worktile Pro IM",
  "version": "1.0.0",
  "main": "app.js",
  "scripts": {
    "run": "babel-node app.js",
    "build": "babel . -d .dist --ignore=\"node_modules\""
  },
  "devDependencies": {
    "babel-cli": "*",
    "babel-core": "*",
    "babel-preset-es2015-node5": "*",
    "babel-preset-stage-3": "*",
    "babel-register": "*"
  }
}
```

Also we need `.babelrc` file to define the behaviours of `Babel.js` as below.

```json
{
  "presets": ["es2015-node5", "stage-3"],
  "plugins": []
}
```

You can copy this file and use `npm install` to prepare your working environment. If you want to run your code with `Babel.js` without compiling, you can simple use `npm run-script run`, then `babel-cli` will compile your source code and execute in memory. This is good when development but not in production, since it will use a lot memory and downside the performance of your application. When you are good to your code, you can run `npm run-script build` to let `Babel.js` build the source code to an output folder (In my case it's `./.dist/`) when you can `node ./.dist/app.js`.

For more information about `Babel.js`, `babel-node` and `Babel-cli` please refer to [Babel's document](https://babeljs.io/docs/usage/cli/).

### Traditional CommomJS
Let's create a very simple application with traditional Node.js modules in `CommonJS` style. Below is `calc.js` file which will be used later.

```js
'use strict';

exports.name = 'calc';
exports.add = (x, y) => {
    return x + y;
};
exports.subtract = (x, y) => {
    return x - y;
};
exports.multiple = (x, y) => {
    return x * y;
};
exports.divide = (x, y) => {
    return x / y;
};
```

Below is how we can use it.

```js
'use strict';

const calc = require('./calc');

const x = 3;
const y = 5;

console.log(`${calc.name}`);

const result_add = calc.add(x, y);
console.log(`${x} + ${y} = ${result_add}`);

const result_subtract = calc.subtract(x, y);
console.log(`${x} - ${y} = ${result_subtract}`);

const result_multiple = calc.multiple(x, y);
console.log(`${x} * ${y} = ${result_multiple}`);

const result_divide = calc.divide(x, y);
console.log(`${x} / ${y} = ${result_divide}`);
```

Since it's in ES5 syntax we can simply execute by `node app.js`.

![001.png]({{site.baseurl}}/img/2016-05-20-import-export-in-es6-with-babel/001.png)

### Export Variables & Import
ES6 `export` works very similar with the way we are using previously. Basically we can use `export` keyword to any variables defined in the module source code. For example, we can export `name` by using `export const name = 'calc';`.
So this module can be upgrade as below.

```js
'use strict';

export const name = 'calc';

export const add = (x, y) => {
    return x + y;
};

export const subtract = (x, y) => {
    return x - y;
};

export const multiple = (x, y) => {
    return x * y;
};

export const divide = (x, y) => {
    return x / y;
};
```

When using this module we need `import` keyword. Similar as `require` we need to specify the path of this module and assign as a variable. Then we can use functions defined in `calc.js` as usual.

```js
'use strict';

import * as calc from './calc';

const x = 3;
const y = 5;

console.log(`${calc.name}`);

const result_add = calc.add(x, y);
console.log(`${x} + ${y} = ${result_add}`);

const result_subtract = calc.subtract(x, y);
console.log(`${x} - ${y} = ${result_subtract}`);

const result_multiple = calc.multiple(x, y);
console.log(`${x} * ${y} = ${result_multiple}`);

const result_divide = calc.divide(x, y);
console.log(`${x} / ${y} = ${result_divide}`);
```

Since we are using `export` and `import` keywords which is **NOT** supported in Node.js v6, we will got error if just run it by `node app.js`.

![002.png]({{site.baseurl}}/img/2016-05-20-import-export-in-es6-with-babel/002.png)

We need use `Babel.js` to compile it to the code Node.js supports and run it by using `npm run-script run`, and you can see it works.

![003.png]({{site.baseurl}}/img/2016-05-20-import-export-in-es6-with-babel/003.png)

In the code below, we use `import * as calc`, which means it will import all variables this module exports, as properties of `calc`. Alternatively we can just import some variables we want and use then as separate variables as below.

```js
'use strict';

import {name, add, subtract} from './calc';

const x = 3;
const y = 5;

console.log(`${name}`);

const result_add = add(x, y);
console.log(`${x} + ${y} = ${result_add}`);

const result_subtract = subtract(x, y);
console.log(`${x} - ${y} = ${result_subtract}`);
```

![004.png]({{site.baseurl}}/img/2016-05-20-import-export-in-es6-with-babel/004.png)

### Default Export
Sometimes we need to export just one variable. In this case we use `export default`. For example, in the code below I wrapped all variables into one object and exported as default.

```js
'use strict';

export default {
    name: 'calc',
    add: (x, y) => {
        return x + y;
    },
    subtract: (x, y) => {
        return x - y;
    },
    multiple: (x, y) => {
        return x * y;
    },
    divide: (x, y) => {
        return x / y;
    }
};
```

Now we can import it into a variable.

```js
'use strict';

import calc from './calc';

const x = 3;
const y = 5;

console.log(`${calc.name}`);

const result_add = calc.add(x, y);
console.log(`${x} + ${y} = ${result_add}`);

const result_subtract = calc.subtract(x, y);
console.log(`${x} - ${y} = ${result_subtract}`);

const result_multiple = calc.multiple(x, y);
console.log(`${x} * ${y} = ${result_multiple}`);

const result_divide = calc.divide(x, y);
console.log(`${x} / ${y} = ${result_divide}`);
```

![005.png]({{site.baseurl}}/img/2016-05-20-import-export-in-es6-with-babel/005.png)

Default export is very useful when exporting a class. For example, the code below we defined our `calc.js` as a class and export.
>Note when exporting a class, do **NOT** append semi-comma at the end of the code.

```js
'use strict';

export default class Calc {
    constructor (x, y) {
        this.x = x;
        this.y = y;
    }

    add () {
        return this.x + this.y;
    }

    subtract () {
        return this.x - this.y;
    }

    multiple () {
        return this.x * this.y;
    }

    divide () {
        return this.x / this.y;
    }
}
```

Below is the code we are using this class.

```js
'use strict';

import Calc from './calc';

const x = 3;
const y = 5;
const calc = new Calc(x, y);

const result_add = calc.add();
console.log(`${x} + ${y} = ${result_add}`);

const result_subtract = calc.subtract();
console.log(`${x} - ${y} = ${result_subtract}`);

const result_mutiple = calc.mutiple();
console.log(`${x} * ${y} = ${result_mutiple}`);

const result_divide = calc.divide();
console.log(`${x} / ${y} = ${result_divide}`);
```

### Summary
In this post I described the `import` and `export` feature in ES6 and how we are using it in Node.js v6 with `Babel.js`. Basically it doesn't provide meaningful enhancement comparing with the original `CommonJS` module system. But with the upgrade of Node.js and web browsers, this feature should be used widely and replace current `CommonJS` and `AMD` I think.

Hope this helps,
Shaun
