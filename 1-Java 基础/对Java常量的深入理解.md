你知道Java中常量是啥意思吗？字符串常量，也就是这个字面量，比如"aaa"，这个字符串常量，final String str = "aaa";这个str也被大多人认为是常量，而这里的常量是说不可变的变量；

所以常量我觉得分为：1.类似于字符串常量，这个字面量本身（“aaa"）；2.不可变的变量(str)

这里又涉及到final关键字，同时又容易与static关键字搞混。。

首先看一下题：

<img src="https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211208153832.png" style="zoom:50%;" />

需要注意，这里是说变量a,b,c存在哪里，不是"aa"“bb""cc"。。

答案就是：a在堆，因为a是类属性，只有当创建对象的时候，才会给当前对象的a属性分配空间，也就是在堆；

b在栈，b已经在方法中了，当执行methodB的时候，以栈帧的方式压入栈中，当然这些b在栈中。

c也在栈，不要因为加了final，你就以为它是常量了，然后这个c应该在常量池？？也就是在元空间？？想多了，c仍然在栈中，**final只能限制c当初始化之后不能再指向另外一片空间，而不能改变被修饰变量的存储区域。**

还有个兄弟说的也不错：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211208154612.png)

我也看了jvm书上说方法区放的是类的信息、静态变量、常量、当然还有.class文件代码，这里说的常量应该是说类属性中加了static final的。。。并不是说我们在方法中加了final就是常量了。。。



也就是说被final修饰的常量，在编译的时候就会直接拿它的值替代。比如下面的n2被final修饰，那在编译的时候，下面计算s2= n2 + "0522"的时候，直接把n2用它的值2019替代了，**所以s2在编译的时候就转成s2 = "20190522"了**，然后运行的时候s==s2自然为true，因为这两个都指向常量池中“20190522”。而n1得到运行这个方法的时候在栈中才能给n1这个变量赋值为2019，编译的时候n1对应的数据并没有在常量池，所以编译的时候s1自然没有直接转成“20190522”，而是运行的时候，会在堆上分配一个对象，这个对象的值是“20190522”，所以**s1指向的是堆上的地址**。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211208155136.png)

参考：https://www.justdojava.com/2019/06/16/java-final/

上面这兄弟还写了关于内部类用外部局部变量时为什么一定要加final关键字:

如果外部方法已经运行完了，而内部线程还在运行，因为方法结束，方法中变量都弹出栈了。。所以这两个a肯定不是同一个。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211208160522.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211208160451.png)