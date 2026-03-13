## Linux 目录介绍

详细查看了一下买的阿里云服务器上装的 Linux 系统（CentOS 7.6 x64）的目录，记录一下各个目录下一般放什么东西。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/Linux/LinuxFileSystemStructure.png)

1.文件系统层次标准：[FHS](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard)

2.[Linux directory structure explained](https://www.howtogeek.com/117435/htg-explains-the-linux-directory-structure-explained/)

FHS 定义了 Linux 或者 UNIX-like 的文件系统结构，但是 Linux 系统中还包含了一些至今未被这个标准定义的目录。

### / -根目录

Linux 系统所有的文件和目录都在根目录 **/** 下面，即使这些文件存在于不同的物理或者虚拟盘上。

### /bin -基本用户二进制文件

这下面的是**单用户模式**下安装系统时必须存在的基本用户二进制文件。比如像 Chrome 这样的应用就放在 /usr/bin 目录下，一些重要的系统级程序 比如 bash shell 就放在 /bin 目录。

### /boot -静态启动文件

这下面放的是启动系统时需要的文件，比如 GRUB boot loader's files 和 Linux kernels 就在这里。但是 boot loader's configuration files 不放在这里，它们放在 /etc 目录下。

### /dev -驱动文件

我们都知道 Linux 系统的思想是“万物皆文件”，在 Linux 系统下，一些外部设备也以文件的形式展示出来，/dev 目录下就放的是这些文件。

### /etc -配置文件

它这里说 **The /etc directory contains configuration files, which can generally be edited by hand in a text editor**，我猜想 etc 会不会是 edited configuration 的缩写:) 注意 /etc 放的是 system-wide configuration files，用户级别的配置文件放在每个用户的 home 目录下。

### /home

这个目录下给每一个用户都分了一个 home 文件夹，文件夹的名字就是你的 username，注意 root 用户的 home 目录并不在 /home 下，而是 /root。比如 Linux 系统下有个 bob 用户，那么就有 /home/bob 文件夹，里面放的是 bob 的 user's data files 和 user-specific configuration files。每一个用户只有对他自己的 home 目录 write 的权限。root 用户才有对所有用户的 home 目录 write 的权限。 

### /lib - Essential Shared Libraries

上面 /bin 目录里面的一些二进制程序所需要的一些库文件就放在 /lib 目录下，当然了 /usr/bin 目录下的二进制程序需要的库就放在 /usr/lib 下咯。

### /lost+found -Recovered Files

每个 Linux 文件系统都有一个 lost+found 目录，当文件系统冲突的时候，那么下一次启动的时候会执行一个文件系统 check。这些不要的文件就会放到 lost+found 目录下，可以从这个目录下去尽可能的恢复数据。

### /media -可移动媒介

当你插入一张 CD 的时候，就会自动在 /media 目录下自动生成你这张光盘的文件夹，通过这个文件夹就可以访问这张 CD 里面的数据。

### /mnt -临时挂载点

系统管理员用这个目录来挂载临时文件系统，比如你要挂载一个 Windows 分区来执行一些文件恢复的操作，你就可以把它挂载到 /mnt/windows 下。

### /opt -Optional Packages

这下面放的是可选软件包的子目录。一些不符合标准文件系统层次结构的专有软件一般用的就是 /opt 目录，比如一些专有程序安装的时候会把它的文件放到 /opt/application 下。

### /proc -内核和进程文件

/proc 目录和 /dev 目录相似，它也不包含这些标准的文件，它包含的是代表系统和进程信息的特殊文件。

### /root

这里是 root 用户的 home 目录，注意 root 用户的 home 目录并不是 /home/root 哦..

### /sbin -System Administration Binaries

它和 /bin 目录相似，它放的也是基本二进制文件，但这里的二进制文件是在 root 用户进行系统管理时用到。

### /srv -Service Data

这下面放的是这个系统提供的一些服务的数据，比如用 Apache HTTP server 来跑一个网站的时候，你网站的一些文件就可以放在 /srv 目录下面。

### /tmp -Tmporary Files

Application 的一些临时文件会放在这下面，系统重新启动或者程序运行当中就可能会把这下面的文件给删除。

### /usr -User Binaries & Read-Only Data

这下面当然放的是用户的 application 和 files，和系统级 application，files 相对立。比如 non-essential application 会放在 /usr/bin 目录下而不是 /bin 下，non-essential system administraion binaries 会放在 /usr/sbin 而不是 /sbin 下。每一个程序所需要的库都会放在 /usr/lib 下面。

/usr/local 是本地编译的应用程序安装的目录。这样可以防止它们破坏系统的其余部分。

### /var -Variable Data Files

/var 目录是 /usr 目录的可写副本，在正常操作中，该目录必须为只读。日志文件和在正常操作期间通常会写入 /usr 的所有其它内容都会写入 /var 目录。

















