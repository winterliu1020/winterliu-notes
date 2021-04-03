### Random类及其局限性

```java
Random random = new Random();
```

我们一般上面这行代码来获得随机数，这里声明一个random对象，如果构造函数中不传递参数则使用默认的种子，这个种子就是一个long类型的数字，有了种子之后。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/concurrent/20201009175236.png)

在单线程下都是先根据老种子生成新种子，然后用新种子传递到一个固定的函数中得到一个随机数。但是如果是多线程，多个线程可能都拿到同一个老的种子去执行步骤4以计算新的种子，导致多个线程得到的新种子是一样的。因为步骤5根据种子获取随机数的算法是固定的，这样就导致多个线程产生相同的随机值。

**所以我们要保证步骤四（产生新种子）的原子性**，也就是说当有多个线程根据同一个老种子计算新种子时，第一个线程的新种子被计算出来之后，第二个线程要丢弃自己老的种子，而使用第一个线程产生的新种子来计算自己的新种子。这样保证了多个线程产生的随机数是随机的。在这里，Random函数使用了一个**原子变量**实现这个效果。



### ThreadLocalRandom

这个就是为了弥补多线程高并发情况下Random的缺陷产生的。和之前说的ThreadLocal类似，这里也是让每个线程都维护一个种子变量，这样每个线程都是根据自己线程上的种子来计算随机数的。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/concurrent/20201016143827.png)

可以看到ThreadLocalRandom和之前的ThreadLocal一样，它只是一个工具类，真正的种子threadLocalRandomSeed变量是在Thread对象中。

ThreadLocalRandom的中种子放在线程里面，ThreadLocalRandom的实例里面只包含与线程无关的通用算法，所以它是线程安全的。































