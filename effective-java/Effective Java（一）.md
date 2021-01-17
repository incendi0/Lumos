# Effective Java（一）

### 一、用静态工厂方法代替构造器

优势：

1. 可读性好，相同签名的构造器可以通过名称区分；
2. 每次调用，不必创造新的对象实例；
3. 可以返回子类对象，适用于基于接口的框架，比如Collections；
4. 根据参数值可以返回不同的类；
5. 方法返回的对象所属的类，在编写包含该静态工厂方法的类时可以不存在。构成了服务者框架（例如JDBC）的基础。

劣势：

1. 如果类的构造器不是public或者protected，无法被继承；
2. 不容易被发现，需要使用者熟悉API，提供者提供JavaDoc。

### 二、多个构造器参数时考虑使用Builder模式

telescoping constructor容易写出冗长代码，JavaBean模式可能在构造过程中导致不一致的状态，同时这个类不可能成为不变类。

Builder模式模拟了具名参数，同时也适用于类层次结构，构建fluent链式API。

```java
public abstract class Pizza {
    public enum Topping {HAM, MUSHROOM, ONION, PEPPER, SAUSAGE};
    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
        public T addToppings(Topping top) {
            toppings.add(Objects.requireNonNull(top));
            return self();
        }

        abstract Pizza build();

//        in subclass, return this
//        simulated self-type
        protected abstract T self();
    }

    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone();
    }

    public static void main(String[] args) {
        NyPizza nyPizza = new NyPizza.Builder(NyPizza.Size.SMALL)
                .addToppings(Topping.HAM).addToppings(Topping.ONION).build();
        Calzone calzone = new Calzone.Builder().addToppings(Topping.MUSHROOM)
                .sausageInside().build();
    }
}

class NyPizza extends Pizza {
    public enum Size {SMALL, MEDIUM, LARGE};
    private final Size size;

    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override
//        子类方法声明返回超级类中声明的返回类型的子类型，这被称作协变返回类型(covariant return type ）。
        NyPizza build() {
            return new NyPizza(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }
}

class Calzone extends Pizza {
    private final boolean sausageInside;

    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sausageInside  = false;

        public Builder sausageInside() {
            sausageInside = true;
            return this;
        }

        @Override
        Calzone build() {
            return new Calzone(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }

    private Calzone(Builder builder) {
        super(builder);
        sausageInside = builder.sausageInside;
    }
}
```

### 三、单例

单例是指仅仅被实例化一次的类，常被用来表示无状态的对象或者本质上是唯一的系统组件。

常见实现单例有两种方法，私有构造器+公有静态成员或者静态工厂方法。

本章引入了第三种方法，生命一个包含单个元素的枚举类型。这种方法提供了序列化机制和阻止反射攻击产生多个实例。

### 四、私有化构造函数

带来副作用，无法被sunclass。

### 五、依赖注入引用资源

### 六、避免创建不必要的对象

不可变对象，例如String，Integer可以被重用，使用静态工厂方法。

### 七、消除过期的引用

内存泄漏问题涞源：

1. 类内部需要管理内存，无用置null；
2. 缓存，使用weakHashMap处理缓存项的生命周期由键的外部引用决定而不是由值决定的情况，使用ScheduledThreadPoolExecutor后台线程或者LinkedHashMap中removeEldestEntry方法来处理不确定“缓存项的生命周期是否有意义”。
3. 监听器和回调。如果客户端不断注册回调，但是没有取消注册，会导致回调积压。确保回调被立即当作垃圾回收的最佳方法是保存弱引用(weak ref)，或者使用WeakHashMap。

### 八、避免使用finalizer和cleaner方法

### 九、使用try-with-resources

