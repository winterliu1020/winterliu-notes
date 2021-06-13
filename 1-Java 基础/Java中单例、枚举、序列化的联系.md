开始，我想写一个单例，发现有饿汉式、懒汉式（或者加锁的懒汉式）、双重校验锁的方式、静态内部类的方式；上面四种方式，虽然在不断改进，避免了线程安全问题，实现了延迟加载，但是依旧不能抵住反射攻击和反序列化攻击。

然后，看到`Effective Java`中说实现单例最好的方式是用枚举，里面只放一个枚举项，这个枚举类也就是单例对应的类，利用枚举实现的底层机制避免了线程安全问题，也实现了延迟加载，最重要的枚举还可以抵住反射攻击和反序列化攻击。

> 这里简单说一下枚举的底层实现：编译器会让枚举类自动继承java.lang.Enum类，里面的枚举项其实是枚举类的实例（static final类型），而且只在第一次使用这个枚举类的时候会初始化所有枚举项实例（这里说明避免了线程安全问题，实现了延迟加载）。

> 再说一下普通的类要实现单例，怎么抵住反射攻击呢：只要在构造方法里面判断是否是第二次构造，是的话抛出异常。怎么实现反序列化攻击呢：只要在单例类中写一个public Object readResolve(){...这里是方法中要做到事} 方法，那么在反序列化的时候，也就是执行readObject(fileInputStream)的时候，这个方法中会去看你当前类中有没有一个叫做readResolve()的方法（注意这个方法的返回值必须是Object，否则反射不到这个方法），如果满足条件的话readObject()返回的就是readResolve()这个方法返回的obj。
>
> 如果单例类中你没有定义一个public Object readResolve()方法，那readObject()就会从流里面读取数据填充到一个新的单例类对象中，并返回这个新的单例类对象。这就违反了单例模式的原则：在JVM中只存在一个实例。
>
> // 以下内容参考：https://juejin.cn/post/6844904142402486285
>
> 这里详细说一下在反序列化的时候readObject()这个方法干了什么（记住这个方法是要返回反序列化得到的对象）：
>
> 1. 调用readObject0()，在这个方法里面用switch(tc) case看当前反序列化的是什么类型，这里是普通的OBJECT类型。
>
>    ```java
>    case TC_ENUM:
>        return checkResolve(readEnum(unshared));
>    
>    case TC_OBJECT:
>        return checkResolve(readOrdinaryObject(unshared));
>    ```
>
> 2. 调用checkResolve(readOrdinaryObject(unshared))：
>
>    其中checkResolve()参数是readOrdinaryObject()方法返回的obj对象，checkResolve主要是对这个对象进行检查，没问题就返回这个obj对象了。那么这个要返回的obj对象哪里new出来并且填充fileInputStream流中的数据的呢？
>
> 3. 这就看readOrdinaryObject()方法：在这个方法中obj先被new出来
>
>    ```java
>    Object obj;
>    try {
>        obj = desc.isInstantiable() ? desc.newInstance() : null;
>    } catch (Exception ex) {
>        throw (IOException) new InvalidClassException(
>            desc.forClass().getName(),
>            "unable to create instance").initCause(ex);
>    }
>    ```
>
>    这里desc.isInstantiable()一定是true，也就是说desc.newInstance()一定会new一个单例类实例（具体来说：在desc.newInstance()中通过`cons = getSerializableConstructor(cl);`这一行代码，用单例类的Class对象可以拿到单例类的构造器，从而new一个单例类实例）。
>
> 4. 拿到一个刚new出来的单例类实例还不够啊，我们现在做的是反序列化，也就是得把流中读取到的数据填充到new出来的单例类实例中。还是在readOrdinaryObject()方法中，obj被new完之后：
>
>    ```java
>    if (desc.isExternalizable()) {
>        readExternalData((Externalizable) obj, desc);
>    } else {
>        readSerialData(obj, desc); // 大多数会执行这一行，
>    }
>    ```
>
>    注意：被new完之后其实**并不是直接看**有没有定义public Object readResolve()这个方法，而是先执行readSerialData(obj, desc)这一行，将流中的数据填充到刚new出来的单例类实例中（这个desc是什么类型呢？它其实是一个ObjectStreamClass类，Java里面万物皆对象，其实是把流里面的filed抽象到这个ObjectStreamClass类中，这个desc里面就有流中的各个field的数据 就可以填充到obj对象了），详细内容在ObjectInputStream类中readSerialData()方法。
>
>    填充完数据之后才会调用desc.hasReadResolveMethod()看单例类中有没有定义readResolve()方法：
>
>    ```java
>    if (obj != null &&
>        handles.lookupException(passHandle) == null &&
>        desc.hasReadResolveMethod())
>    {
>        // 下面这一行是通过反射调用readResolve()方法，并把返回值放到rep
>        Object rep = desc.invokeReadResolve(obj);
>        if (unshared && rep.getClass().isArray()) {
>            rep = cloneArray(rep);
>        }
>        //。。。。
>    }
>    ```
>
>    desc.hasReadResolveMethod()怎么知道你有没有定义readResolve()方法呢？到这个方法里面：
>
>    ```java
>    boolean hasReadResolveMethod() {
>        requireInitialized();
>        return (readResolveMethod != null);
>    }
>    ```
>
>    其中readResolveMethod是Method类，那readResolveMethod什么时候初始化的呢：可以看到在ObjectStreamClass的构造方法里面（说明在desc被new出来的时候就给Method类型的readResolveMethod完成初始化），通过反射获得类名cl中有没有返回值是Object.class的readResolve()方法。
>
>    ```java
>    /**
>     * Creates local class descriptor representing given class.
>     */
>    private ObjectStreamClass(final Class<?> cl) {
>        readResolveMethod = getInheritableMethod(
>                            cl, "readResolve", null, Object.class);
>    ```
>
>    这里再说一下desc:它是ObjectStreamClass类型，它里面有你要写成的Object的Class对象，也有流中的field：
>
>    ```java
>    // 1. desc从哪里来：在readOrdinaryObject()方法中
>    ObjectStreamClass desc = readClassDesc(false);
>    // 2. 再看readClassDesc()方法中有：（如果当前是一个普通的Object进行反序列化）
>    case TC_CLASSDESC:
>                    descriptor = readNonProxyDesc(unshared);
>    // 3. 再看readNonProxyDesc(unshared)方法中有：
>    ObjectStreamClass desc = new ObjectStreamClass(); // 真面目终于出现了
>    // 然后下面是对desc的各项数据初始化赋值
>    desc.initNonProxy(readDesc, cl, resolveEx, readClassDesc(false));
>    // 到此，就有了填充了数据的desc了；至于initNonProxy方法的各项参数就不细说了:)
>    ```
>
>    看完有没有readResolve()方法之后，如果有的话那我们就拿到readResolve()方法返回的Object类型对象rep了，那现在有两个对象了，一个是之前正常从流中读取数据得到的obj对象，一个是刚从eadResolve()方法得到的rep对象，那最终readOrdinaryObject()会返回哪个对象呢？
>
>    ```java
>    if (rep != obj) {
>        // Filter the replacement object
>        if (rep != null) {
>            if (rep.getClass().isArray()) {
>                filterCheck(rep.getClass(), Array.getLength(rep));
>            } else {
>                filterCheck(rep.getClass(), -1);
>            }
>        }
>        handles.setObject(passHandle, obj = rep); // 看这里
>    }
>    ```
>
>    当rep和之前的obj不一样时，会执行上面这段代码，中间都是一些check，看这一行：handles.setObject(passHandle, obj = rep)，说明会把obj指向rep。
>
>    **最终结论**：说明当我们在单例类中定义了返回值时Object类型的readResolve()方法时，下面的readObject()会返回readResolve()中返回的Object对象。
>
>    ```java
>    SingletonModel singletonModel1 = (SingletonModel) objectInputStream.readObject();
>    ```

