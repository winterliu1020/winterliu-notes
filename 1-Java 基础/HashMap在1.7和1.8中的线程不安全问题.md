来源：[csdn](https://blog.csdn.net/swpu_ocean/article/details/88917958)

### 1.7中：

由于在扩容时采用的前插法，所以可能造成死循环或者数据丢失。（下面的3和7就死循环了，5就被丢失了）；而且在多个线程同时put(key, value)的时候还会有数据被覆盖的问题。

> Java中所有的变量存放在主存中，而线程使用变量时会把主内存里面的变量复制到自己的工作内存中，线程读写变量时操作的是自己工作变量中的内存，然后将自己工作内存中的变量刷新到主内存中。因此，当线程A和线程B同时处理一个共享变量时，会存在内存不可见的问题。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/Java%E5%9F%BA%E7%A1%80/20210422172741.png)

其实主要还是因为**Java内存模型**的问题，每个线程都有自己的本地内存，然后所有线程每次更新值需要更新到**主内存**，当多个线程用同一个hashmap时，如果多个线程同时put一个值，好巧，此时这些线程拿到的hashmap的size都到了阙值，所以各个线程都需要对hashmap进行扩容，又由于没有加锁，且CPU分片的原因，当一个线程执行了一部分步骤时，这个线程的CPU时间片用完了（注意A线程用完了CPU时间片后会变成就绪状态，但是它执行一部分步骤产生的数据变化还在本地内存），另外一个线程开始执行一个完整的扩容操作，也就是从主内存加载数据，然后在B线程本地内存进行扩容 最后更新到主内存。

然后A线程拿到CPU时间片恢复执行，后面就会发生死循环问题。具体看：[csdn](https://blog.csdn.net/swpu_ocean/article/details/88917958)和[csdn](https://blog.csdn.net/niu_8865/article/details/108565345)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/Java%E5%9F%BA%E7%A1%80/20210422165409.png)



### 1.8中：

扩容时将前插改成了尾插，避免了上面的死循环和数据丢失问题。但是数据被覆盖问题仍然存在，1.8的源码没有transfer函数，因为JDK1.8直接在resize函数中完成了数据迁移，1.8中put函数：

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null) // 如果没有hash碰撞则直接插入元素
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

第六行判断是否出现hash碰撞（也就是看桶节点是否已经有值），假设线程A,B都put，然后A，B线程中hash函数计算出来的插入下标都是一样的，当A执行完第六行代码后CPU时间片用完了（此时A只是定位到插入位置并且发现该位置是空的，还没有进行插入值），转为就绪状态，然后B拿到时间片完成了正常的插入，然后A重新拿到CPU时间片进行第七步插入，这样导致B插入的数据被A的数据覆盖了。

除此之前，还有就是代码的第38行处有个++size，我们这样想，还是线程A、B，这两个线程同时进行put操作时，假设当前HashMap的zise大小为10，当线程A执行到第38行代码时，从主内存中获得size的值为10后准备进行+1操作，但是由于时间片耗尽只好让出CPU，线程B快乐的拿到CPU还是从主内存中拿到size的值10进行+1操作，完成了put操作并将size=11写回主内存，然后线程A再次拿到CPU并继续执行(此时size的值仍为10)，当执行完put操作后，还是将size=11写回内存，此时，线程A、B都执行了一次put操作，但是size的值只增加了1，所有说还是由于数据覆盖又导致了线程不安全。**从这里也可以看出：重新拿到CPU时间片后不会重新从主内存加载新数据，用的还是自己线程中的本地内存**，其实想想也是，如果会有这种比较操作，其实是用了其它机制（CAS和volatile关键字）。

---

### hashmap为什么容量总是2的N次方？

其实就是将取模操作换成用位操作，但是效果一样，性能更好。

> 作者推算出了如果容量为2的N次方的话，那么hash&(length-1) == hash%length; length转为二进制位一个1+N个0，length-1转为二进制位一个0+N个1；则任意的hash值和length-1做位运算，结构都是后面N个位的的二进制，因为length-1的二进制前面都是0，位运算后都是0，相当于舍弃了高位，保留了后面的N位，后面的N位刚好在0-length之间，也就是等于hash%length取余

length是2的N次方，那么你去hash%length，值都散列在[0, length - 1]上；

同样，如果hash&(length-1)，length-1除了最高位是0，后面全是1，也就是：一个hash值去&(length-1)它能表示的最大值用十进制来说其实就是(length-1)，所以散列范围还是：[0, length - 1]。

### hashmap1.8中为什么选择6，8两个阙值

当每个桶上节点小于6时会退化成链表，当超过8时会将链表转成红黑树。

> 理想状态，受随机分布的hashCode影响，链表中节点遵循柏松分布，所以一般情况下，很少很少会出现一个桶上节点数超过8的情况，超过8如果还用链表的话，性能很差，会转成红黑树，但是转成红黑树的操作需要消耗性能，并且树节点占用的空间是普通节点的两倍。
>
> 正常情况下，都是用的链表，因为当节点过多时会进行扩容。

参考：[csdn](https://blog.csdn.net/niu_8865/article/details/108565345)