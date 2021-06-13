## Java中断

Java中取消一个任务最好的、最合理的办法就是用中断，Java中并没有直接停止一个线程的方法，中断也只是修改线程的中断状态或者抛出一个InterruptException异常，仅此而已，只是向线程发出一个通知，至于采取什么措施要看线程自己如何响应。

### Interrupt status, InterruptedException

Java的中断机制能够使被中断的线程有机会从当前任务中跳脱出来，中断机制最核心的两个概念就是：Interrupt status, InterruptedException。

Java中对中断的大部分操作无外乎：

1. 设置或者清除中断标志位
2. 抛出InterruptException

**1. 关于Interrupt status**

Java中每个线程都有一个中断标志位，表示当前线程是否被中断，boolean类型变量，中断时将其设置为true，清除中断时设置为false。但是在Thread类中并没有InterruptStatus这个属性，只是提供了访问中断标志位的接口：

```java
/**
 * Tests if some Thread has been interrupted.  The interrupted state
 * is reset or not based on the value of ClearInterrupted that is
 * passed.
 */
private native boolean isInterrupted(boolean ClearInterrupted);
```

先返回当前线程中断状态，然后根据参数来判断是否要重置中断状态。

- isInterrupted()：调用native方法isInterrupted(false)，返回线程状态
- interrupted()：这是一个静态方法，调用native方法isInterrupted(true)，返回当前线程状态然后重置线程状态为false
- interrupt()：调用interrupt0()，只是将线程状态改为true

> 可见，`isInterrupted()` 和 `interrupted()` 方法只涉及到中断状态的查询，最多是多加一步重置中断状态，并不牵涉到`InterruptedException`。
>
> 不过值得一提的是，**在我们能使用到的public方法中，`interrupted()`是我们清除中断的唯一方法。**

**2. 关于InterruptException**

如果有其他方法直接或间接的调用了这两个方法，那他们自然也会在线程被中断的时候抛出InterruptedException，并且清除中断状态。例如:

- wait()
- wait(long timeout, int nanos)
- sleep(long millis, int nanos)
- join()
- join(long millis)
- join(long millis, int nanos)

这里值得注意的是，虽然这些方法会抛出InterruptedException，但是并不会终止当前线程的执行，当前线程可以选择忽略这个异常。

也就是说，无论是设置`interrupt status` 还是抛出`InterruptedException`，它们都是给当前线程的**建议**，当前线程可以选择采纳或者不采纳，它们并不会影响当前线程的执行。

至于在收到这些中断的建议后，当前线程要怎么处理，也完全取决于当前线程。

### interrupt

```java
/**
 * Interrupts this thread.
 *
 * <p> Unless the current thread is interrupting itself, which is
 * always permitted, the {@link #checkAccess() checkAccess} method
 * of this thread is invoked, which may cause a {@link
 * SecurityException} to be thrown.
 *
 * <p> If this thread is blocked in an invocation of the {@link
 * Object#wait() wait()}, {@link Object#wait(long) wait(long)}, or {@link
 * Object#wait(long, int) wait(long, int)} methods of the {@link Object}
 * class, or of the {@link #join()}, {@link #join(long)}, {@link
 * #join(long, int)}, {@link #sleep(long)}, or {@link #sleep(long, int)},
 * methods of this class, then its interrupt status will be cleared and it
 * will receive an {@link InterruptedException}.
 *
 * <p> If this thread is blocked in an I/O operation upon an {@link
 * java.nio.channels.InterruptibleChannel InterruptibleChannel}
 * then the channel will be closed, the thread's interrupt
 * status will be set, and the thread will receive a {@link
 * java.nio.channels.ClosedByInterruptException}.
 *
 * <p> If this thread is blocked in a {@link java.nio.channels.Selector}
 * then the thread's interrupt status will be set and it will return
 * immediately from the selection operation, possibly with a non-zero
 * value, just as if the selector's {@link
 * java.nio.channels.Selector#wakeup wakeup} method were invoked.
 *
 * <p> If none of the previous conditions hold then this thread's interrupt
 * status will be set. </p>
 *
 * <p> Interrupting a thread that is not alive need not have any effect.
 *
 * @throws  SecurityException
 *          if the current thread cannot modify this thread
 *
 * @revised 6.0
 * @spec JSR-51
 */
public void interrupt() {
    if (this != Thread.currentThread())
        checkAccess();

    synchronized (blockerLock) {
        Interruptible b = blocker;
        if (b != null) {
            interrupt0();           // Just to set the interrupt flag
            b.interrupt(this);
            return;
        }
    }
    interrupt0();
}
```

线程可以自己中断自己（不需要检查权限），也可以在一个线程中中断另外一个线程的执行，需要通过checkAccess()检查权限，有可能抛出SecurityException异常。

如果当前线程由于如下方法处于阻塞当中，调用interrupt方法后，**线程的中断标志会被清除**，并且收到InterruptException异常。

- Object的方法
  - wait()
  - wait(long)
  - wait(long, int)
- Thread的方法
  - join()
  - join(long)
  - join(long, int)
  - sleep(long)
  - sleep(long, int)

> wait, join, sleep方法在定义的时候都能捕捉InterruptException。

如果线程没有因为上面的函数调用而进入阻塞状态的话，那么中断这个线程interrupt仅仅会设置它的中断标志位，不抛出异常。还有就是：中断一个已经终止的线程没有任何影响。

> 总结：所谓中断线程，并不是让线程停止运行，而仅仅是让线程的中断标志设为true，或者在某些特定情况下抛出一个InterruptedException。

[学习来源](https://segmentfault.com/a/1190000016083002)