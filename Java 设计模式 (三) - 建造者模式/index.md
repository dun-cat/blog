## Java 设计模式 (三) - 建造者模式 
### 概念

建造者模式也是实例化对象的一类设计模式。它不像同类的工厂模式一样抽象化类之后去创建，而是另建一个 `Builder` 帮助类去构建初始化参数而后创建，显然创建类关注的焦点不同。

#### 举例

拿披萨举例

内建Builder类的Pizza类

``` java
public class Pizza {
    private int size;
    private boolean cheese;
    private boolean pepperoni;
    private boolean bacon;

    public static class Builder {
        // 必选项
        private final int size;
        // 可选项
        private boolean cheese = false; // 奶酪
        private boolean pepperoni = false;// 意大利辣香肠
        private boolean bacon = false; // 咸肉

        public Builder(int size) {
            this.size = size;
        }

        public Builder cheese(boolean value) {
            cheese = value;
            return this;
        }
        public Builder pepperoni(boolean value) {
            pepperoni = value;
            return this;
        }
        public Builder bacon(boolean value) {
            bacon = value;
            return this;
        }
        //建造Pizza
        public Pizza build() {
            return new Pizza(this);
        }
    }

    private Pizza(Builder builder) {
        size = builder.size;
        cheese = builder.cheese;
        pepperoni = builder.pepperoni;
        bacon = builder.bacon;
    }
}
```

执行类

``` java
public class RunMain {
    public static void main(String[] args) {
        Pizza.Builder builder=new Pizza.Builder(10)
            .bacon(false)
            .cheese(true)
            .pepperoni(true);
        Pizza pizza =builder.build();
    }
}
```

android自带弹框也是此种设计模式

弹框调用

``` java
AlertDialog.Builder builder =new AlertDialog.Builder(this)
        .setTitle("弹框标题")
        .setMessage("弹框内容")
        .setIcon(android.R.mipmap.sym_def_app_icon);
AlertDialog alertDialog =builder.create();
```
