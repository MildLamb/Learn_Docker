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
