---
id: version-22.4-using-matchers
title: Using Matchers
original_id: using-matchers
---

Jest 使用 "matchers" 以不同方式来测试值.  这份文档会告诉你匹配器通常的一些用法, 查看[`expect` API 文档](ExpectAPI.md).

### Common Matchers

测试一个值最简单的方式就是使用看看它们相等么?

```js
test('two plus two is four', () => {
  expect(2 + 2).toBe(4);
});
```

 `expect(2 + 2)` 返回了一个 "expectation" 对象. 通常你只是在这里调用匹配器, `.toBe(4)` 就是匹配器. 当 Jest 运行时, 它会跟踪所有失败的匹配器, 这样就可以为你打印错误信息.

`toBe` 使用 `Object.is` 来测试是不是绝对相等. 如果你想确认的是个对象, 那就用 `toEqual`:

```js
test('object assignment', () => {
  const data = {one: 1};
  data['two'] = 2;
  expect(data).toEqual({one: 1, two: 2});
});
```

`toEqual` 会递归检查对象或数组的每一个字段:

你也可以测试一个匹配器的反面:

```js
test('adding positive numbers is not zero', () => {
  for (let a = 1; a < 10; a++) {
    for (let b = 1; b < 10; b++) {
      expect(a + b).not.toBe(0);
    }
  }
});
```

### 真值

有时候你要区分 `undefined`, `null`, and
`false`, 但又想同等对待. Jest 有一些
helpers 帮助你.

* `toBeNull` 只匹配 `null`
* `toBeUndefined` 只匹配 `undefined`
* `toBeDefined` 是 `toBeUndefined` 的对立面.
* `toBeTruthy` 匹配任何 `if` 声明返回 true 的情况.
* `toBeFalsy` 匹配任何 `if` 声明返回 false 的情况.

For example:

```js
test('null', () => {
  const n = null;
  expect(n).toBeNull();
  expect(n).toBeDefined();
  expect(n).not.toBeUndefined();
  expect(n).not.toBeTruthy();
  expect(n).toBeFalsy();
});

test('zero', () => {
  const z = 0;
  expect(z).not.toBeNull();
  expect(z).toBeDefined();
  expect(z).not.toBeUndefined();
  expect(z).not.toBeTruthy();
  expect(z).toBeFalsy();
});
```

### 数字

比较数字也有相应的匹配器.

```js
test('two plus two', () => {
  const value = 2 + 2;
  expect(value).toBeGreaterThan(3);
  expect(value).toBeGreaterThanOrEqual(3.5);
  expect(value).toBeLessThan(5);
  expect(value).toBeLessThanOrEqual(4.5);

  // toBe and toEqual are equivalent for numbers
  expect(value).toBe(4);
  expect(value).toEqual(4);
});
```

对于浮点数, 用 `toBeCloseTo` 而不是 `toEqual`.

```js
test('adding floating point numbers', () => {
  const value = 0.1 + 0.2;
  //expect(value).toBe(0.3);           This won't work because of rounding error
  expect(value).toBeCloseTo(0.3); // This works.
});
```

### 字符串

可以用 `toMatch` 来根据正则表达式检查字符串:

```js
test('there is no I in team', () => {
  expect('team').not.toMatch(/I/);
});

test('but there is a "stop" in Christoph', () => {
  expect('Christoph').toMatch(/stop/);
});
```

### 数组

可以使用 `toContain` 来检查数组是否包含某个元素:

```js
const shoppingList = [
  'diapers',
  'kleenex',
  'trash bags',
  'paper towels',
  'beer',
];

test('the shopping list has beer on it', () => {
  expect(shoppingList).toContain('beer');
});
```

### 异常

使用 `toThrow` 来检查 某个函数是否抛出了的异常.

```js
function compileAndroidCode() {
  throw new ConfigError('you are using the wrong JDK');
}

test('compiling android goes as expected', () => {
  expect(compileAndroidCode).toThrow();
  expect(compileAndroidCode).toThrow(ConfigError);

  // You can also use the exact error message or a regexp
  expect(compileAndroidCode).toThrow('you are using the wrong JDK');
  expect(compileAndroidCode).toThrow(/JDK/);
});
```

### And More

这只是一个小简介, 去文档看看: [reference docs](ExpectAPI.md).

了解匹配器之后, 就可以看看 [测试异步代码](TestingAsyncCode.md).
