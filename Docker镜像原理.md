# Docker镜像原理
## 镜像是什么
镜像是一种轻量级、可执行的独立软件包，用来打包软件运行环境和基于运行环境开发的软件，  
它包含运行某个软件所需的所有内容，包括代码、运行时库、环境变量和配置文件。  
如何得到镜像：  
- 从远程仓库下载
- 拷贝文件
- 自己制作一个镜像DockerFile

## 镜像加载原理
### 联合文件系统(UnionFS)
联合文件系统(UnionFS)：Union文件系统(UnionFS)是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改  
作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single  
virtual filesystem)。Union文件系统是Docker镜像的基础。镜像可以通过分层来进行继承，基于基础镜像(没有父镜像)，可以制作  
各种具体的应用镜像。  
特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的  
文件系统会包含所有底层的文件和目录

## 分层理解
```bash
[root@VM-16-14-centos ~]# docker pull redis
Using default tag: latest
latest: Pulling from library/redis
a2abf6c4d29d: Already exists 
c7a4e4382001: Pull complete 
4044b9ba67c9: Pull complete 
c8388a79482f: Pull complete 
413c8bb60be2: Pull complete 
1abfd3011519: Pull complete 
Digest: sha256:db485f2e245b5b3329fdc7eff4eb00f913e09d8feb9ca720788059fdc2ed8339
Status: Downloaded newer image for redis:latest
docker.io/library/redis:latest
[root@VM-16-14-centos ~]# docker image inspect redis:latest
...
"RootFS": {
            "Type": "layers",
            "Layers": [  # 对应6个层级
                "sha256:2edcec3590a4ec7f40cf0743c15d78fb39d8326bc029073b41ef9727da6c851f",
                "sha256:9b24afeb7c2f21e50a686ead025823cd2c6e9730c013ca77ad5f115c079b57cb",
                "sha256:4b8e2801e0f956a4220c32e2c8b0a590e6f9bd2420ec65453685246b82766ea1",
                "sha256:529cdb636f61e95ab91a62a51526a84fd7314d6aab0d414040796150b4522372",
                "sha256:9975392591f2777d6bf4d9919ad1b2c9afa12f9a9b4d260f45025ec3cc9b18ed",
                "sha256:8e5669d8329116b8444b9bbb1663dda568ede12d3dbcce950199b582f6e94952"
            ]
        }
```
### 特点
Docker镜像都是只读的，当容器启动时，一个新的可写层被加载到镜像的顶部   
这一层我们通常称为容器层，容器之下的都叫镜像层  
