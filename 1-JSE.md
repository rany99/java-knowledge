# Java基础

[TOC]

## 一 面向对象

### 1 面向对象和面向过程

#### 面向对象

让计算机有步骤、按顺序地执行任务，这是一种过程化思维，使用面向过程语言在进行大型项目开发，软件复用和维护时存在很大的问题，模块之间的耦合十分严重。

####面向过程

相较于面向过程开发更适合解决规模较大的问题，可以将一个大问题拆解成若干小问题，对现实事物进行抽象并映射为开发对象，通常事物的特征用属性来进行表示，事物的行为和能力则用方法来进行表示，这更接近与人的思维。

### 2 面向对象的三大特性

封装、继承、多态

####在 Java 中有两种形式可以实现多态：

继承：多个子类对同一方法的重写

接口：实现接口并覆盖接口中的同一方法

## 二 数据类型

###1 基本类型和包装类型的区别

Java 中的数据类型分为基本类型和包装类型，主要有以下区别：

- 存储内容：基本类型直接存储数据，包装类型存储指针

- 存储位置：基本类型数据直接存储在栈中，包装类型数据存储在堆中（对象都在堆中）

- 容器使用：因为需要做自动装箱的原因，容器只能存储包装类型，不能存储基本类型

- 默认值：基本类型的默认值为 0 / false， 而包装类型的默认值为 null，即空指针

- 方法：基本类型中没用方法，包装类型有 equal 等各种的方法

  

### 2 基本类型

#### 基本类型的长度

与 C++ 不同， Java 中的基本类型的长度与运行平台无关，因为 Java 运行在 JVM 上，JVM 屏蔽了系统的差异性

| 类型    | 位数 | 类型   | 位数 |
| ------- | ---- | ------ | ---- |
| boolean | -    | byte   | 8    |
| char    | 16   | short  | 16   |
| int     | 32   | float  | 32   |
| long    | 64   | double | 64   |

####boolean 的存储

- boolean 的值在可以使用 1 bit 来进行存储，但其大小没有明确规定
- JVM 在编译时会将 boolean 类型的值转换为 int，其中使用 1 来表示 true， 2 来表示 false
- JVM 支持boolean 数组，但是通过读写 byte 数组来实现的。

#### float 和 double 浮点数的存储

> 浮点数存在精度损失问题，一般不直接使用`==`或`!=`进行比较，而是作差后和一个非常小的数字进行比价`如x-y < eps = 10^-8`则认为`x==y`。

浮点数一般采用科学计数法来表示和存储，单精度和双精度**都分为三个部分：**

- **符号位** (Sign)：0代表正数，1代表为负数；

- **指数位** (Exponent)：用于存储科学计数法中的指数数据；

- **尾数部分** (Mantissa)：采用移位存储尾数部分；

**float**

![img](E:/面经/JavaNotesForInterview/images/float_1.gif)

**double**

![img](E:/面经/JavaNotesForInterview/images/float_2.gif)

如`8.25f`表示为`8.25 * 10^0`，对应的二进制表示为`1000.01b = 1.0001b * 2^3`，<font color="red">（任何小数都可以表示为1.xyz * 2^n，所以第一个1可以去掉，实际上尾数有效位为24位）</font>，所以8.25的存储方式为：

![img](E:/面经/JavaNotesForInterview/images/float_3.gif)

### 3 包装类型

#### 包装类不能被继承

所有的数值类型的包装类都直接继承与父类 Number，非数值类型 Character 和 Boolean 则直接继承与 Object 类。

所有的包装类都不存在无参构造器

所有的包装类都是不可变的（被 final 修饰），因此一旦被创建，包装类的内部值便无法被改变。

#### 为什么要有包装类

Java 作为面向对象语言，包装类让基础类有了对象的特征，便于在各种容器中使用。例如在 HashMap 中的数据操作需要使用到 hashCode() 和 equals() 等方法。

基本类型都有对应的包装类型，基本类型与其对应的包装类型之间的赋值使用自动装箱与拆箱完成。

### 4 缓存池

#### new Integer(123) 和 Integer.valueOf(123) 的区别

new Integer(123)  每次都会创建一个新的对象

Integer.valueOf(123) 会使用缓存池中的对象，多次调用会获得同一个对象的引用

```java
Integer x = new Integer(123);
Integer y = new Integer(123);
System.out.println(x == y);    // false
Integer z = Integer.valueOf(123);
Integer k = Integer.valueOf(123);
System.out.println(z == k);   // true
```

#### Integer.valueOf() 方法的实现

先判断值是否在缓存池中，如果存在则返回缓存池中的对象，否则新建一个包装类对象

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

#### 缓存池范围

- **boolean：** true、false
- **byte:** all values
- **short:** -128-127
- **int:** -128-127
- **char:** \u0000 to \u007F

在使用基本类型对应的包装类型时，如果该数值在缓冲池范围内，就可以直接使用缓冲池中的对象

