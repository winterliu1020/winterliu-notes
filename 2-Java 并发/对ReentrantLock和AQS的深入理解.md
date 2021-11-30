对AQS的总体认知：

一、AQS中一大块方法是private的，也就是说这些方法不能被继承 不能在外部类中使用

（1）这一块方法（被Doug Lea称为Queuing utilities）是用于维护同步队列，包括1.将线程封装成Node加到队列（注意这里是CLH队列 一个虚拟队列，只是通过head和tail两个指针来维护，然后用cas将新节点插入到队尾），2.取出队列的第一个node，也就是head指针指向的node，然后照样用cas去更新head指针指向下一个节点。

（2）还有一些private方法是各种各样的acquire方法，包括共享、独占

二、第二块是public方法

这些方法能在子类中直接调用，比如acquire，release，acquireShared，releaseShared等几个方法，这几个方法其实还是调用了上面的private方法

三、第三块就是给子类进行自定义实现的方法（也就是模版方法），AQS中只有这五个

> isHeldExclusively()//该线程是否正在独占资源。只有用到condition才需要去实现它。
> tryAcquire(int)//独占方式。尝试获取资源，成功则返回true，失败则返回false。
> tryRelease(int)//独占方式。尝试释放资源，成功则返回true，失败则返回false。
> tryAcquireShared(int)//共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
> tryReleaseShared(int)//共享方式。尝试释放资源，成功则返回true，失败则返回false。



![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211109113148.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211109150213.png)



### ReentrantLock中的sync属性

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211109162221.png)

在ReentrantLock中一大块的代码都是实现Sync这个抽象静态内部类，剩下的都是实现继承自Lock接口中的方法，而实现Lock接口中的方法其实就是调用sync这个同步器的方法。（然后ReentrantLock剩下一部分是对Condition及其对应的条件队列的方法，condition也是AQS框架中的）

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211109170412.png)



### 对AQS框架的理解

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211109164547.png)



![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211109163106.png)

可以看到公平锁和不公平锁最终都调用了AQS类中的acquire()方法：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211109165714.png)



毫无疑问，接下来详细看看AQS类中acquireQueued(addWaiter(Node.EXCLUSIVE))，怎么让一个线程阻塞并放到AQS中的同步队列。

先看一下在里面调用的addWaiter(Node.EXCLUSIVE)，这个方法**一定会成功**将当前线程封装成一个node节点，然后将这个node节点放到AQS队列的队尾；用死循环的cas来保证这个节点一定会入队。。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211110103009.png)

那这个节点入队成功后，addWaiter(Node.EXCLUSIVE)会返回这个node，此时这个node就是队尾，也就是tail会指向这个node，然后我们再看**acquireQueued(node)**:

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211110164727.png)

再看shouldParkAfterFailedAcquire(p, node)：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211110170226.png)

### 唤醒和出队（是理解一个线程被封装成node，然后入队之后在干嘛、以及这个node对应线程何时被唤醒，如何出队的）

因为不公平策略是可以在锁空闲的时候立即抢占的，根本不用入队，那原本即将被分配到锁的队列中的第一个线程 在被其它不在队列中的线程抢占锁之后，这个第一个线程又是怎么样呢？（看下面文字可以知道，其实它还是做一次for循环自旋，只有当这个node是第一个节点、并且tryAcquire拿到了锁，自旋转才会结束）

看完这个shouldParkAfterFailedAcquire(pred, node)和parkAndCheckInterrupt()两个函数后有没有想过一个问题，就是线程到这里被park函数阻塞了，也就是不继续往下执行，那这个被阻塞的线程什么时候被唤醒呢？你要知道这个线程已经入队了啊，那它唤醒和出队又是以什么样的方式发生的呢？

其实就是看acquireQueue()函数什么时候返回，下面这张图可以看到只有当node的前驱节点是head的时候并且tryAcquire()抢占到了锁才能返回，就是嘛 这才符合正常逻辑啊，一个线程如果是在不公平策略 可能在锁空闲时立马抢到了，然后在队列中的第一个本来应该拿到锁的线程没抢过这个线程，然后它虽然是队列的第一个线程，但是它没抢过不在队列的一个线程，因此它还是往下走执行shouldParkAfterFailedAcquire(p, node)判断，因为作为队列中第一个线程在第一次被唤醒之后它已经将它的前置节点的waitState设置为-1了，所以shouldPark...这个函数立刻返回true，然后队列中这个第一个线程依旧阻塞，**让抢成功的那个线程执行完毕释放锁并执行unparkSuccessor(h)去唤醒这个队列中第一个线程，然后队列中第一个线程被唤醒了，它继续再次执行for循环，依旧判断p==head,然后尝试获取锁，如果这次成功获取到锁**（注意acquireQueue(node)函数只有这个node的前置节点是head并且这个node对应线程成功抢到锁，node才会从队列中移除，**这个函数才会返回**），不然的话这个node对应的线程就**一直在这个for循环中自旋。。**。。（所以对于大部分不是队列首部的node，这些node其实都被park阻塞了，然后**他们想要继续执行，必须依靠它的前一个节点**，而且前一个节点node的waitState属性还必须是Signal 也就是-1；当然正常情况，一个node前置节点waitState属性不是-1，因为每个节点初始化的时候 waitState属性都为0，这时候第一次进入shouldParkAfterFailedAcquire(p, node)它会将node的前置节点的用cas置为-1，然后返回false，进入到第二次for循环，这时候shouldParkAfterFailedAcquire才返回true，然后才会让这个node对应线程阻塞住；

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211117150243.png)

对shouldParkAfterFailedAcquire(p, node)函数的总结：这个函数保证了node一定在一个waitState值为-1的点的后面，如果第一次执行这个函数前置节点p的waitState值不为-1，如果是大于0的我就一直往前找，找到小于等于0的前置节点，然后第二次执行这个函数，再将这个前置节点的waitState值置为-1，然后这个函数才会返回true，返回true说明同意阻塞当前线程了；从这里可以看到一个线程被阻塞那一定说明这个线程的node的前置节点的waitState值一定是-1；因为我就靠前置节点为-1的点来将我唤醒呢。。。。

### release 释放锁

release释放一个锁又会发生什么呢？记住一句话：**为0的节点是需要被其它非0节点唤醒的；**waitState等于0的节点不能唤醒任何节点

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211117160914.png)

当h不为null，并且h的waitStatus不为0时，会去唤醒下一个节点，我们来看看unparkSuccessor(h)函数如何唤醒？

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211117161813.png)





提几个问题：

1.为什么节点状态会置为cancelled呢，我初始化一个节点时状态不是为0吗，然后虽然在shouldParkAfter...()函数中当发现前驱节点为0时，会把前驱节点状态置为-1，这也没有把节点状态改为cancelled(值是1)啊，那一个节点什么时候状态被改为1了呢，或者说一个node什么时候被标记为取消了呢？

2.我们知道要唤醒一个node，是通过这个node的前驱node来唤醒的，所以当有一个node被需要park的时候，这个node一定需要找一个状态不是取消的前驱节点。



关于LockSupport.park():

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211110180316.png)







