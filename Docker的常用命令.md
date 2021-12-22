# Docker的常用命令
## 帮助命令  
### docker version  # 显示docker的版本信息
### docker info     # 显示docker的系统信息，包括镜像和容器的信息
### docker 命令 --help  # 万能命令

## 镜像命令
### docker images 查看所有本地的主机上的镜像
```shell
[root@VM-16-14-centos ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    feb5d9fea6a5   2 months ago   13.3kB

# 解释
REPOSITORY   镜像的仓库源
TAG          镜像的标签
IMAGE ID     镜像的id
CREATED      镜像的创建时间
SIZE         镜像的大小

# 可选项
Options:
  -a, --all             Show all images (default hides intermediate images)
      --digests         Show digests
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print images using a Go template
      --no-trunc        Don't truncate output
  -q, --quiet           Only show image IDs
  
-a, --a : 显示所有的镜像
-q, --quiet ：只显示镜像的id
```
### docker search 搜索镜像
```shell
[root@VM-16-14-centos ~]# docker search mysql
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   11843     [OK]       
mariadb                           MariaDB Server is a high performing open sou…   4514      [OK]       
mysql/mysql-server                Optimized MySQL Server Docker images. Create…   888                  [OK]

# 可选项，通过收藏过滤
# --filter=STARS=3000  # 搜索出来的镜像就是STARS大于等于3000的
[root@VM-16-14-centos ~]# docker search mysql --filter=STARS=4514
NAME      DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql     MySQL is a widely used, open-source relation…   11843     [OK]       
mariadb   MariaDB Server is a high performing open sou…   4514      [OK]
```
### docker pull 下载镜像
```shell
# 下载镜像：docker pull 镜像名[:tag]
如果需要指定版本：
# docker pull mysql:5.7.28
# docker pull mysql:8.0.18
[root@VM-16-14-centos ~]# docker pull mysql
Using default tag: latest   # 如果不写 tag，版本默认使用最后一个
latest: Pulling from library/mysql
72a69066d2fe: Pull complete  # 分层下载，docker image的核心 联合文件系统
93619dbc5b36: Pull complete 
99da31dd6142: Pull complete 
626033c43d70: Pull complete 
37d5d7efb64e: Pull complete 
ac563158d721: Pull complete 
d2ba16033dad: Pull complete 
688ba7d5c01a: Pull complete 
00e060b6d11d: Pull complete 
1c04857f594f: Pull complete 
4d7cfa90e6ea: Pull complete 
e0431212d27d: Pull complete 
Digest: sha256:e9027fe4d91c0153429607251656806cc784e914937271037f7738bd5b8e7709  # 签名
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest  # 真实地址
```
### docker rmi -f 删除镜像
```shell
# 删除指定的镜像：docker rmi -f imageid
# 批量删除所有镜像: docker rmi -f $(docker images -aq)
[root@VM-16-14-centos ~]# docker rmi -f c20987f18b13
Untagged: mysql:5.7
Untagged: mysql@sha256:f2ad209efe9c67104167fc609cca6973c8422939491c9345270175a300419f94
Deleted: sha256:c20987f18b130f9d144c9828df630417e2a9523148930dc3963e9d0dab302a76
Deleted: sha256:6567396b065ee734fb2dbb80c8923324a778426dfd01969f091f1ab2d52c7989
Deleted: sha256:0910f12649d514b471f1583a16f672ab67e3d29d9833a15dc2df50dd5536e40f
Deleted: sha256:6682af2fb40555c448b84711c7302d0f86fc716bbe9c7dc7dbd739ef9d757150
Deleted: sha256:5c062c3ac20f576d24454e74781511a5f96739f289edaadf2de934d06e910b92
```

