## 前端测试框架：Jest (一) - 介绍 
<!-- * 了解 Jest 的核心功能 -->
<!-- * 编写 Demo 功能验证 -->

### 简介

作为目前流行的前端测试框架之一，Test 框架提供比较完善的断言库，强大的 mock 功能，包含测试覆盖率工具，开箱即用。

### 匹配器 (Matchers)

在验证测试结果和预期值的时，用匹配器验证更加方便，支持的匹配值类型有：Boolean、Number、String、Array、Iterables等常见类型，提供很多匹配方法，还有更多高级其它的匹配用法。

#### 常见类型的匹配

##### Normal

``` javascript
test('测试 2 + 2  是否为 4', () => {
  expect(2 + 2).toBe(4);
});
```

##### Boolean

``` javascript
test('null', () => {
  const n = null;
  expect(n).toBeNull();
  expect(n).toBeDefined();
  expect(n).not.toBeUndefined();
  expect(n).not.toBeTruthy();
  expect(n).toBeFalsy();
});
```

##### Number

``` javascript
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

##### String

``` javascript
test('there is no I in team', () => {
  expect('team').not.toMatch(/I/);
  
});

test('but there is a "stop" in Christoph', () => {
  expect('Christoph').toMatch(/stop/);
});
```

##### Array 和 iterables

``` javascript
const shoppingList = [
  'diapers',
  'kleenex',
  'trash bags',
  'paper towels',
  'milk',
];

test('the shopping list has milk on it', () => {
  expect(shoppingList).toContain('milk');
  expect(new Set(shoppingList)).toContain('milk');
});

```

#### 更多高级的匹配

##### Throw 的 匹配

``` javascript
function compileAndroidCode() {
  throw new Error('you are using the wrong JDK');
}

test('compiling android goes as expected', () => {
  expect(() => compileAndroidCode()).toThrow();
  expect(() => compileAndroidCode()).toThrow(Error);

  // You can also use the exact error message or a regexp
  expect(() => compileAndroidCode()).toThrow('you are using the wrong JDK');
  expect(() => compileAndroidCode()).toThrow(/JDK/);
});
```

> 抛出异常的函数需要被包裹在一个函数里

#### 自定义匹配器

``` javascript
expect.extend({
  yourMatcher(x, y, z) {
    return {
      pass: true,
      message: () => '',
    };
  },
});
```

### 异步匹配

官方提供了 `done` 函数方式和 `promise` 方式，显然 promise 更为简单直接。

``` javascript
// 期望异步数据结果为：'peanut butter'
test('the data is peanut butter', () => {
  return fetchData().then(data => {
    expect(data).toBe('peanut butter');
  });
});
// 支持 Async/Await
test('the data is peanut butter', async () => {
  const data = await fetchData();
  expect(data).toBe('peanut butter');
});

// 期望异步获取一个异常
test('the fetch fails with an error', () => {
  expect.assertions(1); 
  return fetchData().catch(e => expect(e).toMatch('error'));
});

```

> `expect.assertions(1);` 确保当前测试至少执行了1次断言，也就是 catch 函数至少执行了一次。

### Setup & Teardown

在单个 test 的时候，执行之前和执行之后可以通过勾子函数处理一些任务。

#### 多次测试重复设置

``` javascript
beforeEach(() => {
  initializeCityDatabase();
});

afterEach(() => {
  clearCityDatabase();
});

test('测试1', () => {
  expect(isCity('Vienna')).toBeTruthy();
});
test('测试2', () => {
  expect(isCity('San Juan')).toBeTruthy();
});
```

`每`执行一个 test，beforeEach 和 afterEach `都`被调用一次。

#### 一次设置

``` javascript
beforeAll(() => {
  return initializeCityDatabase();
});

afterAll(() => {
  return clearCityDatabase();
});

