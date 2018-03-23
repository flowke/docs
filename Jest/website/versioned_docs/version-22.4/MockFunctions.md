---
id: version-22.4-mock-functions
title: Mock Functions
original_id: mock-functions
---

模拟函数通过擦除函数的实际实现，捕获对函数的调用（以及在这些调用中传递的参数 )，实例化对象时捕获实例, 允许返回值的测试时间配置, 很容易的就测试代码之间的关系.

有两种办法模拟函数: 在 test 代码里面创建个模拟函数 或 写一个 [`手动模拟`](ManualMocks.md) 来覆盖模块依赖.

## 使用一个模拟函数

下面有个 `forEach`函数:

```javascript
function forEach(items, callback) {
  for (let index = 0; index < items.length; index++) {
    callback(items[index]);
  }
}
```

To test this function, we can use a mock function, and inspect the mock's state
to ensure the callback is invoked as expected.

要测试这个函数, 我们可以使用一个模拟函数, 并观察模拟的状态来保证回调函数如期运行:

```javascript
const mockCallback = jest.fn();
forEach([0, 1], mockCallback);

// 模拟函数会调用两次
expect(mockCallback.mock.calls.length).toBe(2);

// 第一次调用函数时, 第一个参数是 0
expect(mockCallback.mock.calls[0][0]).toBe(0);

// 第二次调用函数时, 第一个参数是 0
expect(mockCallback.mock.calls[1][0]).toBe(1);

// 第一次调用时, 返回的结果是 42
expect(mockCallback.mock.returnValues[0]).toBe(42);
```

## `.mock` 属性

所有的模拟函数都有个特别的 `.mock` 属性, 里面有函数调用的数据, 函数返回的值. `.mock` 属性同样跟踪每次执行时 `this` 的值, 所以检查如下代码也同样可以:

```javascript
const myMock = jest.fn();

const a = new myMock();
const b = {};
const bound = myMock.bind(b);
bound();

console.log(myMock.mock.instances);
// > [ <a>, <b> ]
```

这些模拟成员会非常有用:

```javascript
// 函数只被执行一次
expect(someMockFunction.mock.calls.length).toBe(1);

// 第一次调用的时第一个参数是: 'first arg'
expect(someMockFunction.mock.calls[0][0]).toBe('first arg');

// 第一次调用的时第二个参数是: 'second arg'
expect(someMockFunction.mock.calls[0][1]).toBe('second arg');

// 第一次调用的返回值: 'return value'
expect(someMockFunction.mock.returnValues[0]).toBe('return value');

// 被实例化了两次
expect(someMockFunction.mock.instances.length).toBe(2);

// 第一次实例化的实例, 有一个 name 属性, 值是: 'test'
expect(someMockFunction.mock.instances[0].name).toEqual('test');
```

## 模拟返回值

可以插入一些返回值:

```javascript
const myMock = jest.fn();
console.log(myMock());
// > undefined

myMock
  .mockReturnValueOnce(10)
  .mockReturnValueOnce('x')
  .mockReturnValue(true);

console.log(myMock(), myMock(), myMock(), myMock());
// > 10, 'x', true, true
```

比如如下使用场景:

```javascript
const filterTestFn = jest.fn();

// 第一次执行返回 true
// 第二次执行返回 false
filterTestFn.mockReturnValueOnce(true).mockReturnValueOnce(false);

const result = [11, 12].filter(filterTestFn);

console.log(result);
// > [11]
console.log(filterTestFn.mock.calls);
// > [ [11], [12] ]
```

Most real-world examples actually involve getting ahold of a mock function on a
dependent component and configuring that, but the technique is the same. In
these cases, try to avoid the temptation to implement logic inside of any
function that's not directly being tested.

## 模拟模块

