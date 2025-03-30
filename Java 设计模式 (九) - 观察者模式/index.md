## Java 设计模式 (九) - 观察者模式 
### 概念

这个模式的另外一个种叫法是发布订阅模式，这种叫法更佳贴切，表明了主要操作的对象就这两个。基本原理就是向一个类里(发布者)里注入多个实现了相同接口的类(订阅者)

#### 举例

举例中有两个类：Subject 和 Observer，既分别是发布者和订阅者。下面的代码理解上也不是很费力。

订阅者的接口

``` java
public interface IObserver {
    void update(String info);
}
```

订阅者0号

``` java
public class Observer0 implements IObserver{
    @Override
    public void update(String info) {
        System.out.println("我是订阅者0号，我收到了信息：" + info);
    }
}
```

订阅者1号

``` java
public class Observer1 implements IObserver{
    @Override
    public void update(String info) {
        System.out.println("我是订阅者1号，我收到了信息：" + info);
    }
}
```

发布者

``` java
import java.util.ArrayList;
import java.util.List;

public class Subject {
    List<IObserver> observers; // 用列表存储订阅者
    public Subject() {
        observers =new ArrayList<>();
    }
    // 一旦发布者有消息就可以通知订阅者
    public void notifyObserver() {
        for(IObserver observer:observers){
            observer.update("ui had changed");
        }
    }
    // 提供一个公开方法让订阅者可以订阅
    public void register(IObserver observer){
        observers.add(observer);
    }
    // 提供一个公开方法让订阅者取消订阅
    public void unRegister(IObserver observer){
        observers.remove(observer);
    }
}
```

执行类：RunMain

``` bash
我是订阅者0号，我收到了信息：ui had changed
我是订阅者1号，我收到了信息：ui had changed
```
