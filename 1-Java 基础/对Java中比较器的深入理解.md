Compare, Comparator, Comparable, CompareTo, 分得清楚嘛。。。。

中间两个都是接口，Comparable接口是给一些希望自己可以比较的类去实现的，比如Date类，String类，还有八大基本类型的包装类，都默认帮我们实现了Comparable接口，这个接口中只有compareTo方法，当然如果你自定义的类希望可以进行比较，也可以实现这个接口，比如 Class Animal implements Comparable<Animal>{实现compareTo(Animal)方法}，然后后面后面就可以用animal.compareTo(anothAnimal);

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211209214719.png)

但是有没有想过这是一种侵入性的方法，比如要这个类去实现这个接口。可以看到Comparable接口在java.lang中，它从1.2就有了，当然Comparator也是1.2就有，但是你可以看到Comparator是在java.util，它是一个工具类，而前者看成是Java语言的一些东西。。。

插一个话题，实现了Comparable接口后，就可以用Arrays.sort(Object[])或者Collections.sort(List<T>)来对一个序列进行排序，注意Arrays.sort中传的是八大基本数据类型的数组或者Object的数组，Collections.sort传的是List<T>;

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211209221206.png)

### Comparator

我们再来看看非侵入性的，也就是说如果你比较的对象没有实现Comparable接口，那你单纯放到Arrays.sort或者Collection.sort中是无法排序的，因为这两个排序最终还是用的比较对象自身实现的Comparable接口中的compareTo方法。如下图：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211209220826.png)

所以可以用java.util中的外部比较器Comparator接口，你给Arrays.sort()或者Collection.sort()传comparator实现类对象，sort方法时就会判断传的比较器是否有效，有效的话就会**优先**用这个传的比较器。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211209221206.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211209222044.png)

这种传比较器的方式用到的是策略模式。。