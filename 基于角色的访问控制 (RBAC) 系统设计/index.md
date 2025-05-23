## 基于角色的访问控制 (RBAC) 系统设计 
### RBAC

Role-Based Access Control，基于角色的访问控制，这是一种设计思想。

权限设计`针对的主体`是`用户 (User)`和`权限 (Permission)`。最简单的设计只要关联 User 和 Permission 就好了，但显然不方便维护管理。

这里引入`角色 (Role)`的概念，Role 的作用更像是分组 (Group) 的作用。每种 Role 有自己的权限范围，是`一组权限的集合`。将一个 User 分配一个或多个 Role 的角色，那么该 User 将会拥有该集合所拥有的权限。

#### 好处

* Role 使添加、删除和调整权限更加容易。随着用户规模的增加和复杂性的增加，Role 会变得特别有用。

* 使用 Role 来收集为各种API定义的权限。例如，假设您有一个营销模块，该模块允许用户创建新闻稿并将其分发给客户。

目前我们能看到的权鉴设计，基本都是这个设计思路。

#### RBAC1 模型

此模型引入了角色继承(Hierarchical Role)概念，即角色具有上下级的关系，角色间的继承关系可分为`一般继承关系`和`受限继承关系`，他们的区别在于一般继承允许一个角色多继承，而受限继承则只能单继承。

#### RBAC2 模型

此模型约束角色与角色之前的关系，即一个角色所应遵循的强制性规则。

##### 角色互斥

为了避免用户拥有过多权限而产生利益冲突，所以需要做责任分离，包括`静态责任分离`和`动态责任分离`。

* `静态职责分离 (Static Separation of Duty)`： 用户无法同时被赋予有冲突的角色。
* `动态责任分离 (Dynamic Separation of Duty)`：用户在一次会话 (Session) 中不能同时激活自身所拥有的、互相有冲突的角色，只能选择其一。

除此之外还有一些自定义规则：

`基数约束`: 一个角色被分配的用户数量受限；一个用户可拥有的角色数目受限；同样一个角色对应的访问权限数目也应受限，以控制高级权限在系统中的分配

`先决条件角色`: 即用户想获得某上级角色,必须先获得其下一级的角色

#### RBAC3 模型

此模型在以上模型之上，建立用户组 (User Group) 的概念。

##### 用户组

当用户基数大并且一个用户包含多个角色的时候，管理员直接给用户赋予角色也将变成一件繁琐的事。此时就可以建立用户组。

`上下级关系的用户组`：最典型的例子就是部门和职级的关系。
`普通用户组`：无任何关系的组

#### 疑问点

##### 如果一个 User 有多个 Role 会如何？

RBAC 是附加模型，因此，如果 Role 分配重叠，则该 User 拥有的权限是他们的并集。

##### 操作权限判断是基于角色去判断还是基于权限？

在代码上，知道用户是否有功能的操作权限，我们应该`判断用户权限而不是判断用户角色`。

### ABAC

Attribute-Based Access Control，基于属性的权限验证。

> ABAC有时也被称为PBAC (Policy-Based Access Control) 或CBAC (Claims-Based Access Control) 。
