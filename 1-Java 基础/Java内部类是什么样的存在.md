参考：https://blog.csdn.net/theusProme/article/details/50493381

内部类是一种编译时的概念，编译结束之后就会产生两个class文件了。

**成员内部类**（对应外部类的属性、方法）

1. 作为外部类的一个成员存在，与外部类的属性、方法并列。
2. 成员内部类中不能定义static变量，但是可以访问外部类的所有成员。

**局部内部类**（对应外部类方法中的局部变量）

1. 定义在方法内部
2. 在局部内部类中也不可以定义static变量
3. 局部内部类中可以访问外部类的局部变量（即方法内的变量），但是变量必须是final的。
4. 在类外不可直接生成局部内部类（保证局部内部类对外不可见），想要new一个局部内部类，必须在局部内部类所属的方法中去new。
5. **通过内部类和接口达到一个强制的弱耦合，用局部内部类去实现接口，然后在方法中返回接口类型，这样就能让局部内部类不可见，屏蔽实现类的可见性。**

**静态内部类**（可以在任何地方定义）

1. 它只能访问外部类的静态成员

**匿名内部类**

1. 这是一种特殊的局部内部类，通过匿名类实现接口。
2. 匿名内部类一般是为了继承或者实现接口，并不需要增加额外的方法，只是去重写接口中的方法。而且为得也只是一个实例，所以并不需要知道实际类型和类名。

---

非匿名内部类命令规则：OutClass$InnerClass

匿名内部类命名规则：OutClass$1, OutClass$2。。

**关于局部内部类、成员内部类访问外部类属性或者成员变量的问题：** 

`成员内部类肯定不能访问外部类方法中局部变量咯，所以不讨论成员内部类访问局部变量；只讨论局部内部类访问局部变量`

**如果局部内部类访问外部类的局部变量：**

局部变量（不管你是值类型还是引用类型）得加final，如果你不加final，Java8之后，编译器其实会自动给你加effective final的，然后你只能用局部变量，而不能修改局部变量。

**为什么得加final呢？**

**比较好的文章：**https://www.iteye.com/blog/feiyeguohai-1500108

因为它毕竟是局部变量，这个变量的生命周期只存在于这个方法中，当这个方法执行完成，局部变量会被回收。

而如果你在局部内部类中用了局部变量，因为方法可能结束了 然后回收局部变量，但是局部内部类对象在方法结束之后还没死啊，这就会**造成局部内部类对象中拿的是一个已经回收的局部变量**，为了避免这种情况，Java采用给他们加final的措施，因为加了final，局部变量就无法改变，就可以保证在局部内部类中复制的局部变量和外部类的局部变量始终是一样的。具体细节：

> 当变量是final时,通过将final局部变量"复制"一份,复制品直接作为局部内部中的数据成员.这样:当局部内部类访问局部变量 时,其实真正访问的是这个局部变量的"复制品"(即:这个复制品就代表了那个局部变量).因此:当运行栈中的真正的局部变量死亡时,局部内部类对象仍可以 访问局部变量(其实访问的是"复制品"),给人的感觉:好像是局部变量的"生命期"延长了.
>
> 
> 那么:核心的问题是:怎么才能使得:访问"复制品"与访问真正的原始的局部变量,其语义效果是一样的呢?
> 当变量是final时,若是基本数据类型,由于其值不变,因而:其复制品与原始的量是一样.语义效果相同.(若:不是final,就无法保证:复制品与原始变量保持一致了,因为:在方法中改的是原始变量,而局部内部类中改的是复制品)
>
> 当 变量是final时,若是引用类型,由于其引用值不变(即:永远指向同一个对象),因而:其复制品与原始的引用变量一样,永远指向同一个对象(由于是 final,从而保证:只能指向这个对象,再不能指向其它对象),达到:局部内部类中访问的复制品与方法代码中访问的原始对象,永远都是同一个即:语义效 果是一样的.否则:当方法中改原始变量,而局部内部类中改复制品时,就无法保证:复制品与原始变量保持一致了(因此:它们原本就应该是同一个变量.)

