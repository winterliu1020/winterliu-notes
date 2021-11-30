整个Atomic类的核心就是volatile + cas，atomic类中的value的用volatile关键字修饰，让一个线程的修改对其它线程立即可见，同时为了防止并发修改，用cas，其中用到了unsafe类来获取value属性的偏移量，大家都可以通过这个偏移量来获取value的值，然后所有人用cas对这个value值加上一个值，因为cas是原子的，所以只有一个人能添加成功，那其它没有cas成功的人呢？可以看到底层是用了一个do while循环，当cas失败时会再次执行循环，也就是再次cas，再次cas也就是说得再次拿到该地址的一个新值，然后对该新值用cas加上你要添加的值。成功则会退出cas。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211104120230.png)



![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211104115834.png)

再深入看看unsafe.getAndAddInt(this 也就是atomic对象本身, valueOffset偏移量, delta要增加的数值) + delta;

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211104165305.png)

重点是用了一个do while循环 当第一次cas失败时，会再次循环cas，直到成功才会退出循环，这时返回var5就是更新之后的值，也就是执行了add之后的值。