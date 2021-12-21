# Learn_Docker
- Docker概述
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

- Docker的基本组成
![image](https://user-images.githubusercontent.com/92672384/146863067-bc9b00b3-ee9d-4179-a8fc-868f2e990412.png)



- Docker安装
- Docker命令
  - 镜像命令
  - 容器命令
  - 操作命令
  - ... 
- Docker镜像
- 容器数据卷
- DockerFile
- Docker网络原理
- IDEA整合Docker

- Docker Compose
- Docker Swarm
