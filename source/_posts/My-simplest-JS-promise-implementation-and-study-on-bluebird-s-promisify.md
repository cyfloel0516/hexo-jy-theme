---
title: My simplest JS promise implementation and explanation on how does Bluebird promisify works
date: 2016-09-06 01:06:52
tags: 
    - JavaScript
    - Promise
    - Node.JS
---

While I was rebuilding my current project with Node these days, I found myself missing ES6 promises in browser so much. The callback hell in Node is really painful but thanks to [Bluebird](https://github.com/petkaantonov/bluebird), their promisify and promisifyAll function works like a charm to make my life better. 

However, I haven't look deeper into the JavaScript promise and I also wonder how Bluebird's promisify works, so I decide to spend couple of hours to take a look at the implementation of JS promise and Bluebird's promisify function.

<!-- more -->
## Implement a tiny promise function
It is easy to implement a promise class with minimal function. I use ES6 class to write it and there are some points need to pay attentation to:
1. ``then(onResolve, onReject)`` function should track the status of promise and behave different. It is because promise object may be resolved very very fast(right after initialization and faster then all 'then' function), in this case, ``then(onResolve, onReject)`` can be executed immediately without push into the waiting array.
2. Don't allow to invoke resolve or reject repeatly. A promise object can be either resolved or rejected only once.
3. ``Constructor(func)`` function will take the real job function as the parameter. We need to use ``bind`` function to attach current promise object to the context of "worker function". The step can allow us to use other properties or functions of the promise object in the "worker function", in our case, we need to use ``promise._done`` there.

Here is the gist of the simple promise implementation:  

{% gist d58367a3b8ae11f0d252e3c73d31246b SimplestPromise.js %}

## How does Bluebird promisify works
Now it is time to explore the Bluebird promisify function and see how does this function works like a magic. 

This function is really easy on the surface, you pass into a Node style function and it will return a promise style function. It means that we dont need to write terrible callback function even for those function without promise feature.

The promisify function is in this [file](https://github.com/petkaantonov/bluebird/blob/master/src/promisify.js). The entrance of this function actually redirect to two kinds of function behind, one is `makeNodePromisifiedEval` and another is `makeNodePromisifiedClosure`, as you can see from the code:

```js
var makeNodePromisified = canEvaluate
    ? makeNodePromisifiedEval
    : makeNodePromisifiedClosure;
//...
//...
function promisify(callback, receiver, multiArgs) {
    return makeNodePromisified(callback, receiver, undefined,
                                callback, null, multiArgs);
}
```

Why does it have two function here? After go through these, I find the if else case is based on if the code is running in the browser. If yes, then it will use `makeNodePromisifiedClosure`, if not then it will use `makeNodePromisifiedEval` for sure. 

The biggest difference between these two functions is that `makeNodePromisifiedEval` will use JS `new Function()` to generate a new promise function dynamically, but `makeNodePromisifiedClosure` will only wrap the original function just like a closure. 

I am not sure about whay they do this but I guess somehow about performance? I will do some further research and update the post in future if I find anything useful. However for now, I will only go through `makeNodePromisifiedClosure` because the code of `makeNodePromisifiedEval` is a litte it complicated and hard to explain in a small post. Anyway, what I can tell you is that the idea of `makeNodePromisifiedEval` is to generate a piece of code of promise function and use JS [`new function()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function) to generate the wrapper function. You can read the source code if you are really interested at this part.

Now the `makeNodePromisifiedClosure` function:

```js
function makeNodePromisifiedClosure(callback, receiver, _, fn, __, multiArgs) {
    var defaultThis = (function() {return this;})();
    var method = callback;
    if (typeof method === "string") {
        callback = fn;
    }
    function promisified() {
        var _receiver = receiver;
        if (receiver === THIS) _receiver = this;
        ASSERT(typeof callback === "function");
        var promise = new Promise(INTERNAL);
        promise._captureStackTrace();
        var cb = typeof method === "string" && this !== defaultThis
            ? this[method] : callback;
        var fn = nodebackForPromise(promise, multiArgs);
        try {
            cb.apply(_receiver, withAppended(arguments, fn));
        } catch(e) {
            promise._rejectCallback(maybeWrapAsError(e), true, true);
        }
        if (!promise._isFateSealed()) promise._setAsyncGuaranteed();
        return promise;
    }
    util.notEnumerableProp(promisified, "__isPromisified__", true);
    return promisified;
}
```

Let's look at the parameter first. From the source code, I think the two parameters matters to us are `callback` and `receiver`.
* `callback`: the function to be wrapped, like `fs.readFile`
* `receiver`: `this` or say `context` of the function execution

Now here are the steps of the promisify process:
1. Create a `promisified` function. This function will be returned as the final result of promisify.
2. In the `promisified` function, it create a JavaScript Promise instance.
3. In the `promisified` function after step 2, it create a function `fn` using `nodebackForPromise` function. `fn` function is just a replacement of original callback function. In `fn`, it will call promise instance's `fullfill` or `reject` after the job is done or failed.
4. When the `promisified` function is called, it uses JavaScript `apply` function to call the origin function but append the replacement callback function it created in step 3 to the end of argument list. So basically, it replace the original callback function with its own promise callback function and return the Promise instance to us. That's why we can use `then` on it.

Now after all steps, we get a promisified function. Then we get the instance of promisify object by calling it. That is basically how Bluebird do promisify job for us and save us from the callback hell. 

## End
I only take few hours to write this post in order to help me remember and better understand the promise. So if there are errors or you have any questions please leave your comment.

Thank you very much for reading!

