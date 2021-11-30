![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211022151457.png)

讲解最下面的线程池Executor体系

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211020145104.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211020145157.png)

Executor框架提供管理终止的方法和产生一个Future来跟踪一个或多个异步任务。

可以把Runnable直接交给给ExecutorService.execute(Runnable command)执行，也可以把Runnable或者Callable提交给ExecutorService.submit(Runnable command), ExecutorService.submit(Callable call)去执行。

其实最终用线程去跑你的代码都是在线程实现类ThreadPoolExecutor中重写的execute()方法，如下图：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211020155647.png)

只有在这里才真正放到线程池的等待队列中准备跑你的代码，而且你看传给execute()方法的是一个Runnable对象，这也符合最顶层的接口Executor中只有一个Runnable参数的execute方法。

因为FutureTask这个类拥有的属性是Callable对象，而不是Runnable，很明显嘛，我FutureTask的run方法得返回执行结果或者异常，那肯定得用Callable对象。那为什么我们可以用threadPool.submit(runnable, value)或者threadPool.submit(runnable)呢？这里可是runnable对象，其实Doug Lea在AbstractExecutorService这个抽象类中实现了submit方法，他用newTaskFor()方法把你传进去的runnable对象或者runnable和一个value对象封装成了一个Callable的子类叫做AdaptRunnable，这个类中有一个Callable对象和一个value，这样就实现了submit方法在runnable和callable上的通用性。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211025220916.png)



![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211020163524.png)

不管你传一个单独的callable，或者runnable+result，都会用newTaskFor转成一个RunnableFuture对象，RunnableFuture对象中放的是一个Callable对象，runnableFuture类重写的run方法中执行的是callable.call()方法，然后把执行结果放到RunnableFuture的output属性，然后submit()方法返回就是这个RunnableFuture对象，这样你就可以用future.get()获取到你跑的代码的结果，这个结果就是output的值了，注意callable.call()方法会捕获异常，所以如果执行callable.call()时发生异常，这个output就是捕获到的那个异常的结果了。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211026111155.png)



![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211021102946.png)

总结：比较巧妙的点在于用了一个多继承的接口RunnableFuture，它继承了Runnable和Future对象，所以我才可以把RunnableFuture实现类FutureTask的实例传给execute(futureTask)去执行，（注意这里执行也有一个巧妙的地方：这是在AbstractExecutorService这个抽象类中，这个类中并没有实现真正实现execute(runnable)方法，这里只是继承自Executor这个顶层接口中的，但是，我们会在实现类 比如ThreadPoolExecutor中真正实现execute(runnable) 方法，这个方法中涉及到任务队列、线程池中线程调度。

## 定时线程池

看完Executor框架的左边部分，再来看看右半部分定时任务。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211020145104.png)

ScheduledExecutorService也是一个接口，为什么需要创建这样一个接口呢？因为定时任务线程池实现类需要额外用到几类schedule()方法，这是在原先的ExecutorService接口中所没有的方法，原来只有对线程管理的execute, submit, shutDown, invokeAll等方法，不涉及到定时，所以为了对这一部分进行抽象，新创建了这个ScheduledExecutorService接口，增加：schedule(runnable), schedule(callable), scheduleAtFixRate(runnable, initialDelay, delay, timeUnit), scheduleWithFixDelay(runnable, initialDelay, delay, timeUnit)；这四个方法都返回ScheduledFuture<?>。

看到这里，有点好奇为什么scheduleWithFixRate或者scheduleWithFixDelay两个方法啊中不能传callable对象呢？这样我就可以每次定时执行，并且还能拿到callable的执行结果了。很遗憾这个接口中并没有，所以我得继续往ScheduledThreadPoolExecutor这个实现类中看，看它怎么实现的定时执行，看以上增加的四个方法返回的ScheduledFuture是什么。

初步思考：普通的任务调度实现在ThreadPoolExecutor中已经完成了，包括声明多少个核心线程、然后线程池中总的线程多少个，然后选择哪种任务队列，还有拒绝策略。。在这个类中已经可以用threadPoolExecutor.execute(runnable)或者threadPoolExecutor.submit(runnable/callable)来不断接收任务并放在任务队列执行了。

