首先你需要了解HTTPS，这个S指的是安全，那体现在哪些方面呢？

1）会认证客户端、服务器端，确保数据是发到正确的客户端和服务器端

2）当然传输的数据会进行加密，防止被中途窃取

3）维护数据的完整性，**确保数据在传输过程中不被改变**

那HTTPS是怎么做到这些方面呢？用了什么新的协议或者手段呢？

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211214165623.png)

1.对称加密

也叫单密钥加密，加密和解密都用同一个密钥

2.非对称加密

公钥和私钥，公钥加密、私钥解密，同时要求加密内容不能超过公钥的长度，而一般公钥有2048位，所以加密内容不能超过256个字节。

3.摘要算法

就类似于MD5算法，就是不管你内容有多大，执行摘要算法，得到一个128位的密文，就代表了这个内容，通过摘要算法保证HTTPS在传输过程中内容**防止篡改**。

4.数字签名（可以验证信息是由对方发过来的+信息的完整性！！）

可以看到它依赖于非对称加解密、数字摘要两个算法。数字签名是将摘要信息用发送者的私钥先加密，然后与原文一起传送给接收者。

然后接收者首先用发送者的公钥解密，解密就得到了数字摘要，然后再用HASH函数对原文产生一个摘要信息，与解密得到的摘要信息对比，如果一样，说明收到的信息是完整的，没有被中途篡改。

数字签名作用总结：1.确定是由我知道的那个发送方发过来的，因为我是用这个发送方的公钥解的密啊，如果别人发过来那用这个公钥怎么解的开。。2.保证是对方发过来的，而且还能保证是对方发过来的那些数据，中途没有被篡改，所以这里用摘要算法对发过来的原文提取摘要，比一下就知道了。

5.数字证书

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211214173913.png)

---

以上是一些安全知识，那HTTPS是用SSL/TLS来保证安全，而SSL/TLS又是用到上面这些知识来实现。

### SSL 安全套接字层

ssl分为两层：

1）SSL记录协议

2）SSL握手协议

### TLS 传输层安全协议（保密、完整性、认证）

可以理解为SSL3.1;也由TLS记录协议、TLS握手协议组成。为什么要设计TLS呢？这个协议的设计目标是什么呢？它是为了保障传输层（比如tcp）的安全，位于传输层和应用层之间，可以看作是4.5层。。。可以看到它保障了整个传输层的安全，而且TLS是一个独立的协议，它不单单是服务于HTTP的，其它应用层协议比如邮件、Telnet协议都是用TLS。

### TLS协议组件

首先TLS协议也类似于一个框架，它保障了三个安全点，即：认证、保密、完整性。那TLS为了达到这三个安全，里面又分为几层，其实可以把TLS看成由几个组件拼成，每个组件去完成一个安全点。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211215104003.png)

而每个组件又是由特定的密码学算法来解决，比如这里用到了认证算法、加密算法、消息认证算法。。密钥交换算法。。而且TLS设计的非常灵活，它可以看出面向接口的设计，也就是说，底层用哪一种加密算法或者认证算法。。可以由参数来选择，并不固定。。。所以可以不断完善修改这些底层算法，只要提供的接口不变就行。

### SSL，TLS的握手过程

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211214175338.png)

可以看到握手协议是我们常常需要关注的，本质上就是客户端和服务器端端一些交互，以此共享一些信息，比如用哪种**密码算法**、**共享相互之间的密钥**、**还有证书认证**。。。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211215104841.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211214175706.png)

### Client Hello

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211215105833.png)

可以看到是在TCP协议之上的，然后主要传：加密套件、一个随机数；加密套件有一个优先级，就是说客户端把最希望采用的（可能是最新的协议版本）放在最前面，然后服务器优先考虑客户端想选择的加密套件。

### Server Hello

当服务器收到客户端的Hello之后，服务器也必须给客户端发送问候消息。服务器收到Client Hello，那我服务器如果要和你通信，那我肯定得看看你发过来的你支持的加密套件和TLS版本是不是我能够支持的，如果你发过来的所有条件我都可以支持，那我就给你发送证书等消息，**否则，服务器发送握手失败消息。**

