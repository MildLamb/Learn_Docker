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

# 两者的区别
# docker exec：进入容器后开启一个新的终端，可以在里面操作
# docker attach：进入容器正在执行的终端，不会启动新的进程
```

### 从容器内拷贝文件到主机上
- docker cp 容器id:容器内路径 目的的主机路径
```shell
# 先查看主机的home目录
[root@VM-16-14-centos ~]# cd /home
[root@VM-16-14-centos home]# ls
lighthouse  mildlamb  mysql-5.7.26-linux-glibc2.12-x86_64.tar.gz  wildwolf  www

# 进入容器，创建文件
[root@VM-16-14-centos home]# docker attach 9b147c3190f8
[root@9b147c3190f8 /]# cd /home
[root@9b147c3190f8 home]# ls
[root@9b147c3190f8 home]# touch test.java
[root@9b147c3190f8 home]# ls
test.java

# 停止并退出容器，只要容器还存在容器内的文件也就会存在，无论容器是否是运行的状态
[root@9b147c3190f8 home]# exit
exit
[root@VM-16-14-centos home]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@VM-16-14-centos home]# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS                     PORTS     NAMES
9b147c3190f8   centos    "/bin/bash"   21 minutes ago   Exited (0) 6 seconds ago             dreamy_cori

# 从容器内拷贝文件到主机上
[root@VM-16-14-centos home]# docker cp 9b147c3190f8:/home/test.java /home  

# 重新检查主机home目录文件，发现test.java拷贝过来了
[root@VM-16-14-centos home]# ls
lighthouse  mildlamb  mysql-5.7.26-linux-glibc2.12-x86_64.tar.gz  test.java  wildwolf  www
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
## 常用其他命令
### 后台启动容器
```shell
# 命令： docker run -d 容器名
[root@VM-16-14-centos ~]# docker run -d centos
469c10fe7d9b2034e87df2c262927c142880d64b5bd878e609c919288fad8b25
# 问题： 后台启动容器，但docker ps没有发现容器进程
[root@VM-16-14-centos ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
# 常见的坑：docker容器使用后台运行，就必须要有一个前台进程，docker发现没有前台应用就会自动停止
```
### 查看日志
```shell
[root@VM-16-14-centos ~]# docker logs -f -t --tail num d5608c64ed90  # 显示某个容器的近num条日志

[root@VM-16-14-centos ~]# docker logs -tf d5608c64ed90  # 显示某个容器全部的日志
```
### 查看容器中进程信息
```shell
# docker top 容器id
[root@VM-16-14-centos ~]# docker top 9b147c3190f8
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                29487               29468               0                   09:01               pts/0               00:00:00            /bin/bash
```
### 查看镜像的元数据
```shell
# docker inspect 容器id
[root@VM-16-14-centos ~]# docker inspect --help

Usage:  docker inspect [OPTIONS] NAME|ID [NAME|ID...]

Return low-level information on Docker objects

Options:
  -f, --format string   Format the output using the given Go template
  -s, --size            Display total file sizes if the type is container
      --type string     Return JSON for specified type
[root@VM-16-14-centos ~]# docker inspect 9b147c3190f8
[
    {
        "Id": "9b147c3190f8ec0d4bd2e08a44f0997178c9613ad7870cc75c18ca714937cbee",
        "Created": "2021-12-23T01:01:15.719728452Z",
        "Path": "/bin/bash",
        "Args": [],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 29487,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2021-12-23T01:01:16.068768245Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:5d0da3dc976460b72c77d94c8a1ad043720b0416bfc16c52c45d4847e53fadb6",
        "ResolvConfPath": "/var/lib/docker/containers/9b147c3190f8ec0d4bd2e08a44f0997178c9613ad7870cc75c18ca714937cbee/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/9b147c3190f8ec0d4bd2e08a44f0997178c9613ad7870cc75c18ca714937cbee/hostname",
        "HostsPath": "/var/lib/docker/containers/9b147c3190f8ec0d4bd2e08a44f0997178c9613ad7870cc75c18ca714937cbee/hosts",
        "LogPath": "/var/lib/docker/containers/9b147c3190f8ec0d4bd2e08a44f0997178c9613ad7870cc75c18ca714937cbee/9b147c3190f8ec0d4bd2e08a44f0997178c9613ad7870cc75c18ca714937cbee-json.log",
        "Name": "/dreamy_cori",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "host",
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/cd58714f5b077cfb72aa6f38e7970b4771fa69054ca67c40401c7d20369e2d40-init/diff:/var/lib/docker/overlay2/3e72a1fdba3e15d8ec553d81055122953ddcbf95f5cdd41e2ff614f6df4e7457/diff",
                "MergedDir": "/var/lib/docker/overlay2/cd58714f5b077cfb72aa6f38e7970b4771fa69054ca67c40401c7d20369e2d40/merged",
                "UpperDir": "/var/lib/docker/overlay2/cd58714f5b077cfb72aa6f38e7970b4771fa69054ca67c40401c7d20369e2d40/diff",
                "WorkDir": "/var/lib/docker/overlay2/cd58714f5b077cfb72aa6f38e7970b4771fa69054ca67c40401c7d20369e2d40/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "9b147c3190f8",
            "Domainname": "",
            "User": "",
            "AttachStdin": true,
            "AttachStdout": true,
            "AttachStderr": true,
            "Tty": true,
            "OpenStdin": true,
            "StdinOnce": true,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/bash"
            ],
            "Image": "centos",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "org.label-schema.build-date": "20210915",
                "org.label-schema.license": "GPLv2",
                "org.label-schema.name": "CentOS Base Image",
                "org.label-schema.schema-version": "1.0",
                "org.label-schema.vendor": "CentOS"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "672ebd46a95ab6c1255f29ca40a34a671b764fc518fb0f60df68cbf06af82ab9",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/672ebd46a95a",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "561a79ca5c119967900a89ef3c0b2ca4ae16fe4984829d42512c8bafd677779b",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "6c351016746a2af68769fe01f3a446390ed16fc96460c85c7d93eac5ba54ca12",
                    "EndpointID": "561a79ca5c119967900a89ef3c0b2ca4ae16fe4984829d42512c8bafd677779b",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]
```