> *在 jdk 1.8 所有的数值类缓冲池中，Integer 的缓冲池 IntegerCache 很特殊，这个缓冲池的下界是 - 128，上界默认是 127，但是这个上界是可调的，在启动 jvm 的时候，通过 -XX:AutoBoxCacheMax = &lt;size&gt; 来指定这个缓冲池的大小，该选项在 JVM 初始化的时候会设定一个名为 java.lang.IntegerCache.high 系统属性，然后 IntegerCache 初始化的时候就会读取该系统属性来决定上界。*

## 三 String

###1 String 的不可变性

String 被声明为 final，因此它不可以被继承。

Java 8 中，String 的内部采用 char 数组类存储数据

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
}
```

Java 9 之后， String 的内部采用 byte 来存储数据，同时添加了 coder 来标识字符串使用了哪种编码规则

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final byte[] value;

    /** The identifier of the encoding used to encode the bytes in {@code value}. */
    private final byte coder;
}
```

### 2 不可变的好处

#### 可以缓存 hash 值

因为 String 的 hash 值经常被使用，比如经常使用 String 来作为 HashMap 的 key，因此其不可变的特点可以使得 hash 也不可变，因此只需要经过一次 hash 计算。

#### String Pool

如果一个 String 对象已经被创建过了，便会从 String Pool 中取得引用。这便要求 String 是不可变的。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/image-20191210004132894.png"/> </div><br>

#### 安全性

String 常常被作为重要参数使用，不可变性保障了安全性。

#### 线程安全

String 的不可变性保证了其具有线程安全性，可以在多个线程中安全地使用。

###3 String、StringBuffer、StringBuilder 的区别

#### 是否可变

**String** 不可变（成员变量 char[] 和 byte[] 被 final 修饰）

**StringBuffer** 与 **StringBuilder** 可变（提供了相应的修改 char[] 的方法，char[] 支持自动扩容）

#### 是否线程安全

- String 因其不可变性是线程安全的
- StringBuilder 不是线程安全的
- StringBuilder 因其内部使用 synchronized 进行同步，因此是线程安全的

#### 使用场景

不常改变的数据 String

经常改变的数据根据是否需要考虑线程安全使用 StringBuilder 或 StringBuffer

### 4 String Pool

#### 字符串常量池

字符串常量池(String Pool)保存着所有字符串字面量(literal strings)，这些字面量在编译时期便已经确定。

如果像在运行中将字符串添加到字符串常量池（literal strings)，可以使用 String 的 intern() 方法将字符串添加到 String Pool 中。

#### intern() 方法的调用过程

1. 在 String Pool 中判断是否存在一个值能否与该插入的字符串值相等（使用 equals()）；
2. 如果存在相等的字符串，则直接返回 String Pool 中该字符串的引用；
3. 如果不存在相等的字符串，则在 String Pool 中新增该字符串，并返回其引用；

#### 字符串常量池的位置

- 在 Java 7 之前，String Pool 被放在运行时常量池中，属于永久代
- 在 Java 7 之后，String Pool 被放在堆中，因为永久代的空间有限，在 String Pool 中 String 变量越来越多时会导致 OutOfMemoryError 错误

### 5 new String("a")

#### 使用 new String(“a”) 会创建两个字符串对象

- “a” 属于字符串字面量，如果 String Pool 中还没有 “a” 字符串时，则会在 String Pool 中创建一个字符串对象，指向 "a" 的字符串字面量
- 使用 new 的方法在堆中创建一个字符串对象

```
public String(String original) {
    this.value = original.value;
    this.hash = original.hash;
}
```

根据使用 String 构造函数的源码可以看出，在将一个字符串对象作为另一个字符串对象的构造函数参数时，并不会完全复制 value 的内容，而是会指向同一个 value 数组。

## 四 运算

### 1 参数传递

Java 的参数是以值传递的形式传入方法中，而不是引用传递。

当传递一个对象时，本质是将对象的地址以值得形式传递到形参中，这样在方法中改变对象的字段值时也会改变原对象的字段值，因为引用的是同一个对象，对同一个地址的对象进行修改。

### 2 隐式类型转换

Java 不能隐式执行向下转型，这会导致精度降低。

#### int 与 short

```java
short s1 = 1;	//错误的
s1 = s1 + 1;
```

字面量 1 是 int 类型，比 short 的类型精度要高，因此不能隐式地将 int 类型向下转换为 short 类型。

但是在使用 += 或者 ++ 时会执行隐式类型转换

```java
s1 += 1;
s1++;
//s1 = (short) (s1 + 1);
```

#### float 与 double

```java
float f = 1.1; //错误的
```

字面量 1.1 属于 double 类型，因此不能够直接将 1.1 赋值给 float 类型，因为这是向下转型，而 1.1f 才是 float 类型。

```java
float f = 1.1f; //正确的
```