上面探究了一下普通类如何避免反序列化攻击（注意普通类还需要手动加implements Serializable接口，只要实现了这个接口，ObjectInputStream和ObjectOutputStream两个对流操作的类会自动对实现这个接口的类进行序列化和反序列化操作）。下面看看枚举底层机制是如何避免反射攻击和反序列化攻击的：

1.如何避免反射攻击：对于枚举来说，你可以拿到枚举类的构造器对象，但是当构造器对象newInstance()的时候会抛出异常。

```java
if ((clazz.getModifiers() & Modifier.ENUM) != 0)
    throw new IllegalArgumentException("Cannot reflectively create enum objects");
```

2.如何避免反序列化攻击，这就涉及到枚举的反序列化问题了：

对于普通的类，实现Serializable接口来实现序列化，通过反序列化会得到一个新的对象，如果用普通的实现了Serializable接口的类来用作单例类，就不再是单例了。为了保证枚举类型像Java规范中所说的：在JVM中，每一个枚举类对象及枚举项对象都是唯一的，在针对枚举类型的序列化和反序列化上，Java做了特殊的约定：

1. 枚举项的序列化不同于其它普通的Object的序列化。
2. 枚举项的序列化仅仅会对它的name属性进行序列化，枚举项的field values不会被序列化。
3. 序列化枚举项：ObjectOutputStream writes the value returned by the enum constant's name method
4. 反序列化枚举项：ObjectInputStream reads the constant name from the stream; 反序列化的时候通过java.lang.Enum的valueOf方法通过传入enumType和name属性值得到枚举项对象。