## 容器命令
**说明：我们有了镜像才可以创建容器，下载一个linux，centos镜像来测试学习**   
```shell
docker pull centos
```
### 新建容器并启动
```shell
docker run [可选参数] imageId/imageName

# 参数说明
--name="Name"  # 给容器命令，用于区分容器
-d            # 后台方式运行
-it           # 使用交互方式运行，进入容器查看内容
-p            # 指定容器端口
    -p ip:主机端口:容器端口 (常用)
    -p 主机端口:容器端口 (常用)
    -p 容器端口
    容器端口

-P            # 随机指定端口

# 测试
[root@VM-16-14-centos ~]# docker run -it 5d0da3dc9764 /bin/bash   # 启动并进入容器
[root@f84260c46c94 /]# ls  # 查看容器内的centos
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var 
```
### 从容器中退出到主机  
- exit：直接容器停止并退出
- ctrl+p+q：容器不停止退出
```shell
[root@f84260c46c94 /]# exit
exit
[root@VM-16-14-centos ~]# 
```
### 查看正在运行的容器 docker ps
```shell

# docker ps 
# 列出当前正在运行的容器
# -a    列出当前正在运行的容器，以及历史运行过的容器
# -n=?   显示最近创建的?个容器
# -q    只显示容器的编号

[root@VM-16-14-centos ~]# docker ps  # 列出当前正在运行的容器
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@VM-16-14-centos ~]# docker ps -a  # 加-a参数，列出当前正在运行的容器，以及历史运行过的容器
CONTAINER ID   IMAGE          COMMAND       CREATED         STATUS                          PORTS     NAMES
f84260c46c94   5d0da3dc9764   "/bin/bash"   7 minutes ago   Exited (0) About a minute ago             competent_ardinghelli
3b5bd173800c   hello-world    "/hello"      25 hours ago    Exited (0) 25 hours ago                   compassionate_bhaskara
d1abb0e2d385   hello-world    "/hello"      2 weeks ago     Exited (0) 2 weeks ago                    modest_noyce
[root@VM-16-14-centos ~]# docker ps -a -n=2
CONTAINER ID   IMAGE          COMMAND       CREATED          STATUS                     PORTS     NAMES
f84260c46c94   5d0da3dc9764   "/bin/bash"   11 minutes ago   Exited (0) 5 minutes ago             competent_ardinghelli
3b5bd173800c   hello-world    "/hello"      25 hours ago     Exited (0) 25 hours ago              compassionate_bhaskara
```

### 进入正在运行的容器
```shell
# 第一种进入正在运行的容器的方式，通过这种方式进入容器再exit可以关闭容器退出，下面的好像不行
[root@VM-16-14-centos ~]# docker attach fa48d0507d26
[root@fa48d0507d26 /]# 

# 第二种进入正在运行的容器的方式
[root@VM-16-14-centos ~]# docker exec -it fa48d0507d26 /bin/bash
[root@fa48d0507d26 /]# 
```

### 删除容器
- docker rm 容器id  : 删除指定的容器
- docker rm -f $(docker ps -aq)  ： 删除所有的容器
- docker ps -a -q|xargs docker rm -f : 删除所有的容器
```shell
[root@VM-16-14-centos ~]# docker ps -a
CONTAINER ID   IMAGE          COMMAND       CREATED          STATUS                      PORTS     NAMES
fa48d0507d26   centos         "/bin/bash"   7 minutes ago    Exited (0) 2 minutes ago              pedantic_beaver
f84260c46c94   5d0da3dc9764   "/bin/bash"   23 minutes ago   Exited (0) 17 minutes ago             competent_ardinghelli
3b5bd173800c   hello-world    "/hello"      25 hours ago     Exited (0) 25 hours ago               compassionate_bhaskara
d1abb0e2d385   hello-world    "/hello"      2 weeks ago      Exited (0) 2 weeks ago                modest_noyce
[root@VM-16-14-centos ~]# docker rm f84260c46c94  # 删除容器
f84260c46c94
[root@VM-16-14-centos ~]# docker ps -a
CONTAINER ID   IMAGE         COMMAND       CREATED         STATUS                     PORTS     NAMES
fa48d0507d26   centos        "/bin/bash"   8 minutes ago   Exited (0) 2 minutes ago             pedantic_beaver
3b5bd173800c   hello-world   "/hello"      25 hours ago    Exited (0) 25 hours ago              compassionate_bhaskara
d1abb0e2d385   hello-world   "/hello"      2 weeks ago     Exited (0) 2 weeks ago               modest_noyce
```

### 启动和停止容器的操作
- docker start 容器id：启动一个容器
- docker restart 容器id：重启一个容器
- docker stop 容器id：停止一个正在运行的容器
- docker kill 容器id：杀死一个容器，强制停止一个容器

```shell
[root@VM-16-14-centos ~]# docker run -it centos /bin/bash
[root@5000bf9f8fd1 /]# exit
exit
[root@VM-16-14-centos ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS                     PORTS     NAMES
5000bf9f8fd1   centos    "/bin/bash"   15 seconds ago   Exited (0) 7 seconds ago             eager_gauss
[root@VM-16-14-centos ~]# docker start 5000bf9f8fd1
5000bf9f8fd1
[root@VM-16-14-centos ~]# docker ps 
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS         PORTS     NAMES
5000bf9f8fd1   centos    "/bin/bash"   31 seconds ago   Up 5 seconds             eager_gauss
[root@VM-16-14-centos ~]# docker stop 5000bf9f8fd1
5000bf9f8fd1
[root@VM-16-14-centos ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```
