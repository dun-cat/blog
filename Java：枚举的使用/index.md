## Java：枚举的使用 
### 概念

枚举类型是一个特殊类型，这个类型会有预定义好的常量集合。而定义好的变量，它的赋值操作的值都必须是这个类型里预定义中的值。

#### 基本定义

``` java
public enum Fruits {
    APPLE,ORANGE,BANANA
}
```

#### 枚举的使用

1. 使用枚举类型定义变量，并赋值

#### 基本用法

``` java
Fruits fruit= Fruits.APPLE;
```

1. switch语句的使用(JDK1.6之前的switch语句只支持int,char,enum类型)

结合switch

``` java
public void usedBySwitch(Fruits fruit) {
    switch (fruit) {
    case APPLE:
        System.out.println("这是苹果");
        break;
    case ORANGE:
        System.out.println("这是橘子");
        break;
    case BANANA:
        break;
    }
}
```

1. 枚举的另一种方式的使用

其他用法

``` java
public enum Fruits {
        //与简单定义不同，这里的需要加入";"来结尾。并且预定的方式有所不同。
    APPLE("苹果","红色",1),ORANGE("橘子","橘色",2),BANANA("香蕉","黄色",3);

    //枚举可以定义定义一些变量
    private String name;
    private String color;
    private int index;

    //可以自定义构造函数
    private Fruits(String name,String color,int index){
        this.name=name;
        this.index=index;
        this.color=color;
    }

    //可以有自己的方法
    public static void printNameByIndex(int index) {
        for(Fruits f:Fruits.values()){
            if (f.index == index) {
                System.out.println(f.name);
            }
        }
    }
    public static void printColorByName(Fruits name) {
        for(Fruits f:Fruits.values()){
            if (f.name.equals(name.name)) {
                System.out.println(f.color);
            }
        }
    }

    //如同一个普通类一般可以有getter和setter
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getIndex() {
        return index;
    }

    public void setIndex(int index) {
        this.index = index;
    }

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }

}

//调用
public class RunMain {
    public static void main(String arg[]) {
        Fruits.printNameByIndex(2); 
        Fruits.printNameByIndex(3);
        Fruits.printNameByIndex(1);
        Fruits.printColorByName(Fruits.APPLE);
    }
}
```

还有比如可以实现接口，成为接口成员等类似于普通类的操作。

参考资料：

\> 概念参考依据 orcale 中 tutorial 里的[Enum Types](http://docs.oracle.com/javase/tutorial/java/javaOO/enum.html)文章

\> 使用方法参考：[http://www.iteye.com/topic/1116193](http://www.iteye.com/topic/1116193)
