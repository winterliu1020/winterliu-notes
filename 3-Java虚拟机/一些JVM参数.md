调优主要是对JVM一些参数的调整，比如调整堆内存的大小、以及新生代内存大小。。还有元空间/永久代的内存空间。。虽然这块放的是类的数据，一般很少进行GC，但是并不是说永久存在。。

为什么要调整新生代内存大小呢？因为老年代的Major GC成本远大于新生代的Minor GC，所以适当的调大新生代内存大小，避免这些新创建的对象由于对象太大而直接放到老年代。。

当然调优另一块指的是指定垃圾收集器，可以通过JVM参数指定你选择那种回收器。。

还可以进行**GC记录**，啥意思呢？我们必须时刻关注程序运行的状况，所以可以通过GC记录来检查JVM的垃圾回收性能。