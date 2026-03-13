先看四个小点：

1.JVM为每个加载的class及interface创建了对应的Class实例来保存class及interface的所有信息。

2.获取一个class对应的Class实例后，就可以获取class的所有信息。

3.通过Class实例获取class信息的方法称为反射。

4.JVM总是动态加载class，可以在运行期根据条件来控制加载class。

除了八种基本数据类型外，Java的其它类型全部都是class（包括interface）。仔细思考可以得出结论：class（包括interface）的本质是数据类型（Type）。JVM在执行过程中遇到某一种class类型时，才会把它加载到内存。

每加载一种class，JVM就为其创建一个`Class`类型的实例，并关联起来。可以看一下Class这个class:

```java
public final class Class {
    private Class() {}
}
```

以String类为例，我们会把String.java文件编译成String.class文件，然后只有当用到这个String类的时候，JVM才会把它加载到内存，在这里JVM会首先读取String.class文件到内存，然后为String类创建一个Class实例并关联起来。

```java
Class cls = new Class(String);
```

注意这个cls是由JVM内存创建的，看JDK源码就可以发现Class的构造方法是private的，所以只有JVM能创建Class实例，自己的Java程序无法创建。

```java
private Class(ClassLoader loader) {
    // Initialize final field for classLoader.  The initialization value of non-null
    // prevents future JIT optimizations from assuming this final field is null.
    classLoader = loader;
}
```

**所以JVM持有的每个Class实例都指向一个数据类型（class或interface）**

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/Java 基础/20201109234338.png)

JVM会为每个加载的class创建对应的Class实例，并会在实例中保存该class的所有信息，包括类名、包名、父类、实现的接口、所有方法、字段...所以只要拿到某个Class实例就可以通过实例获取到这个实例对应的class的所有信息（反射）。

```java
// 如何拿到一个类的Class实例？
Class cls = String.class;

String s = "Hello";
Class cls = s.getClass();

Class cls = Class.forName("java.lang.String");
```

注意Class实例在JVM中是唯一的，所以不管你用哪一种方法获取Class实例，只要获取的是同一个类，那么就是同一个实例。可以用`==`来比较一下两种方法获取到的实例。

### Class实例比较和instanceof的区别

```java
Integer n = new Integer(123);

boolean b1 = n instanceof Integer; // true，因为n是Integer类型
boolean b2 = n instanceof Number; // true，因为n是Number类型的子类

boolean b3 = n.getClass() == Integer.class; // true，因为n.getClass()返回Integer.class
boolean b4 = n.getClass() == Number.class; // false，因为Integer.class!=Number.class
```

### 一些小细节

注意到数组（比如String[]）也是一种Class，且不同于String.class，它的类名是`[Ljava.lang.String`。此外，JVM为每一种基本类型如int也创建了`Class`，通过`int.class`访问。

可以通过class实例来创建对应类型的实例：

```java
// 获取String的Class实例:
Class cls = String.class;
// 创建一个String实例:
String s = (String) cls.newInstance();
```

### Java动态加载

JVM在执行Java程序的时候，并不是一次性把所有用到的class全部加载到内存，而是第一次需要用到class时才加载。

没用到的类不会进内存，而只是以.class文件的形式躺在那里。

动态加载`class`的特性对于Java程序非常重要。利用JVM动态加载`class`的特性，我们才能在运行期根据条件加载不同的实现类。



本文学习来源：[廖雪峰](https://www.liaoxuefeng.com/wiki/1252599548343744/1264799402020448)