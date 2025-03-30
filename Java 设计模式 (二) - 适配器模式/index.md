## Java 设计模式 (二) - 适配器模式 
### 概念

把一个类的接口转换成被期望的接口。做法是把被适配的对象包裹在适配器类里。

比较适合使用Adapter模式的情况

1. 当你想使用一个已经存在的类时，而它的接口不符合你的需求
2. 你想创建一个可以复用的类，该类可以与其他不相关的类或不可预见的类协同工作
3. 你想使用一些已经存在的子类，但是不可能对每一个都进行子类化以匹配它们的接口，对象适配器可以适配它的父亲接口

#### 例子

已存在需要被适配的类

``` java
public class Adaptee {
        // 需要被对接的行为
    public void specificRequest(){
        // 业务代码
    }
}
```

主动提出来要求对方 (adaptee) 能够按照自己的接口去适配。所以提供一个目标接口给对方。

目标接口

``` java
public interface Target {
    public void request();
}
```

为了能够把被适配对象 `adaptee` 的按照目标接口去适配，需要一个类去实现目标接口，既适配类。

适配类

``` java
public class Adapter implements Target{
    private Adaptee adapteee;
    public Adapter(Adaptee adaptee){
        this.adapteee = adaptee;
    }
    @Override
    public void request() {
        adapteee.specificRequest();
    }
}
```

> 实现适配做法就是把被适配对象 `adaptee` 包裹在适配器 `Adapter` 里。然后在重写的接口函数里调用所需行为。

执行类：RunMain

``` java
public class RunMain {
    public static void main(String[] args){
        Adaptee adaptee =new Adaptee(); // 被适配对象
        Target targetAction = new Adapter(adaptee); // 适配器
        targetAction.request(); // 按照目标接口去执行所需功能
    }
}
```
