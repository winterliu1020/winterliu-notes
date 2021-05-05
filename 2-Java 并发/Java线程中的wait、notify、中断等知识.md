### 参考来源：[Threads and locks](https://javadoop.com/post/Threads-And-Locks-md)

每一个Java对象都关联了一个监视器，也关联了一个等待集合，等待集合中的节点都是一个个线程。

向等待集合中添加或者移除线程的操作都是原子的，有哪些操作可以对这个等待集合进行修改呢？如下：

Object.wait, Object.notify, Object.notifyAll

等待集合可能受到线程的中断状态的影响，也会受到线程中处理中断的方法的影响。

下面讲解Java线程相关知识：

1. Thread中的sleep, join, interrupt
2. 继承自Object的wait, notify, notifyAll
3. Java中断

---

### 一、wait等待, notify通知, interruptions中断

#### 1. 等待wait

方法：wait(), wait(long millisecs), wait(long millisecs, int nanosecs)。如果调用wait方法时没有抛出InterruptedException异常，则表示正常返回。

在线程t中对对象m调用m.wait()方法，n代表加锁编号，同时还没有相匹配的解锁操作，则会发生下面其中之一：

- 如果n等于0（线程t没有持有m对象的锁），会抛出IllegalMonitorStateException异常。wait或者notify操作必须先要获取到监视器锁才能执行，所以这两个方法必须在同步块中调用，否则会抛出IllegalMonitorStateException异常。

- 如果调用wait(long millisecs)或者 wait(long millisecs, int nanosecs)，millisecs不能为负数，millisecs要在[0, 999999]，否则抛出IllegalArgumentException异常。

- 当线程t调用wait方法后，如果t被中断，此时中断状态为true，则wait方法将抛出InterruptException异常，并将中断状态设置为false；

- 否则会顺序发生如下事件（注意：到这里说明首先拿到了对象的监视器锁，wait参数正常，线程t没有被中断）：

  1. 线程t会加入对象m的等待集合中，执行加锁编号n对应的解锁操作

  2. 线程t不会执行任何进一步的指令，直到t从m的等待集合中移出（也就是等待唤醒）。以下情况t会从m的等待集合中移出（**移出队列并不是马上执行，这个线程还需要重新获取锁才行**），然后在之后的某个时间点恢复，继续执行后面的指令。

     - 执行m.notify()，并且在所有等待锁的线程中t被选中从等待集合中移除。
     - 执行m.notifyAll()
     - 线程t发生了interrupt操作
     - 如果t是调用wait(long millisecs)或者 wait(long millisecs, int nanosecs)，millisecs进入等待集合，那么过了millisecs 毫秒或者 (millisecs*1000000+nanosecs) 纳秒后，线程 t 也会从等待集合中移出。
     - JVM假唤醒

  3. 线程t执行编号为n的加锁操作

     2中说线程t刚刚从等待集合中移出，那么t需要再次拿到监视器锁才能继续往下执行。

  4. 如果t在2的时候由于中断从m的等待集合中移出，那么t的**中断状态重置**为false，并且wait抛出InterruptException异常。

  > 总结：当线程a由于等待而正在阻塞的时候，如果另外一个线程执行a.interrupt()，那么a线程的中断状态会重置为false，并且抛出InterruptException异常。

#### 2. 通知Notification

方法：notify, notifyAll

在线程t中对对象m调用m.notify()或者m.notifyAll(), n代表加锁编号，同时对应的解锁操作没有执行，则会发生下面其中之一：

- 如果n等于0，也会抛出IllegalMonitorStateException异常，因为调用notify也要先拿到锁。

- n大于0的话，对于notify，如果m对象的等待集合不为空，那么等待集合中的线程u被选中从等待集合中移出。（虚拟机并不保证哪一个线程会被移出，被移出的线程u可以恢复，但是注意这时候锁还在线程t手上，所以u线程此时对m进行加锁不会成功，直到线程t完全释放锁。

  > 被notify唤醒的线程需要重新获取监视器锁。

