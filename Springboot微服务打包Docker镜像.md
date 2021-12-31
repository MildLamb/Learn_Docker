# Springboot微服务打包Docker镜像
1. 构建springboot项目
2. 打包应用
3. 编写dockerfile
```bash
FROM java:8

COPY *.jar /app.jar

CMD ["--server.port=8080"]

EXPOSE 8080

ENTRYPOINT ["java","-jar","/app.jar"]
```
4. 将jar包和dockerfile传到linux服务器
5. 构建镜像
6. 发布运行
