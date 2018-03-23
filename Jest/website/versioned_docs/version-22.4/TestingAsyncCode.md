---
id: version-22.4-asynchronous
title: Testing Asynchronous Code
original_id: asynchronous
---

运行异步代码的时候, Jest 需要知道所测试的代码什么时候完成, 这样才能进行下一条测试. Jest 有一些办法来处理这种情况.

### 回调

通常的异步模式就是回调.

例如, 你有个 `fetchData(callback)` 函数去请求数据, 完成的时候就会调用 `callback(data)`. 你想测试返回的数据是不是字符串:  `'peanut butter'`.

通常, 代码执行完成后 Jest 就完成测试, 如下测试不会成功:

```js
// Don't do this!
test('the data is peanut butter', () => {
  function callback(data) {
    expect(data).toBe('peanut butter');
  }

  fetchData(callback);
});
```

因为 `fetchData` 执行后就测试完成了. 等不到回调函数的执行.

 `test` 有个备用方法修正这种情况. 函数里添加一个 `done` 参数. `done` 执行之后才是测试完成.


```js
test('the data is peanut butter', done => {
  function callback(data) {
    expect(data).toBe('peanut butter');
    done();
  }

  fetchData(callback);
});
```

如果 `done()` 永远都不会调用, 那么就是测试失败了. 这是我们期望的结果.

### Promises

使用 Promise, 有一个简单的方式处理异步测试. 在 test 里面返回一个 Promise, Jest 就会等待 Promise resolve. 如果 Promise rejected, 测试就失败了.

例如下面代码:

```js
test('the data is peanut butter', () => {
  expect.assertions(1);
  return fetchData().then(data => {
    expect(data).toBe('peanut butter');
  });
});
```

千万记住把 promise return 出去.

如果期望 promise rejected , 就用 `.catch` 方法. 确保添加
`expect.assertions`. 不然, Promise fulfilled 后不会让这一次 test 失败.

```js
test('the fetch fails with an error', () => {
  expect.assertions(1);
  return fetchData().catch(e => expect(e).toMatch('error'));
});
```

### `.resolves` / `.rejects`

##### available in Jest **20.0.0+**

你也可以在 expect 声明里使用 `.resolves` 匹配器, Jest 会等到 promise resolve. 如果 promise rejected, test 会失败.

```js
test('the data is peanut butter', () => {
  expect.assertions(1);
  return expect(fetchData()).resolves.toBe('peanut butter');
});
```

同理, 以下是`.rejects` 匹配器.

```js
test('the fetch fails with an error', () => {
  expect.assertions(1);
  return expect(fetchData()).rejects.toMatch('error');
});
```

### Async/Await

在 test 也可以用 `async` 和 `await`:

```js
test('the data is peanut butter', async () => {
  expect.assertions(1);
  const data = await fetchData();
  expect(data).toBe('peanut butter');
});

test('the fetch fails with an error', async () => {
  expect.assertions(1);
  try {
    await fetchData();
  } catch (e) {
    expect(e).toMatch('error');
  }
});
```

当然, 你可以让 `async` 和 `await` 搭配 `.resolves` 或 `.rejects` 一起用.
(available in Jest **20.0.0+**).

```js
test('the data is peanut butter', async () => {
  expect.assertions(1);
  await expect(fetchData()).resolves.toBe('peanut butter');
});

test('the fetch fails with an error', async () => {
  expect.assertions(1);
  await expect(fetchData()).rejects.toMatch('error');
});
```

这些例子了, `async` and `await` 这个语法糖非常高效.

以上各种测试异步的形式无所谓哪个好于不好. 这取决那种方式能够让你更简单的进行测试.