### 3 switch

#### switch 的作用类型

- switch可作用于**char、byte、short、int 及其对应的包装类**
- switch不可作用于**long、double、float、boolean 及其对应的包装类**
- switch中可以是 **Enum** 类型(Java 5之后)
- switch中可以是 **String **类型(Java 7之后才可以作用在string上)

#### switch 为什么不支持 long、double、float、boolean

switch 的设计初衷是对那些只有少数几个值的类型进行等值判断，如果值过于复杂，那么还是用 if 比较合适。

## 五 关键字

### 1 final

#### final 作用于数据

表示该数据为常量，可以使编译时常量，也可以是在运行时被初始化后不能被改变的常量

- 基本类型：final使其数值不可变
- 引用类型：final使引用不可变，也就是不能应用其他对象（始终指向这一对象），但是被引用的对象本身是可以被修改的。

```java
final int x = 1;
// x = 2;  // cannot assign value to final variable 'x'
final A y = new A();
y.a = 1;
```

#### final 作用于方法

- 被 final 作用的方法不能被子类重写
- private 方法隐式地被指定为 final，如果在子类中定义的方法和基类中的 private 的签名相同，这种写法并不是重写基类方法，而是在子类中定义了一个新的方法。

#### final 作用于类

final 作用的类**不允许被继承**

### 2 static

#### 静态变量

- 静态变量：又被称为类变量，这个类的所有实例共享这一静态变量，可以通过类名直接对它进行防卫。静态变量在内存中只存在一份。
- 实例变量：每创建一个实例就会产生一个实例变量，他与该实例寿命周期一致。

#### 静态方法

- 静态方法在类加载的时候就已经存在了，它不依赖于任何实例。
- 静态方法必须有实现，即静态方法不能是抽象方法。
- 静态方法只能访问所属类的静态字段和静态方法，且静态方法中不能使用 this 和 super 关键字，因为这两个关键字与具体对象关联。

#### 静态语句块

静态语句块只在类初始化时执行一次

#### 静态内部类

静态内部类与非静态内部类的区别

- 非静态内部类依赖于外部类的实例，只有先创建外部类实例，才能用这个实例去创建非静态内部类。
- 静态内部类不依赖于外部类。

```java
public class OuterClass {

    class InnerClass {
    }

    static class StaticInnerClass {
    }

    public static void main(String[] args) {
        // InnerClass innerClass = new InnerClass(); // 'OuterClass.this' cannot be referenced from a static context
        OuterClass outerClass = new OuterClass();
        InnerClass innerClass = outerClass.new InnerClass();
        StaticInnerClass staticInnerClass = new StaticInnerClass();
    }
}
```

静态内部类不能访问外部类的非静态的方法和变量

#### 初始化顺序

- 静态变量和静态语句块的执行优先于实例变量和普通语句块。
- 静态变量和静态语句块的执行顺序取决于他们在代码中的先后顺序。
- 当存在继承情况时，初始化顺序为：
  1. 父类（静态变量、静态语句块）
  2. 子类（静态变量、静态语句块）
  3. 父类（实例变量、普通语句块）
  4. 父类（构造函数）
  5. 子类（实例变量、普通语句块）
  6. 子类（构造函数）

## 六 Objective

Objective 通用方法

### 1 equals()

####equals 的五个特性

(1）自反性

```java
x.equals(x)
```

(2）对称性

```
x.equals(y) == y.equals(x)
```

(3）传递性

```
if (x.equals(y) && y.equals(z)) {
	x.equals(z)
}
```

(4）一致性

多次调用 equals() 不改变结果

(5）与 null 的比较

对于任何不是 null 的对象 x 调用 x.equals(null) 结果均为 false

#### equals() 和 == 的区别

- 基本类型没有 equals() 方法，使用 == 判断值是否相等
- 引用类型使用 == 判断是否引用同一个对象，及地址是否相同
- equals() 默认比较两个对象的地址是否相同，用户可以重写该方法，来实现比较对象的属性是否相同（例如 String 中便重写了 equals()）。

### 2 hashCode()

等价的两个对象散列值一定是相同的，散列值相同的对象不一定等价，因为哈希值得计算具有随机性，两个值不相同的对象也有可能计算得出相同的哈希值。

在覆盖 equals() 方法时总是应当覆盖 hashCode()方法，保证等价的两个对象的哈希值也相等，因为 equals() 判断相等的对象的 hashCode 值也应该相等。

HashSet 和 HashMap 等集合类使用 hashCode() 来计算对象应该存储的位置，因此如果要将对象添加到集合中就需要让对象所属类实现 hashCode() 方法。

#### equals() 和 hashCode() 的联系

- 重写 equals() 方法必须重写 hashCode() 方法；
- 两个对象 equals() 相等则 hashCode() 必需相等；
- 两个对象 hashCode() 相等 equals() 不一定相等，可能是哈希冲突

