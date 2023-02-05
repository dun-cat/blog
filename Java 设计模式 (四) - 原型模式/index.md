## Java 设计模式 (四) - 原型模式 
### 概念

原型模式通过抽象克隆行为，实现 copy 对象的目的。

#### 两种拷贝方式

浅拷贝：拷贝对象当存在引用变量时，只 copy 其引用地址。

深拷贝：拷贝对象当存在引用变量时，拷贝引用变量对应的值。既重新分配内存给引用变量对应的值，并且引用指向新的内存地址。

#### 简单的原型

把克隆行为抽象出来，然后让自定义对象(原型)实现克隆接口。

克隆接口：Cloneable

``` java
public interface ICopying {
    Object copy();
}
```

原型类：A

``` java
public class A implements ICopying{

    private String variableA;
    private int variableB;
    public void setVariableA(String variableA) {
        this.variableA = variableA;
    }

    public void setVariableB(int variableB) {
        this.variableB = variableB;
    }

    @Override
    public Object copy() {
        A a =new A();
        a.setVariableA(this.variableA);
        a.setVariableB(this.variableB);
        return a;
    }
}
```

执行类：RunMain

``` java
public class RunMain {
    public static void main(String[] args) {
        A prototypeA=new A(); // 原型
        A copiedA = (A) prototypeA.copy(); // 复制后的对象
    }
}
```

显然上面的 copy() 函数不太灵活，每次 copy 都要自己手动把原型的值复制过去。但是在java中不用担心这类问题。在基类 Object 类中有个 `native` 的 `clone()` 方法，它是浅拷贝。在 java 的原型接口 ( `Cloneable` ) 中的没有任何接口函数。

#### Java中的原型

Cloneable 接口

``` java
package java.lang;
public interface Cloneable {
}
```

基类：Object

``` java
package java.lang;
    ... // 其它代码
    protected native Object clone() throws CloneNotSupportedException;
    ... // 其它代码
```

原型类：B

``` java
public class B implements Cloneable{
    @Override
    public Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```

执行类：RunMain

``` java
public class RunMain {
    public static void main(String[] args) {
        B prototypeB =new B(); // 创建原型
        try {
            B clonedB=(B) prototypeB.clone(); // 基于原型的克隆
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
    }
}
```

> 原型类必须实现 `Cloneable` 接口和 `override` 基类的 `clone()` 方法。否则会出现运行时异常和编译异常。
