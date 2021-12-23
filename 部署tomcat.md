# 部署tomcat
```bash
# 下载tomcat
[root@VM-16-14-centos /]# docker pull tomcat:9.0

# 启动运行
[root@VM-16-14-centos /]# docker run -d -p 8080:8080 --name tomcat01 tomcat
```
