---
layout: post
title: "Handle List & Array of Rest Parameters"
tags: [nodejs]
---

ECMAScript 6 introduced a new feature called "Rest Parameters" which allow us to represent an indefinite number of arguments as an array. For exmaple the last parameter of function, `others` was prefixing with 3 dots `...`. This means all following arguments will be filled in `others` as an array.

```javascript
const fn = function (base, ...others) {
    console.log(JSON.stringify(others, null, 2));

	let result = base;
	for (let other of others) {
		result += other;
	}
	return result;
};
```

If we invoked `fn` with arguments `fn(1, 2, 3, 4, 5)`, the variant `base` will be `1` while `others` will be `[2, 3, 4, 5]`. 

```javascript
const result_by_param_list = fn(1, 2, 3, 4, 5);
console.log(`fn(1, 2, 3, 4, 5) = ${result_by_param_list}`);
```

The code above will print like this.

```
[
  2,
  3,
  4,
  5
]
fn(1, 2, 3, 4, 5) = 15
```

But in some cases we may want to pass the rest arguments as an array rather than a list. For example

```javascript
const result_by_param_array = fn(1, [2, 3, 4, 5]);
console.log(`fn(1, [2, 3, 4, 5]) = ${result_by_param_array}`);
```

But if we specified the rest parameters as array we got result like this, which is incorrect.

```
[
  [
    2,
    3,
    4,
    5
  ]
]
fn(1, [2, 3, 4, 5]) = 12,3,4,5
```

This is because JavaScript load parameters from the second one and push them into an array as the rest parameter variant `others`. This bahaviour is different from C#. In C# if we define rest parameters it will check the type of parameters, push all followings into an array, or just use that parameter if it's an array.

```csharp
function fn (int base, params int[] others) {
    ......
}
```

How to make JavaScript handle the rest parameters correct when both list or array? That's simple. JavaScript always push rest parameters into an array. So we can just faltten the rest parameters. That means, if we specified parameters as a list, `other` is an array and it will be flattened with no change; if we specified as an array, it will be a 2-level array (an array which the first element is the input array) and will be falttened in 1-level.

This can be done by using the `flatten` method in module `lodash` as below.

```javascript
const _ = require('lodash');

const fn = function (base, ...others) {
	others = _.flatten(others);

	let result = base;
	for (let other of others) {
		result += other;
	}
	return result;
};
```

Or can be done by using the build-in array method `reduce`.

```javascript
const fn = function (base, ...others) {
	others = others.reduce((x, y) => x.concat(y), []);

	let result = base;
	for (let other of others) {
		result += other;
	}
	return result;
};
```
Now the result will be correct when we spcifying the rest parameters in both list style or array style.

```javascript
const result_by_param_list = fn(1, 2, 3, 4, 5);
console.log(`fn(1, 2, 3, 4, 5) = ${result_by_param_list}`);

const result_by_param_array = fn(1, [2, 3, 4, 5]);
console.log(`fn(1, [2, 3, 4, 5]) = ${result_by_param_array}`);
```

The result is

```
fn(1, 2, 3, 4, 5) = 15
fn(1, [2, 3, 4, 5]) = 15
```

Hope this helps,
Shaun