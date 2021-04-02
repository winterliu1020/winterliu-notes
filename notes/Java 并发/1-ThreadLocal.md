```java
static ThreadLocal<String> localVariable = new ThreadLocal<>();
localVariable.get(); // 获取当前线程本地内存中localVariable变量的值
localVariable.remove(); // 清除当前线程的本地内存中localVariable变量

Thread threadOne = new Thread(new Runnable() {
    public void run() {
        localVariable.set("threadOne");
    }
});

Thread threadTwo = new Thread(new Runnable() {
    public void run() {
        localVariable.set("threadTwo");
    }
});
```

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/concurrent/20201008161720.png)

如果你创建了一个ThreadLocal变量，那么访问这个变量的每个线程都会去**复制**这个变量，保存到自己线程的内存中，成为一个**本地副本**（这样别的线程操作threadLocal变量时不会影响这个线程中的threadLocal变量），比如线程one操作这个变量时，其实操作的是线程one的本地内存中的副本。从而避免了线程安全问题。

你不是开了很多个线程吗，但是每个线程的本地变量不是存放在**ThreadLocal实例**里面，而是存放在调用线程的threadLocals变量（Thread类的一个属性）里面。

ThreadLocal就是一个工具壳（包住了Thread类），调用threadLocal变量的set和get方法时再去拿当前线程（Thread）的threadLocals这个属性的值。

因为每个线程（Thread）可以关联很多个threadLocal变量，所以把threadLocals属性设计为ThreadLocalMap（一个定制化的HashMap）。threadLocals是一个HashMap结构，其中key就是当前线程的ThreadLocal的实例对象引用，value是通过set方法传递的值。

如果当前线程不消亡，那么这些本地变量会一直存在当前线程的threadLocals这个map型变量中，所以可能造成内存溢出，因此使用完毕后记得调用threadLocal.remove()，删除当前线程threadLocals中的本地变量。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/concurrent/20201008164923.png)

### ThreadLocal不支持继承性

同一个ThreadLocal变量在父线程中被设置值后，在子线程中是获取不到的。还是和上面说的一样，因为各个线程中的threadLocal变量是相互独立的，存在于各自线程实例的threadLocals属性中。

### InheritableThreadLocal类

用这个类之后，子线程就可以访问父线程中设置的本地变量。这个类继承于ThreadLocal类。它重写了ThreadLocal中的三个方法，它重写了createMap方法，所以当第一次调用set方法时，创建的是当前线程的inheritableThreadLocals变量的实例而不再是threadLocals。当get方法获取当前线程内部的map变量时，获取的也是inheritableThreadLocals而不再是threadLocals。

所以，在InheritableThreadLocal的世界里，变量inheritableThreadLocals代替了threadLocals。当用的是InheritableThreadLocal类时，创建map的ThreadLocalMap构造函数内部会把父线程的inheritableThreadLocals成员变量的值复制到新的ThreadLocalMap对象（也就是子线程的inheritableThreadLocals）中。

总结：

1. InheritableThreadLocal类通过重写getMap(Thread t){ return t.inheritableThreadLocals;}和createMap(Thread t, T firstValue) { t.inheritableThreadLocals = new ThreadLocalMap(this, firstValue);}方法让本地变量保存到了具体线程的inheritableThreadLocals变量中。
2. 当父线程创建子线程时，构造函数会把父线程中inheritableThreadLocals变量里面的本地变量**复制**一份保存到子线程的inheritableThreadLocals变量里面。**所以重点是在创建子线程时（在Thread类的构造方法中），复制了一份一模一样的给子线程**。
3. 详细说明：创建子线程就需要调用Thread的构造方法，而且你还需要帮子线程new一个inheritableThreadLocals变量，因为这个变量是ThreadLocalMap类型的，所以需要调用ThreadLocalMap类的构造方法（最终就是在这个构造方法中将父线程的inheritableThreadLocals的所有<key, value>值复制给子线程的inheritableThreadLocals变量的）。

```java
public static ThreadLocal<String> threadLocal = new InheritableThreadLocal<String>();
```

什么时候需要子线程可以获取父线程的threadlocal变量呢？比如子线程需要使用存放在threadlocal变量中的用户登录信息，再比如一些中间件需要把统一的id追踪的整个调用链路记录下来。

我们还可以手动去做，比如创建线程时传入父线程中的变量，并将其复制到子线程中，或者直接在父线程中将threadLocal中的值存到一个map中，将这个map作为参数传给子线程。

















































