[参考链接1](https://blog.csdn.net/u010010428/article/details/52042644?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-1&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-1)

[参考链接2](https://cloud.tencent.com/developer/article/1595438)

主要是由于多线程下扩容，造成链表死循环。扩容源码：

```java
/**
 * Transfers all entries from current table to newTable. 
 */
void transfer(Entry[] newTable, boolean rehash) {
    int newCapacity = newTable.length;
    for (Entry<K,V> e : table) {

        while(null != e) {
            //（关键代码）
            Entry<K,V> next = e.next;
            if (rehash) {
                e.hash = null == e.key ? 0 : hash(e.key);
            }
            int i = indexFor(e.hash, newCapacity);
            e.next = newTable[i];
            newTable[i] = e;
            e = next;
        } // while  

    }
}
```

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/Java 基础/20201127094231.png)

[参考这篇文章](https://blog.csdn.net/u010010428/article/details/52042644?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-1&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-1)

其实主要是：当线程1（假设线程1中节点的连接顺序是：2-->3）中：e指向数组中的某个节点（假如是2节点），然后next指向该数组节点连接的链表的头节点（假如是3节点）

然后这时突然切换到线程2：线程2会执行一整套的扩容，那么在线程2执行完之后，内存空间中节点的连接顺序就变成了（3-->2）因为在1.7中扩容函数resize()调用的transfer函数转移到新数组，采用的是前插法，（千万注意：线程1和2会在各自的内存空间开创各自的新的newTable[]，但是注意hashmap的节点是线程共用的，所以线程2执行完之后节点变成3-->2，然后线程1获得CPU继续执行，此时在线程1中：e指向2，next指向3，然后执行前叉2节点，变成2-->3, 但是现在next是3，最后3的next会指向2，由此形成环）