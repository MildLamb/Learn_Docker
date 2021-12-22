# run的执行流程
![image](https://user-images.githubusercontent.com/92672384/147015538-881ccf0f-ce4b-4326-bc18-49e7d02edd8b.png)

# 底层原理
Docker是怎么工作的?  
Docker是一个Client - Server结构的系统，Docker的守护进程运行在主机上，通过Socket从客户端访问  
DockerServer接收到DockerClient的指令，就会执行命令  
![image](https://user-images.githubusercontent.com/92672384/147016066-164c6ec2-5340-48af-93fa-085356f47045.png)

# Docker为什么比VM快?
1. Docker有着比虚拟机更少的抽象层，由于Docker不需要Hypervisor实现硬件资源虚拟化，运行在Docker容器上的程序直接使用的都是实际物理机的硬件资源，因此在Cpu、内存利用率上Docker将会在效率上有明显优势。
2. Docker利用的是宿主机的内核，而不需要Guest OS，因此，当新建一个容器时，Docker不需要和虚拟机一样重新加载一个操作系统，避免了引导、加载操作系统内核这个比较费时费资源的过程，当新建一个虚拟机时，虚拟机软件需要加载Guest OS，这个新建过程是分钟级别的，而Docker由于直接利用宿主机的操作系统则省略了这个过程，因此新建一个Docker容器只需要几秒钟。

![image](https://user-images.githubusercontent.com/92672384/147016613-450e5145-ea9d-4eaa-87dd-00fada02addb.png)

|      | Docker容器 | 虚拟机（VM）        |
| :-----------: | :-----------: | :-----------: |
| 操作系统      | 与宿主机共享OS       |  宿主机OS上运行宿主机OS   |
| 存储大小  | 镜像小，便于存储与传输       |  镜像庞大（vmdk等）  |
| 运行性能  | 几乎无额外性能损失     |  操作系统额外的cpu、内存消耗 |
| 移植性  | 轻便、灵活、适用于Linux       |  笨重、与虚拟化技术耦合度高  |
| 硬件亲和性  | 面向软件开发者     |  面向硬件运维者  |
