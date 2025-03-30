## Javascript中的类型判断、空判断 
### 数字
``` javascript
// 数字类型判断
typeof 123 === 'number' // true

// 整型判断
function isInteger(data) {
  return typeof data === 'number' && data % 1 === 0
}
/**
* 引用：[Number.isInteger]
* https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number/isInteger
*/
function isInteger(data) {
  return typeof value === "number" &&
        isFinite(value) &&
        Math.floor(value) === value
}
```
### 字符串
``` javascript
// 字符串类型判断
typeof '123' === 'string' // true

// 空字符串判断
function emptyString(data) {
  reurn data === ''
}

// 非空字符串判断
function nonEmptyString(data) {
  typeof data === 'string' && data !== ''
}
```
### 布尔
``` javascript
// 布尔类型判断
typeof false === 'boolean' // true

function isBoolean(data) {
  return data === false '' data === true
}
```
### 对象
``` javascript
// 对象类型判断
function isObject(data) {
  return data !== null && typeof data === 'object'
}
function isObject(data) {
  return Object.prototype.toString.call(data) === '[object Object]'
}

// 空对象判断
function emptyObject(data) {
  return isObject(data) && Object.keys(data).length === 0
}
// 非空对象判断
function nonEmptyObject(data) {
  return isObject(data) && Object.keys(data).length > 0
}

typeof null === 'object' // true
```
### 数组
``` javascript
// 数组类型判断
Array.isArray([]) // true

// 空数组判断
function emptyArray(data) {
  return Array.isArray(data) && data.length === 0
}

// 非空数组判断
function nonEmptyArray() {
  return Array.isArray(data) && data.length > 0
}
```