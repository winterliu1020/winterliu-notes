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

