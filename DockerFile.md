# DockerFile介绍
dockerfile是用来构建docker镜像的文件，命令参数脚本  
构建步骤：  
1. 编写一个dockerfile文件
2. docker build 构建成为一个镜像
3. docker run 运行镜像
4. docker push 发布镜像 (dockerhub,阿里云等等...)

# DockerFile构建过程
**基础知识:**  
1. 每个保留关键字(指令)都是必须是大写字母
2. 执行从上到下顺序执行
3. #表示注释
4. 每一个指令都会创建提交一个新的镜像层

- DockerFile：构建文件，定义了一切的步骤，源代码
- DockerImages：通过DockerFile构建生成的镜像，最终发布和运行的产品
- Docker容器：容器就是镜像运行起来提供服务

# Dockerfile常用命令
![image](https://user-images.githubusercontent.com/92672384/147425633-dca18df5-3c7a-496c-8898-889a9d59df07.png)

![image](https://user-images.githubusercontent.com/92672384/147426097-10628c9e-bbd9-4c52-95ce-7ae94b5c11c8.png)

```txt
FROM           # 基础镜像，一切从这里开始构建
MAINTAINER     # 镜像是谁写的，姓名 + 邮箱
RUN            # 镜像构建的时候需要运行的命令
ADD            # 步骤，添加tomcat镜像，tomcat压缩包，添加内容
WORKDIR        # 镜像的工作目录
VOLUME         # 挂载的目录
EXPOSE         # 暴露的端口配置
CMD            # 指定这个容器启动的时候要运行的命令，只有最后一个会生效，可以被替代
ENTRYPOINT     # 指定这个容器启动的时候要运行的命令，可以追加命令
ONBUILD        # 当构建一个被继承的Dockerfile，这个时候就会运行ONBUILD的指令，出发指令
COPY           # 类似ADD，将我们的文件拷贝到镜像中
ENV            # 构建的时候设置环境变量
```

## 实战测试
### 创建一个自己的centos
1. 编写dockerfile文件
```bash
FROM centos
MAINTAINER MildLamb<1902524390@qq.com>

ENV MYPATH /usr/local
WORKDIR $MYPATH

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80

CMD echo $MYPATH
CMD echo "----end----"
CMD /bin/bash
```
2. 通过编写的dockerfile文件，构建我们的镜像
```bash
[root@VM-16-14-centos dockerfiles]# docker build -f mydockerfile-centos -t mycentos:0.1 .
```
3. 测试运行
```bash
[root@VM-16-14-centos dockerfiles]# docker run -it mycentos:0.1
[root@2c49583947d2 local]# pwd
/usr/local
[root@2c49583947d2 local]# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.2  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ac:11:00:02  txqueuelen 0  (Ethernet)
        RX packets 8  bytes 656 (656.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
4. docker history可以查看镜像是如何构成的，顺序好像是倒过来的
```bash
# 想要没有...的完整信息可以使用，docker history image_name --no-trunc=true，但可能会有排版问题
[root@VM-16-14-centos dockerfiles]# docker history c35f86d118a0
IMAGE          CREATED        CREATED BY                                      SIZE      COMMENT
c35f86d118a0   25 hours ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "/bin…   0B        
0b342503dfdc   25 hours ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "echo…   0B        
474a4956d12f   25 hours ago   /bin/sh -c #(nop)  VOLUME [volume01 volume02]   0B        
5d0da3dc9764   3 months ago   /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B        
<missing>      3 months ago   /bin/sh -c #(nop)  LABEL org.label-schema.sc…   0B        
<missing>      3 months ago   /bin/sh -c #(nop) ADD file:805cb5e15fb6e0bb0…   231MB
```

## CMD和ENTRYPOINT的区别
### 测试CMD
```bash
# CMD的dockerfile文件
FROM centos
CMD ["ls","-a"]

# 构建镜像，执行镜像
[root@VM-16-14-centos dockerfiles]# vim dockerfile-cmd-test
# 构建镜像
[root@VM-16-14-centos dockerfiles]# docker build -f dockerfile-cmd-test -t cmdtest:1.0 .
Sending build context to Docker daemon  3.072kB
Step 1/2 : FROM centos
 ---> 5d0da3dc9764
Step 2/2 : CMD ["ls","-a"]
 ---> Running in d8b24caa9b0f
Removing intermediate container d8b24caa9b0f
 ---> a18985117cbf
Successfully built a18985117cbf
Successfully tagged cmdtest:1.0
# 直接执行镜像，自动运行dockerfile文件中的cmd命令
[root@VM-16-14-centos dockerfiles]# docker run cmdtest:1.0
.
..
.dockerenv
bin
dev
etc
home
lib
lib64
lost+found
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var

# 当我们想追加一个命令 -l，理想情况是与dockerfile文件的 ls -a组合成为 ls -al，但是docker报错，如下
[root@VM-16-14-centos dockerfiles]# docker run cmdtest:1.0 -l
docker: Error response from daemon: OCI runtime create failed: container_linux.go:380: starting container process caused: exec: "-l": executable file not found in $PATH: unknown.
# cmd的情况下， -l 替换了 CMD["ls","-a"]命令，-l 并不是命令所以报错
ERRO[0000] error waiting for container: context canceled
```
### 测试ENTRYPOINT
```bash
# 创建相同的dockerfile文件，但是使用运行镜像的时候 添加-l参数
[root@VM-16-14-centos dockerfiles]# docker run entrytest:1.0 -l
total 56
drwxr-xr-x   1 root root 4096 Dec 27 03:07 .
drwxr-xr-x   1 root root 4096 Dec 27 03:07 ..
-rwxr-xr-x   1 root root    0 Dec 27 03:07 .dockerenv
lrwxrwxrwx   1 root root    7 Nov  3  2020 bin -> usr/bin
drwxr-xr-x   5 root root  340 Dec 27 03:07 dev
drwxr-xr-x   1 root root 4096 Dec 27 03:07 etc
drwxr-xr-x   2 root root 4096 Nov  3  2020 home
lrwxrwxrwx   1 root root    7 Nov  3  2020 lib -> usr/lib
lrwxrwxrwx   1 root root    9 Nov  3  2020 lib64 -> usr/lib64
drwx------   2 root root 4096 Sep 15 14:17 lost+found
drwxr-xr-x   2 root root 4096 Nov  3  2020 media
drwxr-xr-x   2 root root 4096 Nov  3  2020 mnt
drwxr-xr-x   2 root root 4096 Nov  3  2020 opt
dr-xr-xr-x 128 root root    0 Dec 27 03:07 proc
dr-xr-x---   2 root root 4096 Sep 15 14:17 root
drwxr-xr-x  11 root root 4096 Sep 15 14:17 run
lrwxrwxrwx   1 root root    8 Nov  3  2020 sbin -> usr/sbin
drwxr-xr-x   2 root root 4096 Nov  3  2020 srv
dr-xr-xr-x  13 root root    0 Dec 26 02:08 sys
drwxrwxrwt   7 root root 4096 Sep 15 14:17 tmp
drwxr-xr-x  12 root root 4096 Sep 15 14:17 usr
drwxr-xr-x  20 root root 4096 Sep 15 14:17 var
```
- 使用CMD，我们追加的命令会覆盖原来的命令
- 使用ENTRYPOINT，我们追加的命令会拼接到ENTRYPOINT后面

## 实战测试2：Tomcat镜像
1. 准备镜像文件 tomcat镜像，jdk的压缩包
```bash
# 创建一个文件夹，文件夹里面有 tomcat和jdk的安装包
[root@VM-16-14-centos tomcat]# ls
apache-tomcat-9.0.52.tar.gz  jdk-8u202-linux-x64.tar.gz  readme.txt
```
2. 编写dockerfile文件，官方命名 Dockerfile，build的时候会自动找到这个文件构建镜像
```bash
FROM centos
MAINTAINER mildlamb<1902524390@qq.com>

# 左边是宿主机地址 右边是容器地址
COPY readme.txt /usr/local/readme.txt

# 左边是宿主机地址 右边是容器地址，ADD命令会自动解压文件到右边指定的容器地址
ADD jdk-8u202-linux-x64.tar.gz /usr/local/
ADD apache-tomcat-9.0.52.tar.gz /usr/local/

RUN yum -y install vim

ENV MYPATH /usr/local
WORKDIR $MYPATH

ENV JAVA_HOME /usr/local/jdk1.8.0_202
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.52
ENV CATALINA_BASH /usr/local/apache-tomcat-9.0.52
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin

EXPOSE 8080

CMD /usr/local/apache-tomcat-9.0.52/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.52/bin/logs/catalina.out
```
3. 构建镜像，由于使用了官方的名字Dockerfile，所以可以不用 -f 参数
```bash
[root@VM-16-14-centos tomcat]# ls
apache-tomcat-9.0.52.tar.gz  Dockerfile  jdk-8u202-linux-x64.tar.gz  readme.txt
[root@VM-16-14-centos tomcat]# docker build -t diytomcat:1.0 .
# 过程很漫长，但也太慢了
```
4. 运行容器测试
```bash
[root@VM-16-14-centos tomcat]# docker run -d -p 9090:8080 --name mytomcat -v /home/mildlamb/build/tomcat/test:/usr/local/apache-tomcat-9.0.52/webapps/test -v /home/mildlamb/build/tomcat/tomcatlogs/:/usr/local/apache-tomcat-9.0.52/logs diytomcat:1.0
```
5. 进入容器
```bash
[root@VM-16-14-centos tomcat]# docker exec -it a95778e2a772 /bin/bash
[root@a95778e2a772 local]# ls
apache-tomcat-9.0.52  bin  etc	games  include	jdk1.8.0_202  lib  lib64  libexec  readme.txt  sbin  share  src
[root@a95778e2a772 local]# pwd
/usr/local
```
6. 编写页面发布测试
```bash
# 进入挂载卷的本机文件夹 test目录
[root@VM-16-14-centos tomcat]# pwd
/home/mildlamb/build/tomcat
[root@VM-16-14-centos tomcat]# ls
test  tomcatlogs
[root@VM-16-14-centos tomcat]# cd test

# 创建WEB-INF目录，并在该目录下编写web.xml文件
[root@VM-16-14-centos test]# mkdir WEB-INF
[root@VM-16-14-centos test]# ls
WEB-INF
[root@VM-16-14-centos test]# cd WEB-INF/
[root@VM-16-14-centos WEB-INF]# vim web.xml

# 在WEB-INF 目录下创建一个jsp页面
[root@VM-16-14-centos WEB-INF]# cd ..
[root@VM-16-14-centos test]# vim index.jsp
```
- web.xml
```bash
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
</web-app>
```
- jsp
```bash
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ page import="java.io.*,java.util.*" %>
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>测试自定义的Tomcat镜像</title>
</head>
<body>
<h1>使用Dockerfile构建Tomcat镜像</h1>
<ul>
<li><p><b>站点名:</b>
   https://github.com/MildLamb/Learn_Docker/blob/main/DockerFile.md
</p></li>
</ul>
</body>
</html>
```

# Dockerfile：CentOS 7更换国内源
## 阿里云源地址
[https://developer.aliyun.com/mirror/centos?spm=a2c6h.13651102.0.0.3e221b11BXNk8Q](https://developer.aliyun.com/mirror/centos?spm=a2c6h.13651102.0.0.3e221b11BXNk8Q)
## 宿主机
```bash
#获取国内源
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
#到/etc/yum.repos.d/目录下查找源
cd /etc/yum.repos.d/
#将源拷贝到你Dockerfile所在目录
```
## Dockerfile编写
```bash
#使用 ADD 命令将 CentOS-Base.repo 拷贝到目标基础镜像的目录下
ADD  CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo
#更新yum源|如果你不需要更新版本，可以不执行此命令（升级后的版本太高可能导致原有软件不能运行）
#RUN yum -y update
#运行yum makecache生成缓存，便于查找
RUN yum makecache
#如果觉得占用磁盘空间，可以使用以下指令清除缓存
RUN yum clean
```
