### 1.接口interface：[参考来源](https://blog.csdn.net/ameyume/article/details/6189749)

在interface里面的变量都是public static final 的。所以你可以这样写：
public static final int i=10;
或则
int i=10；（可以省略掉一部分）

注意在声明的时候要给变量赋予初值

**解释：**

首先你要弄清接口的含义.接口就是提供一种统一的’协议’,而接口中的属性也属于’协议’中的成员.它们是公共的,静态的,最终的常量.相当于全局常量.
抽象类是不’完全’的类,相当于是接口和具体类的一个中间层.即满足接口的抽象,也满足具体的实现.

如果接口可以定义变量，但是接口中的方法又都是抽象的，在接口中无法通过行为来修改属性。有的人会说了，没有关系，可以通过实现接口的对象的行为来修改接口中的属性。这当然没有问题，但是考虑这样的情况。如果接口A中有一个public访问权限的静态变量a。按照java的语义，我们可以不通过实现接口的对象来访问变量a，通过A.a = xxx;就可以改变接口中的变量a的值了。正如抽象类中是可以这样做的，那么实现接口A的所有对象也都会自动拥有这一改变后的a的值了，也就是说一个地方改变了a，所有这些对象中a的值也都跟着变了。这和抽象类有什么区别呢，怎么体现接口更高的抽象级别呢，怎么体现接口提供的统一的协议呢，那还要接口这种抽象来做什么呢？所以接口中不能出现变量，如果有变量，就和接口提供的统一的抽象这种思想是抵触的。所以接口中的属性必然是常量，只能读不能改，这样才能为实现接口的对象提供一个统一的属性。

通俗的讲，你认为是要变化的东西，就放在你自己的实现中，不能放在接口中去，接口只是对一类事物的属性和行为更高层次的抽象。对修改关闭，对扩展（不同的实现implements）开放，接口是对开闭原则的一种体现。



### 2.接口放在另一个接口（或者类）中

由于嵌套接口不能直接访问，使用它们的主要目的是通过将相关接口（或相关接口和类）分组在一起来解析命名空间。This way, we can only call the nested interface by using outer class or outer interface name followed by dot( . ), followed by the interface name.所以，你要访问一个接口中的接口，只能通过  外部接口.内部接口  的方式，比如Map.Entry;

```java
// 接口放在接口中
interface MyInterfaceA{  
    void display();  
    interface MyInterfaceB{  
        void myMethod();  
    }  
}  
      
class NestedInterfaceDemo1 
    implements MyInterfaceA.MyInterfaceB{  
     public void myMethod(){
         System.out.println("Nested interface method");
     }  
      
     public static void main(String args[]){  
         MyInterfaceA.MyInterfaceB obj=
                 new NestedInterfaceDemo1(); 
      obj.myMethod();  
     }  
}
```

```java
// 
class MyClass{  
    interface MyInterfaceB{  
        void myMethod();  
    }
}  
    
class NestedInterfaceDemo2 implements MyClass.MyInterfaceB{  
     public void myMethod(){
         System.out.println("Nested interface method");
     }  
    
     public static void main(String args[]){  
        MyClass.MyInterfaceB obj=
               new NestedInterfaceDemo2();  
        obj.myMethod();  
     }  
}
```

### 3.接口可以继承接口吗，抽象类可以继承接口吗，抽象类可以继承实体类吗？

> 1、接口可以继承接口，抽象类不可以继承接口，但可以实现接口。
>
> 2、抽象类可以继承实体类。抽象类可以实现(implements)接口，抽象类可以继承实体类，但前提是实体类必须有明确的构造函数。
>
> 3、抽象类可以继承实体类，就是因为抽象类可以有继承性和有方法。
>
> 4、一个接口可以继承多个接口. interface C extends A, B {}是可以的. 一个类可以实现多个接口: class D implements A,B,C{} 但是一个类只能继承一个类,不能继承多个类 class B extends A{} 在继承类的同时,也可以继承接口: class E extends D implements A,B,C{} 这也正是选择用接口而不是抽象类的原因。



