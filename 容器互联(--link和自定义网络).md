# 容器互联使用 --link
```bash
# 直接使用容器名ping，是ping不通的
[root@VM-16-14-centos ~]# docker exec -it tomcat01 ping tomcat02
ping: unknown host
# 我们可以让容器互联，这样就可以使用容器名ping通了
[root@VM-16-14-centos ~]# docker run -d -P --name tomcat03 --link tomcat01 tomcat
6f086775b2f60e04746410c0389fb6a86258b4793113ea73e5b4983e204b5b7f
[root@VM-16-14-centos ~]# docker exec -it tomcat03 ping tomcat01
PING tomcat01 (172.17.0.2): 56 data bytes
64 bytes from 172.17.0.2: icmp_seq=0 ttl=64 time=0.127 ms
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.093 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.107 ms
^C--- tomcat01 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max/stddev = 0.093/0.109/0.127/0.000 ms

# 反向可以ping通吗？ tomcat01pingtomcat03
[root@VM-16-14-centos ~]# docker exec -it tomcat01 ping tomcat03
ping: unknown host

# 查看docker的网络 docker network
[root@VM-16-14-centos ~]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
f44755dc56ff   bridge    bridge    local
7bbe6257b76e   host      host      local
9751ef0999e3   none      null      local

# 查看网络详细信息
[root@VM-16-14-centos ~]# docker network inspect f44755dc56ff
[
    {
        "Name": "bridge",
        "Id": "f44755dc56ff912f03085a028a3a5acb95d0aacada6d6276f6332afedde6f39b",
        "Created": "2021-12-28T08:54:52.387423741+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "13e3ebc8ee2a0300ca7da26808208563263c51ad32853a42e4c3272c08652b00": {
                "Name": "tomcat02",
                "EndpointID": "60e1c24b9e1456c6a6df76f18875cf66beeede765af02b7a77e31191903c246f",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            },
            "6f086775b2f60e04746410c0389fb6a86258b4793113ea73e5b4983e204b5b7f": {
                "Name": "tomcat03",
                "EndpointID": "11fb9213fad7dab7a170215fb5d47d35d9b59dbfcd782671901199afed5d9900",
                "MacAddress": "02:42:ac:11:00:04",
                "IPv4Address": "172.17.0.4/16",
                "IPv6Address": ""
            },
            "7a1c2461736d2b0894f48edfe49c99fc4409476baac6a4d56519d38f4cbae060": {
                "Name": "tomcat01",
                "EndpointID": "6e6439d0bd5df5bbedcb82eae6886bdfd2097e6c5afa10b840c495b4f6fea57e",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]

# 进入tomcat03容器，查看/etc/hosts文件
[root@VM-16-14-centos ~]# docker exec -it tomcat03 /bin/bash
root@6f086775b2f6:/usr/local/tomcat# cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.2	tomcat01 7a1c2461736d   # tomcat03绑定了tomcat01，因此可以ping通
172.17.0.4	6f086775b2f6
```
本质探究：--link 就是在容器的hosts配置中增加一个被link的容器的ip，使得它们之间能够ping通  
我们现在使用Docker不建议使用--link了  
使用自定义网络！不使用docker0  
docker0问题：不支持容器ming连接访问  

# 自定义网络
### 查看所有的docker网络
```bash
[root@VM-16-14-centos ~]# docker network --help

Usage:  docker network COMMAND

Manage networks

Commands:
  connect     Connect a container to a network
  create      Create a network
  disconnect  Disconnect a container from a network
  inspect     Display detailed information on one or more networks
  ls          List networks
  prune       Remove all unused networks
  rm          Remove one or more networks

Run 'docker network COMMAND --help' for more information on a command.
# 查看所有的docker网络
[root@VM-16-14-centos ~]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
f44755dc56ff   bridge    bridge    local
7bbe6257b76e   host      host      local
9751ef0999e3   none      null      local
```
### 网络模式
- birdge：桥接模式（默认）
- none：不配置网络
- host：和宿主机共享网络
- contaienr：容器网络连通（用的少，局限性大）

