### Java.lang(language) package

> Provides classes that are fundamental to the design of the Java programming language. The most important classes are `Object`, which is the root of the class hierarchy, and `Class`, instances of which represent classes at run time.

里面有最重要的类`Object类`。还包含了基本数据类型的包装类。Math类、Void类、String类、StringBuilder、StringBuffer类。还有类加载（Classes ClassLoader类）、进程类（Process）、ProcessBuilder类、Runtime类、SecurityManager类、还有 System provide "system operations"（比如System类）...

### Void和void

void是一个关键字，Void是一种类型。可以看到Void类型不可以继承与实例化。Void如果作为函数的返回结果则该函数只能返回null。泛型出现之前，Void一般用于反射之中。Void也用于无值的Map中，例如Map<T, Void>这样map将和`Set<T>`有一样的功能。

```java
public final
class Void {

    /**
     * The {@code Class} object representing the pseudo-type corresponding to
     * the keyword {@code void}.
     */
    @SuppressWarnings("unchecked")
    public static final Class<Void> TYPE = (Class<Void>) Class.getPrimitiveClass("void");

    /*
     * The Void class cannot be instantiated.
     */
    private Void() {}
}
```

### 八种基本数据类型

1.整型（byte short int long）；字节数（1，2，4，8）

2.浮点型（单精度float，双精度double）；字节数（4， 8）

3.逻辑性（boolean）（This data type represents one bit of information, but its "size" isn't something that's precisely defined.）

4.字符型（char）；字节数（1）

### 八种基本类型的包装类和常量池

其中Byte,Short,Integer,Long,Character,Boolean这五种包装类（注意包装类都加了final，不可被继承）默认创建了数值范围在[-128, 127]的缓存数据，在这个范围之外的才会创建新的对象。注意浮点型float和double的包装类没有实现常量池技术。具体实现比如IntegerCache:可以看到这里的high是可以配置的，创建的缓存对象会放在cache[]数组中。

```java
private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
    }
```

### Number类

它是java.lang下面的一个抽象类，提供了将包装类型拆箱成基本类型的方法，八个包装类都继承了Number这个抽象类。

```java
package java.lang;

public abstract class Number implements java.io.Serializable {

    public abstract int intValue();

    public abstract long longValue();

    public abstract float floatValue();

    public abstract double doubleValue();

    public byte byteValue() {
        return (byte)intValue();
    }

    public short shortValue() {
        return (short)intValue();
    }

    private static final long serialVersionUID = -8742448824652078965L;
}
```

**怎么看源码**：一个类中，一般最上面是static静态方法，也就是这个类的方法（通过类名.方法直接调用），然后是声明一些变量（属性），然后是构造方法，接下来是一些普通方法（包括重写的方法）。

在Float.java中发现了：

```java
public int compareTo(Float anotherFloat) {
    return Float.compare(value, anotherFloat.value);
}

public static int compare(float f1, float f2) {
    if (f1 < f2)
        return -1;           // Neither val is NaN, thisVal is smaller
    if (f1 > f2)
        return 1;            // Neither val is NaN, thisVal is larger

    // Cannot use floatToRawIntBits because of possibility of NaNs.
    int thisBits    = Float.floatToIntBits(f1);
    int anotherBits = Float.floatToIntBits(f2);

    return (thisBits == anotherBits ?  0 : // Values are equal
            (thisBits < anotherBits ? -1 : // (-0.0, 0.0) or (!NaN, NaN)
             1));                          // (0.0, -0.0) or (NaN, !NaN)
}

// 还有sum, max, min函数，不过它们和上面的几个函数一样都是static方法，所以可以通过Float.sum(float1, float2)来使用；同理Integer，还有其它几种包装类中肯定也有这些静态方法，只不过参数的数据类型不一样而已
```

