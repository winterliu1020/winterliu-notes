### 数组

参考`Java语言规范`。Java中数组是**动态创建的对象**，数组对象可以赋值给Object类型的变量，Object类的所有方法都可以在数组上调用。

数组对象包含大量的变量。数组元素类型自身可以是数组类型。

有些情况下，数组的成员可以是数组：如果成员类型是Object、Cloneable或java.io.Serializable，那么全部或部分成员可以是数组，因为任何数组对象都可以赋值给这些类型的变量。

### 数组类型

数组的长度不是数组类型的一部分。

数组的元素类型可以是任何类型，可以是简单类型、引用类型，特别的：

- 以接口类型作为元素类型的数组是允许的 （这种数组的元素的值可以是空引用，也可以是实现该接口的任意类型的实例）
- 以abstract类类型作为数组的元素类型也是可以的（这种数组的元素可以是空引用，也可以是自身不是abstract的该abstract类的任意子类的实例）

数组类型的超类型关系与超类关系不同。Integer[]的直接超类型是Number[]，但是根据Integer[]的Class对象，Integer[]的直接超类是Object。**因为Object是所有数组类型的超类型**。

一旦数组对象被创建，它的长度就永远不能改变。

数组类型的单个变量可以包含对不同长度的数组的引用，因为数组的长度不是其类型的一部分。

```java
Animal[] animal = new Animal[];
Dog[] dog = new Dog[];
// 如果Dog类可以赋值给Animal类（也就是Dog类是Animal类的子类），那么animal可以持有对任意数组类型为B[]的实例的引用。这可能会在随后的赋值中导致运行时异常。
// 也就是这里的animal可以持有dog
animal = dog;
```

### 数组创建

数组是由数组创建表达式`int[] a = new int[3]`或数组初始化器`int[] a = {1, 2, 3}`创建的。

### 数组初始化器

数组初始化器可以在域声明或局部变量声明中指定。（也就是可以在方法外面或者方法里面，但是用数组创建表达式`int[] a = new int[3]`则必须在方法里面。

数组初始化器是用逗号分隔的表达式列表，用花括号括起来。

### 数组成员

数组类型的成员包括：

- public final 域 length，它包含了数组的元素数量。
- public 方法clone，它覆盖了Object类中同名的方法，并且不会抛出任何受检异常。数组类型T[]的clone方法的返回值是T[]。多维数组的克隆是浅复制，即它只创建单个新数组，里面的子数组是共享的。
- 所有从Object类继承而来的成员，Object中唯一没有被继承的方法就是clone方法。

### 数组的Class对象

**每个数组都与一个Class对象关联，并与其他具有相同元素类型的数组共享该对象**。尽管数组类型不是类，但是每一个数组的Class对象起到的作用看起来都像是：

- 每个数组类型的直接超类都是Object
- 每个数组类型都实现了Cloneable接口和java.io.Serializable接口

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/Java 基础/20201125091945.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/Java 基础/20201125092045.png)

### 深入分析数组对象

