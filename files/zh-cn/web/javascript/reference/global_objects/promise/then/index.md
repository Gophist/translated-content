---
title: Promise.prototype.then()
slug: Web/JavaScript/Reference/Global_Objects/Promise/then
tags:
  - ECMAScript 2015
  - JavaScript
  - Method
  - Promise
  - Prototype
browser-compat: javascript.builtins.Promise.then
---
{{JSRef}}

**`then()`** 方法返回一个{{jsxref("Promise")}}对象。它最多有两个变量，分别是`Promise`在成功和失败情形下的回调函数。

{{EmbedInteractiveExample("pages/js/promise-then.html")}}

> **注意:** 如果上述的一个或两个变量被忽略了或被赋值为了非函数，那么`then` 会缺少处理器函数（handler），但不会报错。
> 当调用`then`的`Promise`被兑现（fulfillment）或拒绝（reject），且`then`没有处理器函数（handler）
> 返回的`Promise`会接受上述调用`then`的`Promise`的最终状态。

## 语法

```js
p.then(onFulfilled[, onRejected]);

p.then(value => {
  // 兑现Promise时，执行的函数
}, reason => {
  // 拒绝Promise时，执行的函数
});
```

### Parameters

- `onFulfilled` `可选`
  - : 当`Promise`被兑现（fulfilled）时所调用的{{jsxref("Function")}}。这个函数有一个参数：`兑现值（fulfillment value）`。
    如果它不是一个函数，它会返回接受的参数，被自身函数替代。
- `onRejected` `可选`
  - : 当`Promise`被拒绝（rejected）时所调用的{{jsxref("Function")}}。这个函数有一个参数：`拒绝原因（rejection reason）`。
    如果它不是一个函数，它会被替换为抛出函数，即抛出它收到的错误作为参数。

### 返回值

一旦兑现或拒绝一个期约{{jsxref("Promise")}}，`兑现（onFulfilled）` 或 `拒绝（onRejected）`的处理函数会被**异步**调用（安排在当前线程循环）。
处理器函数的行为遵从一系列特别的规则。如果一个处理函数：

- 返回一个值，`then`所返回的promise接受返回值
- 不返还任何值，doesn't return anything, the promise returned by `then` gets resolved
  with an `undefined` value.
- 抛出错误throws an error, the promise returned by `then` gets rejected with the
  thrown error as its value.
- 返回一个已经实现的期约promise，returns an already fulfilled promise, the promise returned by `then` gets
  fulfilled with that promise's value as its value.
- returns an already rejected promise, the promise returned by `then` gets
  rejected with that promise's value as its value.
- returns another **pending** promise object, the resolution/rejection of
  the promise returned by `then` will be subsequent to the
  resolution/rejection of the promise returned by the handler. Also, the resolved value
  of the promise returned by `then` will be the same as the resolved value of
  the promise returned by the handler.

Following, an example to demonstrate the asynchronicity of the `then`
method.

```js
// using a resolved promise, the 'then' block will be triggered instantly,
// but its handlers will be triggered asynchronously as demonstrated by the console.logs
const resolvedProm = Promise.resolve(33);

let thenProm = resolvedProm.then(value => {
    console.log("this gets called after the end of the main stack. the value received and returned is: " + value);
    return value;
});
// instantly logging the value of thenProm
console.log(thenProm);

// using setTimeout we can postpone the execution of a function to the moment the stack is empty
setTimeout(() => {
    console.log(thenProm);
});

// logs, in order:
// Promise {[[PromiseStatus]]: "pending", [[PromiseValue]]: undefined}
// "this gets called after the end of the main stack. the value received and returned is: 33"
// Promise {[[PromiseStatus]]: "resolved", [[PromiseValue]]: 33}
```

## 描述Description

