![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211228113526.png)

连接、分析、优化、执行

### 关于重做日志redo log

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211228114346.png)

可以看到是先把操作记录在重做日志文件 此时redo log是prepare状态，然后再去提交数据更新到磁盘，更新成功后redo log才变成提交状态。

两个日志文件 redo log和bin log，还有prepare状态和提交状态：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211228115141.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211228115602.png)

