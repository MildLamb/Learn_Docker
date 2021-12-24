# 提交一个自己的镜像
## commit镜像
```bash
# docker commit 提交容器成为一个新的副本

# docker commit -m="提交的描述信息" -a="作者" 容器id 目标镜像名[:tag]
```

## 实战测试
```bash
# 启动一个默认的tomcat，默认的tomcat的webapps目录下是没有文件的
# 原因是官方的镜像就是webapps目录是空的

# 我们把webapps.dist目录下的文件全部拷贝到webapps目录下，外网就可以正常访问tomcat了

# 现在我想把我这个拷贝文件后的镜像提交为一个新的镜像 dockers commit
# 我们以后就可以直接使用我们自己的tomcat镜像了
[root@VM-16-14-centos ~]# docker commit -a="mildlamb" -m="cp file to webapps" 9a956c61e983 mytomcat:1.0
sha256:fb847f20afb7ed347adc4813d3db0cbedbe98a7a24ecef59270e29c8133b1683
[root@VM-16-14-centos ~]# docker images
REPOSITORY            TAG       IMAGE ID       CREATED         SIZE
mytomcat              1.0       fb847f20afb7   5 seconds ago   685MB
tomcat                9.0       b8e65a4d736d   34 hours ago    680MB
redis                 latest    7614ae9453d1   2 days ago      113MB
nginx                 latest    f6987c8d6ed5   3 days ago      141MB
mysql                 latest    3218b38490ce   3 days ago      516MB
centos                latest    5d0da3dc9764   3 months ago    231MB
elasticsearch         7.12.1    41dc8ea0f139   8 months ago    851MB
portainer/portainer   latest    580c0e4e98b0   9 months ago    79.1MB
```