As the `then` and {{jsxref("Promise.prototype.catch()")}} methods return
promises, they [can be
chained](/en-US/docs/Web/JavaScript/Guide/Using_promises#Chaining) — an operation called _composition_.

## Examples

### Using the `then` method

```js
var p1 = new Promise((resolve, reject) => {
  resolve('Success!');
  // or
  // reject(new Error("Error!"));
});

p1.then(value => {
  console.log(value); // Success!
}, reason => {
  console.error(reason); // Error!
});
```

### 链式调用

The `then` method returns a `Promise` which allows for method
chaining.

If the function passed as handler to `then` returns a `Promise`,
an equivalent `Promise` will be exposed to the subsequent `then`
in the method chain. The below snippet simulates asynchronous code with the
`setTimeout` function.

```js
Promise.resolve('foo')
  // 1. Receive "foo", concatenate "bar" to it, and resolve that to the next then
  .then(function(string) {
    return new Promise(function(resolve, reject) {
      setTimeout(function() {
        string += 'bar';
        resolve(string);
      }, 1);
    });
  })
  // 2. receive "foobar", register a callback function to work on that string
  // and print it to the console, but not before returning the unworked on
  // string to the next then
  .then(function(string) {
    setTimeout(function() {
      string += 'baz';
      console.log(string); // foobarbaz
    }, 1)
    return string;
  })
  // 3. print helpful messages about how the code in this section will be run
  // before the string is actually processed by the mocked asynchronous code in the
  // previous then block.
  .then(function(string) {
    console.log("Last Then:  oops... didn't bother to instantiate and return " +
                "a promise in the prior then so the sequence may be a bit " +
                "surprising");

    // Note that `string` will not have the 'baz' bit of it at this point. This
    // is because we mocked that to happen asynchronously with a setTimeout function
    console.log(string); // foobar
  });

// logs, in order:
// Last Then: oops... didn't bother to instantiate and return a promise in the prior then so the sequence may be a bit surprising
// foobar
// foobarbaz
```

When a value is returned from within a `then` handler, it will effectively
return
`Promise.resolve(<value returned by whichever handler was called>)`.

```js
var p2 = new Promise(function(resolve, reject) {
  resolve(1);
});

p2.then(function(value) {
  console.log(value); // 1
  return value + 1;
}).then(function(value) {
  console.log(value + ' - A synchronous value works'); // 2 - A synchronous value works
});

p2.then(function(value) {
  console.log(value); // 1
});
```

A `then` call will return a rejected promise if the function throws an error
or returns a rejected Promise.

```js
Promise.resolve()
  .then(() => {
    // Makes .then() return a rejected promise
    throw new Error('Oh no!');
  })
  .then(() => {
    console.log('Not called.');
  }, error => {
    console.error('onRejected function called: ' + error.message);
  });
```

In all other cases, a resolving Promise is returned. In the following example, the
first `then()` will return `42` wrapped in a resolving Promise
even though the previous Promise in the chain was rejected.

```js
Promise.reject()
  .then(() => 99, () => 42) // onRejected returns 42 which is wrapped in a resolving Promise
  .then(solution => console.log('Resolved with ' + solution)); // Resolved with 42
```

In practice, it is often desirable to catch rejected promises rather than use
`then`'s two case syntax, as demonstrated below.

```js
Promise.resolve()
  .then(() => {
    // Makes .then() return a rejected promise
    throw new Error('Oh no!');
  })
  .catch(error => {
    console.error('onRejected function called: ' + error.message);
  })
  .then(() => {
    console.log("I am always called even if the prior then's promise rejects");
  });
```

You can also use chaining to implement one function with a Promise-based API on top of
another such function.

```js
function fetch_current_data() {
  // The fetch() API returns a Promise.  This function
  // exposes a similar API, except the fulfillment
  // value of this function's Promise has had more
  // work done on it.
  return fetch('current-data.json').then(response => {
    if (response.headers.get('content-type') != 'application/json') {
      throw new TypeError();
    }
    var j = response.json();
    // maybe do something with j
    return j; // fulfillment value given to user of
              // fetch_current_data().then()
  });
}
```

If `onFulfilled` returns a promise, the return value of `then`
will be resolved/rejected by the promise.

```js
function resolveLater(resolve, reject) {
  setTimeout(function() {
    resolve(10);
  }, 1000);
}
function rejectLater(resolve, reject) {
  setTimeout(function() {
    reject(new Error('Error'));
  }, 1000);
}

var p1 = Promise.resolve('foo');
var p2 = p1.then(function() {
  // Return promise here, that will be resolved to 10 after 1 second
  return new Promise(resolveLater);
});
p2.then(function(v) {
  console.log('resolved', v);  // "resolved", 10
}, function(e) {
  // not called
  console.error('rejected', e);
});

var p3 = p1.then(function() {
  // Return promise here, that will be rejected with 'Error' after 1 second
  return new Promise(rejectLater);
});
p3.then(function(v) {
  // not called
  console.log('resolved', v);
}, function(e) {
  console.error('rejected', e); // "rejected", 'Error'
});
```

### window\.setImmediate style promise-based polyfill

Using a {{jsxref("Function.prototype.bind()")}} `Reflect.apply`
({{jsxref("Reflect.apply()")}}) method to create a (non-cancellable)
{{domxref("window.setImmediate")}}-style function.

```js
const nextTick = (() => {
  const noop = () => {}; // literally
  const nextTickPromise = () => Promise.resolve().then(noop);

  const rfab = Reflect.apply.bind; // (thisArg, fn, thisArg, [...args])
  const nextTick = (fn, ...args) => (
    fn !== undefined
    ? Promise.resolve(args).then(rfab(null, fn, null))
    : nextTickPromise(),
    undefined
  );
  nextTick.ntp = nextTickPromise;

  return nextTick;
})();
```

## Specifications

{{Specifications}}

## 浏览器适配Browser compatibility

{{Compat}}

## See also

- {{jsxref("Promise")}}
- {{jsxref("Promise.prototype.catch()")}}
