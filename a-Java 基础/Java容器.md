![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/Java 基础/20201029172012.png)

### 1. Map.Entry<K, V>是Map声明的一个内部接口，这个接口是泛型接口，定义为Entry<K, V>。它表示Map中的一个实体（也就是一个key-value对)。

**Java1.8中HashMap**：可以看到1.8中里面的节点叫做Node，这个Node是一个内部类，它实现了Map.Entry<K, V>这个接口。因为1.8开始，当发生哈希冲突时，新放进来的节点会放到Node<K, V>[] tab这个数组的节点下面的链表上，如果链表长度过长，则会转化成红黑树。（这里的   Node<K, V>   是一个泛型类）。

> ### java发射机制中，class<T>是什么意思？
>
> Class<Integer> cla;与Class<?> cl;
> 前一个表示cla只能指向Integer这种类型，而后一个cl表示可以指向任意类型。
>
> cla = Integer.class 可以，但cla = Double.class就不可以。
> 但是cl = Integer.class 可以，cl = Double.class也可以 、
>
> 
>
> ?是通配符。
> 最好再去了解下泛型的概念，对这个理解起来比较好

1、我们往HashMap中put值的时候，值其实是存在Node里面的。key和value分别会赋值到Node的key和value中。

2、Node中还有一个字段叫 final int hash；存取这个Node的hash值。这个hash值是最终找数据的关键。

由以上可知，Node对象才是HashMap的核心，存取了数据以及数据的唯一编号hash。

```java
/**
     * Basic hash bin node, used for most entries.  (See below for
     * TreeNode subclass, and in LinkedHashMap for its Entry subclass.)
     */
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }
```

### 2. 一些总结

对于最上面那张容器类图，第一层是Iterator, Collection, Map三个接口，接口就是一种规范，它更加抽象，抽象类抽象程度次之，普通类抽象程度最低。也就是这三个接口最为Java容器中最抽象的东西，其实就是最顶层的规范，接口当中一般是一些方法的声明，变量（属性）按照接口的设计原则来说不应该放到接口当中的。

然后第二层是（List, Set, Queue还是三个接口） AbstractList, AbstractSet, AbstractMap, SortedMap这几个抽象类，在抽象类中已经把第一层对应的接口中定义的方法实现了（不单单是声明了），然后再到第三层的HashSet, ArrayList, HashMap, LinkedHashMap, TreeMap这几个普通的类，把具体的某几个特有的方法分别实现了，也就是抽象程度没有那么高了。

注意：

- HashMap, TreeMap, LinkedHashMap这三个类和HashSet, TreeSet, LinkedHashSet这三个类在一些操作方法、性能是完全一样的。HashSet是HashMap的一种特殊情况，把key-value去掉value就成了Set，本质上是同一种数据结构。
- 使用抽象容器类可以方便的定义类，而不用在每个类中都实现容器接口container 中的所有的方法。

```java
Queue<Integer> queue = new LinkedList<>();
queue.peek(); // 这里其实调用的是LinkedList类中的peek()方法
```

> 为什么用LinkedList来实现Queue?
>
> Java中Queue是一个接口，Queue这个接口继承至Collection接口，LinkedList采用双向链表，本身就有addFirst, getFirst, getLast等功能的需求，而队列Queue是只是特定功能的LinkedList（也就是说Queue按照功能来说其实是LinkedList的一个子集，所以没必要多写一个类），如下图可以看到从Java1.5开始其实LinkedList中已经为Queue operation提供了对应的方法了。
>
> LinkedList其实是实现了Deque（Double End Queue 双端队列）这个接口中的方法，而Deque这个接口继承了Queue这个接口，其实就是：Queue是单端的add, remove, peek, offer等操作，而Deque就是双端，addFirst, addLast, removeFirst...

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/Java 基础/20201029230148.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/Java 基础/20201029230959.png)

### 3.HashMap和LinkedHashMap

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/Java 基础/20201029210807.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/Java 基础/20201029211248.png)



