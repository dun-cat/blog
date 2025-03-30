## Java 设计模式 (十二) - 迭代器模式 
### 概念

迭代器模式 (Iterator) ，它可以让用户透过`特定的接口`巡访容器中的每一个元素而不用了解底层的实现。

#### 举例

##### 定义迭代器接口

``` java
interface Iterator{
    Object first();
    Object next();
    boolean hasNext();
    Object current();
}
```

##### 实现接口

``` java
class ConcreteIterator implements Iterator {
    private List<Object> list = new ArrayList<Object>();
    private int curr = 0;
    public ConcreteIterator(List<Object> list) {
        this.list = list;
    }

    public Object first(){
        return list.get(0);
    }

    public Object next() {
        Object ret = null;
        curr++;
        if(curr < list.size()) {
            ret = list.get(curr);
        }
        return ret;
    }

    public boolean hasNext() {
        return curr >= list.size() ? true : false;
    }

    public Object current() {
        return list.get(curr);
    }
}
```

##### 定义迭代器构造抽象类

```java
abstract class Aggregate {
    abstract Iterator createIterator();
}
```

##### 实现抽象类

``` java
class ConcreteAggregate extends Aggregate {
    private List<Object> list = new ArrayList<Object>();
    public ConcreteAggregate(List<Object> list) {
        this.list = list;
    }
    public Iterator createIterator() {
        return new ConcreteIterator(list);
    }
}
```

##### 执行类：RunMain

``` java
public class RunMain {
    public static void main(String[] args) {
        List<Object> list = new ArrayList<Object>();
        list.add("我是迭代器的第一项");
        list.add("我是迭代器的第二项");
        Aggregate agg = new ConcreteAggregate(list);
        Iterator iterator = agg.createIterator();
        iterator.first();
        while(!iterator.hasNext()) {
                System.out.println(iterator.current());
                iterator.next();
            }
        }
    }
}
```

##### 结果

``` bash
我是迭代器的第一项
我是迭代器的第二项
```