### 测试
```bash
# 我们直接启动的命令，会默认带上 --net bridge，这个就是我们的docker0
[root@VM-16-14-centos ~]# docker run -d -P --name tomcat01 --net bridge tomcat
# docker0特点：默认的，无法使用容器名访问  --link可以实现容器名访问

# 我们可以创建一个自定义网络
[root@VM-16-14-centos ~]# docker network create --help

Usage:  docker network create [OPTIONS] NETWORK

Create a network

Options:
      --attachable           Enable manual container attachment
      --aux-address map      Auxiliary IPv4 or IPv6 addresses used by Network driver (default map[])
      --config-from string   The network from which to copy the configuration
      --config-only          Create a configuration only network
  -d, --driver string        Driver to manage the Network (default "bridge")
      --gateway strings      IPv4 or IPv6 Gateway for the master subnet
      --ingress              Create swarm routing-mesh network
      --internal             Restrict external access to the network
      --ip-range strings     Allocate container ip from a sub-range
      --ipam-driver string   IP Address Management Driver (default "default")
      --ipam-opt map         Set IPAM driver specific options (default map[])
      --ipv6                 Enable IPv6 networking
      --label list           Set metadata on a network
  -o, --opt map              Set driver specific options (default map[])
      --scope string         Control the network's scope
      --subnet strings       Subnet in CIDR format that represents a network segment
      
# 创建网络
# --driver bridge                网络模式
# --subnet 192.168.0.0/16        子网地址
# --gateway 192.168.0.1          网关地址
[root@VM-16-14-centos ~]# docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
fcc80c004cb60f336e18bc39ca438bd17d502ddca397e408b7e6b8d25f0e128a
[root@VM-16-14-centos ~]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
f44755dc56ff   bridge    bridge    local
7bbe6257b76e   host      host      local
fcc80c004cb6   mynet     bridge    local
9751ef0999e3   none      null      local

# 查看自己自定义的网络的信息
[root@VM-16-14-centos ~]# docker network inspect mynet
[
    {
        "Name": "mynet",
        "Id": "fcc80c004cb60f336e18bc39ca438bd17d502ddca397e408b7e6b8d25f0e128a",
        "Created": "2021-12-29T11:21:43.620295815+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]

# 使用我们自定义的网络来启动容器
[root@VM-16-14-centos ~]# docker run -d -P --name tomcat01 --net mynet tomcat
554b51056477938f9e5a8eee7a54068a869a4ac29b9be117eeed4650a6ec47b2
[root@VM-16-14-centos ~]# docker run -d -P --name tomcat02 --net mynet tomcat
d305c145cbd932d98308409ac8c7cb00e2bba0cba222e35829ec8a711415b48a

# 再次查看我们自定义的网络信息，会有我们创建的容器的网络信息
[root@VM-16-14-centos ~]# docker network inspect mynet
[
    {
        "Name": "mynet",
        "Id": "fcc80c004cb60f336e18bc39ca438bd17d502ddca397e408b7e6b8d25f0e128a",
        "Created": "2021-12-29T11:21:43.620295815+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "554b51056477938f9e5a8eee7a54068a869a4ac29b9be117eeed4650a6ec47b2": {
                "Name": "tomcat01",
                "EndpointID": "138e5443986ff9593f627ccd52d517c5c8c02e43118e94f7a9bfc2a7b41f73b2",
                "MacAddress": "02:42:c0:a8:00:02",
                "IPv4Address": "192.168.0.2/16",
                "IPv6Address": ""
            },
            "d305c145cbd932d98308409ac8c7cb00e2bba0cba222e35829ec8a711415b48a": {
                "Name": "tomcat02",
                "EndpointID": "034ab8d774d35c1916ee17be4af7e3dd60509323b5aa3183384b064297a16955",
                "MacAddress": "02:42:c0:a8:00:03",
                "IPv4Address": "192.168.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]

# 在不 --link的情况下，直接使用容器名 ping ，可以ping通
[root@VM-16-14-centos ~]# docker exec -it tomcat01 ping tomcat02
PING tomcat02 (192.168.0.3) 56(84) bytes of data.
64 bytes from tomcat02.mynet (192.168.0.3): icmp_seq=1 ttl=64 time=0.085 ms
64 bytes from tomcat02.mynet (192.168.0.3): icmp_seq=2 ttl=64 time=0.073 ms
64 bytes from tomcat02.mynet (192.168.0.3): icmp_seq=3 ttl=64 time=0.076 ms
^C
--- tomcat02 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2000ms
rtt min/avg/max/mdev = 0.073/0.078/0.085/0.005 ms
```
使用自定义网络的好处：  
不同的集群可以使用不同的网络，保证集群是健康和安全的  

