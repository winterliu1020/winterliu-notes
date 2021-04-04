## springboot 和 Django 项目部署到阿里云

因为我的本科毕业设计是 springboot 工程，但是当中需要用到 OpenCV 来处理图片，由于 Java 版本的 OpenCV 不太方便，所以想到搭建一个 Django 服务，这样就可以利用 Python 来对图片进行预处理。

### 1. springboot 项目部署

我这里用的是 IDEA，所以利用 maven 可以对 springboot 项目直接打包，由于 springboot 项目当中内置了 Tomcat，所以点击右边的 maven package 打包之后在 target 目录下会产生一个 jar 包。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/其它/maven-package.png)

只要把这个 jar 文件上传到服务器上某个目录下，然后在那个目录下运行以下命令：

```shell
# 简单运行，关闭连接窗口后程序会退出
java -jar winterliu-0.0.1-SNAPSHOT.jar
# 让这个程序在后台挂载
nohup java -jar winterliu-0.0.1-SNAPSHOT.jar &
```

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/其它/depoy-to-aliyun.png)

这个就可以让这个程序一直挂载运行，这个端口号就是运行的端口号。

### 2. Django 项目部署

因为我这里 Django 项目非常简单，没有涉及到数据库和界面方面的操作，只是简单的接收 JSON 和 发送 JSON 数据的服务，所以只需要把 Django 项目文件夹上传到服务器上就行了。

![](https://winterliublog.oss-cn-beijing.aliyuncs.com/winterliu-notes/其它/Django-project.png)

因为在 pycharm 中可以简单的利用 install 来下载需要的包，这里需要利用 pip 安装这个项目当中需要的一些包。比如这里用到腾讯 OCR ：

```shell
pip3 install tencentcloud-sdk-python
```

然后到 Django 项目目录下运行以下命令启动 Django 服务：

```shell
python3 manage.py runserver # 这也是简单运行
# 挂载通用命令
nohup "这里是运行的命令" &
```

### 3. 遇到的问题

部署 Django 时我遇到的一些问题：

(1) [django.core.exceptions.ImproperlyConfigured: SQLite 3.8.3 or later is required (found 3.7.17)](https://worldofit.work/error/442/)

(2) [python中import cv2遇到的错误及安装方法](https://blog.csdn.net/yuanlulu/article/details/79017116)

(3) [CentOS 7 Python3.6环境下安装libSM、libXrender、libXext](https://blog.csdn.net/BUGIN/article/details/86692292)

