![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/concurrent/20201027111058.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/concurrent/20210406170900.png)

AQS是抽象同步队列，注意它并不是简单的队列，它是同步队列，所以这个抽象类里面实现了很多用于线程同步的方法，可以看到AQS这个类在java.util.concurrent.locks包下面，是Java并发包的基础工具类，比如ReentrantLock、CountDownLatch、Semaphore、FutureTask类的实现，里面都用到了AQS这个类。

我们来看看ReentrantLock类：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/concurrent/20210406172408.png)

ReentrantLock里面主要定义了一个Sync类，这是一个抽象、静态、内部类，这个类继承了AQS类，AQS主要还是一些同步队列的方法，而ReentrantLock涉及到锁，这个锁本质上还是依赖于AQS同步队列为基础，然后因为Sync也是抽象类，它还增加了一些抽象方法，下面看下Sync类：

```java
abstract static class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = -5179523762034025860L;

        /**
         * Performs {@link Lock#lock}. The main reason for subclassing
         * is to allow fast path for nonfair version.
         */
        abstract void lock();

        /**
         * Performs non-fair tryLock.  tryAcquire is implemented in
         * subclasses, but both need nonfair try for trylock method.
         */
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }

        protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }

        protected final boolean isHeldExclusively() {
            // While we must in general read state before owner,
            // we don't need to do so to check if current thread is owner
            return getExclusiveOwnerThread() == Thread.currentThread();
        }

        final ConditionObject newCondition() {
            return new ConditionObject();
        }

        // Methods relayed from outer class

        final Thread getOwner() {
            return getState() == 0 ? null : getExclusiveOwnerThread();
        }

        final int getHoldCount() {
            return isHeldExclusively() ? getState() : 0;
        }

        final boolean isLocked() {
            return getState() != 0;
        }

        /**
         * Reconstitutes the instance from a stream (that is, deserializes it).
         */
        private void readObject(java.io.ObjectInputStream s)
            throws java.io.IOException, ClassNotFoundException {
            s.defaultReadObject();
            setState(0); // reset to unlocked state
        }
    }
```

主要方法：nonfairTryAcquire()，可以看它说的，Sync这个类在trylock时用的是不公平锁，然后在ReentrantLock类中又有NotfairSync, FairSync两个类，这两个类对于release()方法不管你公平不公平，都是一样用的Sync继承来的方法，但是对于tryLock就不一样了。

> Performs non-fair tryLock.  tryAcquire is implemented insubclasses, but both need nonfair try for trylock method.

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/concurrent/20210406174051.png)

这里再对ReentrantLock总结一下：

1. 这是一个可重入锁，但是在trylock的时候可以根据参数选择公平还是非公平，默认是不公平锁，不管你ReentrantLock里面实现的是公平还是非公平，都是通过内部类Sync的对象sync来调用对应的方法。（这里用了多态）

   ```java
   /** Synchronizer providing all implementation mechanics */
   private final Sync sync;
   
   public ReentrantLock() {
       sync = new NonfairSync();
   }
   
   public ReentrantLock(boolean fair) {
   	sync = fair ? new FairSync() : new NonfairSync();
   }
   ```

2. 前面只讲了ReentrantLock里面的tryAcquire()方法，其实Sync这个类还有一个lock()抽象方法，然后NonFairSync和FairSync对这个lock()方法做了不同的实现，注意两个lock方法里面都用到了**acquire()**方法，这个方法是AQS类里面的final方法，下面会对这个方法做介绍。

   ![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/concurrent/20210406180000.png)

3. 这里介绍一下ReentrantLock类中lock(), tryAcquire()方法的一些实例使用：

---

OK，说回AQS，首先看看AQS有哪些属性：

```java
	/**
     * Head of the wait queue, lazily initialized.  Except for
     * initialization, it is modified only via method setHead.  Note:
     * If head exists, its waitStatus is guaranteed not to be
     * CANCELLED.
     */
    private transient volatile Node head;

    /**
     * Tail of the wait queue, lazily initialized.  Modified only via
     * method enq to add new wait node.
     */
    private transient volatile Node tail;

    /**
     * The synchronization state.
     */
    private volatile int state; // 这个最重要，代表当前锁的状态，0代表没有被占用，大于0代表有线程持有当前锁，每次重入都会加上1

	/**
     * The current owner of exclusive mode synchronization.
     */
    private transient Thread exclusiveOwnerThread; // 注意这个属性并不是AQS里面的，而是AQS父类AbstractOwnableSynchronizer里面的属性，AQS继承而来；代表当前持有独占锁的线程；reentrantLock.lock()可以嵌套调用多次，所以每次用这个来判断当前线程是否已经拥有了锁；if(currentThread == getExclusiveOwnerThread()) {state++}
```

