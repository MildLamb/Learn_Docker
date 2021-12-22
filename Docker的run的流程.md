# run的执行流程
![image](https://user-images.githubusercontent.com/92672384/147015538-881ccf0f-ce4b-4326-bc18-49e7d02edd8b.png)

# 底层原理
Docker是怎么工作的?  
Docker是一个Client - Server结构的系统，Docker的守护进程运行在主机上，通过Socket从客户端访问  
DockerServer接收到DockerClient的指令，就会执行命令  
![image](https://user-images.githubusercontent.com/92672384/147016066-164c6ec2-5340-48af-93fa-085356f47045.png)
