---
layout: post
title: "Compile Node.js Application"
tags: [nodejs]
---

Recently I was working on a project to make our production, [Worktile Pro](https://pro.worktile.com), to be able to on-premise deploy. Besides some code changes one of the problem we were facing is how to protect our source code. This can be resolved by compiling to binary.

There are some tools can compile Node.js. The first one we investigated was [jxcore](https://github.com/jxcore/jxcore). But the bad news was, [it's halting active development](http://www.nubisa.com/nubisa-halting-active-development-on-jxcore-platform/).

The second one was [EncloseJS](http://enclosejs.com/). It can compile Node.js source code into binary very quickly. But it's a commercial production. We can use it for free but the output binary has [network connections and process working time limit](http://enclosejs.com/buy). Since our production is a web application which need to be run 7 * 24 with many client connected, `EncloseJS` is not good for us if we don't want to purchase.

Finally we found [nexe](https://jaredallard.me/nexe/). It's open source with MIT license. The compile duration was a little but longer than `EncloseJS` for the first time, but became quickly then. Hence we decided to use `nexe` to compile our application.

### Use `nexe` Command-line

`nexe` is very easy to use. In our case just install it as development dependency through `npm install --save-dev --verb nexe`. Then we can use `./node_modules/.bin/nexe -i <ENTRY FILE> -o <OUTPUT BINARY>` to compile.

For example, I created a Node.js source file named `app.js` as below.

```javascript
// app.js

'use strict';

console.log(`Hello world!`);

const name = 'nexe';
console.log(`This application was compiled from Node.js to binary by ${name}.`);
```

Then ran command `./node_modules/.bin/nexe -i ./app.js -o ./app`. If this is the first time we use `nexe`, it will download Node.js source code in `./tmp` folder, compile Node.js along with out code. This might take 10 - 20 minutes. At the end we will find a file named `app` was created.

![001.png]({{site.baseurl}}/img/2016-08-29-compile-nodejs-application/001.jpg)

Now we can execute this file, which is compiled from our Node.js source code.

![002.png]({{site.baseurl}}/img/2016-08-29-compile-nodejs-application/002.jpg)

If we tried to view the content of this file we can see it's in binary mode.

![003.png]({{site.baseurl}}/img/2016-08-29-compile-nodejs-application/003.jpg)

### How `nexe` Works

If you are familiar with Node.js you might already know that there are some wrappers written in JavaScript in Node.js source code, which helps developers to invoke internal functionalities such as `fs`, `http`, `os`, etc. These JavaScript code are located in `lib` folder in Node.js source code. When we compiled Node.js, it will load all files in `lib` folder and compiled into Node.js binary, in order to have better performance.

`nexe` leverage this feature. When we ran `nexe` it will firstly download the source code of Node.js (if not found in its temporary folder) into `./tmp` folder. Then `nexe` will read our code file the entry file we specified from the command-line `-i` argument. Based on the `require` statement we wrote, `nexe` will find all dependency files and put all of them into a file named under Node.js source code it downloaded `./tmp/nodejs/latest/node-vN.N.N/lib/nexe.js`. Finally `nexe` will start to compile Node.js from `./tmp` folder. Since all our code (with dependencies) are injected into `./lib/nexe.js`, Node.js will treat it as one of build-in wrappers (as same `fs`, `http`, `os`) and compiled into Node.js binary.

![004.png]({{site.baseurl}}/img/2016-08-29-compile-nodejs-application/004.jpg)

Let's add a dependency file as below.

```javascript
// dependency.js

'use strict';

exports.getName = function () {
    return 'nexe';
};
```

And change `app.js` to use it.

```javascript
// app.js

'use strict';

const dependency = require('./dependency');

const name = dependency.getName();
console.log(`This application was compiled from Node.js to binary by ${name}.`);
```

Compile the source code `./node_modules/.bin/nexe -i ./app.js -o ./app` and execute it.

![005.png]({{site.baseurl}}/img/2016-08-29-compile-nodejs-application/005.jpg)

Let's have a look on `nexe.js` file `nexe` created. Since I'm using the latest Node.js version to compile, in this case it's located at `./tmp/nexe/nodejs/latest/node-v6.4.0/lib/nexe.js`.

```javascript
// ./tmp/nexe/nodejs/latest/node-v6.4.0/lib/nexe.js

(function e(t,n,r){function s(o,u){if(!n[o]){if(!t[o]){var a=typeof require=="function"&&require;if(!u&&a)return a(o,!0);if(i)return i(o,!0);var f=new Error("Cannot find module '"+o+"'");throw f.code="MODULE_NOT_FOUND",f}var l=n[o]={exports:{}};t[o][0].call(l.exports,function(e){var n=t[o][1][e];return s(n?n:e)},l,l.exports,e,t,n,r)}return n[o].exports}var i=typeof require=="function"&&require;for(var o=0;o<r.length;o++)s(r[o]);return s})({1:[function(require,module,exports){
'use strict';

const dependency = require('./dependency');

const name = dependency.getName();
console.log(`This application was compiled from Node.js to binary by ${name}.`);
},{"./dependency":2}],2:[function(require,module,exports){
'use strict';

exports.getName = function () {
    return 'nexe';
};
},{}]},{},[1]);

```

We can see that `app.js` and `dependency.js` are all copied into this file with some additional code `nexe.js` generated.

![006.png]({{site.baseurl}}/img/2016-08-29-compile-nodejs-application/006.jpg)

### Passthrough Application Argument

In some cases we need our application to be able to load execution argument. For example, `node app.js --name=nexe`. If we are using [`minimist`](https://github.com/substack/minimist) module this can be done very simple as below.

```javascript
// dependency.js

'use strict';

const argv = require('minimist')(process.argv.slice(2));

exports.getName = function () {
    return argv.name;
};
```

Then we can use `node ./app.js --name=Shaun` to specify the value of `name` as below.

![007.png]({{site.baseurl}}/img/2016-08-29-compile-nodejs-application/007.jpg)

But if we compiled the code and execute `./app --name=Shaun` it will raise an error.

![008.png]({{site.baseurl}}/img/2016-08-29-compile-nodejs-application/008.jpg)

This is because, `nexe` put our code into Node.js source tree and complile it. In fact when we were running `./app` we were running a special version of Node.js contains our code, and our argument `--name` was accepted by Node.js rather than our application.

To let Node.js passthrough the command-line argument into our code, we just need to append `-f` flag when compiling `./node_modules/.bin/nexe -i ./app.js -o ./app -f`.

Now it works as expected when `./app --name=Shaun`.

![009.png]({{site.baseurl}}/img/2016-08-29-compile-nodejs-application/009.jpg)

### Exclude Required Modules

In the sample code above I added `require('minimist')` in `denpendency.js`. If we opened the generated file `nexe.js` we will find the source of `minimist` was also be copied.

![010.png]({{site.baseurl}}/img/2016-08-29-compile-nodejs-application/010.jpg)

We want to compile our source code, but in most cases we don't want to compile node modules we added as well. In this case we can exclude them by passing a variant to `require` function rather than a string. For exmaple we can change `dependency.js` file from `require('minimist')` to `const minimist = 'minimist'` and then `require(minimist)`;

```javascript
// dependency.js

'use strict';

const minimist = 'minimist';
const argv = require(minimist)(process.argv.slice(2));

exports.getName = function () {
    return argv.name;
};
```

Then compile the code and the source of `minimist` was excluded.

![011.png]({{site.baseurl}}/img/2016-08-29-compile-nodejs-application/011.jpg)

Beside using variant as required module name, we can change `require` to `global.require`. This will also exclude the module from compiling.

```javascript
// dependency.js

'use strict';

const argv = global.require(minimist)(process.argv.slice(2));

exports.getName = function () {
    return argv.name;
};
```

>To be aware that you should deploy the source code of those excluded module along with the compile binary.

### Summary

In this post I introduced how to compile Node.js code into binary by using [nexe](https://jaredallard.me/nexe/). Basically `nexe` join our code as well as all required files into `nexe.js` inside the source of Node.js and compile it. Then our code changed to be part of Node.js binary. I also explain how to passthrough the commmand-line into our application, and how to exclude module we don't want to compile.

Hope this helps,

Shaun

>All documents and related graphics, codes are provided "AS IS" without warranty of any kind.
>Copyright © Shaun Xu. This work is licensed under the [Creative Commons License](https://creativecommons.org/licenses/by/3.0/).