那我们再看看Server Hello消息具体做了什么呢？

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211215111204.png)

第一个，服务器端会从Client Hello中说的所有加密套件中确定一种，这个加密套件包括了：后面客户端和服务器端之间通信使用的对称加密算法、以及生成摘要用的算法；第二，这里服务器也会生成一个随机值(用来创建加密密钥)。。

所以现在服务器端和客户端都有（Random1 + Random2）

### TLS握手的第一阶段总结

通过Client Hello和Server Hello之后，客户端和服务器端相互之间商定了以下内容：

- SSL版本
- 密钥交换、信息验证、加密算法
- 压缩方法
- 后面要用的两个用于生成对称加密密钥的随机数

### TLS建立的第二阶段 （服务器向客户端发送消息）

这个阶段一直是服务器在发消息，客户端在接受消息。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211215112616.png)

经过第一阶段，客户端和服务器端之间只是确定了要用的算法，然后还有两个随机数。我们知道服务器端的证书还没发给客户端呢。。。所以这里服务器端开始把证书发给客户端。。这个阶段具体有这几步：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211215113106.png)

第一次建立TLS握手必须要有证书，当然如果有会话Session id，当然不用握手。。这里服务器会把数字证书发给客户端，而**证书中包含了公钥，1.客户端用这个公钥就可以验证签名**或在**2.密钥交换的时候给消息加密**。

### TLS第三阶段

主要是客户端发送自己的证书给服务器端验证，当然这是可选步骤。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211215115417.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211215115459.png)

这一阶段主要是干客户端密钥交换，在第二阶段，客户端不是已经拿到了服务器端传过来的证书嘛，证书中包含了服务器端的公钥，上面公钥作用的第二点也说了，用于密钥交换的信息加密，什么意思呢？因为我到目前为止客户端和服务器端还没有商量好对称密钥呢。。。所以这里就是客户端传对称密钥给服务器端，客户端会根据之前两个随机数+商量好的对称加密算法，**生成一个对称密钥**，那我肯定不能生成了直接发给服务器端啊，那别人不都知道了，所以这里用上一步中服务器端给客户端传来的证书中的服务器端公钥来对这个对称密钥加密，然后再传给服务器，**这时候只有服务器可以解密，获取到对称密钥**。这里用的是RSA模式进行客户端密钥交换，当然还可以用Diff-Hellman密钥交换协议：（反正就是要么客户端直接生成预备主密码传给服务器端、要么公开一些参数，让服务器端自己生成）

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211215154032.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211215154255.png)





### TLS第四阶段 完成握手协议、建立SSL连接

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211229233911.png)





### 总结一下握手

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211215160606.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211215160841.png)





## 再来详细看看数字证书

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211215145109.png)

网站的证书就是用来标识确实是这个证书对应的网站，而不是别的仿冒网站。也就是说该服务端是合法的，即这个数字证书中的内容是合法的，**比如服务器的公钥**，讲这么多。。。简单点其实就是把服务器的公钥给客户端，然后通过验证证书来说明这个公钥确实是那个合法服务器端的。。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211215151700.png)

那再看看客户端怎么验证传过来的一个证书的合法性（包括确实是这个域名发过来的、而且没有被篡改、没有过期）呢？

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211215150251.png)

因为客户端，比如浏览器或者操作系统会内置很多CA机构的公钥，然后客户端收到一个证书，首先客户端会用对应的CA机构的公钥去对证书中数字签名解密，如果能够解密成功，说明确实是这个CA颁发的，此时得到一个hash值H2；但是现在不能保证内容是否被篡改，所以还得对版本、序列号、服务器端的公钥（整个蓝色部分）进行指定的Hash算法，得到一个Hash值H1，通过比较H1和H2来发现证书内容是否被篡改。

### 证书链

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211215151544.png)





### TLS记录协议

主要负责消息的压缩，加密，数据的认证

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211214180741.png)



![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20220315151503.png)



参考：

https://www.biaodianfu.com/https-ssl-tls.html

https://www.ruanyifeng.com/blog/2014/09/illustration-ssl.html