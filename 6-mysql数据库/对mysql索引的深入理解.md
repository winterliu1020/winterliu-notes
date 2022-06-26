Hash索引

**不支持顺序和范围查找**，所以不太适合用在mysql数据库中。

### 哪些字段适合做索引

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20220122214307.png)



### B树和B+树

B树：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211228110336.png)

B+树：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211228110213.png)

总结：

都采用了平衡树的思想，同时利用二分的思路，节点中包含一个数据，则下一层可以分成两个节点，左节点都比这个数据小，右节点都比这个数据大；如果是两个数据，则下一层可以分成三个节点；

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211228110853.png)



### 索引补充知识

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20220210223821.png)

可以看到在InnoDB中，B+树的非叶子结点其实key虽然还是索引键，但是值却是pageno，也就是数据页的编号，然后指向最下面一层叶子结点，每个节点其实是一页数据，而且每一页中的数据也是有序的，每一页中的数据可以通过链表或者数组存储，这样查找到某一页上后，可以通过二分法快速找到某一行记录。

然后再看看对索引的一些思考：https://www.cnblogs.com/zsql/p/13808417.html

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20220210225437.png)

