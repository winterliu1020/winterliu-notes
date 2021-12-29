![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211213144438.png)

每隔一段时间这个抽奖小程序就突然无法使用了，但是看后台的日志，可以看到定时任务还在执行，所以其实整个程序还是在执行的，但是为什么小程序端无法连接到后端呢？

查了一下服务器出现大量Close_wait主要是后端处理前端请求时间过长，导致前端发起的一次http请求可能在5s内，后端还没有响应，前端就发一个fin给后端，要结束连接了。。此时后端接收到fin，然后后端返回一个ack给前端，然后后端就处于close_wait状态。。由于处理http请求的线程还在处理这个http请求，处于阻塞状态，所以无法给客户端回应。

https://www.cnblogs.com/grey-wolf/p/10936657.html