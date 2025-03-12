## Java 设计模式 (十) - 代理模式 
### 概念

提供给客户机一个代理类，这样可以控制客户机对目标对象的访问。

#### 举例

首先会提供客户机一个Subject接口，作为被访问的对象对外访问控制接口。然后让被访问对象实现Subject接口，实现客户机所需的功能。最后提供一个代理类，让它包含访问接口的实现。

##### 访问对象对外接口

``` java
public interface ISubject {
    void operate();
}
```

##### 被访问的对象

``` java
public class ConcretSubject implements ISubject{
    @Override
    public void operate() {
        System.out.println("我是具体功能模块");
    }
}
```

##### 代理类

``` java
public class Proxy implements ISubject{
    private ISubject iSubject;
    public Proxy(ISubject iSubject) {
        this.iSubject = iSubject;
    }
    @Override
    public void operate() {
        iSubject.operate();
    }
}
```

##### 执行类：RunMain

``` java
public class RunMain {
    public static void main(String[] args){
        ConcretSubject concretSubject = new ConcretSubject(); // 被访问对象
        Proxy proxy = new Proxy(concretSubject); // 设置代理类作为控制访问的过度类
        proxy.operate(); // 操作
    }
}
```

``` bash
我是具体功能模块
```