假设有个类, 里面有个 [axios](https://github.com/axios/axios) 请求:

```js
// users.js
import axios from 'axios';

class Users {
  static all() {
    return axios.get('/users.json').then(resp => resp.data);
  }
}

export default Users;
```

我们可以使用 `jest.mock(...)` 来模拟 axios 模块.

```js
// users.test.js
import axios from 'axios';
import Users from './users';

jest.mock('axios');

test('should fetch users', () => {
  const resp = {data: [{name: 'Bob'}]};
  // 相当于设置模拟请求后返回的数据
  axios.get.mockResolvedValue(resp);

  // or you could use the following depending on your use case:
  // axios.get.mockImplementation(() => Promise.resolve(resp))

  return Users.all().then(users => expect(users).toEqual(resp.data));
});
```

## Mock Implementations

Still, there are cases where it's useful to go beyond the ability to specify
return values and full-on replace the implementation of a mock function. This
can be done with `jest.fn` or the `mockImplementationOnce` method on mock
functions.

```javascript
const myMockFn = jest.fn(cb => cb(null, true));

myMockFn((err, val) => console.log(val));
// > true

myMockFn((err, val) => console.log(val));
// > true
```

The `mockImplementation` method is useful when you need to define the default
implementation of a mock function that is created from another module:

```js
// foo.js
module.exports = function() {
  // some implementation;
};

// test.js
jest.mock('../foo'); // this happens automatically with automocking
const foo = require('../foo');

// foo is a mock function
foo.mockImplementation(() => 42);
foo();
// > 42
```

When you need to recreate a complex behavior of a mock function such that
multiple function calls produce different results, use the
`mockImplementationOnce` method:

```javascript
const myMockFn = jest
  .fn()
  .mockImplementationOnce(cb => cb(null, true))
  .mockImplementationOnce(cb => cb(null, false));

myMockFn((err, val) => console.log(val));
// > true

myMockFn((err, val) => console.log(val));
// > false
```

When the mocked function runs out of implementations defined with
`mockImplementationOnce`, it will execute the default implementation set with
`jest.fn` (if it is defined):

```javascript
const myMockFn = jest
  .fn(() => 'default')
  .mockImplementationOnce(() => 'first call')
  .mockImplementationOnce(() => 'second call');

console.log(myMockFn(), myMockFn(), myMockFn(), myMockFn());
// > 'first call', 'second call', 'default', 'default'
```

For cases where we have methods that are typically chained (and thus always need
to return `this`), we have a sugary API to simplify this in the form of a
`.mockReturnThis()` function that also sits on all mocks:

```javascript
const myObj = {
  myMethod: jest.fn().mockReturnThis(),
};

// is the same as

const otherObj = {
  myMethod: jest.fn(function() {
    return this;
  }),
};
```

## Mock Names

##### available in Jest **22.0.0+**

You can optionally provide a name for your mock functions, which will be
displayed instead of "jest.fn()" in test error output. Use this if you want to
be able to quickly identify the mock function reporting an error in your test
output.

```javascript
const myMockFn = jest
  .fn()
  .mockReturnValue('default')
  .mockImplementation(scalar => 42 + scalar)
  .mockName('add42');
```

## Custom Matchers

Finally, in order to make it simpler to assert how mock functions have been
called, we've added some custom matcher functions for you:

```javascript
// The mock function was called at least once
expect(mockFunc).toBeCalled();

// The mock function was called at least once with the specified args
expect(mockFunc).toBeCalledWith(arg1, arg2);

// The last call to the mock function was called with the specified args
expect(mockFunc).lastCalledWith(arg1, arg2);

// All calls and the name of the mock is written as a snapshot
expect(mockFunc).toMatchSnapshot();
```

These matchers are really just sugar for common forms of inspecting the `.mock`
property. You can always do this manually yourself if that's more to your taste
or if you need to do something more specific:

```javascript
// The mock function was called at least once
expect(mockFunc.mock.calls.length).toBeGreaterThan(0);

// The mock function was called at least once with the specified args
expect(mockFunc.mock.calls).toContain([arg1, arg2]);

// The last call to the mock function was called with the specified args
expect(mockFunc.mock.calls[mockFunc.mock.calls.length - 1]).toEqual([
  arg1,
  arg2,
]);

// The first arg of the last call to the mock function was `42`
// (note that there is no sugar helper for this specific of an assertion)
expect(mockFunc.mock.calls[mockFunc.mock.calls.length - 1][0]).toBe(42);

// A snapshot will check that a mock was invoked the same number of times,
// in the same order, with the same arguments. It will also assert on the name.
expect(mockFunc.mock.calls).toEqual([[arg1, arg2]]);
expect(mockFunc.mock.getMockName()).toBe('a mock name');
```

For a complete list of matchers, check out the [reference docs](ExpectAPI.md).