test('测试1', () => {
  expect(isCity('Vienna')).toBeTruthy();
});
test('测试1', () => {
  expect(isCity('San Juan')).toBeTruthy();
});
```

`无论`执行多少个 beforeAll 和 afterAll `只`被调用一次。

#### 作用域 describe

可以通过 describe 做影响范围的分组。

``` javascript
describe('matching cities to foods', () => {
  // Applies only to tests in this describe block
  beforeEach(() => {
    return initializeFoodDatabase();
  });

  test('Vienna <3 sausage', () => {
    expect(isValidCityFoodPair('Vienna', 'Wiener Schnitzel')).toBe(true);
  });

  test('San Juan <3 plantains', () => {
    expect(isValidCityFoodPair('San Juan', 'Mofongo')).toBe(true);
  });
});
```

#### 执行顺序

同级别下： beforeAll > beforeEach > afterEach > afterAll

任何情况： describe > test，这里指的优先执行 describe 回调函数，但不会执行回调里的 test 的处理方法。

### 强大的 Mock 功能

当测试对象具有外部依赖性时，可能想要“模拟它们”。“模拟”是将代码的某些依赖项替换为自己的实现。

测试绕不过 mock，它能带来多个好处：

* 在开发工作流中，可以使软件开发与测试程序编写`并行`进行；
* mock 出边界情况，提高测试覆盖率 (Testing coverage) ；
* 对于三方受限资源或环境系统不稳定情况下，mock 所需数据，进行测试程序编写；

Jest 中的 mock 功能允许测试代码之间的连接，实现方式有：`擦除函数实现`、`捕获对函数的调用`、`捕获调用构造函数后的实例` (new 方式) 、`测试运行时的返回值配置`等等。

Jest 中，通过 `jest.fn` 可以生成一个 mock 函数。

#### 1. mock 函数调用

##### 1-1. 获取函数调用信息

下面通过 mock 函数， 模拟 callback 函数。在 callback 被调用时，会捕获函数的相关调用信息。

通过 `mockImplementation()` 方法可以 mock `实现 (Implementation)`，在 mock 函数被执行的时候，实现也会被执行。

> jest.fn(implementation) 是 jest.fn().mockImplementation(implementation) 的简写。

``` javascript
function forEach(items, callback) {
  for (let index = 0; index < items.length; index++) {
    callback(items[index]);
  }
}
// 生成 mock 函数
const mockCallback = jest.fn(x => 42 + x);
// 测试 函数
forEach([0, 1], mockCallback);

// 断言：callback 的调用次数期望是2。
expect(mockCallback.mock.calls.length).toBe(2);

// 断言：callback 在第1次被调用时，第1个入参期望是0。
expect(mockCallback.mock.calls[0][0]).toBe(0);

// 断言：callback 在第2次被调用时，第1个入参期望是1。
expect(mockCallback.mock.calls[1][0]).toBe(1);

// 断言：callback 在第1次被调用时，返回值期望是2。
expect(mockCallback.mock.results[0].value).toBe(42);
```

所有的 mock 函数都有一个 mock 属性，如上可以获取调用信息。

##### 1-2. 返回值配置

这里允许在测试函数期间，mock 出`不同的调用次数下，配置不同的返回值`，并支持链式调用，这种风格更加容易阅读和理解。

``` javascript
const myMock = jest.fn();
console.log(myMock()); 
// > undefined

myMock.mockReturnValueOnce(10).mockReturnValueOnce('x').mockReturnValue(true);

console.log(myMock(), myMock(), myMock(), myMock());
// > 10, 'x', true, true
```

##### 1-3. 函数调用和返回值配置共同使用

``` javascript
const filterTestFn = jest.fn();

// mock 出第1次 filterTestFn 被调用时返回 true，第2次被调用时返回 false.
filterTestFn.mockReturnValueOnce(true).mockReturnValueOnce(false);

const result = [11, 12].filter(num => filterTestFn(num));

console.log(result);
// > [11]
console.log(filterTestFn.mock.calls[0][0]); // 11
console.log(filterTestFn.mock.calls[0][1]); // 12
```

##### 1-4. 获取实例

``` javascript
const mockFn = jest.fn();

const a = new mockFn();
const b = new mockFn();

mockFn.mock.instances[0] === a; // true
mockFn.mock.instances[1] === b; // true
```

#### 2. mock 异步

``` javascript
// 语法糖 🍬
jest.fn().mockImplementation(() => Promise.resolve(value));

// 使用
test('async test1', async () => {
  const asyncMock = jest.fn().mockResolvedValue(43);
  await asyncMock(); // 43
});

