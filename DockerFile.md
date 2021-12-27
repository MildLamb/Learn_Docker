# DockerFile介绍
dockerfile是用来构建docker镜像的文件，命令参数脚本  
构建步骤：  
1. 编写一个dockerfile文件
2. docker build 构建成为一个镜像
3. docker run 运行镜像
4. docker push 发布镜像 (dockerhub,阿里云等等...)

# DockerFile构建过程
**基础知识:**  
1. 每个保留关键字(指令)都是必须是大写字母
2. 执行从上到下顺序执行
3. #表示注释
4. 每一个指令都会创建提交一个新的镜像层

- DockerFile：构建文件，定义了一切的步骤，源代码
- DockerImages：通过DockerFile构建生成的镜像，最终发布和运行的产品
- Docker容器：容器就是镜像运行起来提供服务

# Dockerfile常用命令
![image](https://user-images.githubusercontent.com/92672384/147425633-dca18df5-3c7a-496c-8898-889a9d59df07.png)

![image](https://user-images.githubusercontent.com/92672384/147426097-10628c9e-bbd9-4c52-95ce-7ae94b5c11c8.png)

```txt
FROM           # 基础镜像，一切从这里开始构建
MAINTAINER     # 镜像是谁写的，姓名 + 邮箱
RUN            # 镜像构建的时候需要运行的命令
ADD            # 步骤，添加tomcat镜像，tomcat压缩包，添加内容
WORKDIR        # 镜像的工作目录
VOLUME         # 挂载的目录
EXPOSE         # 暴露的端口配置
CMD            # 指定这个容器启动的时候要运行的命令，只有最后一个会生效，可以被替代
ENTRYPOINT     # 指定这个容器启动的时候要运行的命令，可以追加命令
ONBUILD        # 当构建一个被继承的Dockerfile，这个时候就会运行ONBUILD的指令，出发指令
COPY           # 类似ADD，将我们的文件拷贝到镜像中
ENV            # 构建的时候设置环境变量
```
