## Nest.js 脚手架搭建 
### 异常处理

nest 默认提供了一层异常层 (exceptions layer ) ，用于处理所有未作额外处理的异常。当异常未被应用代码捕获处理，会自动发送友好的响应 (response) 。

![filter](filter.png)

#### 异常过滤器 (exception filter)

nest 有个全局的异常过滤器，一个没被识别成 http 异常的 exception 会返回下面的信息：

``` json
{
  "statusCode": 500,
  "message": "Internal server error"
}
```

### 鉴权

### 字段验证

### 常见问题

mysql 不支持 Array 类型

参考资料：

\> [https://bluehorn07.github.io/2020/09/05/How-to-save-JSON-array-in-MySQL-with-TypeORM.html](https://bluehorn07.github.io/2020/09/05/How-to-save-JSON-array-in-MySQL-with-TypeORM.html)
