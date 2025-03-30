## Java 设计模式 (六) - 组合模式 
### 概念

GOF 的《设计模式》一书中对使用组合模式的意图描述如下：将对象组合成树形结构以表示"部分 - 整体"的层次结构。Composite 使得用户对单个对象和组合对象的使用具有一致性。

#### 举例

下面有几个类：

Component抽象类：用来定义统一行为和类型；

Leaf类：继承了 Component 抽象了，并 override 其行为；

Composite类：继承了 Component 的部件，override 其行为，并且维护一个用于存储子部件的列表；

抽象组件类

``` java
abstract class Component {
    protected String name;

    public Component(String name) {
        this.name = name;
    }
    public abstract void Add(Component c);
    public abstract void Remove(Component c);
    public abstract void Display(int depth);
}
```

叶子类

``` java
class Leaf extends Component {

    public Leaf(String name) {
        super(name);
    }

    @Override
    public void Add(Component c) {}
    @Override
    public void Remove(Component c) {}

    @Override
    public void Display(int depth) {
        String temp = "";
        for (int i = 0; i < depth; i++) 
            temp += '-';
        System.out.println(temp + name);
    }
}
```

部件类

``` java
import java.util.ArrayList;
import java.util.List;

public class Composite extends Component {
    private List<Component> childrens;

    public Composite(String name) {
        super(name);
        childrens = new ArrayList<>();
    }

    @Override
    public void Add(Component c) {
        childrens.add(c);
    }

    @Override
    public void Remove(Component c) {
        childrens.remove(c);
    }

    @Override
    public void Display(int depth) {
        String temp = "";
        for (int i = 0; i < depth; i++)
            temp += '-';
        System.out.println(temp + name);

        for (Component c : this.childrens) {
            c.Display(depth + 2);
        }
    }
}
```

执行类：RunMain

``` java
public class RunMain {
    public static void main(String[] asgs){
        Composite root = new Composite("root"); // 根部件

        root.Add(new Leaf("Leaf A"));
        root.Add(new Leaf("Leaf B"));

        Composite compX = new Composite("Composite X"); // 子部件
        compX.Add(new Leaf("Leaf C"));
        compX.Add(new Leaf("Leaf D"));
        root.Add(compX); // 添加到根部件里

        Composite compXY = new Composite("Composite Y"); // 子部件
        compXY.Add(new Leaf("Leaf E"));
        compXY.Add(new Leaf("Leaf F"));
        root.Add(compXY); // 添加到根部件里

        root.Display(1); // 显示结构
    }
}
```

``` bash
- root
- - - Leaf A
- - - Leaf B
- - - Composite X
- - - - - Leaf C
- - - - - Leaf D
- - - Composite Y
- - - - - Leaf E
- - - - - Leaf F
```
