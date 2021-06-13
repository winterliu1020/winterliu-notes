枚举代码：

```java
enum Season{
    aaa,
    bbb,
    ccc,
    ddd
}
```

编译之后的class文件用javap -v Season.class命令查看：

```java
警告: 二进制文件Season包含javaBasic.Season
Classfile /Users/liuwentao/githubRepo/Code/JavaSE_Practice/src/javaBasic/Season.class
  Last modified 2021-5-25; size 914 bytes
  MD5 checksum 18210985803665e62859c37166b03a17
  Compiled from "MyEnum.java"
final class javaBasic.Season extends java.lang.Enum<javaBasic.Season>
  minor version: 0
  major version: 52
  flags: ACC_FINAL, ACC_SUPER, ACC_ENUM
Constant pool:
   #1 = Fieldref           #4.#38         // javaBasic/Season.$VALUES:[LjavaBasic/Season;
   #2 = Methodref          #39.#40        // "[LjavaBasic/Season;".clone:()Ljava/lang/Object;
   #3 = Class              #23            // "[LjavaBasic/Season;"
   #4 = Class              #41            // javaBasic/Season
   #5 = Methodref          #16.#42        // java/lang/Enum.valueOf:(Ljava/lang/Class;Ljava/lang/String;)Ljava/lang/Enum;
   #6 = Methodref          #16.#43        // java/lang/Enum."<init>":(Ljava/lang/String;I)V
   #7 = String             #17            // aaa
   #8 = Methodref          #4.#43         // javaBasic/Season."<init>":(Ljava/lang/String;I)V
   #9 = Fieldref           #4.#44         // javaBasic/Season.aaa:LjavaBasic/Season;
  #10 = String             #19            // bbb
  #11 = Fieldref           #4.#45         // javaBasic/Season.bbb:LjavaBasic/Season;
  #12 = String             #20            // ccc
  #13 = Fieldref           #4.#46         // javaBasic/Season.ccc:LjavaBasic/Season;
  #14 = String             #21            // ddd
  #15 = Fieldref           #4.#47         // javaBasic/Season.ddd:LjavaBasic/Season;
  #16 = Class              #48            // java/lang/Enum
  #17 = Utf8               aaa
  #18 = Utf8               LjavaBasic/Season;
  #19 = Utf8               bbb
  #20 = Utf8               ccc
  #21 = Utf8               ddd
  #22 = Utf8               $VALUES
  #23 = Utf8               [LjavaBasic/Season;
  #24 = Utf8               values
  #25 = Utf8               ()[LjavaBasic/Season;
  #26 = Utf8               Code
  #27 = Utf8               LineNumberTable
  #28 = Utf8               valueOf
  #29 = Utf8               (Ljava/lang/String;)LjavaBasic/Season;
  #30 = Utf8               <init>
  #31 = Utf8               (Ljava/lang/String;I)V
  #32 = Utf8               Signature
  #33 = Utf8               ()V
  #34 = Utf8               <clinit>
  #35 = Utf8               Ljava/lang/Enum<LjavaBasic/Season;>;
  #36 = Utf8               SourceFile
  #37 = Utf8               MyEnum.java
  #38 = NameAndType        #22:#23        // $VALUES:[LjavaBasic/Season;
  #39 = Class              #23            // "[LjavaBasic/Season;"
  #40 = NameAndType        #49:#50        // clone:()Ljava/lang/Object;
  #41 = Utf8               javaBasic/Season
  #42 = NameAndType        #28:#51        // valueOf:(Ljava/lang/Class;Ljava/lang/String;)Ljava/lang/Enum;
  #43 = NameAndType        #30:#31        // "<init>":(Ljava/lang/String;I)V
  #44 = NameAndType        #17:#18        // aaa:LjavaBasic/Season;
  #45 = NameAndType        #19:#18        // bbb:LjavaBasic/Season;
  #46 = NameAndType        #20:#18        // ccc:LjavaBasic/Season;
  #47 = NameAndType        #21:#18        // ddd:LjavaBasic/Season;
  #48 = Utf8               java/lang/Enum
  #49 = Utf8               clone
  #50 = Utf8               ()Ljava/lang/Object;
  #51 = Utf8               (Ljava/lang/Class;Ljava/lang/String;)Ljava/lang/Enum;
{
  public static final javaBasic.Season aaa;
    descriptor: LjavaBasic/Season;
    flags: ACC_PUBLIC, ACC_STATIC, ACC_FINAL, ACC_ENUM

  public static final javaBasic.Season bbb;
    descriptor: LjavaBasic/Season;
    flags: ACC_PUBLIC, ACC_STATIC, ACC_FINAL, ACC_ENUM

  public static final javaBasic.Season ccc;
    descriptor: LjavaBasic/Season;
    flags: ACC_PUBLIC, ACC_STATIC, ACC_FINAL, ACC_ENUM

  public static final javaBasic.Season ddd;
    descriptor: LjavaBasic/Season;
    flags: ACC_PUBLIC, ACC_STATIC, ACC_FINAL, ACC_ENUM

  public static javaBasic.Season[] values();
    descriptor: ()[LjavaBasic/Season;
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=1, locals=0, args_size=0
         0: getstatic     #1                  // Field $VALUES:[LjavaBasic/Season;
         3: invokevirtual #2                  // Method "[LjavaBasic/Season;".clone:()Ljava/lang/Object;
         6: checkcast     #3                  // class "[LjavaBasic/Season;"
         9: areturn
      LineNumberTable:
        line 9: 0

  public static javaBasic.Season valueOf(java.lang.String);
    descriptor: (Ljava/lang/String;)LjavaBasic/Season;
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=1, args_size=1
         0: ldc           #4                  // class javaBasic/Season
         2: aload_0
         3: invokestatic  #5                  // Method java/lang/Enum.valueOf:(Ljava/lang/Class;Ljava/lang/String;)Ljava/lang/Enum;
         6: checkcast     #4                  // class javaBasic/Season
         9: areturn
      LineNumberTable:
        line 9: 0

  static {};
    descriptor: ()V
    flags: ACC_STATIC
    Code:
      stack=4, locals=0, args_size=0
         0: new           #4                  // class javaBasic/Season
         3: dup
         4: ldc           #7                  // String aaa
         6: iconst_0
         7: invokespecial #8                  // Method "<init>":(Ljava/lang/String;I)V
        10: putstatic     #9                  // Field aaa:LjavaBasic/Season;
        13: new           #4                  // class javaBasic/Season
        16: dup
        17: ldc           #10                 // String bbb
        19: iconst_1
        20: invokespecial #8                  // Method "<init>":(Ljava/lang/String;I)V
        23: putstatic     #11                 // Field bbb:LjavaBasic/Season;
        26: new           #4                  // class javaBasic/Season
        29: dup
        30: ldc           #12                 // String ccc
        32: iconst_2
        33: invokespecial #8                  // Method "<init>":(Ljava/lang/String;I)V
        36: putstatic     #13                 // Field ccc:LjavaBasic/Season;
        39: new           #4                  // class javaBasic/Season
        42: dup
        43: ldc           #14                 // String ddd
        45: iconst_3
        46: invokespecial #8                  // Method "<init>":(Ljava/lang/String;I)V
        49: putstatic     #15                 // Field ddd:LjavaBasic/Season;
        52: iconst_4
        53: anewarray     #4                  // class javaBasic/Season
        56: dup
        57: iconst_0
        58: getstatic     #9                  // Field aaa:LjavaBasic/Season;
        61: aastore
        62: dup
        63: iconst_1
        64: getstatic     #11                 // Field bbb:LjavaBasic/Season;
        67: aastore
        68: dup
        69: iconst_2
        70: getstatic     #13                 // Field ccc:LjavaBasic/Season;
        73: aastore
        74: dup
        75: iconst_3
        76: getstatic     #15                 // Field ddd:LjavaBasic/Season;
        79: aastore
        80: putstatic     #1                  // Field $VALUES:[LjavaBasic/Season;
        83: return
      LineNumberTable:
        line 10: 0
        line 11: 13
        line 12: 26
        line 13: 39
        line 9: 52
}
Signature: #35                          // Ljava/lang/Enum<LjavaBasic/Season;>;
SourceFile: "MyEnum.java"
```

