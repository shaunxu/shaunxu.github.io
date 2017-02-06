---
layout: post
title: "Try RxJS in Node.js"
tags: [nodejs]
---

Reactive Extension (Rx), which is introduced by Microsoft, is a library for composing asynchronous and event-based programs, by using `observable`, `observer` with LINQ-style query operators. If LINQ is the first generation on how people deal with data, Rx should be next generation.

In LINQ, everything can be considered as iterables. Records in database are iterable, elements in an array are iterable, nodes in an XML file are iterable, and even more, response from an HTTP request could be iterable, too. But LINQ only takes care of synchronous operation. When we need to deal with asynchronous operation how can we leverage the idea of LINQ? The answer is Rx.

In Rx, everything can be considered as observable, any input can be considered as stream. User input and click events are stream, HTTP requests are stream, timer elapsed events are steam. We can use Rx to subscribe these steams, append hander function when data came in.

Rx is implemented in many language, such as C# (Rx.NET), C++ (RxCpp), Python (RxPy), Objective-C (Rx.ObjC) and JavaScript (RxJS). There are many examples on how to use RxJS in the internet, but I found almost all of them are front-end, which means it shows how to use RxJS to simplify DOM operations and browser events. But as a backend developer I would like to show how to use RxJS in Node.js.

### Use RxJS to Handle Readline Input

As I mentioned, in Rx everything can be considered as stream and what we need to do is to subscribe them. Below I would like to use `readline` as the first example. This small application will display what user input with the date in front of. The code below were implemented in traditional way, I mean, `callback`.

```javascript
'use strict';

const readline = require('readline');
const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout
});

rl.on('line', (line) => {
    // feature 1: echo
    console.log(`[${Date.now()}] you said: "${line}".`);

    rl.prompt();
}).on('error', (error) => {
    console.log(`[${Date.now()}] error: "${error}".`);
}).on('close', () => {
    console.log(`[${Date.now()}] bye.`);
});

rl.prompt();
```

Now let's add one feature, upper-case and reverse user's input. Well we can add 2 lines of code after the code for our first feature.

```javascript
'use strict';

const readline = require('readline');
const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout
});

rl.on('line', (line) => {
    // feature 1: echo
    console.log(`[${Date.now()}] you said: "${line}".`);

    // feature 2: uppercase
    const reversed_line = line.toUpperCase();
    console.log(`[${Date.now()}] you said (reversed): "${reversed_line}".`);

    rl.prompt();
}).on('error', (error) => {
    console.log(`[${Date.now()}] error: "${error}".`);
}).on('close', () => {
    console.log(`[${Date.now()}] bye.`);
});

rl.prompt();
```

The code is clean and it seems no necessary to introduce Rx. Now let's add another feature. With this feature, we don't want our application to show user's input at once. In face we want our application to `buffer` user's input and show it every 3 seconds, but do not show anything if no input in the past 3 seconds.

In order to implement this new feature we need to initialize an array to buffer user's input. We need to push user's input into this buffer. We also need a timer which will be triggered every 3 seconds. And once it triggered, we need to check if there's anything in the buffer, show and clear. We also need to stop the timer when error occurred. The code would be like this.

```javascript
'use strict';

const readline = require('readline');
const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout
});

// feature 3 (partial): batch echo every 3 seconds
let buf = [];
const interval = setInterval(() => {
    if (buf && buf.length > 0) {
        console.log(`[${Date.now()}] you said (in past 3 seconds): "${buf.join(', ')}".`);
        buf = [];
    }
}, 3000);

rl.on('line', (line) => {
    // feature 1: echo
    console.log(`[${Date.now()}] you said: "${line}".`);

    // feature 2: uppercase
    const reversed_line = line.toUpperCase();
    console.log(`[${Date.now()}] you said (reversed): "${reversed_line}".`);

    // feature 3 (partial): batch echo every 3 seconds
    buf.push(line);

    rl.prompt();
}).on('error', (error) => {
    clearInterval(interval); // feature 3 (partial): stop timer if error occurred
    console.log(`[${Date.now()}] error: "${error}".`);
}).on('close', () => {
    clearInterval(interval); // feature 3 (partial): stop timer if application was closed
    console.log(`[${Date.now()}] bye.`);
});

rl.prompt();
```

