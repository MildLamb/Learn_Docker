# 初识Dockerfile
dockerfile就是用来构建docker镜像的构建文件！命令脚本！  
通过这个脚本可以生成镜像，镜像是一层一层的，脚本是一个个命令，每个命令都是一层！  

## 测试Dockerfile
- 编写一个DockerFile文件
```shell
# 指令都是大写的
FROM centos

VOLUME ["volume01","volume02"]

CMD echo "-----end-----"

CMD /bin/bash
# 每个命令都是一层操作
```
- 构建DockerFile
```bash
[root@VM-16-14-centos docker-volume-test]# docker build -f dockerFile-01 -t mildlamb/centos:1.0 .
Sending build context to Docker daemon  2.048kB
Step 1/4 : FROM centos
 ---> 5d0da3dc9764
Step 2/4 : VOLUME ["volume01","volume02"]
 ---> Running in b7957ad7171d
Removing intermediate container b7957ad7171d
 ---> 474a4956d12f
Step 3/4 : CMD echo "-----end-----"
 ---> Running in 52dc48bfec41
Removing intermediate container 52dc48bfec41
 ---> 0b342503dfdc
Step 4/4 : CMD /bin/bash
 ---> Running in afef08278b37
Removing intermediate container afef08278b37
 ---> c35f86d118a0
Successfully built c35f86d118a0
Successfully tagged mildlamb/centos:1.0
```
- 启动镜像查看目录
```bash
[root@VM-16-14-centos docker-volume-test]# docker run -it c35f86d118a0 /bin/bash
[root@b70a252159a2 /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var  volume01	volume02
# volume01,volume02就是我们生成镜像的时候自动挂载的，数据卷目录
```
- 查看一下卷挂载的目录
```bash
[root@VM-16-14-centos ~]# docker ps
CONTAINER ID   IMAGE                 COMMAND        CREATED         STATUS         PORTS                                       NAMES
6d588111780e   c35f86d118a0          "/bin/bash"    2 minutes ago   Up 2 minutes                                               elegant_galileo
6cd2c89647a3   portainer/portainer   "/portainer"   46 hours ago    Up 46 hours    0.0.0.0:8088->9000/tcp, :::8088->9000/tcp   adoring_brattain
[root@VM-16-14-centos ~]# docker inspect 6d588111780e
...
"Mounts": [
            {
                "Type": "volume",
                "Name": "7e8be568cb900ecb50b5b4190be782064918d1ab78e08477c4f08e59e24aa159",
                "Source": "/var/lib/docker/volumes/7e8be568cb900ecb50b5b4190be782064918d1ab78e08477c4f08e59e24aa159/_data",
                "Destination": "volume01",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            },
            {
                "Type": "volume",
                "Name": "01b9c239dea48906816d46b24d1ff0f3ced40a957549f04680ac73e182e48f7a",
                "Source": "/var/lib/docker/volumes/01b9c239dea48906816d46b24d1ff0f3ced40a957549f04680ac73e182e48f7a/_data",
                "Destination": "volume02",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ]
...
```
