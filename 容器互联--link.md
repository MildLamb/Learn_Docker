# 容器互联使用 --link
```bash
# 直接使用容器名ping，是ping不通的
[root@VM-16-14-centos ~]# docker exec -it tomcat01 ping tomcat02
ping: unknown host
# 我们可以让容器互联，这样就可以使用容器名ping通了
[root@VM-16-14-centos ~]# docker run -d -P --name tomcat03 --link tomcat01 tomcat
```
