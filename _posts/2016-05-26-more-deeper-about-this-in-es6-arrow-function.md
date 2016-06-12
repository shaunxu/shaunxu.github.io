---
layout: post
title: "More Deeper about 'this' in ES6 Arrow Function"
tags: [nodejs]
---

When I'm developing [Worktile Pro](https://pro.worktile.com) with ES6 and ES7, arrow function is one of features I like most. This makes me to use arrow function as anywhere as I could. But I also encountered some problem by using it. For example, in [one my previous post](http://geekswithblogs.net/shaunxu/archive/2016/03/15/pay-attention-to-use-es6-arrow-function-with-arguments.aspx), I found `arguments` are not defined in arrow function. Several days ago I found another problem about `this` in arrow function.

Let's have a look on the sample code below. Here I have a module exports two method, one was defined in arrow function, the other was in traditional function.

```javascript
// arrow-fn-and-this.js
'use strict';

exports.arrowFn = () => {
    console.log(this && this.name);
};

exports.normalFn = function () {
    console.log(this && this.name);
};
```

Below is how I was using it.

```javascript
// app.js
'use strict';

const fns = require('./arrow-fn-and-this.js');

const that = {
    name: 'Shaun Xu'
};

const bindArrowFn = fns.arrowFn.bind(that);
const bindNormalFn = fns.normalFn.bind(that);

bindArrowFn();
bindNormalFn();
```

As you can see I `bind` these two function with `that` variable. What we will get when I executed `app.js`?

![001.png]({{site.baseurl}}/img/2016-05-26-more-deeper-about-this-in-es6-arrow-function/001.png)

The method defined in traditional way worked as we expected, but the arrow function worked strange. `this` variable inside the arrow function was `undefined` even though I `bind` to `that`. Why?

The reason is, different from traditional function, `this` in an arrow function is **NOT** a variable. In [MDN document](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions), `this` arrow function is **lexical**. What that mean is, when we define an arrow function, `this` value will be assigned based on where the arrow function was defined, rather than what it will be assigned later when the application was running. `this` in arrow function works more like a constant, rather than a variable. The value of a constant was being decided based on the code, rather than the runtime assignment.

### Update 2016-06-12

>Thanks to [nelsonlaquet](https://github.com/nelsonlaquet/) who pointed me the example below is **NOT** correct. I created a gist as better example and please refer the [comments below](http://blog.shaunxu.me/2016-05-26-more-deeper-about-this-in-es6-arrow-function/#comment-2702277629).

The code below partially explained what `this` works in an arrow function.

```javascript
const wrapper = {
    fn: () => {
        // `this` was defined as the enclosing context, which is `wrapper`, as a constant
        const this = wrapper;
        // ... some codes else
    }
}

fn = () => {
    // since there's no enclosing context, `this` would be `undefined`
    const this = undefined;
    // ... some codes
}
```

But in traditional function, `this` is just a variable which can be assigned in runtime. It works like this.

```javascript
const wrapper = {
    fn: function () {
        var this = retrieve_this_magically(thisArg);
        // ... some codes else using `this`
    }
}
```

When invoking function through `object.method` approach, `retrieve_this_magically` will be the `object` and assigned to `this`. When using `Function.prototype.bind`, `Function.prototype.apply` and `Function.prototype.call`, it will assign `this` based the first parameter of `bind`, `apply` and `call`.~~

So to be summary, arrow function is great and it reduce the complexity of our code. But when using arrow function, pay attention to its **lexical** variables:
- arguments
- super
- this
- new.target
They are defined and assigned based on the code, and cannot be changed in runtime.

Hope this helps,
Shaun