AQS就是这四个属性，再看下AQS的等待队列：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/concurrent/20210406214230.png)

注意阻塞队列不包含head节点。等待队列中每个线程被包装成一个Node实例，这里用的是双向链表。

```java
static final class Node {
    // 标识节点当前在共享模式下
    static final Node SHARED = new Node();
    // 标识节点当前在独占模式下
    static final Node EXCLUSIVE = null;

    // ======== 下面的几个int常量是给waitStatus用的 ===========
    /** waitStatus value to indicate thread has cancelled */
    // 代表此线程取消了争抢这个锁
    static final int CANCELLED =  1;
    /** waitStatus value to indicate successor's thread needs unparking */
    // 官方的描述是，其表示当前node的后继节点对应的线程需要被唤醒
    static final int SIGNAL    = -1;
    /** waitStatus value to indicate thread is waiting on condition */
    // 本文不分析condition，所以略过吧，下一篇文章会介绍这个
    static final int CONDITION = -2;
    /**
     * waitStatus value to indicate the next acquireShared should
     * unconditionally propagate
     */
    // 同样的不分析，略过吧
    static final int PROPAGATE = -3;
    // =====================================================


    // 取值为上面的1、-1、-2、-3，或者0(以后会讲到)
    // 这么理解，暂时只需要知道如果这个值 大于0 代表此线程取消了等待，
    //    ps: 半天抢不到锁，不抢了，ReentrantLock是可以指定timeouot的。。。
    volatile int waitStatus;
    // 前驱节点的引用
    volatile Node prev;
    // 后继节点的引用
    volatile Node next;
    // 这个就是线程本尊
    volatile Thread thread;

}
```

Node类中就是thread + waitStatus + pre + next四个属性。

---

总结：上面介绍了AQS的一些数据结构，其实就是上面这个等待队列，再加上前面稍微介绍了一下ReentrantLock类如何继承自AQS类，并实现了Sync静态内部类，提供了公平锁和非公平锁。下面看下**ReentrantLock的使用方式**：

```java
public class OrderService {
    private static ReentrantLock reentrantLock = new ReentrantLock(true); // 开一个公平锁
    public void createOrder() {
        // 比如同一时间只能一个线程创建订单
        reentrantLock.lock();
        // 通常，lock()之后紧跟try语句
        try {
            // 这块代码只能由获取到锁的线程执行，其他线程在lock()方法上阻塞，等拿到锁才能进入try
            // 执行代码...
        } finally {
            // 释放锁
            reentrantLock.unlock();
        }
    }
}
```

### 线程抢锁

再贴一下这张图：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/concurrent/20210406180000.png)

不管是公平还是不公平加锁都用到了AQS类中的acquire()方法，这里看下这个方法：

