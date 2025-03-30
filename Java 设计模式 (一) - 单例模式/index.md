## Java 设计模式 (一) - 单例模式 
### 简介

对于整个程序来讲，这个类只能被实例化一次，一旦实例化就会在程序运行期一直存在，直到程序结束运行。

对于不用考虑多线程的情况下(例:javascript 在浏览器的运行环境就是单线程情况，代码不存在同时被执行的情况)

单例实现必要条件：

1. 需要一个私有的静态的类 这个类的类型就是单例类本身
2. 需要一个受保护的构造函数，因为单例类的实例化操作不能由外部控制
3. 需要一个公开的静态的获取单例类实例的方法

#### 单例基本写法

``` java
public class ClassicSingleton {
    private static ClassicSingleton instance = null; // 第一步

    protected ClassicSingleton() {} // 第二步 采用 private 还是 protected 视封装而定

    public static ClassicSingleton getInstance() { // 第三步
        if (instance == null) {
            instance = new ClassicSingleton();
        }
        return instance;
    }
}
```

这个例子也是懒加载概念的一种体现，只有在用得着的时候才去做对应的工作。这个单例类只有在需要的时候才被实例化，分配内存空间。

一般情况下，都会加入同步 (synchronized) 代码，来解决多线程编程中有可能让单例丧失唯一性的问题。

### 模拟多线程问题

测试单例类

``` java
package simpleTest;

public class Singleton {
    private static Singleton singleton = null;
    private static boolean firstThread = true;

    protected Singleton() {}

    public static Singleton getInstance() {
        if (singleton == null) {
            simulateRandomActivity();// 随机活动的模拟第一个线程会睡个50毫秒
            singleton = new Singleton();
        }
        return singleton;
    }

    public boolean isSingletonInitialized() { // 检测单例是否已经实例化
        return singleton == null;
    }

    private static void simulateRandomActivity() {
        try {
            // 留足够时间在第一个线程实例化之前去执行第二个线程
            if (firstThread) {
                firstThread = false;
                System.out.println("线程1开始 sleeping...");
                Thread.sleep(50);
            }
        } catch (InterruptedException ex) {
            System.out.println("Sleep interrupted");
        }
    }
}
```

测试运行类

``` java
package simpleTest;

public class RunMain {
    private static Singleton singleton = null;

    public static void main(String args[]) throws InterruptedException {
        // 两个线程都会调用 Singleton.getInstance()方法
        Thread threadOne = new Thread(new Runnable1()), threadTwo = new Thread(new Runnable2());
        System.out.println("开始执行两个线程\n");

        threadOne.start();
        threadTwo.start();
        threadOne.join();
        threadTwo.join();

        System.out.println();
        System.out.println("两个线程执行结束");
    }

    private static class Runnable1 implements Runnable {
        public void run() {
            Singleton s = Singleton.getInstance();

            if (singleton == null){
                System.out.println("全局的singleton：在线程1中已创建");
                singleton = s;
            }

            // 检测单例类实例是否唯一
            boolean isUnique = true;
            if (s!=singleton) {
                isUnique=false;
            }

            System.out.println("线程1中检测实例是否唯一：" + isUnique);
        }
    }

    private static class Runnable2 implements Runnable {
        public void run() {

            Singleton s = Singleton.getInstance();

            if (singleton == null){
                singleton = s;
                System.out.println("全局的singleton：在线程2中已创建");
            }

            //检测单例类实例是否唯一
            boolean isUnique=true;
            if (s!=singleton) {
                isUnique=false;
            }

            System.out.println("线程2中检测实例是否唯一：" + isUnique);
        }
    }

}
```

> 因为线程1被休眠类了50ms，所以导致线程2先实例化了单例类。于是在全局变量singleton和通过getInstance()对比中为true，本身就是赋值所以一样。而当线程1休眠结束后又创建了一个单例类的实例，这时其实存在两个单例，结果通过对比为false。出现了两个不同实例也就形成线程安全问题。

#### 多线程单例基本写法

``` java
public synchronized static Singleton getInstance() {
    if(singleton == null) {
        singleton = new Singleton();
    }
    return singleton;
}
```

机智的你一定发现这里存在的问题。因为一个同步方法执行一百次也有可能比不上这个方法的非同步方法的资源开销，所以每次调用函数getInstance 都得进行一次同步操作是不合算的。我们只是需要在第一次实例化的赋值阶段给予同步即可。

#### 多线程单例写法思考 1

``` java
public static Singleton getInstance() {
    if(singleton == null) {
        synchronized(Singleton.class) {
            singleton = new Singleton();
        }
    }
    return singleton;
}
```

但是这样的缺失去了线程安全。我们来发挥脑力想象一下，当线程1进入到同步语块时，线程1被占用。此时假如有线程2进入到if语句，那么线程2将会等待线程1执行结束。但是这样线程2又会调用 new instance() 实例化，而产生两个不同的实例。于是我们要使用名叫 (Double-checked locking) 的双重锁定技术(本身技术简单，但是依然需要记住专业名词)，代码如下：

#### 多线程单例写法思考 2

``` java
public static Singleton getInstance() {
    if(singleton == null) {
        synchronized(Singleton.class) {
            if(singleton == null) {
                singleton = new Singleton();
            }
        }
    }
    return singleton;
}
```

以上的代码看起来已经不错了，但不幸的事还是发生了，它依旧不能保证正常工作。这是由于指令重排序 (instruction reordering) (编译器优化程序执行的一种方法)导致的。先不管重排序概念如何，已下给予基本概述。

* * *

当线程1进入到同步语块时，语句 singleton = new Singleton(); 的操作顺序可能不像平时理解的那样。
一般来说编译器会有3个步骤：