数组的是 Object 的直接子类,它属于“第一类对象”，但是它又与普通的 Java 对象存在很大的不同，从它的类名就可以看出：[I，这是什么东东？？在 JDK 中我就没有找到这个类，话说这个”[I”都不是一个合法标识符。怎么定义成类啊？所以我认为 sun 那帮天才肯定对数组的底层肯定做了特殊的处理。

```java
public class Test {
        public static void main(String[] args) {
            int[] array_00 = new int[10];
            System.out.println("一维数组：" + array_00.getClass().getName());
            int[][] array_01 = new int[10][10];
            System.out.println("二维数组：" + array_01.getClass().getName());

            int[][][] array_02 = new int[10][10][10];
            System.out.println("三维数组：" + array_02.getClass().getName());
        }
    }
    -----------------Output:
    一维数组：[I
    二维数组：[[I
    三维数组：[[[I
```

通过这个实例我们知道：[代表了数组的维度，一个[表示一维，两个[表示二维。可以简单的说数组的类名由若干个’[‘和数组元素类型的内部名称组成。不清楚我们再看:

```java
public class Test {
        public static void main(String[] args) {
            System.out.println("Object[]:" + Object  [].class);
            System.out.println("Object[][]:" + Object[][].class);
            System.err.println("Object[][][]:" + Object[]  [][].class);
            System.out.println("Object:" + Object.class);
        }
    }
    ---------Output:
    Object[]:class [Ljava.lang.Object;
    Object[][]:class [[Ljava.lang.Object;
    Object[][][]:class [[[Ljava.lang.Object;
    Object:class java.lang.Object
```

从这个实例我们可以看出数组的“庐山真面目”。同时也可以看出数组和普通的 Java 类是不同的，普通的 Java 类是以全限定路径名 + 类名来作为自己的唯一标示的，而数组则是以若干个 [+L+ 数组元素类全限定路径+类来最为唯一标示的。这个不同也许在某种程度上说明了数组也普通 Java 类在实现上存在很大的区别，也许可以利用这个区别来使得 JVM 在处理数组和普通 Java 类时作出区分。

我们暂且不论这个[I 是什么东东，是由谁来声明的，怎么声明的（这些我现在也不知道！但是有一点可以确认：这个是在运行时确定的）。先看如下

```java
public class Test {
        public static void main(String[] args) {
            int[] array = new int[10];
            Class clazz = array.getClass();   
            System.out.println(clazz.getDeclaredFields ().length);   
            System.out.println(clazz.getDeclaredMethods ().length);   
            System.out.println(clazz.getDeclaredConstructors().length);   
            System.out.println(clazz.getDeclaredAnnotations().length);   
            System.out.println(clazz.getDeclaredClasses().length);   
        }
    }
    ----------------Output：
    0
    0
    0
    0
    0
```

从这个运行结果可以看出，我们亲爱的 [I 没有生命任何成员变量、成员方法、构造函数、Annotation 甚至连 length 成员变量这个都没有，它就是一个彻彻底底的空类。没有声明 length，那么我们 array.length 时，编译器怎么不会报错呢？确实，数组的 length 是一个非常特殊的成员变量。我们知道数组的是 Object 的直接之类，但是 Object 是没有 length 这个成员变量的，那么 length 应该是数组的成员变量，但是从上面的示例中，我们发现数组根本就没有任何成员变量，这两者不是相互矛盾么？

```java
public class Main {
        public static void main(String[] args) {
            int a[] = new int[2];
            int i = a.length;
        }
    }
```

打开class文件，可以看到main方法的字节码：

```java
0 iconst_2                   //将int型常量2压入操作数栈  
        1 newarray 10 (int)          //将2弹出操作数栈，作为长度，创建一个元素类型为int, 维度为1的数组，并将数组的引用压入操作数栈  
        3 astore_1                   //将数组的引用从操作数栈中弹出，保存在索引为1的局部变量(即a)中  
        4 aload_1                    //将索引为1的局部变量(即a)压入操作数栈  
        5 arraylength                //从操作数栈弹出数组引用(即a)，并获取其长度(JVM负责实现如何获取)，并将长度压入操作数栈  
        6 istore_2                   //将数组长度从操作数栈弹出，保存在索引为2的局部变量(即i)中  
        7 return                     //main方法返回
```

在这个字节码中我们还是没有看到 length 这个成员变量，但是看到了这个:arraylength ,这条指令是用来获取数组的长度的，所以说 JVM 对数组的长度做了特殊的处理，它是通过 arraylength 这条指令来实现的。

其实就是说你在Java代码中用到了arr.length，那么到字节码中会翻译成执行arraylength这条指令，由JVM去执行这条指令来获取数组长度。（所以说到底还是没有声明length这个属性）



参考来源：

1. Java语言规范
2. [极客学院](https://wiki.jikexueyuan.com/project/java-enhancement/java-eighteen.html)