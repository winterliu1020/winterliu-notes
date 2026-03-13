### 1. LockSupport工具类

JDK中的rt.jar包里面的LockSupport是一个工具类，主要作用是**挂起和唤醒线程**，该工具类是创建锁和其它同步类的基础。（其实锁的本质就是挂起和唤醒线程）

LockSupport类和每个使用它的线程都会关联一个许可证，但是默认情况下调用LockSupport类的方法的线程是不持有许可证的。所以说呢：如果你某个线程拿到了许可证，那调用比如LockSupport.park()时会马上返回，但是如果没拿到许可证，那么调用线程会被禁止参与线程的调度，即阻塞挂起。

**LockSupport工具类中的park(), unpark(thread) 方法理解**：

假设有thread1,thread2，在thread1中调用LockSupport.park()那么线程1会被挂起，然后需要在thread2中调用LockSupport.park(thread1)，这样线程1会被唤醒（因为调用unpark之后会让thread1持有与LockSupport的许可证）。如果thread1之前没调用park,则在thread2中调用LockSupport.unpark(thread1)之后，thread1再去调用park会立刻返回（因为thread1已经拿到了许可证了啊）。

需要注意：因为调用park方法而被阻塞的线程被其它线程中断而返回时并不会抛出InterruptedException异常。

调用LockSupport.park(this); // 这样当打印线程堆栈排查问题时就能知道是哪个类被阻塞了。

Thread类中有个变量volatile Object parkBlocker,用来存放park方法传递的对象，也就是说当你在thread1中调用LockSupport.park(blocker)时，blocker变量会放到thread1这个线程的成员变量里。

LockSupport.park()方法其实是调用的UNSAFE类中的UNSAFE.park()方法实现的，而UNSAFE.park()是一个native方法，也就是说它会去调用C语言的代码。

### 2. 抽象同步队列（AQS -- 锁的底层支持）概述

AQS（AbstractQueuedSynchronizer)抽象同步队列，是实现同步器的基础组件，并发包中锁的底层就是使用AQS实现的。

这是一个双向队列，其中Node放的是thread变量，Node节点中有：SHARED, EXCLUSIVE, waitStatus, prev, next.

对于AQS来说，线程同步的关键是对状态值state进行操作，根据state是否属于同一个线程，操作state的方式分为独占式和共享方式。**独占方式**获取的**资源是与具体线程绑定的**，某个线程获取到了资源，就会标记是这个线程获取到了，那么其它线程再次尝试操作state获取资源时会发现当前该资源不是自己持有，就会在获取失败后被阻塞。**共享方式**的资源与具体线程是不相关的，多个线程通过CAS方式去竞争获取资源，一个线程获取资源后，另外的线程再次去获取时，只要当前资源还能满足需要，另外的线程只要使用CAS方式进行获取即可，不满足的话则把这个线程放到阻塞队列。

**条件变量**

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/concurrent/20201027111058.png)

### 3. 独占锁ReentrantLock的原理

ReentrantLock是可重入独占锁。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/concurrent/20201027113630.png)

### 4. 读写锁ReentrantReadWriteLock原理

ReentrantLock是独占锁，某时只能有一个线程可以获取该锁，而实际中会有写少读多的场景，所以提出ReentrantReadWriteLock，它采用读写分离，允许多线程同时获取该锁。





















