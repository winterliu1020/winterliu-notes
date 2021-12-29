创建、删除目录：

```shell
$mkdir testDir
## rmdir只能删除空目录
$rmdir testDir
$rmdir -r testDir # 会将testDir里面的东西删掉，然后再把testDir这个文件夹删掉
# 显示文件夹中的东西
$ls -al （可以加个路径）
```

检查文件的内容：

```shell
# cat, tac, more, less, head, tail
```

修改文件权限：

```shell
$chmod 777 test.txt
```

复制：

```shell
cp [-ai] 来源文件 目标文件 #如果来源文件有两个以上，目标文件一定得是一个目录;i参数是会询问是否覆盖同名文件，a参数（相当于-dr）是指会把:1.如果文件是链接文件（-d），它相当于会再复制一个快捷方式。2.如果来源文件是个文件夹，它会递归持续复制（-r）
cp ~/.bashrc  /tmp/bashrc 
cp ~/.bashrc . #将文件复制到当前目录
cp -r /etc/ /tmp #将etc这个目录下所有内容复制到/tmp目录下面，注意-r复制出来的文件的权限可能会被改变，所有用cp时一般用-a
```

压缩、解压缩：

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211223113124.png)

```shell
# zip、unzip
zip test.zip test.txt # 讲test.txt压缩成test.zip
zip /root/test.zip test.txt
unzip test.zip #默认将文件解压到当前目录（就是把这个压缩包里面的东西拿到当前目录）
unzip test.zip -d /root/testDir #将压缩包里面的东西放到testDir这个指定的目录中
```

软链接：相当于给原文件创建了一个快捷方式，如果删除了原文件，对应的链接文件也会消失。

```shell
ln -s test.txt test_softlink
```

硬链接：相当于给原文件取了个别名，两者是同一个文件，删除其中一个，另一个不会消失，但是对其中一个修改，另一个也会随之改变。

```shell
ln test.txt test_hardlink
```

观察文件的类型：

```shell
$file test
```

指令文件名的搜索：就是找命令对应文件所在的位置（which是在PATH这个环境变量里面找）

```shell
$which ls
```

文件文件名的搜索：

```shell
## whereis 只是查找某几个目录下面的，所以比find快
$ whereis -[bmsu] 文件或目录名

## locate用这个命令去查找的原理是通过数据库，所以查找之前可能需要用updatedb命令来更新一下数据库  
$ updatedb
$ locate test.txt

## find [PATH] [option] [action]; 注意用find找数据的时候相当的操硬盘，所以一般先使用whereis和locate去找
$find / -name passwd
$find / -name *passwd* // *通配符

```

移动文件：

```shell
## 可以用mv命令来重命名
$ mv test1.txt test2.txt # 将test1重命名为test2
$ mv file1.txt file2.txt file3.txt folder # 移动多个文件到某个文件夹

# 加了-u 表示foo.txt bar.txt这两个文件中只有比bar文件夹中的文件更加新的才会移动到bar中
$ mv -u foo.txt bar.txt bar # 注意：bar文件夹中也有foo.txt bar.txt文件；The file foo.txt is not moved as it is older than the file in the destination folder.

# 移动多个文件(夹)到 某个目录  加-t即可
mv build config test1.txt -t folder

## mv 一个文件夹下的所有东西（文件、文件夹）到另一个目录时发生报错，可以换成cp复制的方式，然后删除原目录即可
mv ./backup/* ./backupArchives
mv: cannot move './backup/base' to './backupsArchive/base': Directory not empty
cp -r ./backup/* ./backupArchives && rm -R ./backup/*

```

```bash
# ps命令 Process Status
# 显示所有进程
ps -A 或者 -e
ps -f 全格式显示进程

# grep命令 global regular expression print
# grep [option] pattern file
ps -ef | grep -f adb # 也可以加字符串，然后这个字符串作为正则匹配
grep -f test.txt # 后面可以加文件，test.txt文件中每一行都作为正则匹配项

# wc命令 wordCount
wc testfile # 统计行数 单词数 字节数
wc -l # 统计行数

# du命令 diskUseage
du -h # 输出当前文件夹中文件的大小，单位是M

# awk命令 三个人名字的首字母 主要用于对字符串进行处理
输入流 ｜ awk -F '=' '{print $1 $2}' # 根据‘=’等号来切割 $1 $2是切割得到的第1，2个元素
awk '这里是脚本' 文件名 # 对文件中每一行利用脚本进行过滤，得到符合的行
awk '$1>2 && $2=="Are" {print $1, $2}' log.txt
awk '$1>2 && $2=="Are"' log.txt # 后面不加print就是默认输出符合脚本的整行


# sort命令
# uniq -c
ls -l | sort | uniq -c | sort -k 1 -nr
sort # 会将输入流所有行按照ascii排序
uniq -c # 排完序之后，这个命令会合并相邻的重复行，并统计重复数；输出(前面是频率 后面是内容)：
2 hello
3 my
4 thanks
sort -k 1 # 表示按照每行的第一个字段排序 从小到大
sort -nr # -n表示指定按照数值大小进行排序，后面加r表示逆序

cat simpleLocker.txt | awk -F ' ' '{print $2}' | sort | uniq | while read line
do
	adb uninstall $line
done

# top命令
top # 看当前机器中的进程状态
uptime # 看CPU负载情况

# ps命令 process status进程状态
ps -a # 显示所有有终端控制下执行的进程
ps -A 或者 ps -ax # 显示所有进程 无论是否运行在终端
ps -au # 显示所有有终端控制下执行的进程的详细信息 包括CPU使用情况等等
ps -aux # 所有进程的详细信息 无论是否运行在终端
ps -e # 和-A效果一样 会显示所有进程的信息 但是只看得到几项信息
ps -f # 把全部列都给显示出来，通常和其它选项联用
ps -fe 和 ps -aux效果差不多
ps -u root # 看root用户下的进程

# uname命令 unix name 用于查看一些系统信息
uname -a # 看内核 操作系统 CPU信息

# netstat命令 显示Linux系统的网络情况
netstat -a # 详细网络情况 哪个ip和哪个ip用哪种协议建立连接，当前连接状态是什么样的
netstat -apu # -all protocal udp
netstat -apt # -all protocal tcp 看tcp协议端口使用情况
netstat -i # 显示网卡列表 就是ifconfig看到的网卡名称
netstat -tunlp | grep 8080 # 查看端口占用情况 -tcp udp n(拒绝显示别名，全用数字显示) listen(仅列出在listen的服务) p(显示建立相关连接的process)
kill -9 PID # 杀掉进程

# lsof命令 list open files 列出当前系统打开文件的工具
lsof -i:端口号 # 这个命令得root用户才能执行
```

### Linux权限命令

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/notes/20211223114047.png)