```java
public final void acquire(int arg) { // 此时arg == 1
    // 首先调用tryAcquire(1)一下，这个只是试一试，如果直接就成功了，就不需要进入队列排队了
    // 对于公平锁的语义就是：本来就没有人持有锁，所以根本就没必要进入队列等待
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt(); // tryAcquire()没有成功，就需要把当前线程挂起，并且放入等待队列
}
// 注意这里如果tryAcquire(arg)返回true，也就结束了，否则的话acquireQueued()方法会将线程压到队列中
// 这里插入一个细节：这是在ReentrantLock类中调用的tryAcquire()方法，其实AQS类里面也有tryAcquire()方法，但是我们知道ReentrantLock类重写了tryAcquire()方法，所以这里调用的是ReentrantLock类中的tryAcquire()方法。

//--------------------------------------------------------------------------------
// 我们来看一下ReentrantLock类中的tryAcquire()方法：
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    // c == 0则此时没有线程持有锁
    if (c == 0) {
        // 那我们再看看队列中是不是有人等了好久了，因为这是公平锁，得按照队列顺序
        if (!hasQueuedPredecessors() && compareAndSetState(0, acquires)) {
            // 没有线程在等待，就用CAS尝试一下，成功了就获取到锁了，不成功的话只能说明就在刚刚几乎同一时刻被另外一个线程抢先了
            // 到这里就是获取到了锁，标记一下，告诉大家，现在是我占用了锁
            setExclusiveOwnerThread(current);
            return true;
        }
    } 
    // 这里不会存在并发问题
    else if (current == getExclusiveOwnerThread()) { // 进入这个分支，说明是重入了，则需要操作state += 1；
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    
    // 如果到这里，说明上面的if、else if都没有返回true，也就是没有拿到锁
    return false;
    // 那么再看外层调用，接着就会执行：acquireQueued(addWaiter(Node.EXCLUSIVE), arg)
    // if (!tryAcquire(arg) 
    //        && acquireQueued(addWaiter(Node.EXCLUSIVE), arg)) 
    //     selfInterrupt();
}



//---------------------------------------------------------------------------------
// 下面是AQS类中的tryAcquire()方法：
/**
     * Attempts to acquire in exclusive mode. This method should query
     * if the state of the object permits it to be acquired in the
     * exclusive mode, and if so to acquire it.
     *
     * <p>This method is always invoked by the thread performing
     * acquire.  If this method reports failure, the acquire method
     * may queue the thread, if it is not already queued, until it is
     * signalled by a release from some other thread. This can be used
     * to implement method {@link Lock#tryLock()}.
     *
     * <p>The default
     * implementation throws {@link UnsupportedOperationException}.
     *
     * @param arg the acquire argument. This value is always the one
     *        passed to an acquire method, or is the value saved on entry
     *        to a condition wait.  The value is otherwise uninterpreted
     *        and can represent anything you like.
     * @return {@code true} if successful. Upon success, this object has
     *         been acquired.
     * @throws IllegalMonitorStateException if acquiring would place this
     *         synchronizer in an illegal state. This exception must be
     *         thrown in a consistent fashion for synchronization to work
     *         correctly.
     * @throws UnsupportedOperationException if exclusive mode is not supported
     */
    protected boolean tryAcquire(int arg) {
        throw new UnsupportedOperationException();
    }
```

-----

### acquireQueued(addWaiter(Node.EXCLUSIVE), arg)方法

如果tryAcquire(arg)尝试拿锁没有成功，就会调用：acquireQueued(addWaiter(Node.EXCLUSIVE), arg)把当前线程加入到阻塞队列中，这里看看里面的addWaiter()方法：

```java
	/**
     * Creates and enqueues node for current thread and given mode.
     *
     * @param mode Node.EXCLUSIVE for exclusive, Node.SHARED for shared
     * @return the new node
     */
    // 此方法的作用是把线程包装成node，同时进入到队列中
    // 参数mode此时是Node.EXCLUSIVE，代表独占模式
    private Node addWaiter(Node mode) {
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
        // 以下几行代码想把当前node加到链表的最后面去，也就是进到阻塞队列的最后
        Node pred = tail;

        // tail!=null => 队列不为空(tail==head的时候，其实队列是空的，不过不管这个吧)
        if (pred != null) { 
            // 将当前的队尾节点，设置为自己的前驱 
            node.prev = pred; 
            // 用CAS把自己设置为队尾, 如果成功后，tail == node 了，这个节点成为阻塞队列新的尾巴
            if (compareAndSetTail(pred, node)) { 
                // 进到这里说明设置成功，当前node==tail, 将自己与之前的队尾相连，
                // 上面已经有 node.prev = pred，加上下面这句，也就实现了和之前的尾节点双向连接了
                pred.next = node;
                // 线程入队了，可以返回了
                return node;
            }
        }
        // 仔细看看上面的代码，如果会到这里，
        // 说明 pred==null(队列是空的) 或者 CAS失败(有线程在竞争入队)
        // 读者一定要跟上思路，如果没有跟上，建议先不要往下读了，往回仔细看，否则会浪费时间的
        enq(node);
        return node;
    }
```

