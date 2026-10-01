# Introduction
Node.js is a javascript runtie environment.
It runs v8 js engine of google chrome outside the browser. So it allows server side development also in js.
Runs in a single main process. But we can use worker threads for creating multi threads. But it uses multi threaded system internally. But it cannot create a seperate thread for each request.

# Event Loop
It allows node.js to perform non blocking i/o operations.

# Module
Original way is to package js code in node.js is `commonJS modules system`. It is the default one.
```javaScript
const circle = require('./circle.js');
console.log(`The area of a circle of radius 4 is ${circle.area(4)}`);
```
package.json has type `commonjs`
Calling `require()` uses the common js module loader.
Using require() we can load ecmaScript module also but the module should be fully synchronous.


