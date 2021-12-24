### 什么是portainer?
Docker图形化界面管理工具！提供一个后台面板供我们操作
```bash
[root@VM-16-14-centos ~]# docker run -d -p 8088:9000 \
> --restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
```
- 外网访问后，第一次要先先设置密码登录
- 登录后选local
- 就可以看到服务器的docker信息了