1. 先分配对象的内存空间
2. 初始化对象
3. 把实例赋值给 singleton

但是经过指令重排后改变了执行顺序：

1. 先分配对象的内存空间
2. 把实例赋值给 singleton
3. 初始化对象
所以就会导致 singleton 虽然已被赋值，但是被赋值的 singleton 却未被初始化。

* * *

所以结合上面代码的来说，在单例的构造函数被调用之前，编译器可以随便访问的单例的成员变量。如果发生这样的情况，虽然线程1在 singleton 赋值后已被占用，但是在 singleton 被初始化之前，线程2依然可以得到一个未初始化的 singleton。

#### Volatile 修饰符

在java5开始提供了一个修饰符 `volatile` 它可以解决这个问题。官方说明如下：

> Java 编程语言允许线程访问共享变量，为了确保共享变量能被准确和一致的更新，线程应该确保通过`排他锁`单独获得这个变量。Java 语言提供了 volatile，在某些情况下比锁更加方便。如果一个字段被声明成 volatile，java 线程内存模型确保所有线程看到这个变量的值是一致的。于是我们继续改进：

带 volatile 的单例

``` java
public class Singleton {
    private volatile static Singleton singleton = null;

    public static Singleton getInstance() {
        if (singleton == null) {
            synchronized (Singleton.class) {
                if (singleton == null) {
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```

上面的代码就可以作为库或程序的一部分使用了。

下面提供了一个可选择性的写法，保证简单，快速，线程安全。

### 静态单例

``` java
public class Singleton {
    public final static Singleton INSTANCE = new Singleton();
    private Singleton() {
        // Exists only to defeat instantiation.
    }
}
```

虽然这样的代码在技术上无懈可击的解决了问题，但是实际上采用这样写法的还是比较少。上面是在程序启动时就被初始化，所以是先占用内存，没有延迟概念。

上面的可以看到我们的单例都是在编译器期指定的，不够灵活，而且通常 getInstance() 是静态方法，不能在子类重新定义它。

#### 注册型单例

``` java
import java.util.HashMap;

public class RegesitorSingleton {
private static HashMap map = new HashMap();

protected RegesitorSingleton() {}

public static synchronized RegesitorSingleton getInstance(String classname) {
    if (classname == null)
        throw new IllegalArgumentException("Illegal classname");

    RegesitorSingleton singleton = (RegesitorSingleton) map.get(classname);

    if (singleton != null) {
        System.out.println("got singleton from map: " + singleton);
        return singleton;
    }
    if (classname.equals("SingeltonSubclass_One"))
        singleton = new SingletonSubclass_One();
    else if (classname.equals("SingeltonSubclass_Two"))
        singleton = new SingletonSubclass_Two();
    map.put(classname, singleton);

    System.out.println("created singleton: " + singleton);
    return singleton;
}
}

class SingletonSubclass_One extends RegesitorSingleton {}// 子类1

class SingletonSubclass_Two extends RegesitorSingleton {}// 子类2
```

单例基类创建并把子类存储一个map里。但是基类却是高维护的，每添加一个子类就得修改 getInstance() 函数。于是我们用到了反射机制代码如下：

### 反射构建单例

``` java
import java.util.HashMap;

public class RegesitorSingleton {
    private static HashMap map = new HashMap();

    protected RegesitorSingleton() {}

    public static synchronized RegesitorSingleton getInstance(String classname) {
        if (classname == null)
            throw new IllegalArgumentException("Illegal classname");

        RegesitorSingleton singleton = (RegesitorSingleton) map.get(classname);

        if (singleton != null) {
            System.out.println("got singleton from map: " + singleton);
            return singleton;
        }
        try {
            singleton = (RegesitorSingleton) Class.forName(classname).newInstance();
        } catch (ClassNotFoundException cnf) {
            System.out.println("Couldn't find class " + classname);
        } catch (InstantiationException ie) {
            System.out.println("Couldn't instantiate an object of type " + classname);
        } catch (IllegalAccessException ia) {
            System.out.println("Couldn't access class " + classname);
        }
        map.put(classname, singleton);
        System.out.println("created singleton: " + singleton);
        return singleton;
    }
}

class SingletonSubclass_One extends RegesitorSingleton {}// 子类1

class SingletonSubclass_Two extends RegesitorSingleton {}// 子类2
```

不清楚反射的使用，可以参考[Java反射机制](http://lumin.tech/blog/java-reflection/)，当然以上最后还有一个被人认为是最好的方法就是枚举实现单例，看到代码如下：

### 枚举单例

``` java
public enum Singleton {
    INSTANCE;

    private Resource instance;
    private Singleton() {
        instance = new Resource();
    }
    public Resource getInstance() {
        return instance;
    }
}
class Resource{}
//调用方式

public class RunMain {
    public static void main(String arg[]) {
        Singleton.INSTANCE.getInstance();
    }
}
```

可以达到目的原因有下：

1. 线程安全
因为 INSTANCE 最后会被编译器处理成 static final 的，并且在 static 模块中进行的初始化，因此它的实例化是在 class 被加载阶段完成，是线程安全的。这个特性也决定了枚举单例不是 lazy 的。
2. 不可被反射实例化
在 Java 中，不仅通过限制 enum 只能声明 private 的构造方法来防止 Enum 被使用 new 进行实例化，而且还限制了使用反射的方法不能通过 Constructor 来 newInstance 一个枚举实例。在你尝试使用反射得到的 Constructor 来调用其 newInstance 方法来实例化 enum 时，回得到一个 exception。
3. 阻止反序列化
传统的单例模式要防止序列化/反序列化的攻击必须要手动来实现 readObject 或者 readResolve 方法，这一点 Enum 已经我们保证了。