得到以下总结：

1. 枚举类Season默认继承了java.lang.Enum类，字节码格式并没有禁止继承java.lang.Enum，但是我们写代码的时候会发现不能extends java.lang.Enum类，说明是编译器不允许继承Enum类的。也就是说编译器会自动帮我们给枚举类继承java.lang.Enum类。

2. 再说枚举里面的值，其实枚举就是一个类，而里面的值就是这个类的对象，比如枚举类Season，可以直接通过Season.SPRING拿到这个枚举值，底层实现其实就是Season枚举类里面有四个static final的Season对象（静态域对象）。

3. 而且我们可以调用values()方法得到定义的枚举量的数组，那这个values()方法又是哪来的呢？我们在枚举类中并没有声明values()方法，其实也是编译器自动生成的，它会返回常量池中的Season[]变量，而Season变量就存储了所有的枚举值；那枚举值又是什么时候放到Season[]数组中的呢？其实这个枚举类有一块static代码区域（随着类加载的时候就会执行），在这块区域中会取枚举类的所有静态域（也就是SPRING, SUMMER...）对象放到Season[]数组中，即完成了这个数组的初始化。

4. 再看Ordinal()方法，因为每个枚举类都会自动继承自java.lang.Enum类，Ordinal()方法是Enum类中的方法，会返回每个枚举值的序号ordinal，其实每一个加入到枚举类中值都会有name和ordinal属性（这两个属性其实是Enum类中的，枚举类通过继承Enum类得到的）。

5. 枚举类也支持方法的重写，枚举类的字节码也是普通的类，但是枚举类是加了final修饰的。枚举类中的枚举值其实是枚举类的子类，这样我们就可以在枚举值这个类中重写枚举类中的方法了。其实在编译Season的时候，会把aaa, bbb...这些编译成Season的内部类，这些内部类extends了Season类，并且重写了fun1()方法。

   ```java
   enum Season{
       aaa{
           @Override
           public void fun1() {
               return;
           }
       },
       bbb,
       ccc,
       ddd;
       public void fun1() {
           return;
       }
   }
   ```

   