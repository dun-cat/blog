## Java 设计模式 (七) - 装饰者模式 
### 概念

装饰模式处理两种类：装饰类和被装饰类。为了让多个装饰类装饰一个对象，让装饰类的 `interface` 去 `extends` 被装饰类的 `interface`，保持实例类型一致。

下面拿一碗面来举例，并且采用 `interface` 去实现，而没有用 `abstract` 。简单的抽象行为，抽象的方式依据具体业务。

#### 例子

面条接口

``` java
//显示面条信息的接口
public interface INoodles {
    String info();
}
```

装饰面条的接口

``` java
//继承装饰者接口实现实例的组合
public interface INoodlesDecorator extends INoodles{}
```

裸面类

``` java
public class Noodles implements INoodles{
    @Override
    public String info() {
        return "裸面";
    }
}
```

荷包蛋类

``` java
public class PoachedEggsDecorator implements INoodlesDecorator{
    private INoodles iNoodles;

    public PoachedEggsDecorator(INoodles iNoodles) {
        this.iNoodles = iNoodles;
    }
    @Override
    public String info() {
        return iNoodles.info()+" + 荷包蛋";
    }
}
```

卷心菜类

``` java
public class CabbageDecorator implements INoodlesDecorator{
    private INoodles iNoodles;

    public CabbageDecorator(INoodles iNoodles) {
        this.iNoodles = iNoodles;
    }
    @Override
    public String info() {
        return iNoodles.info()+" + 卷心菜";
    }
}
```

执行类：RunMain

``` java
public class RunMain {
    public static void main(String[] args){
        Noodles noodles =new Noodles();                                                // 创建裸面

        PoachedEggsDecorator poachedEggsDecorator =new PoachedEggsDecorator(noodles);  // 加入荷包蛋，成为了被装饰者
        CabbageDecorator cabbageDecorator =new CabbageDecorator(poachedEggsDecorator); // 加入卷心菜

        System.out.println(cabbageDecorator.info());                                   // 打印点菜信息
    }
}
```

``` bash
裸面 + 荷包蛋 + 卷心菜
```
