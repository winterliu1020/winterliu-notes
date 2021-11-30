昨天看hashmap中putval函数看到`transient`关键字，然后发现transient关键字和Java中的序列化（Serilizable接口）有关系。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/Java%20%E5%9F%BA%E7%A1%80/20201203102401.png)

### 关于序列化和transient

Java中一个类实现了Serilizable接口，那么这个类的所有对象的属性和方法都可以被自动序列化，我们可以不用关注序列化内部过程。

但是有时候我们会遇到这样的需求：某个类的有些属性需要序列化、其它的属性不需要被序列化（比如一些密码等敏感信息），我们不希望这些敏感信息在网络操作（主要涉及序列化操作，本地序列化缓存也适用）中被传输。那么这种变量就可以加transient关键字。

**所以transient关键字就是可以让变量不会被序列化，也就是不会被持久化到磁盘中，让某个字段的生命周期仅存在于调用者的内存中。**

比如：

```java
read before Serializable: 
username: Alexia
password: 123456

read after Serializable: 
username: Alexia
password: null
```

将User对象序列化之后写入到文件中，但是password属性加了transient关键字，那么它就不能被序列化，所以再次从文件中读取对象进行反序列化后，对应的password就是null。（password只在内存中存在，不能写入到文件）

### transient使用小结

1.变量被transient修饰，该变量不能被序列化，不再是对象持久化的一部分，该变量内容在序列化后无法获得访问。

2.transient只能修饰成员变量，不能修饰方法和类。也不能修饰本地变量（就是方法中的形参和方法体中声明的参数）。

3.静态变量（也就是类变量，加了static的）不管是否被transient修饰，均不能被序列化。反序列化后类中static型变量username的值为当前JVM中对应static变量的值，这个值是JVM中的不是反序列化得出的。

所以从上面三点来看，能够序列化的变量一般是指new出一个对象，这个对象所拥有的那些属性。

### HashMap中如何使用transient关键字？

首先看一下hashmap中存在的问题，然后再看如何用transient关键字来解决这个问题。

hashmap存储键值对，它的存储其实是依赖于hashcode()方法的，但是当我们把一个hashmap进行序列化/反序列化的时候，我们本身是不会保存计算出来的hashcode值的，而计算hashcode值调用的hashcode()方法是Native方法，也就是说它和底层实现相关，不同虚拟机可能有不同的HashCode算法，比如同一个key值在虚拟机A上的hashcode值为1，在虚拟机B上的hashcode值为2，在虚拟机C上的hashcode值为3，这样就失去了Java的跨平台性。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/Java%20%E5%9F%BA%E7%A1%80/20201203155411.png)

**总结**：其实就是利用不同虚拟机计算的HashCode值不同，如果采用Java自带的序列化，反序列化方法（ObjectInputStream, ObjectOutputStream），那么在两台不同的虚拟机上虽然hashmap里面的entry的内存分布一模一样，但是就是这个一模一样带来了问题，因为到新的机器上，比如key="abc"，然后计算得到的是一个新的hashcode值，那么通过这个hashcode值去get对应的value肯定不是一样的啊。所以

所以HashMap类重写了序列化，反序列化（hashmap类中反序列化其实是重新hash，用字节流中的key, value数据重构一个新的hashmap，就是将所有key, value键值对一个一个执行putValue方法放到新的hashmap中，这样你通过key=“abc"去get值，肯定可以拿到对应的值了 ）的方法，也就是writeObject, readObject。

`HashMap`仍然可以像通常那样处理非transient字段，但是它们一个接一个地在字节数组的末尾写入存储的键值对。在反序列化时，它会通过默认的反序列化过程处理非transient变量，然后逐个读取键值对。**对于每个键，哈希和索引再次计算并插入到表中的正确位置，**以便可以再次检索它而不会出现任何错误。 

> 为什么hashmap要重写序列化和反序列化方法：
>
> 下文是摘自《Effective Java》：
>
> *For example, consider the case of a hash table. The physical representation is a sequence of hash buckets containing key-value entries. The bucket that an entry resides in is a function of the hash code of its key, which is not, in general, guaranteed to be the same from JVM implementation to JVM implementation. In fact, it isn’t even guaranteed to be the same from run to run. Therefore, accepting the default serialized form for a hash table would constitute a serious bug. Serializing and deserializing the hash table could yield an object whose invariants were seriously corrupt.*

writeObject:

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/Java%20%E5%9F%BA%E7%A1%80/20201203160031.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211130175046.png)

readObject:

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/Java%20%E5%9F%BA%E7%A1%80/20201203155957.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/Java%20%E5%9F%BA%E7%A1%80/20201203160253.png)



参考：

[图解HashMap](https://www.cnblogs.com/xrq730/p/5030920.html)

[Java transient关键字使用示例](https://www.jianshu.com/p/2911e5946d5c)



---

### 以下是effective Java第二版十一章关于序列化的内容

对象序列化API，提供了一个框架，用来将对象编码成字节流（序列化），并从字节流编码中重新构建对象（反序列化）。

一旦对象被序列化之后，它的编码就可以从一台正在运行的虚拟机被传递到另一台虚拟机上，或者被存储到磁盘上，供以后反序列化时用。

序列化技术为远程通信提供了标准的线路级对象表示法，也为JavaBeans组件结构提供了标准的持久化数据格式。

### 关于该不该实现序列化接口

序列化接口提供了一些好处：如果一个类将要加入到某个框架中，并且该框架依赖于序列化来实现`对象传输`或者`持久化`,那对于这个类来说，实现Serializable接口就很有必要。

虽然只要在某个类的声明中加入“implements Serializable”就可以让这个类的实例被序列化（加了这个只是说明这个类可以序列化，序列化和反序列化真正用的是Java自带的ObjectOutputStream.putFields和ObjectInputStream.readFields方法，或者你也可以自己写writeObject, readObject方法来序列化，反序列化，像HashMap一样），但是实现这个接口付出的最大的代价是，一旦一个类被发布，就大大降低了“改变这个类的实现”的灵活性。

这里其实就像项目当中使用到Entity层，因为每一个Entity类的属性都会与数据库中的字段对应，但是一旦项目发布，你就不能随便改（增加或者删除，修改）Entity层中的属性了（其实序列化的作用之一就是容易持久化到数据库），灵活性降低。

### 关于序列版本ID

序列化会使类的演变受到限制，也就是每一个实现序列化接口的类会有一个唯一的与这个类对应的序列版本UID（其实有点像文件的md5码），如果你没有在一个名为serialVersionUID的私有静态final的long域中显式的指定该标识号，系统会自动根据（该类的名称、所实现接口的名称、所有的公有的和受到保护的成员的名称）并调用复杂运算来在运行时产生该标识号。

UID其实就是一个号码，对应某个类的状态，当该类发生变化那么通过计算产生的UID也会变化，所以如果你没有声明一个显式的序列版本UID，兼容性将会遭到破坏，在运行时导致InvalidClassException异常。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/Java%20%E5%9F%BA%E7%A1%80/20201203170515.png)

