上面这个方法中：如果**队列为空**(pred != null) 或者**队列不为空但是用CAS把自己设置为队尾没成功**（说明有其他线程也在用CAS竞争入队，那么就会执行下面的enq(node)方法，采用自旋的方式入队：

自旋在这里的语义是：CAS设置tail过程中，竞争一次竞争不到，就多次竞争，总会排到的

```java
	private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            // 之前说过，队列为空也会进来这里
            if (t == null) { // Must initialize
                // 初始化head节点
                // 细心的读者会知道原来 head 和 tail 初始化的时候都是 null 的
                // 还是一步CAS，你懂的，现在可能是很多线程同时进来呢
                if (compareAndSetHead(new Node()))
                    // 给后面用：这个时候head节点的waitStatus==0, 看new Node()构造方法就知道了

                    // 这个时候有了head，但是tail还是null，设置一下，
                    // 把tail指向head，放心，马上就有线程要来了，到时候tail就要被抢了
                    // 注意：这里只是设置了tail=head，这里可没return哦，没有return，没有return
                    // 所以，设置完了以后，继续for循环，下次就到下面的else分支了
                    tail = head;
            } else {
                // 下面几行，和上一个方法 addWaiter 是一样的，
                // 只是这个套在无限循环里，反正就是将当前线程排到队尾，有线程竞争的话排不上重复排！！！
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }

```

现在又回到这里：

```java
if (!tryAcquire(arg) 
	&& acquireQueued(addWaiter(Node.EXCLUSIVE), arg)) 
	selfInterrupt();
```

我们看完了addWaiter(Node.EXCLUSIVE)方法，如果执行成功它会返回最新的tail，那么回过来看acquireQueued()方法：

```java
	// 下面这个方法，参数node，经过addWaiter(Node.EXCLUSIVE)，此时已经进入阻塞队列
    // 注意一下：如果acquireQueued(addWaiter(Node.EXCLUSIVE), arg))返回true的话，
    // 意味着上面这段代码将进入selfInterrupt()，所以正常情况下，下面应该返回false
    // 这个方法非常重要，应该说真正的线程挂起，然后被唤醒后去获取锁，都在这个方法里了
    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                // p == head 说明当前节点虽然进到了阻塞队列，但是是阻塞队列的第一个，因为它的前驱是head
                // 注意，阻塞队列不包含head节点，head一般指的是占有锁的线程，head后面的才称为阻塞队列
                // 所以当前节点可以去试抢一下锁
                // 这里我们说一下，为什么可以去试试：
                // 首先，它是队头，这个是第一个条件，其次，当前的head有可能是刚刚初始化的node，
                // enq(node) 方法里面有提到，head是延时初始化的，而且new Node()的时候没有设置任何线程
                // 也就是说，当前的head不属于任何一个线程，所以作为队头，可以去试一试，
                // tryAcquire已经分析过了, 忘记了请往前看一下，就是简单用CAS试操作一下state
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                // 到这里，说明上面的if分支没有成功，要么当前node本来就不是队头，
                // 要么就是tryAcquire(arg)没有抢赢别人，继续往下看
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            // 什么时候 failed 会为 true???
            // tryAcquire() 方法抛异常的情况
            if (failed)
                cancelAcquire(node);
        }
    }
```

这里面又涉及到shouldParkAfterFailedAcquire()方法：

