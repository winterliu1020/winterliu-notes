### 为什么要有泛型？

Java编程思想中说有很多原因，但是一个重要原因是为了创建容器类。C++中的模板就是泛型的思想，模板的精神：**参数化类型**。

泛型的本质就是参数化类型，也就是将原来具体的类型参数化。泛型的出现避免了强转操作，可以在编译器中完成类型的转化，避免运行时强制类型转换而出现ClassCastException类型转换异常。注意Java泛型也是一种语法糖，在JDK1.5时增加了泛型，很大程度上方便在集合上的使用。

### 泛型的使用

三种使用方式：泛型类、泛型方法、泛型接口。

1.泛型类：把泛型定义在类上

```java
// 注意泛型类型必须是引用类型
public class 类名 <泛型类型1,...> {
	
}
```

2.泛型方法：把泛型定义在方法上

```java
public <泛型类型> 返回类型 方法名 (泛型类型 变量名) {

}
```

```java
class Demo{  
  public <T> T fun(T t){   // 可以接收任意类型的数据  
   return t ;     // 直接把参数返回  
  }  
}
// 注意：当调用fun()方法时，根据传入的实际对象，编译器就会判断出类型形参T所代表的实际类型，所以在上面的fun函数中会根据传来的t这个形参的类型返回不同的数据类型。
public class GenericsDemo26{  
  public static void main(String args[]){  
    Demo d = new Demo() ; // 实例化Demo对象  
    String str = d.fun("汤姆") ; // 传递字符串  
    int i = d.fun(30) ;  // 传递数字，自动装箱  
    System.out.println(str) ; // 输出内容  
    System.out.println(i) ;  // 输出内容  
  }  
}
```

3.泛型接口：把泛型定义在接口

```java
public interface 接口名<泛型类型> {

}
```

```java
// 泛型接口
public interface Inter<T> {
	public abstract void show(T t);	
}
// 子类是泛型类
public class InterImpl<E> implements Inter<E> {
    @Override
    public void show(E t) {
        System.out.println(t);
    }
}
// 注意：这里左边可以用接口，但是右边new肯定是实例类了，比如常用List list = new LinkedList(); 这里的List其实就是一个接口
Inter<String> inter = new InterImpl<String>();
inter.show("hello");
```

源码中泛型的使用：

```java
//定义接口时指定了一个类型形参，该形参名为E
public interface List<E> extends Collection<E> {
   //在该接口里，E可以作为类型使用
   public E get(int index) {}
   public void add(E e) {} 
}

//定义类时指定了一个类型形参，该形参名为E
public class ArrayList<E> extends AbstractList<E> implements List<E> {
   //在该类里，E可以作为类型使用
   public void set(E e) {
   .......................
   }
}
```

### 泛型类派生子类

父类派生子类的时候不能再包含类型形参，需要传入具体的类型

错误：`public class A extends Container<K, V> {}`

正确：`public class A extends Container<Integer, String> {}`

也可以不指定具体的类型，这样系统就会把K, V形参当成Object类型处理：`public class A extends Container {}`

### 泛型构造器、高级通配符

。。。

### 泛型擦除

指编译器编译带类型说明的集合时会去掉类型信息。

```java
public class GenericTest {
    public static void main(String[] args) {
        new GenericTest().testType();
    }

    public void testType(){
        ArrayList<Integer> collection1 = new ArrayList<Integer>();
        ArrayList<String> collection2= new ArrayList<String>();
        
        System.out.println(collection1.getClass()==collection2.getClass());
        //两者class类型一样,即字节码一致
        
        System.out.println(collection2.getClass().getName());
        //class均为java.util.ArrayList,并无实际类型参数信息
    }
}

// 输出
true
java.util.ArrayList
```

分析：1.这是因为不管为泛型的类型形参传入哪一种类型实参，对于Java来说，它们依然被当成同一类处理，在内存中也只占用一块内存空间。从Java泛型的这一概念提出的目的来看，其**只是作用于代码编译阶段**，在编译过程中，对于正确检验泛型结果后，会将泛型的相关信息擦出，也就是说，成功编译过后的class文件中是不包含任何泛型信息的。泛型信息不会进入到运行时阶段。

2.在静态方法、静态初始化块或者静态变量的声明和初始化中不允许使用类型形参。由于系统中并不会真正的生成泛型类，所以instanceof运算符后不能使用泛型类。



---

泛型就是定义一种模板，例如`ArrayList<T>`,然后在代码中为用到的类创建对应的ArrayList<类型>;

```java
ArrayList<String> strList = new ArrayList<String>();
```

泛型实现了编写一次，万能匹配，通过编译器保证了类型安全。

### 泛型的向上转型

`ArrayList<T>`实现了`List<T>`接口，它可以向上转型为`List<T>`;

但是要注意：不能把`ArrayList<Integer>`向上转型为`ArrayList<Number>`或者`List<Number>`。

`ArrayList<Integer>`和`ArrayList<Number>`两者完全没有继承关系。

### 总结

1.泛型就是编写模板代码来适应任意类型；

2.泛型的好处是使用时不必对类型进行强制转换，它通过编译器对类型进行检查；

3.注意泛型的继承关系：可以把`ArrayList<Integer>`向上转型为`List<Integer>`(T不能变！)，但是不能转成`ArrayList<Number>`。

学习来源：

[廖雪峰](https://www.liaoxuefeng.com/wiki/1252599548343744/1265102638843296)

[掘金](https://juejin.im/post/6844903925666021389)

