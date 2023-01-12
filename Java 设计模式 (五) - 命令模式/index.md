## Java 设计模式 (五) - 命令模式 
### 概念

简单来说就是任务执行的细分。分别得设置了任务请求对象和任务执行对象。而关联它们的方法就是设置一个接口，让实现接口的实例包裹被执行的任务，然后它再把自己送到执行者那里去执行。

虽然叫命令模式，但显然容易让思维限制住。名字不怎么好，而且网络上举的例子不太能够让人明白过来。

#### 举例

需要执行的任务

``` java
public class Receiver {
    public void task(){
        System.out.println("我是任务本体");
    }
}
```

桥梁作用的任务接口

``` java
public interface ITask {
    void execute();
}
```

把任务包裹在实现接口的任务体里

``` java
public class MyTask implements ITask{
    private Receiver receiver;
    public MyTask(Receiver receiver) {
        this.receiver = receiver;
    }
    @Override
    public void execute() {
        if (receiver != null) {
            receiver.task();
        }
    }
}
```

任务的执行者

``` java
public class Invoker {
    private ITask task;

    public void setTask(ITask task) {
        this.task = task;
    }
    public void run(){
        task.execute();
    }
}
```

执行类：RunMain

``` java
public class RunMain {
    public static void main(String[] args){
        Receiver receiver = new Receiver(); // 需要执行的任务

        ITask iTask = new MyTask(receiver); // 把任务封装到一个接口里

        Invoker invoker = new Invoker(); // 任务的执行者只需要包含接口，然后执行
        invoker.setTask(iTask);
        invoker.run();
    }
}
```