总结一点：在编译的时候，编译器会为每个枚举类自动生成一个values()方法，而且常量池中会有一个枚举项类型的数组 比如Season[]，当首次使用一个枚举类的时候，比如用到这个Season枚举类，会把SPRING, SUMMER...这些枚举项自动填充到Season数组，而values方法返回的就是这个Season[]数组。

在readObject0()中，如果反序列化的是Enum对象：

```java
case TC_ENUM:
    return checkResolve(readEnum(unshared));
```

进入readEnum()方法：它会读取流中的name属性值，再利用Enum.valueOf(cl, name)得到某一个枚举类中的枚举项对象。

```java
String name = readString(false);
Enum<?> result = null;
Class<?> cl = desc.forClass();
if (cl != null) {
    try {
        @SuppressWarnings("unchecked")
        Enum<?> en = Enum.valueOf((Class)cl, name);
        result = en;
    } catch (IllegalArgumentException ex) {
        throw (IOException) new InvalidObjectException(
            "enum constant " + name + " does not exist in " +
            cl).initCause(ex);
    }
    if (!unshared) {
        handles.setObject(enumHandle, result);
    }
}
```

我们再进入valueOf(class, name)：通过枚举类.enumConstantDirectory()可以得到一个Map<String, T>，key是枚举项的name，value是枚举项对象。那这个map里面的数据哪里来的呢？其实enumConstantDirectory()函数中就是调用编译器自动生成的value()方法拿到枚举项数组 比如Season[]，然后构造一个map。

```java
public static <T extends Enum<T>> T valueOf(Class<T> enumType,
                                            String name) {
    T result = enumType.enumConstantDirectory().get(name);
    if (result != null)
        return result;
    if (name == null)
        throw new NullPointerException("Name is null");
    throw new IllegalArgumentException(
        "No enum constant " + enumType.getCanonicalName() + "." + name);
}
```

参考：

[普通类为什么加了readResolve()方法就可以避免单例的反序列化攻击？](https://juejin.cn/post/6844904142402486285)

[枚举序列化](https://www.jianshu.com/p/d3d797c3cd45)