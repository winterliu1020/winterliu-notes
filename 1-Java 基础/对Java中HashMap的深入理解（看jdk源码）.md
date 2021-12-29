首先看看HashMap和HashTable的区别：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211224173419.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211224173541.png)

### 那我们正式来看看HashMap

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211224174426.png)

可以看到Node实现了Map中的Entry接口：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211224174729.png)

一些属性：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211224175108.png)

比较：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211224175424.png)

### 其它

涉及到扩容、链表转成红黑树。。。