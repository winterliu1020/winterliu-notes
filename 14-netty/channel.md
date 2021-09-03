在bootstrap.channel(NioServerSocket.class)或者serverBootstrap.channel(NioServerSocket.class)时，只是设置了通道类型，并没有初始化channel实例，那么初始化channel实例是在什么时候呢？

- 客户端：在执行bootstrap.connect()函数的时候
- 服务器端：在执行serverBootstrap.bing()函数的时候

客户端执行bootstrap.channel(NioServerSocket.class)时，只是设置Bootstrap的channelFactory属性为一个ReflectiveChannelFactory对象，而这个ReflectiveChannelFactory对象可以用于创建你传进去的channelClass对应的Channel实例。

> 关于ChannelFactory类，bootstrap类中有一个ChannelFactory属性，用于反射生成传入bootstrap.channel(xxxChannel.class)中的对应的Channel实例，参考：https://skyao.gitbooks.io/learning-netty/content/channel/channel_factory.html
>
> 疑问：这里生成的channel实例是和bootstrap关联的吗？等后面讨论ChannelPromise的时候看看。。

> 以下探究netty实例化一个NioSocketChannel对象底层实现，底层竟然实例化的还是java.nio包中SocketChannel对象，NioSocketChannel类只是多了一个config内部类，存储channel的配置参数而已，参考：https://www.javadoop.com/post/netty-part-2

```java
// AbstractBootstrap
public B channel(Class<? extends C> channelClass) {
    if (channelClass == null) {
        throw new NullPointerException("channelClass");
    }
    // 这个channelFactory是一个方法，用于设置所在的Bootstrap中的channelFactory属性为:new ReflectiveChannelFactory<C>(channelClass)
    return channelFactory(new ReflectiveChannelFactory<C>(channelClass));
}
```

那ReflectiveChannelFactory是个什么类呢？它是一个反射channel工厂，其实就是一个channel工厂类，用于帮助创建channel实例，只不过是通过反射的方式创建，所以叫反射channel工厂，我们传入要创建的channel的class对象就可以构建实例了，如下：

```java
package io.netty.channel;

public class ReflectiveChannelFactory<T extends Channel> implements ChannelFactory<T> {
    // 内部保存一个Class
    private final Class<? extends T> clazz;

    // 构造函数传入
    public ReflectiveChannelFactory(Class<? extends T> clazz) {
        if (clazz == null) {
            throw new NullPointerException("clazz");
        }
        this.clazz = clazz;
    }

    @Override
    public T newChannel() {
        try {
            // 简单通过Class的newInstance()方法创建实例
            // 这也要求传入的Channel Class必须有默认构造函数
            return clazz.newInstance();
        } catch (Throwable t) {
            throw new ChannelException("Unable to create Channel from class " + clazz, t);
        }
    }
    ......
}
```

---

以上可以看到只是设置了Bootstrap对应的channel类型，还没有为Bootstrap实例化一个传入类型的channel实例，对于客户端来说，当connect()函数调用时，才会实例化channel实例，我们来看connect()函数：

```java
// Bootstrap
public ChannelFuture connect(String inetHost, int inetPort) {
    return connect(InetSocketAddress.createUnresolved(inetHost, inetPort));
}
// 调用这个方法
public ChannelFuture connect(SocketAddress remoteAddress) {
    if (remoteAddress == null) {
        throw new NullPointerException("remoteAddress");
    // validate 只是校验一下各个参数是不是正确设置了
    validate();
    return doResolveAndConnect(remoteAddress, config.localAddress());
}
    

// 真正要做的事情在这里！
private ChannelFuture doResolveAndConnect(final SocketAddress remoteAddress, final SocketAddress localAddress) {
    final ChannelFuture regFuture = initAndRegister();
    final Channel channel = regFuture.channel();
    ......
}
    
// 看一下initAndRegister()方法
final ChannelFuture initAndRegister() {
    Channel channel = null;
    try {
        // 前面我们说过，这里会进行 Channel 的实例化；比如我们一般是实例化一个NioSocketChannel
        channel = channelFactory.newChannel();
        init(channel);
    } catch (Throwable t) {
        ...
    }
    ...
    return regFuture;
}
```

我们知道netty中的NioSocketChannel其实是对java nio包中的SocketChannel的再次封装，那么在netty中实例化一个NioSocketChannel，发生了什么呢：

我们看NioSocketChannel的构造方法：

```java
public NioSocketChannel() {
    // SelectorProvider 实例用于创建 JDK 的 SocketChannel 实例
    this(DEFAULT_SELECTOR_PROVIDER); // DEFAULT_SELECTOR_PROVIDER会拿到一个SelectorProvider实例，然后这个this方法其实会调用下面这个构造方法
}

public NioSocketChannel(SelectorProvider provider) {
    // 看这里，newSocket(provider) 方法会创建 JDK 的 SocketChannel
    this(newSocket(provider));
}
```

