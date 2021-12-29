### io多路复用

可以用多进程、多线程；但是多进程fork一个进程消耗资源多，且需要上下文切换，而且进程之间资源不共享；而多线程虽然可以共享资源，且创建消耗资源少一些，但仍需上下文切换。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211229100133.png)

### select, poll, epoll

关于select:其实就是设置一个组长，不断的去轮询它管理的成员，轮询的事件：可读、可写、或者超时啥的。。但是确实是只能管理1024个文件描述符

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211229112006.png)

关于poll：能够管理的成员更多

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211229112238.png)

epoll:

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211229112732.png)

epoll不是轮询了，你可以看到它遍历完一次之后，下次只要check指定的关键节点即可。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211229145108.png)



### 关于Redis中所用的io多路复用模型

redis中的io多路复用本质上就是用的Linux中io多路复用的select, poll, epoll函数，redis只是对底层这些函数进行了封装，来处理接收的文件事件和时间事件。

首先你要知道redis服务器需要处理哪些类型的事件：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211229113816.png)

我们知道redis中文件事件是用单进程、单线程来处理的，那它怎么处理这种多客户端情况下的多事件呢？答案就是io多路复用

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211229114152.png)

比如某一个套接字产生了连接请求，那io多路复用程序会监听到这个事件，然后把这个套接字传送到文件事件分派器，分派器再把这个套接字分派到连接请求处理器。

那我们知道io多路复用程序是负责监听所有套接字的这些监听事件，**那它怎么去监听的呢？**是像上面说的用多个进程？多个线程？都不是！其实还是用的Linux内核中的select, poll, epoll。。只不过封装了一层：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211229114749.png)

也就是说不管你下面的任意一个socket产生什么事件，redis都去统一管理这些事件，并把这些事件分派到不同的事件处理器。

### 总结

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211229114956.png)

单线程处理对于redis来说足够了，因为瓶颈不是CPU，而是网络io；但是考虑到一些大键值对的删除操作，用多线程更快。

