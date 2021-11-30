Java中对象头：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211030155648.png)





**自旋锁、偏向锁、轻量级锁、重量级锁**

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211030154411.png)

详细看看轻量级锁加锁过程：还有解锁过程，其中涉及到锁对象中的mark word复制到每个尝试获取该锁的线程的栈帧，然后这些线程线程中就有了这个锁对象的mark word（复制到线程栈帧中叫 lock recode锁记录），然后呢？为什么要复制呢？因为这还是一个轻量级锁，线程并发数不多，采用的是线程CAS自旋的方式去获取锁，具体流程：这些程序都有了mark word，然后都用CAS尝试去修改锁对象的mark word，让锁对象的mark word指向自己线程的lock record锁记录，并将lock record的owner指向锁对象的mark word。

这一部分如果更新成功，说明此时是轻量级锁，会将锁对象中mark word中锁标志位改为“00”。当然CAS自旋了一定次数还是不成功，说明已经有其它线程已经占用了这个锁，也就是说有多个锁在同步竞争，这次轻量级锁要膨胀为重量级锁。锁标记位是“10”，同时锁对象的对象头存储的就是指向重量级锁的指针。这个重量级锁本质上就是C++层面的ObjectMonitor对象，这次如果锁标记位是10了，然后来一个线程，这个线程查看锁标记位是10，**然后再看看这个Java层的锁对象关联的C++层面的ObjectMonitor对象的owner指向是否为null，如果是null，说明没有线程占用这个锁，那就直接获取，如果不为null，那么这个线程会被放到ObjectMonitor对象中的entrylist属性，等待其他线程释放锁后去用CAS抢夺。**

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211030163847.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211030165422.png)

以上轻量级锁加锁过程、轻量级锁转成重量级锁 参考：https://www.cnblogs.com/paddix/p/5405678.html

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/image-20211103175039544.png)

轻量级锁时，一个线程在执行完同步代码块的时候，会用CAS将自己线程的栈帧中锁记录改回锁对象的Mark word，如果修改成功，说明还没有升级为重量级锁，改回之后，那么另外一个线程在不断重试CAS获取轻量级锁就可以成功拿到这个轻量级锁了，如果第二个线程在CAS循环结束之前还没有拿到轻量级锁，那这个锁对象的Mark word中的记录就会被修改为重量级锁 然后第二个线程被挂起，然后第一个线程执行完之后用CAS去将自己线程栈帧中的锁记录改回锁对象中Mark word时，会失败（因为锁对象的Mark word变成重量级锁了），这时第一个线程会释放锁，并且唤起被挂起的333445555555555w333ee3344线程。

推荐：关于各个锁之间加锁、解锁的详细过程：https://www.cnblogs.com/wuqinglong/p/9945618.html



关于ObjectMonitor：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211030161742.png)

Synchronized是可重入锁，它利用ObjectMonitor对象中的 _owner来标识拥有该monitor监视器锁的线程，然后用 _recursions 来标识拥有这个monitor监视器锁的线程重入次数。

参考：https://www.cnblogs.com/aobing/p/12906927.html



![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211030154453.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211030162350.png)



从synchronized 在C++层面源码分析它的锁升级过程：

https://www.cnblogs.com/cscw/p/13769404.html#3-object%E7%9A%84wait%E5%92%8Cnotify%E6%96%B9%E6%B3%95%E5%8E%9F%E7%90%86 (重点看)

https://www.cnblogs.com/hongdada/p/14513036.html



参考：

http://wangweiqing.github.io/blog/2016/05/09/liao-liao-bing-fa-suo/

https://segmentfault.com/a/1190000037645482