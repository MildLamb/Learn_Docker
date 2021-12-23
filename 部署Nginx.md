# 部署Nginx
1. 搜索镜像 docker search nginx
2. 下载镜像 docker pull nginx
3. 运行测试 
```bash
[root@VM-16-14-centos /]# docker search nginx
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
nginx                             Official build of Nginx.                        16002     [OK]       
jwilder/nginx-proxy               Automated Nginx reverse proxy for docker con…   2103                 [OK]
richarvey/nginx-php-fpm           Container running Nginx + PHP-FPM capable of…   820                  [OK]
jc21/nginx-proxy-manager          Docker container for managing Nginx proxy ho…   296                  
linuxserver/nginx                 An Nginx container, brought to you by LinuxS…   161                  
tiangolo/nginx-rtmp               Docker image with Nginx using the nginx-rtmp…   148                  [OK]
jlesage/nginx-proxy-manager       Docker container for Nginx Proxy Manager        147                  [OK]
alfg/nginx-rtmp                   NGINX, nginx-rtmp-module and FFmpeg from sou…   112                  [OK]
nginxdemos/hello                  NGINX webserver that serves a simple page co…   80                   [OK]
privatebin/nginx-fpm-alpine       PrivateBin running on an Nginx, php-fpm & Al…   61                   [OK]
nginx/nginx-ingress               NGINX and  NGINX Plus Ingress Controllers fo…   59                   
nginxinc/nginx-unprivileged       Unprivileged NGINX Dockerfiles                  56                   
nginxproxy/nginx-proxy            Automated Nginx reverse proxy for docker con…   31                   
staticfloat/nginx-certbot         Opinionated setup for automatic TLS certs lo…   25                   [OK]
nginx/nginx-prometheus-exporter   NGINX Prometheus Exporter for NGINX and NGIN…   22                   
schmunk42/nginx-redirect          A very simple container to redirect HTTP tra…   19                   [OK]
centos/nginx-112-centos7          Platform for running nginx 1.12 or building …   16                   
centos/nginx-18-centos7           Platform for running nginx 1.8 or building n…   13                   
bitwarden/nginx                   The Bitwarden nginx web server acting as a r…   12                   
flashspys/nginx-static            Super Lightweight Nginx Image                   11                   [OK]
mailu/nginx                       Mailu nginx frontend                            10                   [OK]
webdevops/nginx                   Nginx container                                 9                    [OK]
sophos/nginx-vts-exporter         Simple server that scrapes Nginx vts stats a…   7                    [OK]
ansibleplaybookbundle/nginx-apb   An APB to deploy NGINX                          3                    [OK]
wodby/nginx                       Generic nginx                                   1                    [OK]
[root@VM-16-14-centos /]# docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
a2abf6c4d29d: Pull complete 
f3409a9a9e73: Pull complete 
9919a6cbae9c: Pull complete 
fc1ce43285d7: Pull complete 
1f01ab499216: Pull complete 
13cfaf79ff6d: Pull complete 
Digest: sha256:366e9f1ddebdb844044c2fafd13b75271a9f620819370f8971220c2b330a9254
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
[root@VM-16-14-centos /]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
nginx         latest    f6987c8d6ed5   47 hours ago   141MB
mysql         latest    3218b38490ce   47 hours ago   516MB
hello-world   latest    feb5d9fea6a5   3 months ago   13.3kB
centos        latest    5d0da3dc9764   3 months ago   231MB
[root@VM-16-14-centos /]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

# -d 后台运行
# --name 给容器起名
# -p 宿主机端口与容器端口映射
[root@VM-16-14-centos /]# docker run -d --name nginx01 -p 4399:80 nginx
cb72bafd2e4819a3c78125d4dced652e7aa927f0c52de3415ef2080dd7f48389
[root@VM-16-14-centos /]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                                   NAMES
cb72bafd2e48   nginx     "/docker-entrypoint.…"   3 seconds ago   Up 2 seconds   0.0.0.0:4399->80/tcp, :::4399->80/tcp   nginx01
[root@VM-16-14-centos /]# curl localhost:4399
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
