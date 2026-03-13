## 一、Java线程类Thread

### 1.1 Java线程和os线程关系

一个Java线程会对应一个操作系统线程。操作系统实现线程有三种方式 [具体看这里](https://blog.csdn.net/CringKong/article/details/79994511)：

- 内核线程实现
- 用户线程实现
- 用户线程加轻量级进程混合实现

> Java线程在JDK1.2之前，是基于称为“绿色线程”（Green Threads）的用户线程实现的，而在JDK1.2中，线程模型替换为基于操作系统原生线程模型来实现。
>
> JDK1.2之前，**程序员们为JVM开发了自己的一个线程调度内核，而到操作系统层面就是用户空间内的线程实现。**而到了JDK1.2及以后，JVM选择了更加稳健且方便使用的操作系统原生的线程模型，**通过系统调用，将程序的线程交给了操作系统内核进行调度**。
>
> 也就是说，**现在的Java中线程的本质，其实就是操作系统中的线程**，Linux下是基于`pthread`库实现的**轻量级进程**，Windows下是原生的系统`Win32 API`提供系统调用从而实现**多线程**。

### 1.2 Java线程类Thread中六种线程状态

调用某个方法后所对应的threadState值（注意这里按照jdk源码中Thread类的六种状态为准）

**几种状态之间的转换**

NEW, RUNNABLE, WAITING, BLOCKED, TIMED_WAITING, TERMINATED

除去NEW和TERMINATED，

Java线程状态RUNNABLE ：操作系统线程的ready和running状态

Java线程WAITING, BLOCKED, TIMED_WAITING ：操作系统线程的waiting

> 由于不同的操作系统在线程的设计上存在差异，所以JVM在设计上就已经声明：虚拟机中的线程状态，不反应任何操作系统线程状态。

#### JVM线程：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/concurrent/20210428095939.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/concurrent/20210428170733.png)

#### 操作系统线程：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/concurrent/20210428100136.png)

**其中阻塞和等待的区别**（阻塞就是**JVM**去调度恢复，等待则是让**另外一个线程去发出信号**恢复执行）

- 阻塞

  阻塞是尝试进入synchronized块，也就是尝试去获取对象锁，但是该锁被其它线程持有了，那么这个线程就会进入阻塞状态。**处于阻塞状态的线程由JVM调度器来决定是否唤醒，不需要由另外一个线程来显式唤醒，不会响应中断**。

- 等待

  **指的是当一个线程需要等待另一个线程发出信号从而进入可执行状态时，这个时候就是等待状态，等待状态可以响应中断**，例如调用：Object.wait(), Thread.join()及等待Lock或者Condition。

> 需要注意虽然synchronized和JUC里的Lock都实现锁的功能，但线程进入的状态不一样。synchronized会让线程进入阻塞态，而JUC里的Lock是用LockSupport.park()/unpark()来实现阻塞/唤醒的，会让线程进入等待态。
>
> 而LockSupport的park和unpark实际调用的是Unsafe类中native方法park, unpark()，与Object类的wait，notify机制相比，park和unpark的两个优点：
>
> - 它是以thread为操作对象，更加符合阻塞线程的直观定义，不像Object.wait()是通过一个锁资源来表示线程阻塞。
> - 操作更精确，park可以准确唤醒具体某个线程，notify是随机唤醒一个线程。
>
> **理解park：**关于“许可”
>
> - 其实park, unpark的设计原理核心是“许可”：park是让线程等待一个许可，而unpark是为线程提供一个许可。
>
>   如果LockSupport.park(), 那么除非另外一个线程中调用LockSupport.unpark(threadA)，否则A将阻塞在park操作上（注意这里的threadState是waiting）。
>
> - 先unpark再park也是可以的
>
>   先unpark说明先给了许可，再park就会消费这个许可，然后还可以继续执行。（但是许可不能叠加）
>
> #### Unsafe.park和Unsafe.unpark的底层实现原理
>
> 在Linux系统下，是用的Posix线程库pthread中的mutex（互斥量），condition（条件变量）来实现的。 **mutex和condition保护了一个_counter的变量，当park时，这个变量被设置为0，当unpark时，这个变量被设置为1。**

### 1.3 Java线程类Thread中常用方法

**1. sleep()** 这是一个native方法

> 1. Causes the currently executing thread to sleep
> 2. The thread does not lose ownership of any monitors.

Thread.sleep() 与 Thread.currentThread().sleep() 有什么区别

没区别，执行效果一样的。一个用类调用静态方法，一个用实例调用静态方法。

sleep函数会让当前函数让出CPU，但是，当前线程仍然持有它所获得的监视器锁，这与同时让出CPU资源和监视器锁资源的wait方法不一样。

> 对比一下obj.wait()方法：obj.wait()会调用wait(0)这个native方法，其实wait(long timeout, int nanos)和sleep(long millis, int nanos)在实现等待时间上很相似，但是底层分别调用native方法wait(timeout)和sleep(millis)是不一样的。

**2. yield()方法** 也是native方法

> A hint to the scheduler that the current thread is willing to yield its current use of a processor. The scheduler is free to ignore this hint.
>
> yield不会让线程退出RUNNABLE状态，最多从running变成ready，但是sleep(millis)会把线程状态转成TIMED_WAITING。

这只是给CPU一个建议说当前线程愿意让出CPU给其它线程，CPU采不采纳取决于不同厂商，可能刚yield让出CPU然后又立马抢夺到CPU。但是sleep一定会让出CPU，并且休眠指定时间不参与CPU竞争。

**3. join()方法**

方法介绍（sleep, join方法的理解）

> Waits at most {@code millis} milliseconds for this thread to die. A timeout of {@code 0} means to wait forever.

join底层用的是wait方法实现等待，而wait方法一定得先拿到监视器锁，可以发现join是synchronized方法，而且它是非静态的，所以这里的锁是子线程实例的监视器锁。

同时在主线程中会调用isAlive()方法检查子线程是否还存活（注意isAlive是子线程的方法）。如果还子线程还存活，则主线程会执行wait(0)让出监视器锁，进入WAITING状态；然后主线程需要等待子线程的notify才能从wait位置继续执行，注意：**当子线程m yThread停止执行时，会调用this.notifyAll，所以子线程结束，主线程会拿到子线程实例对象的监视器锁，继续往下执行**。

[学习来源](https://segmentfault.com/a/1190000016056471)