- n大于0，对于notifyAll，等待集合中所有线程都将从等待集合中移出，然后恢复。但是这些线程恢复后只有一个线程可以获得监视器锁。

#### 3. 中断Interruptions

中断发生于Thread.interrupt方法的调用。

令线程t调用线程u上的方法u.interrupt()，其中t和u可以是同一个线程（自己可以中断自己），这个操作会将u的中断状态设置为true。

> thread.interrupt()方法并不是暂停线程，这个方法只是设置线程的中断状态。特殊之处在于设置了中断状态为true之后，下面这几个方法会感知到：
>
> - wait(), wait(long), wait(long, int), join(), join(long, int), sleep(long), sleep(long, int)
>
>   这些方法的方法签名都有throws InterruptedException，用来响应中断状态修改。
>
> - 如果线程阻塞在InterruptibleChannel类的IO操作中，那么这个channel会被关闭。
>
> - 如果线程阻塞在一个Selector中，那么select方法会立即返回。
>
> 如果线程阻塞在以上三种情况，那么当线程感知到中断状态后（此线程的interrupt()方法被调用），会将中断状态重新设置为false，然后执行相应的操作（通常是跳到catch异常处）
>
> 如果不是上面三种情况，那么，线程的interrupt()方法被调用，会将线程的中断状态设置为true。
>
> 当然，除了这几个方法，我知道的是 LockSupport 中的 park 方法也能自动感知到线程被中断，当然，它不会重置中断状态为 false。我们说了，只有上面的几种情况会在感知到中断后先重置中断状态为 false，然后再继续执行。

> **下面的不太理解：**
>
> 另外，如果有一个对象 m，而且线程 u 此时在 m 的等待集合中，那么 u 将会从 m 的等待集合中移出。这会让 u 从 wait 操作中恢复过来，u 此时需要获取 m 的监视器锁，获取完锁以后，发现线程 u 处于中断状态，此时会抛出 InterruptedException 异常。
>
> > 这里的流程：t 设置 u 的中断状态 => u 线程恢复 => u 获取 m 的监视器锁 => 获取锁以后，抛出 InterruptedException 异常。
> >
> > 这个流程在前面 **wait** 的小节已经讲过了，这也是很多人都不了解的知识点。如果还不懂，可以看下一小节的结束，我的两个简单的例子。
> >
> > 一个小细节：u 被中断，wait 方法返回，并不会立即抛出 InterruptedException 异常，而是在重新获取监视器锁之后才会抛出异常。

实例方法thread.isInterrupted()可以知道线程的中断状态。

调用静态方法Thread.interrupted()可以返回当前线程的中断状态，同时将中断状态设置为false。

#### 4. 等待、通知、中断三者的交互

如果一个线程在等待期间，同时发生了**通知和中断**，它将发生下面情况的一种：

- 从wait方法中正常返回，同时不改变中断状态。（也就是说调用Thread.interrupted方法将会返回true）

  这种情况就是说线程虽然因为调用thread.interrupt()让线程的中断状态设置为true，但是这个线程是从notify被唤醒，而不是在wait方法中捕捉到InterruptedException被返回（这样返回就是不正常返回，会把状态设置为false），被notify唤醒会从wait方法中正常返回。

- 由于抛出InterruptedException异常而从wait方法中返回，中断状态会被设置为false。

