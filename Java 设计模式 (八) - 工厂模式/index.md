## Java 设计模式 (八) - 工厂模式 
### 概念

首先，工厂模式是用来创建类的实例的。既然是工厂模式，"工厂"一词就表明其创建的类不只一个，还有也表明我们对于开发人员来说关注的焦点不在于如何创建一个类，而是在于如何更好的去封装工厂，让其具备很好的扩展性和可维护性，并且大家为此津津乐道(just kidding)。

### 简单的工厂

用一个静态方法来创建实例。 (鞋匠举例)

定义一个鞋的接口：Shoes

``` java
public interface IShoes {
    void printNameOfShoes();
}
```

三种款式的鞋类：Derbys  Monks  Oxford

``` java
public class Derbys implements IShoes {
    private String name = "德比鞋";

    @Override
    public void printNameOfShoes() {
        System.out.println(this.name);
    }
}

public class Monks implements IShoes{
    private String name ="孟克鞋";

    @Override
    public void printNameOfShoes() {
        System.out.println(this.name);
    }
}

public class Oxford implements IShoes{
    private String name = "牛津鞋";
    @Override
    public void printNameOfShoes() {
        System.out.println(this.name);
    }
}
```

工厂类：ShoesFactory

``` java
public class ShoesFactory {
    public static IShoes createShoes(String style) {
        IShoes shoes = null;
        switch (style) {
        case "01":
            shoes= new Derbys();
            break;
        case "02":
            shoes = new Monks();
            break;
        case "03":
            shoes =new Oxford();
            break;
        }
        return shoes;
    }
}
```

执行类：RunMain

``` java
public class RunMain {
    public static void main(String arg[]) {
        Shoes derbys = ShoesFactory.createShoes("01");
        Shoes monks  = ShoesFactory.createShoes("02");
        Shoes oxford = ShoesFactory.createShoes("03");
        derbys.printNameOfShoes();
        monks.printNameOfShoes();
        oxford.printNameOfShoes();
    }
}
```

``` bash
德比鞋
孟克鞋
牛津鞋
```

> 创建了一个接口，为了能让打印这个行为被抽象化来作为多个鞋款类的输出规范，之后让各款式鞋子实现其接口便可，接下来就是创建了一个工厂类，用其内部的一个静态方法来生产类的实例，最后在主调函数里执行实例化和各自的打印输出。

建立多个工厂，适应业务变化，抽象出工厂，规范化创建工厂。

增加了一间工厂，并且创建新品牌：乐福鞋。

### 抽象的工厂

工厂类的接口：IShoesFactory '新的鞋款类：Loafer '新的工厂类: NewShoesFactory

``` java
public interface IShoesFactory {
    IShoes createShoes(String style);
}

public class Loafer implements IShoes {
    private String name = "乐福鞋";
    @Override
    public void printNameOfShoes() {
        System.out.println(this.name);
    }
}

public class NewShoesFactory implements IShoesFactory{
    @Override
    public IShoes createShoes(String style) {
        IShoes shoes = null;
        switch (style) {
        case "new_01":
            shoes= new Loafer();
            break;
        }
        return shoes;
    }
}
```

修改之前的ShoesFactory工厂类

``` java
public class ShoesFactory implements IShoesFactory {
    @Override
    public IShoes createShoes(String style) {
        IShoes shoes = null;
        switch (style) {
        case "01":
            shoes= new Derbys();
            break;
        case "02":
            shoes = new Monks();
            break;
        case "03":
            shoes =new Oxford();
            break;
        }
        return shoes;
    }
}
```

执行类

```java
public class RunMain {
    public static void main(String arg[]) {

        ShoesFactory oldfactory = new ShoesFactory();
        IShoes derbys = oldfactory.createShoes("01");
        derbys.printNameOfShoes();

        NewShoesFactory newFactory = new NewShoesFactory();
        IShoes loafer =newFactory.createShoes("new_01");
        loafer.printNameOfShoes();
    }
}
```

``` bash
德比鞋
乐福鞋
```

> 上面的工厂类中没有了静态方法，IShoesFactory中createShoes接口函数不能用static修饰符，不然会编译异常，因为但凡是静态方法必须实现方法体。抽象出工厂类中创建鞋子这个行为后，工厂类的创建便由用户自行实例化。