# 网络连通
```bash
[root@VM-16-14-centos ~]# docker network --help

Usage:  docker network COMMAND

Manage networks

Commands:
  connect     Connect a container to a network   # 连接一个容器到一个网络
  create      Create a network
  disconnect  Disconnect a container from a network
  inspect     Display detailed information on one or more networks
  ls          List networks
  prune       Remove all unused networks
  rm          Remove one or more networks

Run 'docker network COMMAND --help' for more information on a command.
```
## 测试打通 tomcat01 到 tomcat-docker-01
```bash
# 查看帮助
[root@VM-16-14-centos ~]# docker network connect --help

Usage:  docker network connect [OPTIONS] NETWORK CONTAINER

Connect a container to a network

Options:
      --alias strings           Add network-scoped alias for the container
      --driver-opt strings      driver options for the network
      --ip string               IPv4 address (e.g., 172.30.100.104)
      --ip6 string              IPv6 address (e.g., 2001:db8::33)
      --link list               Add link to another container
      --link-local-ip strings   Add a link-local address for the container
      
# 连通容器到网络
[root@VM-16-14-centos ~]# docker network connect mynet tomcat-docker-01

# 观察网络的变化
[root@VM-16-14-centos ~]# docker network inspect mynet
[
    {
        "Name": "mynet",
        "Id": "fcc80c004cb60f336e18bc39ca438bd17d502ddca397e408b7e6b8d25f0e128a",
        "Created": "2021-12-29T11:21:43.620295815+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "554b51056477938f9e5a8eee7a54068a869a4ac29b9be117eeed4650a6ec47b2": {
                "Name": "tomcat01",
                "EndpointID": "138e5443986ff9593f627ccd52d517c5c8c02e43118e94f7a9bfc2a7b41f73b2",
                "MacAddress": "02:42:c0:a8:00:02",
                "IPv4Address": "192.168.0.2/16",
                "IPv6Address": ""
            },
            "bb0e1063aa9e303aa6465098384c778175f7d8a3c6cb1b16e6bcbc8a6dafb51b": {
                # 发现tomcat-docker-01，直接被加入到了网络中
                "Name": "tomcat-docker-01",
                "EndpointID": "428840e2e6f5a30d2f402870ecf1a4b00d8931608669dd2a4712bb493699aa84",
                "MacAddress": "02:42:c0:a8:00:04",
                "IPv4Address": "192.168.0.4/16",
                "IPv6Address": ""
            },
            "d305c145cbd932d98308409ac8c7cb00e2bba0cba222e35829ec8a711415b48a": {
                "Name": "tomcat02",
                "EndpointID": "034ab8d774d35c1916ee17be4af7e3dd60509323b5aa3183384b064297a16955",
                "MacAddress": "02:42:c0:a8:00:03",
                "IPv4Address": "192.168.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]


# 测试tomcat01(mynet网络)能否ping通tomcat-docker-01(docker0网络)?可以
[root@VM-16-14-centos ~]# docker exec -it tomcat01 ping tomcat-docker-01
PING tomcat-docker-01 (192.168.0.4) 56(84) bytes of data.
64 bytes from tomcat-docker-01.mynet (192.168.0.4): icmp_seq=1 ttl=64 time=0.090 ms
64 bytes from tomcat-docker-01.mynet (192.168.0.4): icmp_seq=2 ttl=64 time=0.071 ms
64 bytes from tomcat-docker-01.mynet (192.168.0.4): icmp_seq=3 ttl=64 time=0.069 ms
^C
--- tomcat-docker-01 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2000ms
rtt min/avg/max/mdev = 0.069/0.076/0.090/0.009 ms
```
**结论：**  
假设我们要跨网络操作，就需要使用docker network connect 将容器连接到网络
