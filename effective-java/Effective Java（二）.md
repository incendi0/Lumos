# Effective Java（二）

## 通用方法设计

Object被设计用来扩展，非final方法（equals，hashCode，toString，clone和finalizer）都被设计成可以override的，只要遵守通用约定。类似的Comparable.compareTo也有类似约定

### 十、Override euqals方法时遵守的约定

如果不覆盖，类的实例只能和自身相等。如下情况：

1. 类的每个实例本质时唯一的。例如Thread类，代表活动实体而不是值；
2. 没必要提供逻辑相等的功能。例如Pattern类，检测两个正则表达式是否相同，可能设计者认为这样的功能不会有使用者需要；
3. 超类已经覆盖了equals，而且适用于子类。例如集合中List 实现从AbstractList 继承equals 实现， Map 实现从AbstractMap 继承equals 实现；
4. 类是私有的，或者package级别私有，equals方法不会被调用。

当类有逻辑相等测试的需求，而且超类并没有覆盖equals时，这种类被称为值类。例如Integer，String。

覆盖equals时，要遵守它的约定，实现等价关系。如下：

1. 自反性。非null引用，x.equals(x)返回true；
2. 对称性。非null引用x，y，满足x.equals(y) == true时，y.equals(x) == true；
3. 传递性。非null引用x，y，z，满足x.equals(y) == ture，且y.equals(z) == true时，x.equals(z) == true；
4. 一致性。equals方法内比较的属性没被修改时，多次调用x.equals(y)应该返回一致的结果；
5. 对于非null引用x，x.equals(null) == false。

我们无法做到，在扩展可实例化的类同时，既增加了属性，同时还能保证equals约定。例如Point类和ColoredPoint类。

但是可以通过“复合优于继承”的方式扩展Point类，即ColoredPoint有两个属性color和Point，而不是直接继承Point。

里氏替换原则（ Liskov substitution principle ）认为， 一个类型的任何重要属性也将适用于它的子类型，因此为该类型编写的任何方法，在它的子类型上也应该同样运行得很好。

实现高质量的equals方法：

1. == 检查参数是否就是该对象的引用；
2. instanceof操作符检查参数是否时正确的类型；
3. 类型转换；
4. 对比关键属性。

同时，有一些建议：

1. 覆盖equals时总要覆盖hashCode;
2. 不要企图让equals方法过于智能，不要寻求复杂的等价关系；
3. 不要改变equals方法签名。

工作中可以考虑使用IDE或者工具类自动生成equals方法。

### 十一、覆盖equals时总要覆盖hashCode

如果不这么做，违反hashCode的通用约定，在使用HashSet，HashMap等基于散列的集合就不会正常工作。

约定如下：

1. 应用执行期间，如果equals方法使用的信息没有改动，多次调用返回一致的值；
2. 如果equals方法返回true，则hashCode相等；
3. 如果equals方法返回false，则hashCode不一定要求返回不同的值。

具体编写可以使用IDE生成或者参考Guava的com.google.common.hash.Hashing。

### 十二、始终覆盖toString

默认它包含类的名称，以及一个“＠”符号，接着是散列码的无符号十六进制表示法。toString 方法应该以美观的格式返回一个关于对象的简洁、有用的描述。

### 十三、谨慎地覆盖clone

Cloneable 接口决定了Object中受保护的clone 方法实现的行为：如果一个类实现了Cloneable, Object 的clone方法就返回该对象的逐域拷贝，否则就会抛出CloneNotSupportedException 异常。

事实上，实现Cloneable 接口的类是为了提供一个功能适当的公有的clone 方法。

clone方法的约定是非常弱的，如下：

1. x.clone() != x && x.clone().getClass() == x.getClass() 返回true
2. x.clone().equals(x)

上述也不是必须的。

要实现一个功能良好的clone方法：调用super.clone()，由此得到的对象将是原始对象功能完整的克隆（clone ）。不可变的类永远不需要提供clone方法。实际上， clone 方法就是另一个构造器；必须确保它不会伤害到原始的对象， 并确保正确地创建被克隆对象中的约束条件 。

在数组上调用clone 返回的数组，其编译时的类型与被克隆数组的类型相同。这是复制数组的最佳习惯做法。事实上，数组是clone 方法唯一吸引人的用法。

Cloneable 架构与引用可变对象的final 域的正常用法是不相兼容的。

如果你扩展一个实现了Cloneable 接口的类，那么你除了实现一个行为良好的clone 方法外，没有别的选择。否则，最好提供某些其他的途径来代替对象拷贝。对象拷贝的更好的办法是提供一个拷贝构造器（ copy constructor）或拷贝工厂（ copy factory ） 。

像构造器一样， clone 方法也不应该在构造的过程中，调用可以覆盖的方法。如果clone 调用了一个在子类中被覆盖的方法，那么在该方法所在的子类有机会修正它在克隆对象中的状态之前，该方法就会先被执行，这样很有可能会导致克隆对象和原始对象之间的不一致。

### 十四、考虑实现Comparable

将这个对象与指定的对象进行比较。当该对象小于、等于或大于指定对象的时候，分别返回一个负整数、零或者正整数。如果由于指定对象的类型而无法与该对象进行比较，则抛出ClassCastException 异常。

在下面的说明中，符号sgn(expr巳ssion）表示数学中的signum 函数，它根据表达式（ expression ）的值为负值、零和正值，分别返回-1、0或l 。

1. 实现者必须确保所有的x 和y 都满足sgn(x.compareTo(y)) == -sgn (y.compareTo (x)) 。（这也暗示着，当且仅当y.compareTo (x)抛出异常时， x . compareTo(y)才必须抛出异常）；
2. 可传递；
3. 实现者要确保当x.compareTo(y) == 0时，满足对于所有的z，sgn(x.compareTo(z)) == sgn(y.compareTo(z))；
4. 建议(x.compareTo(y)) == x.equals(y)。