// 支持链式调用
test('async test2', async () => {
  const asyncMock = jest
    .fn()
    .mockResolvedValue('default')
    .mockResolvedValueOnce('first call')
    .mockResolvedValueOnce('second call');

  await asyncMock(); // first call
  await asyncMock(); // second call
  await asyncMock(); // default
  await asyncMock(); // default
});
```

#### 3. mock 模块

可以通过 `jest.mock('module_name')`，来模拟模块。

``` javascript
// users.js
import axios from 'axios';

class Users {
  static all() {
    return axios.get('/users.json').then(resp => resp.data);
  }
}

export default Users;
```

``` javascript
// users.test.js
import axios from 'axios';
import Users from './users';

jest.mock('axios'); // 显示调用

test('should fetch users', () => {
  const users = [{name: 'Bob'}];
  const resp = {data: users};
  axios.get.mockResolvedValue(resp);

  // or you could use the following depending on your use case:
  // axios.get.mockImplementation(() => Promise.resolve(resp))

  return Users.all().then(data => expect(data).toEqual(users));
});
```

### 常用工具库

#### 1. jest-changed-files

用于校验最后一次 commit 中，哪些文件发生改变。

``` javascript
import {getChangedFilesForRoots} from 'jest-changed-files';

getChangedFilesForRoots(['/path/to/test'], {
  lastCommit: true,
  withAncestor: true,
}).then(files => {
  /*
  {
    repos: [],
    changedFiles: []
  }
  */
});

```

#### 2. jest-diff

比较两个任意值，并打印美化过的不同的地方。

``` javascript
const diff = require('jest-diff').default;

const a = {a: {b: {c: 5}}};
const b = {a: {b: {c: 6}}};

const result = diff(a, b);

// print diff
console.log(result);

```

#### 3. jest-docblock

导出 javascript 文件顶部的注释。

``` javascript
const {parseWithComments} = require('jest-docblock');

const code = `
/**
 * This is a sample
 *
 * @flow
 */

 console.log('Hello World!');
`;

const parsed = parseWithComments(code);

// prints an object with two attributes: comments and pragmas.
console.log(parsed);
```

#### 4. jest-get-type

获取值类型。

``` javascript
const getType = require('jest-get-type');

const array = [1, 2, 3];
const nullValue = null;
const undefinedValue = undefined;

// prints 'array'
console.log(getType(array));
// prints 'null'
console.log(getType(nullValue));
// prints 'undefined'
console.log(getType(undefinedValue));
```

#### 5. jest-validate

用于处理验证中的错误、警告、废弃等消息，并可以指定用户正确的配置。

``` javascript
const {validate} = require('jest-validate');

const configByUser = {
  transform: '<rootDir>/node_modules/my-custom-transform',
};

const result = validate(configByUser, {
  comment: '  Documentation: http://custom-docs.com',
  exampleConfig: {transform: '<rootDir>/node_modules/babel-jest'},
});

console.log(result);
```

#### 6. jest-worker

给测试提供多线程能力

``` javascript
// heavy-task.js
module.exports = {
  myHeavyTask: args => {
    // long running CPU intensive task.
  },
};
```

``` javascript
// main.js
async function main() {
  const worker = new Worker(require.resolve('./heavy-task.js'));

  // run 2 tasks in parallel with different arguments
  const results = await Promise.all([
    worker.myHeavyTask({foo: 'bar'}),
    worker.myHeavyTask({bar: 'foo'}),
  ]);

  console.log(results);
}

main();
```

#### 7. pretty-format

格式化输出可序列化的 javascript 内建类型。

``` javascript
const {format: prettyFormat} = require('pretty-format');

const val = {object: {}};
val.circularReference = val;
val[Symbol('foo')] = 'foo';
val.map = new Map([['prop', 'value']]);
val.array = [-0, Infinity, NaN];

console.log(prettyFormat(val));
/*
Object {
  "array": Array [
    -0,
    Infinity,
    NaN,
  ],
  "circularReference": [Circular],
  "map": Map {
    "prop" => "value",
  },
  "object": Object {},
  Symbol(foo): "foo",
}
*/
```
