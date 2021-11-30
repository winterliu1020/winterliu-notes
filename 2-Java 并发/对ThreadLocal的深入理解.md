首先想一下为什么要有threadLocal类？比如在主线程中有一个变量，各个子线程是可以直接访问这个变量的，那如果我想要在每一个线程中单独要一份呢？而且各个线程中对自己线程的该变量修改不会影响到其他线程的该变量的值。这就要用到threadLocal类了。

```java
public class ThreadLocalPractice {
    ThreadLocal<Animal> animalThreadLocal = new ThreadLocal<>();
    public static void main(String[] args) {
        // 开辟两个线程，分别去set这个threadLocal，然后get获取值
        ExecutorService executorService = new ThreadPoolExecutor(
                5,
                10,
                3,
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<>(1),
                new ThreadFactory() {
                    @Override
                    public Thread newThread(Runnable r) {
                        Thread thread = new Thread(r); // 线程工厂，注意要把r传给thread
                        return thread;
                    }
                },
                new ThreadPoolExecutor.AbortPolicy()
        );

        executorService.execute(new ThreadLocalPractice().new DogRunnable());
        executorService.execute(new ThreadLocalPractice().new PigRunnable());
        // 详细讲讲threadLocal.get(), set()在thread类发生了什么
    }

    class DogRunnable implements Runnable {
        @Override
        public void run() {
            animalThreadLocal.set(new Animal("dog"));
            System.out.println("Dog:" + animalThreadLocal.get().getName());
        }
    }
    class PigRunnable implements Runnable {
        @Override
        public void run() {
            animalThreadLocal.set(new Animal("pig"));
            System.out.println("Pig:" + animalThreadLocal.get().getName());
        }
    }
}

```

要知道threadLocal针对的是thread，可以看到在thread类中有ThreadLocal.ThreadLocalMap类型的 threadLocals和 inheritableThreadLocals，后面这个inheritableThreadLocals是一个可继承的threadLocalMap，什么意思呢？如果我们只有一个threadLocals这一个map的话，我们想在一个子线程中获得创建这个子线程的父线程的threadLocalMap存储的所有的<threadLocal, value>怎么办呢？那就得用到inheritableThreadLocals了，在一个线程中，如果你想把这个线程中的对象传到它的子线程，你应该声明**inheritableThreadLocal类的实例，它是继承ThreadLocal类的，只是重写了createMap和getMap方法，让这两个方法返回的是当前线程中的inheritableThreadLocals变量，**这样这个线程中声明的所有的inheritableThreadLocal实例就会存放在inheritableThreadLocals这个map了。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211027172553.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211028112927.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211028113950.png)

InheritableThreadLocal类，只是重写了createMap和getMap方法，让这两个方法返回的是当前线程中的inheritableThreadLocals变量：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/image-20211028113353450.png)





ThreadLocalMap是ThreadLocal类中一个静态内部类：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/image-20211027172928614.png)



threadLocal在主线程和子线程的关系：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211027173627.png)

我们知道一个Thread实例中包含threadLocalMap属性，但是初始化的时候是null的，那什么时候线程中的这个map会放入key, value呢？

可以看到threadLocalMap的构造是懒加载，如果在线程中没有执行put，或者get,这个线程中的threadLocalMap是一直为null的，当你执行某个threadLocal对象的put或者get方法的时候，它才会去找一下这个线程的threadLocalMap对象中有没有这个threadLocal对象的key，在这个线程第一次访问threadLocal对象的时候才会创建threadLocalMap对象。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/image-20211028105628376.png)

我们再来看看ThreadLocal类的get, set, remove等方法：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211028111016.png)

这是threadLocal对象的set，get方法，这些方法底层会调用threadLocalMap这个map维护的put(key, value), get(key)方法。



![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211028114512.png)

想一下为什么这个map中的key的类型threadLocal要设计成弱引用呢？

threadLocal在map中被设计成弱引用，是为了当这个threadLocal实例在所有线程中都被回收了，这时候设计成弱引用 这些线程中的map存储的对应于这个threadLocal的entry就会被回收，因为Java线程中的threadLocalMap属性并不能调用这个map的set, put等方法，这个map的这些方法都被设计为private，只能通过threadLocal对象的public类型的get(), set()方法间接调用map的这些private方法。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211028154031.png)

那这个map为什么不提供public方法供外界使用呢？我看到下面这篇文章里说是因为thradLocalMap这个map存放的数据是整个线程都在用，设计threadLocal这个类出来也就是为了方便在同一个线程的不同方法栈中使用数据，那么我直接用threadLocal获取数据一是更加方便，二是不破坏其它地方保存到这个map中的数据。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211028155832.png)

https://zhuanlan.zhihu.com/p/304240519