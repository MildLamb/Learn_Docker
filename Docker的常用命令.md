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