```java
	/**
     * Checks and updates status for a node that failed to acquire.
     * Returns true if thread should block. This is the main signal
     * control in all acquire loops.  Requires that pred == node.prev
     *
     * @param pred node's predecessor holding status
     * @param node the node
     * @return {@code true} if thread should block
     */
    // 刚刚说过，会到这里就是没有抢到锁呗，这个方法说的是："当前线程没有抢到锁，是否需要挂起当前线程？"
    // 第一个参数是前驱节点，第二个参数才是代表当前线程的节点
    private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        int ws = pred.waitStatus;
        // 前驱节点的 waitStatus == -1 ，说明前驱节点状态正常，当前线程需要挂起，直接可以返回true
        if (ws == Node.SIGNAL)
            /*
             * This node has already set status asking a release
             * to signal it, so it can safely park.
             */
            return true;

        // 前驱节点 waitStatus大于0 ，之前说过，大于0 说明前驱节点取消了排队。
        // 这里需要知道这点：进入阻塞队列排队的线程会被挂起，而唤醒的操作是由前驱节点完成的。
        // 所以下面这块代码说的是将当前节点的prev指向waitStatus<=0的节点，
        // 简单说，就是为了找个好爹，因为你还得依赖它来唤醒呢，如果前驱节点取消了排队，
        // 找前驱节点的前驱节点做爹，往前遍历总能找到一个好爹的
        if (ws > 0) {
            /*
             * Predecessor was cancelled. Skip over predecessors and
             * indicate retry.
             */
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
            /*
             * waitStatus must be 0 or PROPAGATE.  Indicate that we
             * need a signal, but don't park yet.  Caller will need to
             * retry to make sure it cannot acquire before parking.
             */
            // 仔细想想，如果进入到这个分支意味着什么
            // 前驱节点的waitStatus不等于-1和1，那也就是只可能是0，-2，-3
            // 在我们前面的源码中，都没有看到有设置waitStatus的，所以每个新的node入队时，waitStatu都是0
            // 正常情况下，前驱节点是之前的 tail，那么它的 waitStatus 应该是 0
            // 用CAS将前驱节点的waitStatus设置为Node.SIGNAL(也就是-1)
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        // 这个方法返回 false，那么会再走一次 for 循序，
        //     然后再次进来此方法，此时会从第一个分支返回 true
        return false;
    }

//-----------------------------------------------------------------------------------
// private static boolean shouldParkAfterFailedAcquire(Node pred, Node node)
    // 这个方法结束根据返回值我们简单分析下：
    // 如果返回true, 说明前驱节点的waitStatus==-1，是正常情况，那么当前线程需要被挂起，等待以后被唤醒
    //        我们也说过，以后是被前驱节点唤醒，就等着前驱节点拿到锁，然后释放锁的时候叫你好了
    // 如果返回false, 说明当前不需要被挂起，为什么呢？往后看

    // 跳回到前面是这个方法
    // if (shouldParkAfterFailedAcquire(p, node) &&
    //                parkAndCheckInterrupt())
    //                interrupted = true;

    // 1. 如果shouldParkAfterFailedAcquire(p, node)返回true，
    // 那么需要执行parkAndCheckInterrupt():

    // 这个方法很简单，因为前面返回true，所以需要挂起线程，这个方法就是负责挂起线程的
    // 这里用了LockSupport.park(this)来挂起线程，然后就停在这里了，等待被唤醒=======
    private final boolean parkAndCheckInterrupt() {
        LockSupport.park(this);
        return Thread.interrupted();
    }

    // 2. 接下来说说如果shouldParkAfterFailedAcquire(p, node)返回false的情况

   // 仔细看shouldParkAfterFailedAcquire(p, node)，我们可以发现，其实第一次进来的时候，一般都不会返回true的，原因很简单，前驱节点的waitStatus=-1是依赖于后继节点设置的。也就是说，我都还没给前驱设置-1呢，怎么可能是true呢，但是要看到，这个方法是套在循环里的，所以第二次进来的时候状态就是-1了。

    // 解释下为什么shouldParkAfterFailedAcquire(p, node)返回false的时候不直接挂起线程：
    // => 是为了应对在经过这个方法后，node已经是head的直接后继节点了。剩下的读者自己想想吧。
}
```

**说到这里，也就明白了，多看几遍 `final boolean acquireQueued(final Node node, int arg)` 这个方法吧。自己推演下各个分支怎么走，哪种情况下会发生什么，走到哪里。**

## 解锁操作

线程唤醒，正常情况下，如果线程没获取到锁，线程会被LockSupport.park(this)挂起停止，等待被唤醒。

