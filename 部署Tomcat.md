# 部署tomcat
```bash
# 下载tomcat
[root@VM-16-14-centos /]# docker pull tomcat:9.0

# 启动运行
[root@VM-16-14-centos /]# docker run -d -p 8080:8080 --name tomcat01 tomcat:9.0

# 发现外网访问不了

# 进入容器，查看目录，进入webapps目录，发现是空的
# 原因：阿里云镜像，默认是最小的镜像，所有不必要的都剔除掉，只保证最小的可运行的环境
# 解决方法，把webapps.dist目录下的文件拷贝到webapps下即可
[root@VM-16-14-centos /]# docker exec -it tomcat01 /bin/bash
root@3b73888aeb56:/usr/local/tomcat# ls -all
total 172
drwxr-xr-x 1 root root  4096 Dec 22 17:16 .
drwxr-xr-x 1 root root  4096 Dec 22 17:00 ..
-rw-r--r-- 1 root root 18970 Dec  2 14:30 BUILDING.txt
-rw-r--r-- 1 root root  6210 Dec  2 14:30 CONTRIBUTING.md
-rw-r--r-- 1 root root 57092 Dec  2 14:30 LICENSE
-rw-r--r-- 1 root root  2333 Dec  2 14:30 NOTICE
-rw-r--r-- 1 root root  3378 Dec  2 14:30 README.md
-rw-r--r-- 1 root root  6898 Dec  2 14:30 RELEASE-NOTES
-rw-r--r-- 1 root root 16507 Dec  2 14:30 RUNNING.txt
drwxr-xr-x 2 root root  4096 Dec 22 17:16 bin
drwxr-xr-x 1 root root  4096 Dec 23 03:21 conf
drwxr-xr-x 2 root root  4096 Dec 22 17:16 lib
drwxrwxrwx 1 root root  4096 Dec 23 03:21 logs
drwxr-xr-x 2 root root  4096 Dec 22 17:16 native-jni-lib
drwxrwxrwx 2 root root  4096 Dec 22 17:16 temp
drwxr-xr-x 2 root root  4096 Dec 22 17:16 webapps
drwxr-xr-x 7 root root  4096 Dec  2 14:30 webapps.dist
drwxrwxrwx 2 root root  4096 Dec  2 14:30 work
root@3b73888aeb56:/usr/local/tomcat# cd webapps
root@3b73888aeb56:/usr/local/tomcat/webapps# ls
```
