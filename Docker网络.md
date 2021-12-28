# Docker网络
```shell
[root@VM-16-14-centos /]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo    # 本机回环地址
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 52:54:00:28:a5:c9 brd ff:ff:ff:ff:ff:ff
    inet 10.0.16.14/22 brd 10.0.19.255 scope global eth0   # 腾讯云内网地址
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fe28:a5c9/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:a9:a8:8c:d9 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0   # docker0地址
       valid_lft forever preferred_lft forever
    inet6 fe80::42:a9ff:fea8:8cd9/64 scope link 
       valid_lft forever preferred_lft forever
```
### 三个网络，docker是如何处理容器网络访问的?
```shell
# 启动镜像
[root@VM-16-14-centos /]# docker run -d -P --name tomcat01 tomcat:9.0
a2e81768215ac16a1618407ab001c1ac54ff5bd9e850768c3f669ac14deb0aa9

# 想不进入容器查看ip信息，但是报错OCI runtime exec failed，说明容器内没有这个命令，进容器装一个就行了apt update && apt install -y iproute2
[root@VM-16-14-centos /]# docker exec -it tomcat01 ip addr
OCI runtime exec failed: exec failed: container_linux.go:380: starting container process caused: exec: "ip": executable file not found in $PATH: unknown

# 等待安装完毕后，再次执行命令
# 发现容器启动的时候会得到一个 eth0@if95 的随机ip地址，由docker分配
[root@VM-16-14-centos /]# docker exec -it tomcat01 ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
94: eth0@if95: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
       
# 思考，linux能不能ping通容器内部，可以
[root@VM-16-14-centos /]# ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.058 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.066 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.058 ms
64 bytes from 172.17.0.2: icmp_seq=4 ttl=64 time=0.058 ms
64 bytes from 172.17.0.2: icmp_seq=5 ttl=64 time=0.067 ms
64 bytes from 172.17.0.2: icmp_seq=6 ttl=64 time=0.061 ms
^C
--- 172.17.0.2 ping statistics ---
6 packets transmitted, 6 received, 0% packet loss, time 4999ms
rtt min/avg/max/mdev = 0.058/0.061/0.067/0.007 ms
```
### 原理
1. 我们每启动一个docker容器，docker就会给docker容器分配一个ip，我们只要安装了docker，就会有一个网卡 docker0
2. 桥接模式，使用的技术是 evth-pair 技术
3. 启动容器后，再次测试 ip addr
```shell
[root@VM-16-14-centos /]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 52:54:00:28:a5:c9 brd ff:ff:ff:ff:ff:ff
    inet 10.0.16.14/22 brd 10.0.19.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fe28:a5c9/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:a9:a8:8c:d9 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:a9ff:fea8:8cd9/64 scope link 
       valid_lft forever preferred_lft forever
95: vethe732b8f@if94: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether 16:bf:b5:4e:03:85 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::14bf:b5ff:fe4e:385/64 scope link 
       valid_lft forever preferred_lft forever
```
4. 我们发现这个容器带来的网卡，都是一对一对的，比如上面的 95---94  
veth-pair 就是一对的虚拟设备接口，他们都是成对出现的，一段连着协议，一段彼此相连  
正因为这个特性，evth-pair充当一个桥梁，连接各种虚拟网络设备  
5. 再启动一个容器tomcat02，测试tomcat01和tomcat02之间是否可以ping通
```bash
# 和上面安装ip addr命令一样，先进入容器，准备安装 ping 工具
# 安装ping工具的命令 apt-get install inetutils-ping
# 安装后测试是否可以ping通，可以
[root@VM-16-14-centos /]# docker exec -it tomcat01 ping 172.17.0.3
PING 172.17.0.3 (172.17.0.3): 56 data bytes
64 bytes from 172.17.0.3: icmp_seq=0 ttl=64 time=0.120 ms
64 bytes from 172.17.0.3: icmp_seq=1 ttl=64 time=0.113 ms
64 bytes from 172.17.0.3: icmp_seq=2 ttl=64 time=0.107 ms
^C--- 172.17.0.3 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max/stddev = 0.107/0.113/0.120/0.000 ms
```
![image](https://user-images.githubusercontent.com/92672384/147530185-f63f09c6-1c1f-493e-a841-6d04069bd7a3.png)

结论：tomcat01和tomcat02是共用一个路由器docker0  
所有的容器不指定网络的情况下，都是用docker0路由的，docker会给我们的容器分配一个默认的可用ip  

![image](https://user-images.githubusercontent.com/92672384/147530530-14fc2ff2-cd69-47d2-a535-3ef007467a52.png)
Docker使用的是Linux的桥接，宿主机中是一个Docker容器的网桥docker0  
只要容器删除，对应的网桥对也删除了  
