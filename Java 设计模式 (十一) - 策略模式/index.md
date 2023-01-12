## Java 设计模式 (十一) - 策略模式 
### 概念

Strategy 用来允许在运行时动态地选择算法或操作的不同实现。通常，在抽象类中实现任何公共行为，而具体子类提供那些有差异的行为。客户机一般知道可用的不同策略，并且可以在其中选择。

#### 举例

##### 策略接口

``` java
public interface IStrategy {
    void operate();
}
```

##### 策略A

``` java
public class StrategyA implements IStrategy{
    @Override
    public void operate() {
        System.out.println("我是解决问题的策略A");
    }
}
```

##### 策略B

```java
public class StrategyB implements IStrategy{

    @Override
    public void operate() {
        System.out.println("我是解决问题的策略B");
    }

}
```

##### 过渡类(中间层)

``` java
public class Context {
    private IStrategy iStrategy;
    public Context(IStrategy iStrategy) {
        this.iStrategy = iStrategy;
    }
    public void execute(){
        iStrategy.operate();
    }
}
```

##### 执行类：RunMain

``` java
public class RunMain {
    public static void main(String[] args){
        IStrategy iStrategyA = new StrategyA();
        Context context0 =new Context(iStrategyA);
        context0.execute();
        IStrategy iStrategyB = new StrategyB();
        Context context1 = new Context(iStrategyB);
        context1.execute();
    }
}
```

##### 结果

``` bash
我是解决问题的策略A
我是解决问题的策略B
```
