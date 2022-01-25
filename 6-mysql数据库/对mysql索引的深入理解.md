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