```java
public void unlock() {
    sync.release(1);
}

public final boolean release(int arg) {
    // 往后看吧
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}

// 回到ReentrantLock看tryRelease方法
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    // 是否完全释放锁
    boolean free = false;
    // 其实就是重入的问题，如果c==0，也就是说没有嵌套锁了，可以释放了，否则还不能释放掉
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}

/**
 * Wakes up node's successor, if one exists.
 *
 * @param node the node
 */
// 唤醒后继节点
// 从上面调用处知道，参数node是head头结点
private void unparkSuccessor(Node node) {
    /*
     * If status is negative (i.e., possibly needing signal) try
     * to clear in anticipation of signalling.  It is OK if this
     * fails or if status is changed by waiting thread.
     */
    int ws = node.waitStatus;
    // 如果head节点当前waitStatus<0, 将其修改为0
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);
    /*
     * Thread to unpark is held in successor, which is normally
     * just the next node.  But if cancelled or apparently null,
     * traverse backwards from tail to find the actual
     * non-cancelled successor.
     */
    // 下面的代码就是唤醒后继节点，但是有可能后继节点取消了等待（waitStatus==1）
    // 从队尾往前找，找到waitStatus<=0的所有节点中排在最前面的
    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
        // 从后往前找，仔细看代码，不必担心中间有节点取消(waitStatus==1)的情况
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        // 唤醒线程
        LockSupport.unpark(s.thread);
}
```

唤醒waitStatus<=0的所有节点中排在最前面的节点对应的线程，被唤醒的线程从以下的代码中继续往前走：

```java
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this); // 刚刚线程被挂起在这里了
    return Thread.interrupted();
}
// 又回到这个方法了：acquireQueued(final Node node, int arg)，这个时候，node的前驱是head了
```

## 总结

并发环境下，加锁和解锁需要三个部件的协调：

1. 锁状态state。它为0时代表没有线程占有锁，可以去争抢这个锁，用CAS将state设为1，如果CAS成功，说明抢到锁了，那么其他线程就抢不到了，锁重入的话，state需要+1，解锁就是减1，直到state又变为0，代表释放锁，所以lock()和unlock()方法必须配对。然后唤醒等待队列中的第一个线程，让其占有锁。
2. 线程的阻塞和接触阻塞。AQS中采用了LockSupport.park(thread)来挂起线程，用unpark来唤起线程。
3. 阻塞队列。没有抢到锁的线程要等待，放到一个queue中，AQS用的是一个FIFO队列。AQS采用了CLH锁的变体来实现。

## 示例图解析

首先，第一个线程调用 reentrantLock.lock()，翻到最前面可以发现，tryAcquire(1) 直接就返回 true 了，结束。只是设置了 state=1，连 head 都没有初始化，更谈不上什么阻塞队列了。要是线程 1 调用 unlock() 了，才有线程 2 来，那世界就太太太平了，完全没有交集嘛，那我还要 AQS 干嘛。

如果线程 1 没有调用 unlock() 之前，线程 2 调用了 lock(), 想想会发生什么？

线程 2 会初始化 head【new Node()】，同时线程 2 也会插入到阻塞队列并挂起 (注意看这里是一个 for 循环，而且设置 head 和 tail 的部分是不 return 的，只有**入队成功**才会跳出循环)

```java
private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        if (t == null) { // Must initialize
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

首先，是线程 2 初始化 head 节点，此时 head==tail, waitStatus==0

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/concurrent/20210407145228.png)

然后线程 2 入队：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/concurrent/20210407145134.png)

同时我们也要看此时节点的 waitStatus，我们知道 head 节点是线程 2 初始化的，此时的 waitStatus 没有设置， java 默认会设置为 0，但是到 shouldParkAfterFailedAcquire 这个方法的时候，线程 2 会把前驱节点，也就是 head 的waitStatus设置为 -1。

那线程 2 节点此时的 waitStatus 是多少呢，由于没有设置，所以是 0；

如果线程 3 此时再进来，直接插到线程 2 的后面就可以了，此时线程 3 的 waitStatus 是 0，到 shouldParkAfterFailedAcquire 方法的时候把前驱节点线程 2 的 waitStatus 设置为 -1。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/concurrent/20210407145256.png)

这里可以简单说下 waitStatus 中 SIGNAL(-1) 状态的意思，**Doug Lea 注释的是：代表后继节点需要被唤醒。也就是说这个 waitStatus 其实代表的不是自己的状态，而是后继节点的状态，我们知道，每个 node 在入队的时候，都会把前驱节点的状态改为 SIGNAL，然后阻塞，等待被前驱唤醒。**这里涉及的是两个问题：有线程取消了排队、唤醒操作。其实本质是一样的，可以顺着 “waitStatus代表后继节点的状态” 这种思路去看一遍源码。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/concurrent/20210407145510.png)

