## Java：反射机制 
### 概念

根据oracle中的一篇文章[Using Java Reflection](http://www.oracle.com/technetwork/articles/java/javareflection-1536171.html)中说明，还是98年发表的。

反射是Java编程语言的一项功能。它允许一个正在执行的程序去检查或自我"反省"，并且还可以修改内部的程序属性。例如，它可以获取一个类的所有成员的名字并显示它们。

有一个反射的具体使用便是JavaBeans，它可以通过构建工具让软件组件被可视化的修改。当Java组件(classes)动态加载后，这个工具可以通过反射获取它们的属性。

我们其实只需这样简单理解就好了。

#### 简单例子

``` java
import java.lang.reflect.*;
public class DumpMethods {
  public static void main(String args[])
  {

   try {
     Class c = Class.forName(args[0]);
     Method m[] = c.getDeclaredMethods();
     for (int i = 0; i < m.length; i++)
     System.out.println(m[i].toString());
   }
   catch (Throwable e) {
     System.err.println(e);
   }
  }
}
```

#### 阐述例子

反射的各种类，像 Method 这样的类都可以在 java.lang.reflect 包里找到。

从一个你需要检查的类中获取 java.lang.Class 对象 (java.lang.Class 是用来表现正在运行程序的类和接口)
其中一个获取 Class 对象的方法如下面所示：

``` java
Class c = Class.forName("java.lang.String");  //获取 String 的 Class 对象
Class c = int.class; //获取整型的Class对象
//or
Class c = Integer.TYPE;
```

通过这个类获取已声明的所有方法的列表，像 getDeclaredMethods 方法一样;

一旦这信息握在手里，就可以用反射API去检查这信息。以下通过文本显示第一个方法;

``` java
Class c = Class.forName("java.lang.String");
Method m[] = c.getDeclaredMethods();
System.out.println(m[0].toString());
```

**模拟 instanceof 操作符：**( * 类名和文件名保持一致，规范来讲类名需要首字母大写)

``` java
class A {}
public class instance1 {
  public static void main(String args[]) {
    try {
      Class cls = Class.forName("A");
      boolean b1 = cls.isInstance(new Integer(37));
      System.out.println(b1);
      boolean b2 = cls.isInstance(new A());
      System.out.println(b2);
    } catch (Throwable e) {
      System.err.println(e);
    }
  }
}
```

#### 找出类的方法

反射最有价值的并且也是最基础的使用是找出定义在类里面的方法，如下代码：

``` java
import java.lang.reflect.*;

public class method1 {
  private int f1(Object p, int x) throws NullPointerException {
    if (p == null)
      throw new NullPointerException();
    return x;
  }

 public static void main(String args[]) {
   try {
     Class cls = Class.forName("method1");

     Method methlist[] = cls.getDeclaredMethods();
     for (int i = 0; i < methlist.length; i++) {
       Method m = methlist[i];
       System.out.println("name  = " + m.getName());//打印方法名
       System.out.println("decl class = " + m.getDeclaringClass());//打印方法所在类名

       Class pvec[] = m.getParameterTypes();
       for (int j = 0; j < pvec.length; j++)
         System.out.println("   param #" + j + " " + pvec[j]);//打印形参名

       Class evec[] = m.getExceptionTypes();
       for (int j = 0; j < evec.length; j++)
         System.out.println("exc #" + j + " " + evec[j]);//打印异常类型

       System.out.println("return type = " + m.getReturnType());//打印返回参数类型
       System.out.println("-----");
     }
   } catch (Throwable e) {
     System.err.println(e);
   }
 }
}
```

#### 找出类的字段

``` java
import java.lang.reflect.*;

public class field1 {
  private double d;
  public static final int i = 37;
  String s = "testing";

  public static void main(String args[]) {
    try {
      Class cls = Class.forName("field1");

      Field fieldlist[] = cls.getDeclaredFields();
      for (int i = 0; i < fieldlist.length; i++) {

        Field fld = fieldlist[i];
        System.out.println("name  = " + fld.getName()); //获取变量名
        System.out.println("decl class = " + fld.getDeclaringClass()); //获取变量所在类名
        System.out.println("type = " + fld.getType());//获取变量类型

        int mod = fld.getModifiers();
        System.out.println("modifiers = " + Modifier.toString(mod)); //获取修饰符
        System.out.println("-----");
      }
    } catch (Throwable e) {
      System.err.println(e);
    }
  }
}
```

#### 执行类的方法

``` java
import java.lang.reflect.*;

public class method2 {
  public int add(int a, int b) {
    return a + b;
  }

  public static void main(String args[]) {
    try {
      Class<?> cls = Class.forName("method2");

      Class partypes[] = new Class[2];
      partypes[0] = Integer.TYPE;
      partypes[1] = Integer.TYPE;
      Method meth = cls.getMethod("add", partypes); //获取方法名为"add"的方法，并且加入自定义形参

      method2 methobj = new method2();
      Object arglist[] = new Object[2];
      arglist[0] = new Integer(37);
      arglist[1] = new Integer(47);
      Object retobj = meth.invoke(methobj, arglist); //加入实参，执行add方法，返回参数。
      Integer retval = (Integer) retobj;
      System.out.println(retval.intValue());  //打印输出结果
    } catch (Throwable e) {
      System.err.println(e);
    }
  }
}
```

#### 执行类的构造函数

``` java
import java.lang.reflect.*;

public class constructor2 {
  public constructor2() {}

  public constructor2(int a, int b) {
    System.out.println("a = " + a + " b = " + b);
  }

  public static void main(String args[]) {
    try {
      Class<?> cls = Class.forName("constructor2");

      Class partypes[] = new Class[2];
      partypes[0] = Integer.TYPE;
      partypes[1] = Integer.TYPE;
      Constructor ct = cls.getConstructor(partypes);

      Object arglist[] = new Object[2];
      arglist[0] = new Integer(37);
      arglist[1] = new Integer(47);
      Object retobj = ct.newInstance(arglist);

    } catch (Throwable e) {
      System.err.println(e);
    }
  }
}
```

#### 修改字段的值

``` java
import java.lang.reflect.*;

public class field2 {
  public double d;

  public static void main(String args[]) {
    try {
      Class cls = Class.forName("field2");

      Field fld = cls.getField("d"); //获取filed2类中变量名为d的字段

      field2 f2obj = new field2(); //实例化自定义类

      System.out.println("d = " + f2obj.d);

      fld.setDouble(f2obj, 12.34); //在指定的已实例化filed2类中修改其值
      System.out.println("d = " + f2obj.d);
    } catch (Throwable e) {
      System.err.println(e);
    }
  }
}
```

#### 数组的使用

``` java
import java.lang.reflect.*;

public class array1 {
  public static void main(String args[]) {
    try {
      Class cls = Class.forName("java.lang.String");

      Object arr = Array.newInstance(cls, 10);//实例化长度为10的字符串

      Array.set(arr, 5, "this is a test"); //设置下标在5的值为"this is a test"

      String s = (String) Array.get(arr, 5);//获取下标在5的值

      System.out.println(s);
    } catch (Throwable e) {
      System.err.println(e);
    }
  }
}
```

``` java
 import java.lang.reflect.*;

 public class array2 {
   public static void main(String args[]) {
     int dims[] = new int[] { 5, 10, 15 };
     Object arr = Array.newInstance(Integer.TYPE, dims); // 第二参数为维度，这里创建了5x10x15
                               // int数组

     Object arrobj = Array.get(arr, 3); // 获取10x15数组

     Class cls = arrobj.getClass().getComponentType();
     System.out.println(cls);

     arrobj = Array.get(arrobj, 5); // 获取15一维数组
     Array.setInt(arrobj, 10, 37); // 设置[3][5][10] 的值为37

     int arrcast[][][] = (int[][][]) arr;
     System.out.println(arrcast[3][5][10]);
   }
 }
```

> 这里的数组的类型都是动态得被创建了，而不是在编译期。

#### 总结

反射对于特定功能需求的实现很有用，通过名字可以检索类和数据结构，允许在运行期审查程序自身信息。
