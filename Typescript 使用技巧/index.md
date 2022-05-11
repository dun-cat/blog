## Typescript 使用技巧 
### 如何获取函数参数类型？

``` ts
// 函数
const foo = (input: string,next: number) => {}

// 获取 foo 函数的参数
type FooParams = Parameters<typeof foo>

// foo 函数第二个参数类型
type SecondParamType = FooParamsType[1] // number
```
