## MySQL常用笔记 
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

``` bash
grant all privileges on testDB.* to test@localhost identified by '1234';
grant select,update on testDB.* to test@localhost identified by '1234';
grant select,delete,update,create,drop on *.* to test@"%" identified by "1234";
flush privileges; # 刷新系统权限表
# 格式：grant 权限 on 数据库.* to 用户名@登录主机 identified by "密码";
```

### 增

``` bash
create database test;
# 指定字符集来创建数据库；
create database publish_system_test DEFAULT CHARACTER SET utf8mb4;

insert into mysql.user(Host,User,Password) values("localhost","test",password("123")); # 添加用户
```

### 删

``` bash
drop databases test
delete from test
```

### 查

``` bash
show databases
show tables
```

### 改

``` bash
update user set password=PASSWORD("123") where User='root'; # 更新密码
# or 
update user set authentication_string=PASSWORD("123") where User='root'; # 更新密码

update user set Host='%' where user='test';
flush privileges;
```