其实就是一个**局部变量**和**局部内部类对象**的生命周期不同的问题，Java采用复制的方式来解决。具体来说：

> ```java
> public class Test {
>    public void test(final int b) { 
>     final int a = 10;
>  
>     new Thread(){  
>     public void run() {
>          System.out.println(a);
>     }
>     }.start();
>   }
> }
> ```
>
> 如果局部变量的值在编译期间就可以确定，则直接在匿名内部里面创建一个拷贝。如果局部变量的值无法在编译期间确定，则通过构造方法传参的方式来对拷贝进行初始化赋值（可以反编译查看，发现Test$1的构造方法有两参数，一个是指向外部类对象的引用，一个是int型变量。很显然，int型变量就是Test成员变量a的拷贝。）
>
> 参考：https://zhuanlan.zhihu.com/p/57859259

---

根据Oracle官方文档：https://docs.oracle.com/javase/tutorial/java/javaOO/localclasses.html

Nested Class：非静态（Inner Class), 静态（static nested class)

为什么要用Nested Class?

1. **It is a way of logically grouping classes that are only used in one place**:Nesting such "helper classes" makes their package more streamlined.
2. **It increases encapsulation**:外部类A, 内部类B; B可以直接访问A的private属性，而且B itself can be hidden from the outside world.
3. **It can lead to more readable and maintainable code**

一、**关于Inner Class（在外部类中 方法外定义的内部类，也叫成员内部类**）:在一个外部类中定义的Inner Class其实和外部类中定义的instance methods and variables一样，都得通过外部类的一个实例才能去访问这个Inner class。

二、**local Class（方法中定义的内部类，也叫局部内部类）**

三、**[Anonymous Classes](https://docs.oracle.com/javase/tutorial/java/javaOO/anonymousclasses.html)**

---

**内部类如何访问到外部类的（私有）属性？**

参考：

https://www.cnblogs.com/datamining-bio/p/13870270.html

https://www.jianshu.com/p/3132b0641883

1. 编译器会自动为内部类添加一个成员变量`final Outer this$0`, 这个成员变量的类型就是外部类的类型，而且这个成员变量就是指向外部类对象的引用。
2. 编译器会自动为内部类的构造方法添加一个参数，这个参数的类型就是外部类的类型，然后在构造方法中使用这个参数为1中添加的成员变量this$0赋值；
3. 在调用内部类的构造函数初始化内部类对象时，**会默认传入外部类的引用。**

内部类：多了一个外部类的final字段和一个带参构造器，外部类的引用有了；外部类的引用有了，但是还不能访问外部类的private属性啊，那怎么办？

外部类：其实编译器给外部类也多了一个static方法，比如：static int access$000(onehundred.OuterAndInnerClass);这里是拿int类型的属性；内部类就是通过调用这个static方法得到了外部类的private属性。

----

这篇关于内部类底层实现原理的文章也很好：https://www.cnblogs.com/dolphin0520/p/3811445.html

**final关键字：**关于局部内部类访问局部变量时，局部变量得加上final，这篇文章讲了final的一些实现原理：https://www.cnblogs.com/dolphin0520/p/3736238.html

关于final的一点细节：当final修饰一个引用变量（对象）的时候，虽然这个引用变量不能再指向另一片存储空间（也就是另一个对象），但是这个引用变量指向的空间，也就是这个对象中的属性是可以修改的。我在局部内部类的方法中使用外部类的局部变量（一个对象）时，给这个对象中的属性修改值，我发现可以修改，我还奇怪为什么加了final还能修改值。。。其实final作用是在这个引用变量上，它不能重新指向别的空间，而原来指向的那片空间里的属性是没加final的，当然可以修改了。