It works. But the code for feature 3 are separated into 4 pieces: the buffer array and timer construction, put user's input into buffer, clear timer when error occurred or when application was closed.

Now let's have a look on how to implement it using Rx. First of all we need to import Rx module `const Rx = require('rx');`. Then we need to treat user's input as a steam. In RxJS, `Subject` is an object can wrap anything as a stream.

> In theory `Subject` implements `Observable` and `Observer` which can be `subscribe`ed. You can find more information about `Subject` at https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/subjects/subject.md

```javascript
'use strict';

const readline = require('readline');
const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout
});

const Rx = require('rx');
const subject = new Rx.Subject();
rl.on('line', (line) => {
    subject.onNext(line);
    rl.prompt();
}).on('error', (error) => {
    subject.onError(error);
}).on('close', () => {
    subject.onCompleted();
});
```

In the code above we created a `Subject` and when user input through `readline`, we invoked 3 methods in `Subject`. `onNext` was invoked when user input something, which will make our `subject` push data; `onError` was invoked when `readline` failed, which will make `subject` sent an error. And `onCompleted` was invoked when `readline` was close, which will close our `subject` as well.

Now we can implement features based on the `subject` we created, rather than the `readline` module. In order to display user's input, we just need to `subscribe` our `subject` and display the content.

![Screen Shot 2017-02-06 at 13.50.56.png]({{site.baseurl}}/img/2017-01-21-try-rxjs-in-nodejs/Screen Shot 2017-02-06 at 13.50.56.png)

```javascript
'use strict';

const readline = require('readline');
const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout
});

const Rx = require('rx');
const subject = new Rx.Subject();
rl.on('line', (line) => {
    subject.onNext(line);
    rl.prompt();
}).on('error', (error) => {
    subject.onError(error);
}).on('close', () => {
    subject.onCompleted();
});

// feature 1: echo
subject.subscribe((line) => {
    console.log(`[${Date.now()}] you said: "${line}".`);
});

rl.prompt();
```

In order to upper-case user's input, we can `subscribe` our `subject`, modify the content and display. But this is not the best way. In Rx, any stream can be filtered, mapped or buffered, etc., and the result is another stream that we can subscribe. So what we should do is to based on the original subject, generate the new stream we interested in, `subscribe` it.

![Screen Shot 2017-02-06 at 13.54.32.png]({{site.baseurl}}/img/2017-01-21-try-rxjs-in-nodejs/Screen Shot 2017-02-06 at 13.54.32.png)

The code below shows how to `map` the original stream to upper-case and `subscribe`.

```javascript
...

// feature 1: echo
...

// feature 2: uppercase
subject.map((line) => {
    return line.toUpperCase();
}).subscribe((line) => {
    console.log(`[${Date.now()}] you said (reversed): "${line}".`);
});

rl.prompt();
```

Similarly, for feature 3 we could `buffer` the original stream every 3 seconds. It will group user's input during this 3 seconds in an array. Then we `filter` this new stream if the content array is not empty, which only shows if user input in 3 seconds. Finally we `subscribe` it and display this stream, which is buffered every 3 seconds and not empty.

![Screen Shot 2017-02-06 at 14.06.06.png]({{site.baseurl}}/img/2017-01-21-try-rxjs-in-nodejs/Screen Shot 2017-02-06 at 14.06.06.png)

```javascript
...

// feature 1: echo
...

// feature 2: uppercase
...

// feature 3: batch echo every 3 seconds
subject.buffer(() => {
    return Rx.Observable.timer(3000);
}).filter((lines) => {
    return lines.length > 0;
}).subscribe((lines) => {
    console.log(`[${Date.now()}] you said (in past 3 seconds): "${lines.join(', ')}".`);
});

rl.prompt();
```

### Benefit of Using Rx

From the code below we can see that by using Rx, we only need to focus our independent business logic. Rx abstracts inputs as stream that can be mapped. This means we do not need to combine different logic into one place with the same data source. For example the feature 3 we previously talked. In traditional way we have to put the code alone with 2 other features, but some parts of the implementation had to be outside.

