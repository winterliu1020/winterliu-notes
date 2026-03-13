Thread这个类在java.lang这个包下面，之前以为在JUC下面呢。。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211017201747.png)

可以看到JDK中对Thread的初始定义，它说一开始JVM就只有一个非守护线程，这个就是main thread，然后慢慢的Java虚拟机会继续创建一些守护线程，比如一些用于垃圾收集的守护线程、一些执行辅助操作的守护线程，守护线程和普通的线程有什么不同呢？Java虚拟机如果停止执行，它会在停止执行之前等待非守护线程执行完毕，JVM不会care 守护线程是否还在执行，只要非守护线程执行完了，JVM就可以停止执行。而且一般只有守护线程才会创建守护线程。

从图可以看到JVM跑起来之后，会一直执行执行，直到Runtime这个类中的exit()方法被执行，并且Security manager执行checkExit(status)检查状态码，并且运行exit；然后JVM才会关闭（当然其中还涉及到hook，addHook之类的知识）；

或者当JVM中所有非守护线程全死了，并且那些跑起来的线程的run方法以及return了或者已经抛出一个异常了。这样JVM才会停止执行。

以上是对Thread的初步认识，再看这个类，有threadName, 然后它所属的**threadGroup**（threadGroup也有子group和父group），**stackSize**也就是这个线程中规定的最大栈的字节数，**threadSeqNumber**：tId thread id 线程id 这是一个long字段，从0开始自增，jdk中用的是一个synchronized方法来对threadSeqNumber这个long字段自增1，而不用单纯的volatile修饰，保证原子性。

还有priority线程的**优先级**，一般来说线程的优先级就是它父线程的优先级，而且取值是：1，5，10；还有是否是守护线程 一个boolean类型的字段daemon；还有一个比较重要的**线程状态threadStatus**，有0-5六种取值，分别是NEW, RUNNABLE（其实就对应了操作系统线程的READY, RUNNING状态）, WAITTING, TIMEED WAITING, BLOCK;

还有一个**target字段**，是一个Runnable对象，主要用于传runnable对象来创建thread实例时保存传进来的runnable对象，然后这个thread执行start()方法其实真正要跑的代码是传过来的runnable对象中的run()方法中的代码，如果是以继承Thread类的方式创建thread实例，那执行thread.start()跑的代码就是子类重写的run()方法。

还有一些不太用到的属性，另外一部分就是Thread类中的方法了，比如最基本的init(一些thread配置参数)方法，在初始化一个thread的时候会初始化所属的group，分配一个tid，是否是daemon，优先级啊，target...

当然Thread类中还有sleep(), yield(), stop(), exit()方法。。大部分都会和一个native方法对应，其中比较重要的就是thread.start()方法了，详细说一下：

Thread.java --> jvm.cpp --> thread.cpp --> os_linux.cpp

看一下这个文件调用顺序，thread.java中就是在Java层面我们调用thread.start()他会调用一个start0()的native方法，众所周知，Java当中的native方法使用c,c++来实现的，所以本质上调用native start0()就是用JNI调用jvm.cpp中的JVM_StartThread()方法，这相当于已经在C++层面了，在JVM_StartThread()方法中会执行native_thread = new JavaThread(&thread_entry, sz)创建一个C++层面的thread，在C++层面new 一个 JavaThread其实是到thread.cpp中执行new JavaThread()对应的构造方法，构造方法中会调用os:: create_thread(this, ...)，而creat_thread()是在os_linux.cpp中，**这个方法会执行pthread_create()方法真正创建一个系统线程。**

pthread_create(&tid, &attr, thread_native_entry, thread)第三个参数是回调方法，第四个是回调方法的参数，这个回调方法传入的是C++层面的thread，然后在回调方法中会执行thread.run()也就是在c++层面执行thread的run方法，后面就是执行JavaCalls回调方Java层的方法 run方法。

**具体看，写的很好**：https://www.cnblogs.com/tera/p/13937611.html

---

当然Thread比较重要的还有中断，Java当中的中断只是设置中断标志字段，或者抛出中断异常。并不会让程序停止，要让程序停止你要在程序中响应中断异常，或者响应中断标志位，由你来控制线程的结束。

还有：线程不能重复start，重复start 线程会依据threadStatus字段判断是否抛出illageThreadStatusException异常；线程的stop方法也已经被丢弃了。还有thread.exit()方法，线程正常的退出之前这个方法会将一些资源给释放。