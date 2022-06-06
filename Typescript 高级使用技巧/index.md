## Typescript 高级使用技巧 
### 函数

#### 获取函数参数类型

获取`foo`函数的第二个参数的类型：

``` ts
// 函数
const foo = (input: string,next: number) => {}

// 获取 foo 函数的参数
type FooParams = Parameters<typeof foo>

// foo 函数第二个参数类型
type SecondParamType = FooParamsType[1] // number
```

#### 函数参数设置泛型 T，并 T 是多个可选

`log`函数只允许`A`或者`B`的其中一个：

``` ts
type A = {
  value: number;
}

type B = {
  value: number;
}

const log = <T extends A | B>(obj: T) => {
  console.log(obj.value)
}
```

### 数据类型

#### 设置一个 map 的 key 类型

设置`myMap`的`key`的含义为`userName`，对应的 value 类型为`string`类型：

``` ts
const myMap: {[userName : string ] : string} = {};
```

#### 获取 array 的元素类型

获取`list`的元素类型

``` ts
const list = [1, 2, 'hello']

typeof list[0]
// number

typeof list[2]
// string
```