```javascript
'use strict';

...

// feature 3
let buf = [];
const interval = setInterval(() => {
    if (buf && buf.length > 0) {
        console.log(`[${Date.now()}] you said (in past 3 seconds): "${buf.join(', ')}".`);
        buf = [];
    }
}, 3000);

rl.on('line', (line) => {
    // feature 1: echo
    ...

    // feature 2: uppercase
    ...

    // feature 3
    buf.push(line);

    rl.prompt();
}).on('error', (error) => {
    clearInterval(interval); // feature 3
    ...
}).on('close', () => {
    clearInterval(interval); // feature 3
    ...
});

...
```

If we uses RxJS, we only need to subscribe the stream (or mapped stream) a feature needs and put all of the code inside its handler function.

```javascript
'use strict';

...

// feature 1: echo
subject.subscribe((line) => {
    console.log(`[${Date.now()}] you said: "${line}".`);
});

// feature 2: uppercase
subject.map((line) => {
    return line.toUpperCase();
}).subscribe((line) => {
    console.log(`[${Date.now()}] you said (reversed): "${line}".`);
});

// feature 3: batch echo every 3 seconds
subject.buffer(() => {
    return Rx.Observable.timer(3000);
}).filter((lines) => {
    return lines.length > 0;
}).subscribe((lines) => {
    console.log(`[${Date.now()}] you said (in past 3 seconds): "${lines.join(', ')}".`);
});

rl.prompt();
```

And we can evenly append more features without changing any existing code. For example, if I want to add a debounce feature which means if user submit input too frequently (within 0.5 second), it will not be displayed. I only need to `debounce` the original `subject` and subscribe it.

```javascript
'use strict';

...

// feature 1: echo
...

// feature 2: uppercase
...

// feature 3: batch echo every 3 seconds
...

// feature 4: debounce 1 second
subject.debounce(500).subscribe((line) => {
    console.log(`[${Date.now()}] you said (debounce 0.5 second): "${line}".`);
});

rl.prompt();
```

Besides, when using built-in functions of RxJS we can perform operations against a stream such as filter, buffer, map, etc. But they will **NOT** modify the original stream. This means for a stream we can operate and subscribe it multiple times.

![Screen Shot 2017-02-06 at 14.28.14.png]({{site.baseurl}}/img/2017-01-21-try-rxjs-in-nodejs/Screen Shot 2017-02-06 at 14.28.14.png)

### Use RxJS with Socket.IO

Now let's take another example more reality. As you may know our production [Worktile](https://worktile.com) uses Socket.IO to implement realtime communication. I will demonstrate how to use RxJS with Socket.IO.

In order to focus this post about Node.js I created 2 files, `ws-server.rx.js` is Socket.IO server and `ws-client.rx.js` is a Socket.IO console client.

In server side, we firstly initialize socket service.

```javascript
// ws-server.rx.js
'use strict';

const Rx = require('rx');

const app = require('http').createServer();
const io = require('socket.io')(app);
```

Then we created a RxJS `Subject` instance. And when a client was connected we emit a connection data element to our `subject`. When the client was disconnected we emit a disconnect data element to our `subject` as well.

```javascript
// ws-server.rx.js

...

const subject = new Rx.Subject();

io.on('connection', (socket) => {
    console.log(`CONNECTION: ${socket.id}`);

    subject.onNext({
        socket: socket,
        type: 'online'
    });

    socket.on('disconnect', () => {
        subject.onNext({
            socket: socket,
            type: 'offline'
        });
    });
});
```

And when we received message from the client, we emit a message data element to our `subject`.

```javascript
// ws-server.rx.js

...

const subject = new Rx.Subject();

io.on('connection', (socket) => {
    console.log(`CONNECTION: ${socket.id}`);

    subject.onNext( ... );

    socket.on('disconnect', () => { ... });

    socket.on('message', (message) => {
        subject.onNext({
            socket: socket,
            type: 'message',
            index: message.index,
            content: message.content
        });
    });
});
```

**Feature 1: Send message to all clients when a new client connected.**

