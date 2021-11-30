Java对象在内存中的布局分为三块区域：对象头、实例数据、对齐填充。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/Java%20%E5%9F%BA%E7%A1%80/20210326162711.png)

Synchronized用的锁就是存在Java对象头里的，hotspot虚拟机的对象头主要包括两部分数据：Mark Word（标记字段）、Class Pointer（类型指针），Class Pointer是对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。**Mark Word用于存储对象自身的运行时数据，它是实现轻量级锁和偏向锁的关键。**

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/image-20210326165009522.png)

### 对象头中的Mark word与线程中Lock Record

线程进入同步代码块的时候，如果此同步对象没有被锁定，即这个同步对象的锁标志位是01，则虚拟机首先会在当前线程的栈中创建我们称之为“锁记录Lock record”的空间，用于存储锁对象的Mark word的拷贝。

Lock record是线程私有数据结构，每一个线程都有一个可用Lock record列表，每一个被锁住的对象的Mark word都会与一个Lock record关联（对象头的Mark Word中的Lock Word指向Lock Record的起始地址），同时Lock Record中有一个Owner字段存放拥有该锁的线程的唯一标识，表示该锁被这个线程占有。

### 监视器（monitor）

任何一个对象都有一个monitor与之关联，当且一个monitor被持有后，它将处于锁定状态。synchronized在JVM里的实现都是基于进入和退出monitor对象来实现方法的同步和代码块的同步，都可以通过成对的MonitorEnter和MonitorExit指令来实现。

1. **MonitorEnter指令：插入在同步代码块的开始位置，当代码执行到该指令时，将会尝试获取该对象Monitor的所有权，即尝试获得该对象的锁；**
2. **MonitorExit指令：插入在方法结束处和异常处，JVM保证每个MonitorEnter必须有对应的MonitorExit；**

那什么是Monitor？可以把它理解为 一个同步工具，也可以描述为 一种同步机制，它通常被 描述为一个对象。

与一切皆对象一样，所有的Java对象是天生的Monitor，每一个Java对象都有成为Monitor的潜质，因为在Java的设计中 ，**每一个Java对象自打娘胎里出来就带了一把看不见的锁，它叫做内部锁或者Monitor锁**。

也就是通常说Synchronized的对象锁，MarkWord锁标识位为10，其中指针指向的是Monitor对象的起始地址。在Java虚拟机（HotSpot）中，Monitor是由ObjectMonitor实现的，其主要数据结构如下（位于HotSpot虚拟机源码ObjectMonitor.hpp文件，C++实现的）：

```c++
ObjectMonitor() {
    _header       = NULL;
    _count        = 0; // 记录个数
    _waiters      = 0,
    _recursions   = 0;
    _object       = NULL;
    _owner        = NULL;
    _WaitSet      = NULL; // 处于wait状态的线程，会被加入到_WaitSet
    _WaitSetLock  = 0 ;
    _Responsible  = NULL ;
    _succ         = NULL ;
    _cxq          = NULL ;
    FreeNext      = NULL ;
    _EntryList    = NULL ; // 处于等待锁block状态的线程，会被加入到该列表
    _SpinFreq     = 0 ;
    _SpinClock    = 0 ;
    OwnerIsThread = 0 ;
  }
```

**Monitor对象存在于每个Java对象的对象头Mark Word中（存储的指针的指向），Synchronized锁便是通过这种方式获取锁的，也是为什么Java中任意对象可以作为锁的原因，同时notify/notifyAll/wait等方法会使用到Monitor锁对象，所以必须在同步代码块中使用。**





