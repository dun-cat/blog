## MySQL 常用笔记 
### 登录

``` bash
mysql -u root -p

use mysql
```

### 远程连接

需要修改配置文件 /etc/mysql/my.cn #bind-address           = 127.0.0.1  注释掉

``` bash
mysql -h 114.55.136.250 -P 3306 -u test -p
```

### 授权

``` mysql
GRANT ALL PRIVILEGES ON testDB.* to test@localhost IDENTIFIED BY '1234';
GRANT select,update on testDB.* to test@localhost IDENTIFIED BY '1234';
GRANT select,delete,update,create,drop on *.* TO test@"%" IDENTIFIED BY "1234";
FLUSH privileges; # 刷新系统权限表
# 格式：grant 权限 on 数据库.* to 用户名@登录主机 identified by "密码";
```

### 增

``` mysql
CREATE DATABASE test;
# 指定字符集来创建数据库；
CREATE DATABASE publish_system_test DEFAULT CHARACTER SET utf8mb4;

INSERT INTO mysql.user(Host,User,Password) VALUES("localhost","test",password("123")); # 添加用户
```

#### 添加表字段

``` mysql


```

### 删

``` mysql
DROP DATABASES test;
DELETE FROM test;

# 删除记录
DELETE FROM table_name;
```

数据库若在 `safe update mode` 模式下工作，你是删除不了的。此时你可以执行以下脚本关闭它：

``` mysql
# 查看模式
SHOW VARIABLES LIKE '%safe_updates%';
```

``` mysql
# 关闭更新安全模式
SET SQL_SAFE_UPDATES=0;
```

### 查

``` mysql
SHOW DATABASES;
SHOW TABLES;
```

### 改

#### 更新用户密码

``` mysql

UPDATE user SET password=PASSWORD("123") WHERE User='root'; # 更新密码
# or 
UPDATE user SET authentication_string=PASSWORD("123") WHERE User='root'; # 更新密码

UPDATE user SET Host='%' WHERE user='test';
flush privileges;
```

#### 修改表字段

``` mysql
ALTER TABLE test CHANGE project_id p_id int(11) NOT NULL COMMENT '项目 id';
```

参考资料：

\> [https://dev.mysql.com/doc/refman/8.0/en/](https://dev.mysql.com/doc/refman/8.0/en/)