This can be implemented by filter the `subject` that data type equals `online`, subscribe and broadcast message to all clients.

```javascript
// ws-server.rx.js

...

subject.filter((x) => {
    return x.type === 'online';
}).subscribe(
    (x) => {
        console.log(`${x.socket.id} joint.`);
        x.socket.broadcast.emit('message', `${x.socket.id} online.`);
    }
);
```

**Feature 2: Send message to all clients when a new client disconnected.**

Similarly, this can be implemented by filter the `subject` that data type equals `offline`, subscribe and broadcast message to all clients.

```javascript
// ws-server.rx.js

...

...

subject.filter((x) => {
    return x.type === 'offline';
}).subscribe(
    (x) => {
        console.log(`${x.socket.id} left.`);
        x.socket.broadcast.emit('message', `${x.socket.id} offline.`);
    }
);
```

**Feature 3: Broadcast message from any clients every 3 seconds.**

This is very similar to the 3rd feature in previous example. In that case our application will buffer user's input every 3 seconds and display. In this case, our server will buffer all clients' message every 3 seconds and broadcast.

```javascript
// ws-server.rx.js

...

...

...

subject.filter((x) => {
    return x.type === 'message';
}).buffer(() => {
    return Rx.Observable.timer(3000);
}).filter((xs) => {
    return xs.length > 0;
}).subscribe(
    (xs) => {
        for (const x of xs) {
            console.log(`${x.socket.id} say: "(${x.index}) ${x.content}".`);
            x.socket.broadcast.emit('message', `${x.socket.id} say: "(${x.index}) ${x.content}".`);            
        }
    }
);
```

Finally, start the server and listen port `22222`. The full code would be like this.
```javascript
'use strict';

const Rx = require('rx');

const app = require('http').createServer();
const io = require('socket.io')(app);
const subject = new Rx.Subject();
io.on('connection', (socket) => {
    console.log(`CONNECTION: ${socket.id}`);

    subject.onNext({
        socket: socket,
        type: 'online'
    });

    socket.on('disconnect', () => {
        subject.onNext({
            socket: socket,
            type: 'offline'
        });
    });

    socket.on('message', (message) => {
        subject.onNext({
            socket: socket,
            type: 'message',
            index: message.index,
            content: message.content
        });
    });
});

subject.filter((x) => {
    return x.type === 'online';
}).subscribe(
    (x) => {
        console.log(`${x.socket.id} joint.`);
        x.socket.broadcast.emit('message', `${x.socket.id} online.`);
    }
);

subject.filter((x) => {
    return x.type === 'offline';
}).subscribe(
    (x) => {
        console.log(`${x.socket.id} left.`);
        x.socket.broadcast.emit('message', `${x.socket.id} offline.`);
    }
);

subject.filter((x) => {
    return x.type === 'message';
}).buffer(() => {
    return Rx.Observable.timer(3000);
}).filter((xs) => {
    return xs.length > 0;
}).subscribe(
    (xs) => {
        for (const x of xs) {
            console.log(`${x.socket.id} say: "(${x.index}) ${x.content}".`);
            x.socket.broadcast.emit('message', `${x.socket.id} say: "(${x.index}) ${x.content}".`);            
        }
    }
);

app.listen(22222);
console.log(`ready`);
```

Switch to the client side code. Firstly we create a `Subject` which wrap `readline` module to accept user's input. It will be subscribed in future once connected to the server.

```javascript
// ws-client.rx.js

'use strict';

const Rx = require('rx');

let index = 0;

const readline = require('readline');
const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout
});
const readlineSubject = new Rx.Subject();
rl.on('line', (line) => {
    readlineSubject.onNext({
        index: index++,
        content: line
    });
    rl.prompt();
}).on('error', (error) => {
    readlineSubject.onError(error);
}).on('close', () => {
    readlineSubject.onCompleted();
});
```

Then connect to Socket.IO server.

```javascript
// ws-client.rx.js

...

const socketSubject = new Rx.Subject();
const socket = require('socket.io-client')('ws://localhost:22222/', {
    transports: [
        'websocket'
    ]
});
```