> 总结：
>
> **1.wait和notify的关系：**wait 方法返回后，需要重新获取监视器锁，才可以继续往下执行。
>
> **2.wait和interrupt的关系：**如果线程调用 wait 方法，当此线程被中断的时候，wait 方法会返回，然后重新获取监视器锁，**然后抛出 InterruptedException 异常**。
>
> thread1.wait()方法会释放锁，然后 thread2拿到锁开始执行 -> thread2中调用thread1.interrupt()，那么thread1中的wait方法就会返回，但是thread1返回并不会立即往下执行，而要抢到监视器锁，抢到监视器锁之后在这里也不会向下执行了，因为它得响应中断，所以这里thread1拿到锁之后会立即抛出InterruptedException 异常。
>
> **3.interrupt和notify的关系：**当线程1在等待的时候，如果另外一个线程执行了thread1.interrupt()操作和object.notify()操作，那么它可能会发生两种情况里面的一种。
>
> 看原文例子：[第三个例子](https://javadoop.com/post/Threads-And-Locks-md)
>
> > 有可能发生 线程1 是正常恢复的，虽然发生了中断，它的中断状态也确实是 true，但是它没有抛出 InterruptedException，而是正常返回。**此时，thread2 将得不到唤醒，一直 wait。**

---

### 二、休眠和礼让（Sleep and Yield）

Thread.sleep()让当前执行线程休眠，休眠期间不会释放任何的监视器锁。

注意：**Thread.sleep和Thread.yield都不具有同步的语义**，在Thread.sleep和Thread.yield方法调用之前，不要去虚拟机将寄存器中的缓存刷出到共享内存中，同时也不要求在这两个方法调用之后，重新从共享内存中读取数据到缓存。

> *例如，我们有如下代码块，this.done 定义为一个 non-volatile 的属性，初始值为 false。*
>
> ```java
> while (!this.done)
>     Thread.sleep(1000);
> ```
>
> *编译器可以只读取一次 this.done 到缓存中，然后一直使用缓存中的值，也就是说，这个循环可能永远不会结束，即使是有其他线程将 this.done 的值修改为 true。*

> yield 是告诉操作系统的调度器：我的cpu可以先让给其他线程。**注意，调度器可以不理会这个信息。**
>
> 这个方法太鸡肋，几乎没用。

---

### 三、内存模型（JMM)

什么是Java内存模型？

内存模型主要是为了规范多线程程序中修改（写）或者访问（读）同一个值的时候的行为，定义了对共享内存的写操作对于读操作的可见性。

具体看[这里17.4](https://javadoop.com/post/Threads-And-Locks-md)

### 四、final属性的语义

关于final：用 final 修饰的类不可以被继承，用 final 修饰的方法不可以被覆写，用 final 修饰的属性一旦初始化以后不可以被修改。

看一看final声明的属性：

final声明的属性正常情况下初始化之后就不会被改变，final属性的语义和普通属性的语义有些不同，比如对于final属性的读操作，由于final属性值根本就不会变，所以compilers可以自由去除不必要的同步，而且compilers还可以将final属性值直接缓存在寄存器中，而不用像普通属性一样从内存中重新读取。

如果对某个属性加了final，那么我就不需要使用同步就可以实现线程安全的不可变对象。

> 对象只有在构造方法结束了才被认为`完全初始化`了。如果一个对象**完全初始化**以后，一个线程持有该对象的引用，那么这个线程一定可以看到正确初始化的 final 属性的值。
>
> **这个隐含了，如果属性值不是 final 的，那就不能保证一定可以看到正确初始化的值，可能看到初始零值。**

```java
class FinalFieldExample { 
    final int x;
    int y; 
    static FinalFieldExample f;

    public FinalFieldExample() {
        x = 3; 
        y = 4; 
    } 

    static void writer() {
        f = new FinalFieldExample();
    } 

    static void reader() {
        if (f != null) {
            int i = f.x;  // 程序一定能得到 3  
            int j = f.y;  // 也许会看到 0
        } 
    } 
}
```

这个类`FinalFieldExample`有一个 final 属性 x 和一个普通属性 y。我们假定有一个线程执行 writer() 方法，另一个线程再执行 reader() 方法。

因为 writer() 方法在对象完全构造后将引用写入 f，那么 reader() 方法将一定可以看到初始化后的 f.x : 将读到一个 int 值 3。然而， f.y 不是 final 的，所以程序不能保证可以看到 4，可能会得到 0。