那么ScheduledThreadPoolExecutor它执行的是scheduleXXX()方法，虽然它传进来的也是runnable或者callable，但是它还会传delay这种延迟执行时间，那它怎么实现这种延迟执行的呢？

首先ScheduledThreadPoolExecutor是在ThreadPoolExecutor基础上扩展一些定时方法，那么它相比于ThreadPoolExecutor多了些什么呢？

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211022160100.png)

1.使用了一个自定义的task类型，叫做ScheduledFutureTask，我们知道，这个ScheduledThreadPoolExecutor继承了ThreadPoolExecutor类，所以它也可以用继承得来的execute(runnable)或者submit(runnable/callable)方法，但是在ScheduledThreadPoolExecutor类中，所有提交过来的runnable或者callable都会封装在ScheduledFutureTask中，然后这些不需要延时执行的task就设置成delay时间是0。

2.使用一个自定义的队列，叫做DelayWorkQueue，是无边界DelayQueue的变体，相比于ThreadPoolExecutor，ScheduledThreadPoolExecutor中的任务队列没有容量约束，而且简化了corePoolSize和maximumPoolSize。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211022171106.png)

再看下这个队列怎么把任务加入：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211130174534.png)

可以看到用到了ReentrantLock保证入队的同步，再看下任务出队：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211022181326.png)

任务出队思路，以及如何实现延时：可以看到任务队列加了锁，这里用的是ReentrantLock，当很多活动线程去向队列中取task的时候，首先需要拿到这把锁，拿到之后，看一下队列的第一个任务的delay时间，如果还没到当前时间，则拿到该任务的线程会阻塞至到期时间，然后再取任务。

首先这个任务队列是一个继承自PriorityQueue的队列，定时时间越短的任务放在越前面。

具体：https://cloud.tencent.com/developer/article/1697106



## ThreadPoolExecutor类的理解

这个类实现了线程的管理、任务的添加、对线程池的监控。。

首先想一下以下几个问题：

1.任务队列满了怎么办？

2.线程池中什么时候会创建一个新的线程？

3.提交任务时，线程池中线程满了怎么办？

4.线程池中空闲线程什么时候会关闭？

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211026154510.png)

可以看到线程池状态为Running的时候，是用-1代表的，shutdown是0，所以如果有任务提交到线程池执行的时候，需要先检查一下当前线程池状态，是ctl是-1的时候才能提交。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211130174708.png)



![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211130174735.png)

重点看上图的run()方法，可以看到doug lea抽象出了一个worker，worker是runnable的子类，同时有一个thread属性，然后重写了父类Runnable的run方法。Worker中还有一部分是AQS的方法，看注释可以知道它实现了一个简易的锁，因为worker里面有thread，需要对这个线程的执行权加锁，这里用AQS实现了一个独占锁，而不是拿ReentrantLock这个可重入锁。



上面介绍了一下Worker这个内部类，要知道我们不管是threadPoolExecutor.execute()还是threadPoolExecutor.submit()，最终都会调用execute()方法，

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211130174823.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211130174854.png)

上面最后一行应该是如果任务队列满了，会尝试直接addWorker(command, false)，表示创建非核心线程。

那么传的command 或者 true/false 怎么创建对应的worker对象呢？线程池中怎么存储这些worker对象呢？所以看看addWorker()方法：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211026174214.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211130174922.png)

addWorker()中执行t.start()会发生什么呢？如下图，其实就是调用worker中的run()方法

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211026180110.png)

再来看看runWorker()方法，这个方法传了worker对象自己，这个方法是在run()方法中，所以说这其实已经是在子线程中执行runWork()方法了。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211026185444.png)

再看看getTask()，getTask()有哪些情况会返回呢？

1. 阻塞直到有任务，而且核心线程是不会被回收的，会一直等待任务
2. 超时关闭：对于非核心线程，keepAliveTime如果到了，还没有getTask还没有返回一个任务，非核心线程会被回收
3. 还有对一些条件的判断，会返回null

具体看：https://javadoop.com/post/java-thread-pool

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211130174952.png)

对于线程池中线程的回收：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211026195300.png)

