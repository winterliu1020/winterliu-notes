CyclicBarrier有点类似于CountDownLatch，但是CountDownLatch只能倒计时一次，而CyclicBarrier能够reset，也就是类似于能够多次倒计时；他们两更深一点的区别是：CountDownLatch是一个线程（或者几个线程）**阻塞等待**其它线程去完成任务，当其它线程完成任务之后，这一个线程才会往下执行；而CyclicBarrier是**几个线程相互等待**，只有等到达到了这个栅栏的要求数才会放行，这几个线程会同时被唤醒往下执行；

而CyclicBarrier类的最核心的方法是dowait()：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211117223728.png)

CyclicBarrier稍微复杂一点，它依赖的是ReentrantLock，然后用到了ReentrantLock中的一个condition，把这个condition称为trip，当达到栅栏要求时，也就是这个condition队列的长度等于CyclicBarrier设置的初始值时，condition条件队列所有节点放到阻塞队列中等待被唤醒。。。