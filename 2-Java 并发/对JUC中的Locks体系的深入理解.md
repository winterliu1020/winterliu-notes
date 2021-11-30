![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211108160619.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211108162652.png)

那Lock接口中有哪些方法呢？因为所有Locks框架下的锁的实现类都会继承这个lock接口。

方法：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211108170748.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211108171227.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211108171531.png)

可以看到tryLock()方法一般是作为if语句的条件，然后把同步代码块放到if语句，这样起到对共享资源的同步的作用。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211108172455.png)









