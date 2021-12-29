![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211224105119.png)

LinkedList是比较常用的，既可以做list，又可以用作队列Queue的实现，还可以做Stack的实现，因为Java中Stack被做成一个实体类了，而且继承自Iterator接口，效率不高，所以要用栈的时候常常用LinkedList。

可以看到还有一个Deque接口，双端队列，LinkedList和ArrayDeque都实现了这个接口。所以你要用栈或者队列的数据结构的时候LinkedList和ArrayDeque都可以考虑。我们可以看到Deque接口中居然tmd定义了栈的一些方法，难怪在LinkedList中实现了push, pop这类栈方法。。。

Deque主要定义了以下几类接口方法：

1.Queue的方法，其实和他继承自Queue的方法一模一样

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211224154101.png)

2.Stack的方法，因为Deque是双端队列，所以可以实现栈的效果

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211224154307.png)

3.Collection接口的方法

4.再就是属于双端队列的方法

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211224154505.png)

因为是双端队列，所以，之前Queue都是简单的add，remove，现在就是可以在两端，addFirst, addLst。。。



### LinkedList

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211224152316.png)

LinkedList是一个双向链表，里面都是Node连在一起，那我们来看看node：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211224152709.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211224155020.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211224155319.png)

### 总结：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211224105119.png)

再看这张图就清楚了，因为Deque这个接口太强大了，定义了Queue的方法，又有双端队列的方法，同时还定义了push, pop这两个栈的方法，所以LinkedList就实现了普通队列、双端队列、栈，同时还有List接口的方法。。。牛皮。。

说回本质，LinkedList就是用了一个内部类Node，然后两个指针first, last, 一个size，就这几个属性。。。

---

### 再来说说ArrayDeque

注意虽然底层是用Array来存储，但是你记住它本质上还是一个队列，也就是说它提供的方法还是队列的方法。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211224160215.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211224162611.png)

实现了一个循环数组的效果。同时实现了**双端队列**的方法。但是注意ArrayDeque不能存储null，LinkedList就可以。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211224164850.png)

