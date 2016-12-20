---
layout: post
title: "Partial Class in JavaScript"
tags: [nodejs]
---

We should NOT write a lot of code inside one file, or in one class, not only in JavaScript in theory. But in practice we might need to write a huge class. I know in C# we can split one class in multiple files, which they called _partial class_. How can we do this in JavaScript?

That's simple. Assuming I have a class called `Huge`. Below is the definition file `huge.js`.

```javascript
// huge.js

'use strict'

const HugeClass = function () {};

HugeClass.prototype.method1 = function () {
    // ...
};

HugeClass.prototype.method2 = function () {
    // ...
};

HugeClass.prototype.method2 = function () {
    // ...
};

module.exports = HugeClass;
```

Now I would like to move method `huge1` into `huge.partial-1.js`, but also has it defined in `Huge` class. `huge.partial-1.js` would be like this.

```javascript
// huge.partial-1.js

'use strict';

module.exports = function (HugeClass) {
    HugeClass.prototype.method1 = function () {
        // ...
    };
};
```

As you can see, I passed the class definition `HugeClass` into this file and defined its method. Then in `huge.js` what I need to do is to simply require `huge.partial-1.js` and invoke it.

```javascript
// huge.js

'use strict'

const HugeClass = function () {};

require('./huge.partial-1')(HugeClass);

HugeClass.prototype.method2 = function () {
    // ...
};

HugeClass.prototype.method2 = function () {
    // ...
};

module.exports = HugeClass;
```

Similarly I can put `method2` and `method3` into 2 partial files.

```javascript
// huge.partial-2.js

'use strict';

module.exports = function (HugeClass) {
    HugeClass.prototype.method2 = function () {
        // ...
    };
};
```

```javascript
// huge.partial-3.js

'use strict';

module.exports = function (HugeClass) {
    HugeClass.prototype.method3 = function () {
        // ...
    };
};
```

And innvoke (combine) them into `huge.js`.


```javascript
// huge.js

'use strict'

const HugeClass = function () {};

require('./huge.partial-1')(HugeClass);
require('./huge.partial-2')(HugeClass);
require('./huge.partial-3')(HugeClass);

module.exports = HugeClass;
```

Hope this helps,
Shaun
