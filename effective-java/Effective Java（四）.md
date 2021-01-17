# Effective Java（四）

### 泛型

Java5开始，引入泛型。泛型信息在运行时会被擦除，但是正确编写的泛型程序，编译器会在需要的地方自动插入类型转换。

### 二十六、不要使用Raw Type

使用Raw type会失去编译期带来的类型安全优势。

使用Raw Type可能会带来运行时异常，参考下面代码：

```java
public static void main(String[] args) {
    List<String> xs = new ArrayList<>();
    unsafeAdd(xs, 1);
//. Exception in thread "main" java.lang.ClassCastException: java.lang.Integer cannot be cast to java.lang.String  
    String s = xs.get(0);
}
private static void unsafeAdd(List xs, Object toAdd) {
    xs.add(toAdd);
}
```

如果使用泛型，不确定或者不关心泛型参数，可以使用unbound wild card，如方法：

```java
static int numElementsInCommon(Set<?> s1, Set<?> s2);
```

为了防止加入元素破坏泛型约束，除null外任何元素都不能加入到Collection<?>中，否则会编译报错。如果无法接受这个限制，则可以使用泛型方法或者有限制的通配符类型。

有些情况是例外:

1. 类字面量（class literal）使用原生类型，例如，List.class、String[].class、int.class。类字面量不允许参数化类型，List\<String\>.class是不合法的。
2. instanceof操作符，运行时，泛型信息被擦除，在参数化类型而非无限制通配符类型上使用instanceof是不合法的。

### 二十七、消除非受检异常

```java
@SuppressWarnings("unchecked")
public <T> T[] toArray(T[] a) {
    if (a.length < size)
// Make a new array of a's runtime type, but my contents:
        return (T[]) Arrays.copyOf(elementData, size, a.getClass());
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}

public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
    @SuppressWarnings("unchecked")
    T[] copy = ((Object)newType == (Object)Object[].class)
        ? (T[]) new Object[newLength]
        : (T[]) Array.newInstance(newType.getComponentType(), newLength);
    System.arraycopy(original, 0, copy, 0,
                     Math.min(original.length, newLength));
    return copy;
}
```

非受检警告很重要，不要忽略它们。每一条警告都表示可能在运行时抛出ClassCastException 异常。要尽最大的努力消除这些警告。如果无法消除非受检警告，同时可以证明引起警告的代码是类型安全的就可以在尽可能小的范围内使用＠ Suppress Warnings("unchecked")注解禁止该警告。要用注释把禁止该警告的原因记录下来。

### 二十八、列表优于数组

数组和泛型有以下不同：

1. 数组是协变的（covariant），列表是不变的（invariant）。如果sub是sup的子类，sub[]也是sup[]的子类，但是List\<sub\>和List\<sup\>没关系。参考下列代码：

   ```java
   //        运行时异常
           Object[] oa = new Long[1];
           oa[0] = "asdfasdf";
   //        编译期异常
           List<Object> ol = new ArrayList<Long>();
           ol.add("asdfasdf");
   ```

2. 数组是具体化的（reified）。但泛型只在编译时强化它们的类型信息，并在运行时丢弃（或者擦除）它们的元素类型信息。不可具体化的（ non-reifiable ）类型是指其运行时表示法包含的信息比它的编译时表示法包含的信息更少的类型。唯一可具体化的(reifiable)参数化类型是无限制的通配符类型，如List \<?\>和Map\<?, ?\>数组在运行时知道元素类型，是创建无限制通配类型的数组是合法的。

这些不同让数组和泛型无法很好的一起工作。可以声明泛型数组，但是创建泛型数组是不合法的。

### 二十九、优先考虑泛型

无法创建不能具体化类型的数组，但是底层是数组的话，怎么解决这个问题呢？

第一种方法是，创建Object数组，转换成泛型数组类型，编译器会给unchecked cast的警告，如果数组是类内private属性，永远不会暴露给客户端或者其他方法，则可以证明这个cast没有破坏程序类型安全性，此时可以增加@SuppressWarning注解来禁止这个警告；

第二种方法是，底层使用Object数组，每次手动cast类型。

### 三十、优先考虑泛型方法

泛型方法也可以获得类型安全的便利。

```java
private static UnaryOperator<Object> identity = t -> t;
@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identityFunction() {
    return (UnaryOperator<T>) identity;
}
System.out.println(identityFunction().apply(1));
System.out.println(identityFunction().apply("asdf"));
```

恒等函数很特殊： 它返回未被修改的参数，因此我们知道无论T 的值是什么，用它作为Unary Function\<T\>都是类型安全的。因此，我们可以放心地禁止由这个转换所产生的未受检转换警告。

### 三十一、利用有限制通配符（bounded wild card）来提升API 的灵活性

为了获得最大限度的灵活性，要在表示生产者或者消费者的输入参数上使用通配符类型。

PECS 表示producer-extends，consumer-super 。

```java
// HashMap方法
public void putAll(Map<? extends K, ? extends V> m)
// Collection默认方法
default boolean removeIf(Predicate<? super E> filter)
```

```java
public static void swap(List<?> list, int i, int j) {
    swapHelper(list, i, j);
}
public static <E> void swapHelper(List<E> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
```

### 三十二、慎用同时使用泛型和可变参数

可变参数和泛型不能良好地合作，这是因为可变参数设施是构建在顶级数组之上的一个技术露底，泛型数组有不同的类型规则。虽然泛型可变参数不是类型安全的，但它们是合法的。如果选择编写带有泛型（或者参数化）可变参数的方法，首先要确保该方法是类型安全的，然后用＠ SafeVarargs 对它进行注解，这样使用起来就不会出现不愉快的情况了。