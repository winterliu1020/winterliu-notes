字符串，list, 哈希，set, zset

1 字符串Simple Dynamic String

统筹：len, free, but-->['a', 'b', 'c'...] 字符数组

**空间预分配** 每次修改之后如果长度变大了（修改之前的长度是len）：

1. 如果修改之后长度<1M，那么修改之后长度变成2*len
2. 如果修改之后长度>1M，那么修改之后除了存储所有字符串之外，还得多分配1M空间。

**惰性释放** 如果修改后减少了存储的内容，多出来的那些空间不会立即释放。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211228145806.png)

### 2 双向链表

统筹：head, tail, len  实施：node[pre, next]

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211228145938.png)

### 3 ziplist

是什么？它如何最大限度的节省内存？它的优缺点？

其实类似于数组，但是核心又是一个在连续内存空间的链表，因为它每个元素的大小是可以不相等的。

数据量小的时候会使用压缩列表，因为每添加一个元素都需要新申请一块内存，而且如果当前这片连续空间用完了，需要申请一块更大的空间，然后把之前的数据复制进来。

而且压缩列表是redis中列表键和哈希键的底层实现。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/concurrent/20210908144823.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211228150857.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211228150917.png)

通过统筹部分的尾指针找到最后一个节点，然后从后往前遍历

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211228151041.png)

要么一个字节、要么五个字节，根据前一节点的长度。。

### 4 Hash

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211228151549.png)



### 5 zset

有序集合。里面的每一个元素都包括[score和member]

底层可以用ziplist或者skiplist来实现。看是否满足：节点数小于128并且节点大小小于64字节。

不满足的话，底层就用skiplist编码，它包括一个字典和一个skiplist，这个跳表按照score从小到大排序，跳表包含了所有集合元素。而字典存储了member到score的映射，所以可以O(1)查找member的score。

其中dict和skiplist如果有相同的score，member，他们会共享这块区域，以此节省内存。

#### skiplist

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/concurrent/20210908151711.png)

如果你按照这种固定间隔去形成跳表，那么每次插入或者删除一个节点都需要去调整，又退化成O(n)。

那么跳表它在插入一个节点时会产生一个随机数来作为这个节点的层数，就不要求上面一层节点数是下面那层节点数的一半。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/concurrent/20210908153222.png)

为什么不用哈希或者平衡树？

1. 哈希只适用于查找单个元素，而skiplist还可以方便查找某个范围内的元素。
2. 查找某个范围所有元素时，用skiplist首先需要找到左区间所在位置，然后遍历skiplist的第一层就可以，而平衡树找到左区间后，如果想用中序遍历去找该区间所有元素，你还得对平衡树做一定调整。比较复杂。而且平衡树需要存储左右两个指针，跳表如果p取四分之一，每个节点平均只需存储1.33个指针。

#### zset的设计

```c
//定义跳表的基本数据节点
typedef struct zskiplistNode {
    robj *obj; // zset value
    double score;// zset score
    struct zskiplistNode *backward;//后向指针
    struct zskiplistLevel {//前向指针
        struct zskiplistNode *forward;
        unsigned int span;
    } level[];
} zskiplistNode;

typedef struct zskiplist {
    struct zskiplistNode *header, *tail;
    unsigned long length;
    int level;
} zskiplist;

//有序集数据结构
typedef struct zset {
    dict *dict;//字典存放value,以value为key
    zskiplist *zsl;
} zset;

```

参考：https://segmentfault.com/a/1190000022028505

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/concurrent/20210909205149.png)







链表、跳表