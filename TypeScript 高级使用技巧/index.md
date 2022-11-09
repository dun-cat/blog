## TypeScript 高级使用技巧 
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

### 泛型

通用范例

``` ts
function identity<T, U>(value: T, message: U): T {
  console.log(message);
  return value;
}

console.log(identity<number, string>(18, 'lumin'));
```

### 常用工具类

下面介绍 TypeScript 内置的工具类型，并展示他们源码实现。

#### Partial\<Type\>

构造一个所有属性类型都设置为`可选`的类型。

```ts
type User = {
  name: string;
  password: string;
  address: string;
  phone: string;
};

type PartialUser = Partial<User>;
```

实现[源码](https://github.com/microsoft/TypeScript/blob/0993c017bace34cbcbf8a19b830b22db95676ca4/lib/lib.es5.d.ts#L1549)：

``` ts
/**
 * Make all properties in T optional
 */
type Partial<T> = {
    [P in keyof T]?: T[P];
};
```

#### Required\<Type\>

构造一个所有属性为`必填`的类型，与 Partial 含义相反。

``` ts
type User = {
  name: string;
  password: string;
  address: string;
  phone: string;
};

type RequiredUser = Required<User>;
```

实现[源码](https://github.com/microsoft/TypeScript/blob/0993c017bace34cbcbf8a19b830b22db95676ca4/lib/lib.es5.d.ts#L1556)：

``` ts
/**
 * Make all properties in T required
 */
type Required<T> = {
    [P in keyof T]-?: T[P];
};
```

#### Readonly\<Type\>

构造一个所有属性为`只读`的类型，这意味着该类型的属性不能被赋值。

``` ts
type User = {
  name: string;
  password: string;
  address: string;
  phone: string;
};

type ReadonlyUser = Readonly<User>;
```

实现[源码](https://github.com/microsoft/TypeScript/blob/0993c017bace34cbcbf8a19b830b22db95676ca4/lib/lib.es5.d.ts#L1563)：

``` ts
/**
 * Make all properties in T readonly
 */
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
};
```

#### Record\<Keys, Type\>

构造一个对象类型，其属性键为 Keys，其属性值为 Type。用于将一种类型的属性映射到另一种类型。

``` ts
type User = {
  name: string;
  password: string;
  address: string;
  phone: string;
};

type UserIds = 1000 | 1001 | 1002;

type UserMap = Record<UserIds, User>;
```

实现[源码](https://github.com/microsoft/TypeScript/blob/0993c017bace34cbcbf8a19b830b22db95676ca4/lib/lib.es5.d.ts#L1577)：

``` ts
/**
 * Construct a type with a set of properties K of type T
 */
type Record<K extends keyof any, T> = {
    [P in K]: T;
};
```

#### Exclude\<UnionType, ExcludedMembers\>

从 UnionType 类型中`排除`已存在的其中成员类型。

```ts
type T0 = Exclude<1 | 2 | 3, 2>;
// type T0 = 1 | 3

type T1 = Exclude<1 | 2 | 3, 2 | 3>;
// type T0 = 1

type T2 = Exclude<string | number | (() => void), Function>;
//  T2 = string | number

```

实现[源码](https://github.com/microsoft/TypeScript/blob/0993c017bace34cbcbf8a19b830b22db95676ca4/lib/lib.es5.d.ts#L1584)：

``` ts
/**
 * Exclude from T those types that are assignable to U
 */
type Exclude<T, U> = T extends U ? never : T;
```

#### Extract\<Type, Union\>

从 Type 中提取可以赋值给 Union 的类型。也就是说提取的类型即属于 Type，又属于 Union，属于他们共有类型。

```ts
type T0 = Extract<'a' | 'b' | 'c', 'a' | 'f'>;
// type T0 = 'a'

type T1 = Extract<string | number | (()=> void), Function>;
//  T1 = () => void

```

实现[源码](https://github.com/microsoft/TypeScript/blob/0993c017bace34cbcbf8a19b830b22db95676ca4/lib/lib.es5.d.ts#L1589)：

``` ts
/**
 * Extract from T those types that are assignable to U
 */
type Extract<T, U> = T extends U ? T : never;
```
