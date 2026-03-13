### wait(), notify(), notifyAll() 和 synchronized关键字 和 锁 的关系

wait(), notify(), notifyAll()都是Object类的方法，而不是Thread类，因为wait(), notify(), notifyAll()这三个本质上等待的是对象monitor，由于Java中每个对象都有一个内置的monitor，自然所有类理应有wait(), notify()方法。

注意这三个方法都要在同步块里面调用，不然会报illegal Monitor State Exception，因为调用这三个方法必须要拿到同步对象的monitor。

再说一下：synchronized是一个关键字，synchronized内置了锁，这个锁是一种对象锁（锁的是对象而非引用变量），粒度是对象，实现对临界资源的同步互斥访问，可重入（可重入避免了死锁）。

synchronized可以把任何一个非null对象作为锁，在HotSpot JVM实现中，锁有个专门的名字：对象监视器（Object monitor）。

synchronized可以加在实例方法（锁住了实例方法对应的实例对象）、静态方法（锁住了静态方法对应的类的class对象）、或者synchronized(object)。可以看到synchronized锁住的都是对象。

我们如果要实现同步，需要依赖锁来实现，那么锁的同步又依赖谁呢？**synchronized给出的答案是在软件层面依赖JVM，而j.u.c.Lock给出的答案是在硬件层面依赖特殊的CPU指令。**

synchronized同步代码块：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/concurrent/20210326153503.png)

synchronized同步方法：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/concurrent/20210326153850.png)

相比于同步代码块，同步方法多了一个ACC_SYNCHRONIZED标识符，JVM根据该标识符实现方法同步。