## 前端测试框架: Jest，较为详细的窥探 
<!-- * 了解 Jest 的核心功能 -->
<!-- * 编写 Demo 功能验证 -->

### 简介

作为目前流行的前端测试框架之一，Test 框架提供很多前端测试任务方法。

### 匹配器（Matchers）

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

### mock 函数

通常我们只要对结果值使用匹配器验证即可，但是也存在一个对`回调函数的验证`。在验证回调函数的正确性可以有几个维度：`回调次数`、`每次回调参数`、`每次回调结果`。

在 Jest 通过 `Jest.fn` 可以完成以上任务的校验，执行后返回一个 mock 函数。
