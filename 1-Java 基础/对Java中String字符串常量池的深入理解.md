首先记住核心：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211213170321.png)

来源：https://tech.meituan.com/2014/03/06/in-depth-understanding-string-intern.html

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211213170429.png)



对字符串常量池有一个大致的认识：在1.6中字符串常量池放在永久代中，最大只有4M的空间，所以1.7就把字符串常量池放到堆中了。并且**常量池的数据结构是由C++写的一个类似于HashTable的东西**（所以常量池中也是存的对象！！），也就是数组+链表，但是数组长度固定1009，所以当字符串常量过多时，导致链表过长，String.intern()这个函数执行时间就变长。。（intern方法：如果当前String在字符串常量池中存在则直接返回字符串常量池中这个字符串对象的地址；如果字符串常量池不存在该字符串，则先在字符串常量池创建，再返回地址）

### 想想用intern方法目的是什么？？

我知道了，其实就是减少在堆中创建越来越多的String对象，而将String对象放到字符串常量池中，这个intern是一个native方法，比如 这两句：

String str = new String("aaa");  

String str = new String("aaa").intern();

前一句无论常量池中有没有"aaa"这个常量，都会在堆上创建一个新的String对象，而后一句会先看常量池中有没有这个对象，如果有则直接返回常量池中这个字符串对象的地址，如果没有则会在常量池中创建一个"aaa"对象，然后返回常量池这个对象的地址。。。并不会在堆上创建对象。。

所以说用intern方法本质上就是用字符串常量池，而常量池其实就是C++写的一个哈希表，用于快速返回已经有了的字符串对象，因为如果没有常量表，也就是如果没有这个哈希表，你怎么在整个堆上知道出现过这个字符串呢。。