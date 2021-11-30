![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211118115856.png)

最开始http1.0只负责传输简单的html文本，然后有了css和js，以及网络环境更加复杂，所以提出了http1.1，1.1做了几个优化，主要围绕改进http的带宽和延迟：

首先你得明白http1.0在带宽和延迟方面什么原因导致网络速度慢，主要是：

1.早期的网络基础设施差，带宽低；

2.延迟方面

1）你想想一下一次http网络请求会发生哪些步骤，首先浏览器 比如chrome浏览器，同时时刻只允许与同一个域名保持四个tcp连接，如果你超过了4个连接，同一时刻下后面对该域名的所有请求都会被浏览器阻塞，所以你后面这些请求都还卡在浏览器这里呢。。

2）加入当前请求没被浏览器阻塞，那接下来就得去DNS服务器找IP，这里也有一定延迟，但可以通过IP缓存解决。

3）然后找到了IP，就必须先开始发起tcp三次握手，http请求报文最快可以放在第三次tcp握手报文中，然后如果你每次http请求都要执行tcp三次握手的话，那这就带来了比较高的延迟，所以后面设计了tcp复用。。。

### http1.1

为了降低延迟，并且符合更多的应用场景，http1.1设计了更多的字段，有更多的错误状态码，最重要的是**支持长连接**和请求的流水线处理。

1. 长连接：为了解决1.0中每次http请求都要先发起三次tcp握手，在1.1中支持长连接（persistentConnection），和流水线pipeline处理，这样的话我多个http请求可以在一个tcp连接上进行，这样是不是就减少了每次建立tcp连接和关闭连接的延迟呢。。。
2. 缓存处理：在1.0中也有缓存，主要是用head中的If-Modified-Since, Expires来作为缓存判断的标准，1.1中加了Entity tag, If-Unmodified-Since, If-match等字段来达到更准确的缓存策略。。
3. 带宽优化：1.0中客户端只能请求一个完整的对象，而1.1中可以请求一个对象中的一部分，它是在请求头中引入了range字段，然后响应包中响应码是206；
4. 1.1相比1.0还新增了24个错误状态码，以及可以在请求头中加hostname；

### http2.0 （重点在于低延迟）

1.0和1.1只基于文本，而2.0协议的解析是基于二进制格式；

支持多路复用，多个http请求可以同时在一条tcp连接上发送，也就是multiplexing；一个request会对应一个id；多个request可以随机的混杂在同一个tcp连接上；

2.0还支持header压缩，因为1.1中header带有了大量信息，每次都传的话占用了带宽，2.0中通讯双方各自cache一份header fields表，避免重复header传输。

2.0还支持服务端推送，比如一个页面中包含100个资源，那就要发送100个请求，2.0就在第一个请求页面时，响应这个页面资源，随后客户端不在发起页面中包含的资源请求，而是有服务器端自动推送；

注意：tcp有慢启动这一回事，因为tcp连接会随时间自我调节，tcp连接刚开始会限制传输速度，当数据传送成功时，才会提高传输速度。所以2.0通过多路复用tcp连接，避免了慢启动带来的延迟，充分利用了高带宽。

### 重点看看2.0

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211118163513.png)

从外层向里层看，在一个tcp连接上可以同时并行多个stream，这些流中传输这不同的http请求或者响应，也就是说一个tcp连接可以同时发送多个http请求，然后流里面传的是message（请求和响应都叫message），然后一个message里面包括header帧和body帧，所以可以看到2.0中将报文分成不同的帧，帧是最小单元；而1.1中拆包是基于\r\n分隔符。

我们知道1.x中是基于文本的，所以http头没有一个固定的大小，而2.0中基于二进制，报文被封装成了一个个帧，而帧是有固定格式的：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211118164807.png)

1.1中一个请求包括header，body，然后2.0中用一个header帧来承载1.1中的header，然后一个body帧来承载1.1中的body；其实就是把原本header中哪些字段的数据放到header帧的Frame payload部分，原本body中数据放到body帧的Frame payload部分；

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211118165312.png)

思考：可以看到在http2.0中也去做了拥塞控制、多路复用、server push，这其实是传输层协议tcp做的事情，然后现在应用层协议去干了，更好的利用了**单条**tcp连接，有种tcp over tcp的感觉；既然我只用了单条tcp连接，而且我在http层就去干了拥塞控制啥的，为什么不直接用在udp上面呢，实现一个tcp over udp。。

想象一下，比如做一个new tcp over udp, 我们这里的http2就是这个new tcp，它底层直接用udp做，然后grpc这样的应用层协议再over http2，这样http2就在应用层和传输层之间，传输层使用udp, 然后http2仅实现可靠的udp传输。这样是不是就把tcp消灭了呢。。。

参考：https://juejin.cn/post/6844903489596833800

https://www.arloor.com/posts/http2/