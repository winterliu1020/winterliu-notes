之前一直没在代码中遇到过弱引用这一块，今天在看ThreadLocal类中的ThreadLocalMap的key时，发现这个key（也就是threadLocal）是一个弱引用，弱引用是不管内存空间足够与否，都会回收这部分空间。也就是为了防止内存泄漏，当一片空间仅仅由一个弱引用指向时，gc就会把这片空间回收。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/image-20211028160800975.png)

再来看看弱引用WeakReference类是怎么实现的呢？它不也就是一个类嘛，一个泛型类，这个类继承自Reference类，Reference类中的referent属性会被GC区别对待。。。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211028161314.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211028162011.png)

这样看来不管是WeakReference还是其它什么引用，这些类都是一个容器，真正的还要看这个容器中的referent属性，这个属性才会被GC区别对待。

再说一句，引用是引用，类还是类，并没有说这个类是弱引用，类并不会被Reference类或者它的子类修饰，而应该是一个类中我的一个属性，比如说一个类中有一个Animal animal属性，我为了防止这个animal指向的Animal实例空间内存泄漏，把这个属性引用设置为弱引用，这样如果在这个类的其它地方没有一个强引用指向这一片animal实例内存空间，那么gc的时候就会回收这片空间。