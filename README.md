# Learn_Docker
## Docker概述
  - 比较Docker和虚拟机技术的不同：
    - 传统虚拟机，虚拟出一套硬件，运行一个完整的操作系统，然后在这个系统上安装和运行软件
    - 而Docker容器内的应用直接运行在 宿主机上，容器是没有自己的内核的，也没有虚拟我们的硬件，所以轻便
    - Docker容器间是相互隔离的，每个容器内都有一个属于自己的文件系统，互不影响的
  - 应用更快速的交付和部署
    - 传统：阅读帮助文档，安装程序
    - Docker：打包镜像发布测试，一键运行
  - 更便捷的升级和扩缩容
  - 更简单的系统运维
    - 容器化后，我们的开发，测试环境都是高度一致的
  - 更高效的计算资源利用
    - Docker是内核级别的虚拟化，可以在一个物理机上运行很多的容器实例

## Docker的基本组成
![image](https://user-images.githubusercontent.com/92672384/146863067-bc9b00b3-ee9d-4179-a8fc-868f2e990412.png)

- 镜像(image)：docker镜像就好比是一个模板，可以通过这个模板来创建容器服务，romcat镜像 ==> run ==> tomcat01容器(提供服务)  
通过镜像可以创建多个容器(最终服务或项目的运行是在容器进行的)
- 容器(container)：Docker利用容器技术，独立运行一个或者一组应用，通过镜像来创建，启动、停止、删除，等基本命令
- 仓库(repostory)：存放镜像的地方，仓库分为共有仓库和私有仓库



## Docker安装
- 环境准备
  - 需要一点点Linux基础
  - CentOS 7
  - Xshell
- 环境查看
```bash
# 系统内核是 3.10 及以上的
[root@VM-16-14-centos /]# uname -r
3.10.0-1160.11.1.el7.x86_64

[root@VM-16-14-centos /]# cat /etc/os-release
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"
```
- 安装
  - 1. 卸载原来的Docker
```bash
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```
  - 2. 安装需要的安装包
```bash
yum install -y yum-utils
```
  - 3. 设置镜像的仓库
```bash
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
  - 更新yum软件包索引
```bash
yum makecache fast
```

  - 4. 安装docker相关的内容
```bash
yum install docker-ce docker-ce-cli containerd.io
```
  - 5. 启动docker
```bash
systemctl start docker
```
  - 6. 使用docker version 查看是否安装成功
```bash
[root@VM-16-14-centos /]# docker version
Client: Docker Engine - Community
 Version:           20.10.12
 API version:       1.41
 Go version:        go1.16.12
 Git commit:        e91ed57
 Built:             Mon Dec 13 11:45:41 2021
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.12
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.16.12
  Git commit:       459d0df
  Built:            Mon Dec 13 11:44:05 2021
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.4.12
  GitCommit:        7b11cfaabd73bb80907dd23182b9347b4245eb5d
 runc:
  Version:          1.0.2
  GitCommit:        v1.0.2-0-g52b36a2
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```
  - 7. 测试hello-world
```bash
[root@VM-16-14-centos /]# docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```
  - 8. 查看一下 hello-world 镜像
```bash
[root@VM-16-14-centos /]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    feb5d9fea6a5   2 months ago   13.3kB
```
了解：卸载docker
```bash
# 卸载依赖
yum remove docker-ce docker-ce-cli containerd.io

# 删除资源
rm -rf /var/lib/docker
rm -rf /var/lib/containerd
```

## 配置阿里云镜像加速
- 登录阿里云，在产品中找到容器镜像服务，创建一个个人版的镜像托管
- 在镜像工具中，找到镜像加速器
- 执行按照提示执行下面4个命令
```bash
sudo mkdir -p /etc/docker

sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://9ku8hjj8.mirror.aliyuncs.com"]
}
EOF

sudo systemctl daemon-reload

sudo systemctl restart docker
```
- 执行完命令就完成了阿里云的镜像加速

## Docker命令
  - 镜像命令
  - 容器命令
  - 操作命令
  - ... 
## Docker镜像
## 容器数据卷
## DockerFile
## Docker网络原理
## IDEA整合Docker

- Docker Compose
- Docker Swarm
