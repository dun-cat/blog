## ECMAScript 标准提纲手册 
下面是从 ES5 到最新的 ECMAScript 标准（ES2024）中新增的主要内容的提纲。这些提纲涵盖了每个版本中的主要特性和改进。

### ECMAScript 5 (ES5) - 2009

- 严格模式 (`"use strict";`)
- JSON 支持
- 更好的属性定义
  - `Object.defineProperty`
  - `Object.defineProperties`
- 新的数组方法
  - `Array.isArray`
  - `Array.prototype.forEach`
  - `Array.prototype.map`
  - `Array.prototype.filter`
  - `Array.prototype.reduce`
  - `Array.prototype.reduceRight`
  - `Array.prototype.every`
  - `Array.prototype.some`
  - `Array.prototype.indexOf`
  - `Array.prototype.lastIndexOf`
- 新的对象方法
  - `Object.keys`
  - `Object.create`
  - `Object.getPrototypeOf`
  - `Object.preventExtensions`
  - `Object.isExtensible`
  - `Object.seal`
  - `Object.isSealed`
  - `Object.freeze`
  - `Object.isFrozen`
- 更好的属性访问控制
- 支持多行字符串
- 增强的错误处理

### ECMAScript 6 (ES6 / ES2015) - 2015

- 块级作用域 (`let` 和 `const`)
- 箭头函数 (`=>`)
- 类 (`class`)
- 模板字面量（反引号字符串）
- 解构赋值
- 默认参数值
- 剩余参数和扩展运算符
- 模块 (`import` 和 `export`)
- 迭代器和生成器 (`function*` 和 `yield`)
- `Promise`
- `Map` 和 `Set`
- `WeakMap` 和 `WeakSet`
- 新的字符串方法
- 新的数组方法
- 新的对象方法
- `Symbol`
- `for...of` 循环

### ECMAScript 7 (ES2016) - 2016

- 指数运算符 (`**`)
- `Array.prototype.includes`

### ECMAScript 8 (ES2017) - 2017

- 异步函数 (`async` / `await`)
- `Object.values`
- `Object.entries`
- `Object.getOwnPropertyDescriptors`
- 字符串填充 (`String.prototype.padStart` 和 `String.prototype.padEnd`)
- `Trailing commas` in function parameter lists and calls
- `SharedArrayBuffer` 和 `Atomics`

### ECMAScript 9 (ES2018) - 2018

- 异步迭代 (`for-await-of`)
- 对象扩展操作符 (`...`)
- `Promise.prototype.finally`
- 正则表达式改进
  - `s` 修饰符（dotAll 模式）
  - Unicode 属性转义 (`\p{...}` 和 `\P{...}`)
  - 后行断言（lookbehind assertions）
  - 命名捕获组

### ECMAScript 10 (ES2019) - 2019

- `Array.prototype.flat` 和 `Array.prototype.flatMap`
- `Object.fromEntries`
- `String.prototype.trimStart` 和 `String.prototype.trimEnd`
- `Symbol.prototype.description`
- `Function.prototype.toString` 修正
- `try...catch` 中的可选 `catch` 绑定
- `JSON.stringify` 改进
- `well-formed` JSON.stringify 输出
- `Array.prototype.sort` 稳定排序

### ECMAScript 11 (ES2020) - 2020

- `BigInt`
- 动态 `import`
- 可选链操作符 (`?.`)
- 空值合并操作符 (`??`)
- `Promise.allSettled`
- 全局对象 (`globalThis`)
- 新的 `import.meta`
- `for-in` 枚举顺序
- `String.prototype.matchAll`
- 禁止使用 `Function` 构造函数创建函数

### ECMAScript 12 (ES2021) - 2021

- `String.prototype.replaceAll`
- 逻辑赋值操作符 (`&&=`, `||=`, `??=`)
- 数字分隔符（Numeric separators）
- `Promise.any`
- WeakRef 和 FinalizationRegistry
- `Array.prototype.at`

### ECMAScript 13 (ES2022) - 2022

- `top-level await`
- 类字段（Class fields）
- 私有字段和方法（Private fields and methods）
- 静态块（Static blocks）
- 正则表达式匹配索引（match indices）
- `Error.cause`

### ECMAScript 14 (ES2023) - 2023

- 装饰器（Decorators）
- 允许 Symbol 作为 WeakMap 键
- 新的 Array 方法
  - `Array.prototype.toSorted`
  - `Array.prototype.toReversed`
  - `Array.prototype.toSpliced`
  - `Array.prototype.with`
- 新的 `Map` 和 `Set` 方法
  - `Map.prototype.emplace`
  - `Set.prototype.symmetricDifference`
  - `Set.prototype.difference`
  - `Set.prototype.intersection`

### ECMAScript 15 (ES2024) - 2024 (预计)

- 元素类型转换（Element Type Annotations）
- 模式匹配（Pattern Matching）
- 标准模块支持（Standard Library Modules）

这个提纲展示了每个 ECMAScript 版本引入的主要特性和改进，有助于理解 JavaScript 的演进过程和新特性。