所以我们主要看newSocket(provider)方法：

```java
// 可以看到在provider.openSocketChannel()中其实创建的是java.nio.channels.SocketChannel实例
private static SocketChannel newSocket(SelectorProvider provider) {
    try {
        // 创建 SocketChannel 实例
        return provider.openSocketChannel();
    } catch (IOException e) {
        throw new ChannelException("Failed to open a socket.", e);
    }
}
```

总结：所以说NioSocketChannel在实例化的过程中会先实例化java.nio中的SocketChannel对象，NioServerSocketChannel也一样。

---

再继续看构造方法：

```java
// 刚才newSocket()方法返回java.nio.channels.SocketChannel对象后，会接着调用下面的构造方法：
public NioSocketChannel(java.nio.channels.SocketChannel socket) {
    this((Channel)null, socket);
}

// 所以整个New NioSocketChannel()构造方法最终会执行到这个真正的构造方法：
// 第二行代码：实例化NioSocketChannel对象中的NioSocketChannelConfig属性，这个config用于保存channel的配置信息
// 第一行代码：调用父类构造器（往下看）
public NioSocketChannel(Channel parent, java.nio.channels.SocketChannel socket) {
    super(parent, socket);
    this.config = new NioSocketChannel.NioSocketChannelConfig(this, socket.socket());
}
```

看看调用父类构造器中发生了什么：1.设置属性，也就是设置SelectableChannel、以及关心哪种事件  2.还设置了channel的非阻塞模式

```java
protected AbstractNioByteChannel(Channel parent, SelectableChannel ch) { // 说明：上面传来的SocketChannel是SelectableChannel这个抽象类的一个实现子类
    // 毫无疑问，客户端关心的是 OP_READ 事件，等待读取服务端返回数据；socketChannel.read()
    super(parent, ch, SelectionKey.OP_READ);
}

// 然后是到这里
protected AbstractNioChannel(Channel parent, SelectableChannel ch, int readInterestOp) {
    super(parent);
    this.ch = ch;
    // 我们看到这里只是保存了 SelectionKey.OP_READ 这个信息，在后面的时候会用到
    this.readInterestOp = readInterestOp;
    try {
        // ******设置 channel 的非阻塞模式******
        ch.configureBlocking(false);
    } catch (IOException e) {
        ......
    }
}
```

NioServerSocketChannel的构造方法类似，也是设置非阻塞，设置服务器端关心的SelectionKey.OP_ACCEPT 事件。

总结：其实实例化netty中的NioSocketChannel时，主要就是实例化了jdk层的SocketChannel对象，然后设置该SocketChannel通道为非阻塞模式。NioServerSocketChannel同理。









Bootstrap中的init()函数：

```java
void init(Channel channel) {
    ChannelPipeline p = channel.pipeline();
    p.addLast(new ChannelHandler[]{this.config.handler()});
    setChannelOptions(channel, (Entry[])this.options0().entrySet().toArray(newOptionArray(0)), logger);
    setAttributes(channel, (Entry[])this.attrs0().entrySet().toArray(newAttrArray(0)));
}
```

ServerBootstrap中的init()函数：

```java
void init(Channel channel) {
    setChannelOptions(channel, (Entry[])this.options0().entrySet().toArray(newOptionArray(0)), logger);
    setAttributes(channel, (Entry[])this.attrs0().entrySet().toArray(newAttrArray(0)));
    ChannelPipeline p = channel.pipeline();
    final EventLoopGroup currentChildGroup = this.childGroup;
    final ChannelHandler currentChildHandler = this.childHandler;
    final Entry<ChannelOption<?>, Object>[] currentChildOptions = (Entry[])this.childOptions.entrySet().toArray(newOptionArray(0));
    final Entry<AttributeKey<?>, Object>[] currentChildAttrs = (Entry[])this.childAttrs.entrySet().toArray(newAttrArray(0));
    p.addLast(new ChannelHandler[]{new ChannelInitializer<Channel>() {
        public void initChannel(final Channel ch) {
            final ChannelPipeline pipeline = ch.pipeline();
            ChannelHandler handler = ServerBootstrap.this.config.handler();
            if (handler != null) {
                pipeline.addLast(new ChannelHandler[]{handler});
            }

            ch.eventLoop().execute(new Runnable() {
                public void run() {
                    pipeline.addLast(new ChannelHandler[]{new ServerBootstrap.ServerBootstrapAcceptor(ch, currentChildGroup, currentChildHandler, currentChildOptions, currentChildAttrs)});
                }
            });
        }
    }});
}
```