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

### CMD和ENTRYPOINT的区别
- 测试CMD
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