Next, we will create another `subject` based on this `socket` client object when it's connected, disconnected and received any messages from server, and send data with different types.

```javascript
// ws-client.rx.js

...

...

socket.on('connect', () => {
    socketSubject.onNext({
        type: 'online'
    });

    socket.on('disconnect', () => {
        socketSubject.onNext({
            type: 'offline'
        });
    });

    socket.on('message', (message) => {
        socketSubject.onNext({
            type: 'message',
            content: message
        });
    });
});
```

We filter the socket subject and accept those type equals `offline` and display `bye` in console, which means the client will show `bye` when disconnected. We also filter the socket subject and accept those type equals `message` and display it, which means when any messages sent from server (say something from any other clients), it will be displayed on this client.

```javascript
// ws-client.rx.js

...

...

...

socketSubject.filter((x) => {
    return x.type === 'message';
}).subscribe((x) => {
    console.log(x.content);
});

socketSubject.filter((x) => {
    return x.type === 'offline';
}).subscribe(() => {
    console.log(`bye.`);
});
```

And we should filter the socket subject and accept those type equals `online`, which means when this client was connected to the server, we started to subscribe the readline subject and emit the content to server.

```javascript
// ws-client.rx.js

...

...

...

...

socketSubject.filter((x) => {
    return x.type === 'online';
}).subscribe(() => {
    console.log(`${socket.id} joint.`);

    readlineSubject.subscribe((line) => {
        console.log(`${socket.id} say: "(${line.index}) ${line.content}".`);
        socket.emit('message', line);
    }, () => {}, () => {
        socket.disconnect();
    });
});
```

Finally we start to prompt the readline for user input. The full code would be like this.

```javascript
// ws-client.rx.js

'use strict';

const Rx = require('rx');

let index = 0;

const readline = require('readline');
const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout
});
const readlineSubject = new Rx.Subject();
rl.on('line', (line) => {
    readlineSubject.onNext({
        index: index++,
        content: line
    });
    rl.prompt();
}).on('error', (error) => {
    readlineSubject.onError(error);
}).on('close', () => {
    readlineSubject.onCompleted();
});

const socketSubject = new Rx.Subject();
const socket = require('socket.io-client')('ws://localhost:22222/', {
    transports: [
        'websocket'
    ]
});
socket.on('connect', () => {
    socketSubject.onNext({
        type: 'online'
    });

    socket.on('disconnect', () => {
        socketSubject.onNext({
            type: 'offline'
        });
    });

    socket.on('message', (message) => {
        socketSubject.onNext({
            type: 'message',
            content: message
        });
    });
});

socketSubject.filter((x) => {
    return x.type === 'online';
}).subscribe(() => {
    console.log(`${socket.id} joint.`);

    readlineSubject.subscribe((line) => {
        console.log(`${socket.id} say: "(${line.index}) ${line.content}".`);
        socket.emit('message', line);
    }, () => {}, () => {
        socket.disconnect();
    });
});

socketSubject.filter((x) => {
    return x.type === 'message';
}).subscribe((x) => {
    console.log(x.content);
});

socketSubject.filter((x) => {
    return x.type === 'offline';
}).subscribe(() => {
    console.log(`bye.`);
});

rl.prompt();
```

Now we can start the server and 3 clients, chatting.

![Screen Shot 2017-02-06 at 16.23.15.gif]({{site.baseurl}}/img/2017-01-21-try-rxjs-in-nodejs/Screen Shot 2017-02-06 at 16.23.15.gif)

### Summary

Microsoft said `Rx = Observables + LINQ + Schedulers`. I don't think it's easy to understand. In my mind, Rx treat everything in program are stream. A stream is a subject that can be subscribed. And what we should do is to build a stream based on one or more original stream, implement what should do when data feed. Your code should never be executed unless data came. And even more, you should only perform your code from a specific steam, which might be an original stream, or being built on top of multiple originals.

This post only demonstrated how to try RxJS with Node.js, not within a browser. More information about Rx and RxJS please find its official website https://github.com/Reactive-Extensions. And there's an online book for RxJS I recommended https://xgrommx.github.io/rx-book/.

Hope this helps,
Shaun
