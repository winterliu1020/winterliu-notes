下面会介绍Java中的代理类Proxy和代理类中需要传递的参数InvocationHandler类。

Proxy类是所有生成的代理类的父类，**为什么生成的代理类需要去继承Proxy类呢**？因为代理类中没有InvocationHandler属性啊，没有这个属性我怎么实现后面的：在代理类重新生成的接口中的方法中调用this.h.invoke(this, m3, null)，this表示生成的代理类，h表示这个代理类的InvocationHandler对象；这样就会让代理类调用接口中的方法时都会走handler类中的invoke()方法，你再去**在invoke()方法中实现方法调用的增强和约束**。

代理类还会“实现”你定义的接口，因为我们想让代理类执行方法和被代理类执行方法对于用户来说是无感知的，比如你用代理类实例调用添加学生方法studentProxy.addStudent()，它和被代理类实例studentImpl.addStudent()是一样的，因为代理类和student类都实现了**同一个Student接口。** 这样我就可以通过 Student studentProxy = Proxy.newInstance(studentImpl.getClass().getClassLoader, studentImpl.getClass().getInterfaces(), studentInvocationHandler);拿到代理类。

> Proxy.newInstance()方法如何产生一个代理类会在下面说明。

然后studentProxy.addStudent()代理类中调用的接口的所有方法都会转向去调用代理类自动生成的对应同名方法，每个同名方法中都是去调用this.h.invoke(this, m3, null)方法，这里this是代理类实例、h是代理类指定的InvocationHandler类实例；再说invoke方法中三个参数：this还是代理类实例，第二个参数是调用的接口中的方法的Method实例，第三个参数是调用接口中的方法的参数。（所以说代理类会让你调用的所有方法都截断，然后去调用代理指定的handler类中的invoke()方法）

**为什么自动生成的代理类需要去implements接口呢？**

其实代理分为静态代理和动态代理，静态代理要自己去手动写代理类，需要对被代理类中的每一个方法都实现代理，所以很繁琐；所以提出动态代理（比如：jdk动态代理、cglib动态代理），首先我们想一下，所谓的代理模式，就是让一个代理类和被代理类关联起来，让用户用代理类去调用方法时是和直接用被代理类调用方法是无感知的，**那怎么让两个类关联起来呢？**一般有两种方式：一是一个类继承另外一个类，二是一个类和另外一个类共同实现同一个接口；这样用一个类调用方法就和用另外一个类调用方法一样，用户感知不到。

jdk动态代理用的是第二种实现同一个接口的方式，为什么呢？因为用jdk动态代理的话，我的代理类还得继承Proxy这个父类啊，所以没办法，jdk动态代理就采用实现同一个接口的方式让代理类和被代理类之间产生关联。而cglib动态代理是基于类的，它不要求某个类必须有接口，它的**代理类是被代理类的子类**，因为它不用继承Proxy类啊，所以cglib采用这种直接继承被代理类的方法；但是同样的，cglib也得有一个类似于jdk动态代理中的InvocationHandler类的东西来处理动态代理类调用方法，所以cglib用的是实现MethodInterceptor接口的方式，实现这个接口中intercept(Object obj, Method method, Object[] args, MethodProxy methodProxy)方法，其实和jdk代理中代理类指定的handler类实现InvocationHandler接口一样。

**jdk动态代理和cglib动态代理的优缺点：**

jdk动态代理在生成.class文件然后加载到内存中产生类对象这个过程相比cglib更快，但是jdk动态代理在调用方法时采用反射方式，相对于cglib直接继承被代理类来调用方法，采用反射调用方法更慢；而且从jdk1.6到1.8，jdk在不断优化，使得通过反射调用方法的方式也越来越快，而cglib有点停滞不前。

在spring中，如果被代理类有接口，那么默认采用jdk动态代理的方法，如果没有接口，才采用cglib实现动态代理。

**一些注意点：**

在new一个代理类的时候，就需要把具体的实例对象传给代理类的InvocationHandler对象的target属性，这样在进入到handler对象的invoke()方法中时，就可以直接执行method.invoke(target, args)，target就是方法的具体执行实例。

---

**参考的文章：**

第一块文章：

首先有静态代理，然后发现代理业务的复杂，产生了动态代理：动态代理又包括jdk动态代理，cglib动态代理，可以从下面这篇文章仔细看自动生成的代理类中生成的同名方法，看这些东西：this.h.invoke(this, m3, null);m3 = Class.forName("proxy.UserService").getMethod("addUser", new Class[0]);可以看到m3还是接口UserService的方法，而不是接口实现类的方法，噢，其实接口实现类中getMethod拿到的addUser方法的method对象和通过接口拿到的method是一样吧；不管你通过接口还是具体实现类拿，我要得只是这个方法的method实例，便于我后面利用反射调用这个方法而已。

https://juejin.cn/post/6844903811861970958#heading-8

第二块文章：

搞清代理类如何与被代理类之间产生关联，并调用方法之后，我想深入看看Java中的Proxy类如何产生我们的代理类的，主要通过Proxy.newInstance(classload, interfaces, handler);具体来说（参考下面这篇文章），jdk会做以下事情：

1. 获取被代理类上的所有接口列表；
2. 确定要生成的代理类的类名，默认为：com.sun.proxy.$ProxyXXX;
3. 根据需要实现的接口信息，在代码中动态创建 该代理Proxy类的字节码；
4. 用newInstance中传递的classload加载器将对应的字节码转换成对应的class对象；
5. 创建InvocationHandler实例handler，用来处理Proxy所有方法调用；
6. Proxy的class对象 以创建的handler对象为参数，实例化一个proxy对象。

https://www.cnblogs.com/jing99/p/7004769.html

下面这两篇文章讲解Proxy类及其如何产生代理类对象，主要涉及到：getProxyClass0(loader, intfs)方法、proxyClassCache.get(loader, interfaces)方法 （这是代理类的缓存策略）、supplier.get() 也就是java.lang.reflect.WeakCache.Factory#get方法、java.lang.reflect.Proxy.ProxyClassFactory#apply方法 这个方法中会生成代理类字节文件 然后加载代理类。

继续**深入 生成的代理类 和 InvocationHandler 实例的 invoke()方法的关系**，可以在sun.misc.ProxyGenerator.ProxyMethod#generateMethod方法中看到生成代理类中的方法时，会把调用handler的invoke()方法的语句 也就是this.h.invoke(...)写到生成的代理类的方法体中。

https://juejin.cn/post/6904069753409634312

https://juejin.cn/post/6934463255469359111#heading-14 （源码解析）



看了一些Proxy的实现细节之后，我想为什么会产生两种动态代理模式呢？一种基于接口、一个基于直接继承被代理类；可以看看**jdk动态代理和cglib动态代理的区别**，以及生成的两种代理类文件是什么样的：

https://juejin.cn/post/6844903641753583624



Spring中的AOP也用了动态代理模式，下面这篇文章的后面部分可以看看AOP用jdk和cglib两种动态代理实现的具体实例：

https://segmentfault.com/a/1190000037648064

---

其它的一些参考：

jdk的动态代理及为什么需要接口：

https://blog.csdn.net/zxysshgood/article/details/78684229

https://blog.csdn.net/H_X_P_/article/details/112217659



手动实现代理模式：

https://www.cnblogs.com/youzhibing/p/10464274.html

mybatis中mapper代理对象生成源码解析：https://www.cnblogs.com/youzhibing/p/10451680.html