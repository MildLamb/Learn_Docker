# Redis集群部署
```bash
for port in $(seq 1 6); \
do \
mkdir -p /mydata/redis/node-${port}/conf
touch /mydata/redis/node-${port}/conf/redis.conf
cat  << EOF > /mydata/redis/node-${port}/conf/redis.conf
port 6379 
bind 0.0.0.0
cluster-enabled yes 
cluster-config-file nodes.conf
cluster-node-timeout 5000
cluster-announce-ip 172.38.0.1${port}
cluster-announce-port 6379
cluster-announce-bus-port 16379
appendonly yes
EOF
done
```
```bash
# 创建一个自定义网络
[root@VM-16-14-centos conf]# docker network create redis --subnet 172.38.0.0/16
2ca312b985182be28a8c58c489ac9065e54811964366c23830d58e4106378abe

# 执行脚本，编写redis集群配置文件
[root@VM-16-14-centos ~]# for port in $(seq 1 6); \
> do \
> mkdir -p /mydata/redis/node-${port}/conf
> touch /mydata/redis/node-${port}/conf/redis.conf
> cat << EOF >/mydata/redis/node-${port}/conf/redis.conf
> port 6379 
> bind 0.0.0.0
> cluster-enabled yes 
> cluster-config-file nodes.conf
> cluster-node-timeout 5000
> cluster-announce-ip 172.38.0.1${port}
> cluster-announce-port 6379
> cluster-announce-bus-port 16379
> appendonly yes
> EOF
> done
[root@VM-16-14-centos ~]# cd /mydata
[root@VM-16-14-centos mydata]# ls
redis
[root@VM-16-14-centos mydata]# cd redis
[root@VM-16-14-centos redis]# ls
node-1  node-2  node-3  node-4  node-5  node-6
[root@VM-16-14-centos redis]# cd node-1
[root@VM-16-14-centos node-1]# ls
conf
[root@VM-16-14-centos node-1]# cd conf
[root@VM-16-14-centos conf]# ls
redis.conf

# 启动redis集群
[root@VM-16-14-centos conf]# docker run -p 6371:6379 -p 16371:16379 --name redis-1 -v /mydata/redis/node-1/data:/data -v /mydata/redis/node-1/conf/redis.conf:/etc/redis/redis.conf -d --net redis --ip 172.38.0.11 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf

[root@VM-16-14-centos conf]# docker run -p 6372:6379 -p 16372:16379 --name redis-2 -v /mydata/redis/node-2/data:/data -v /mydata/redis/node-2/conf/redis.conf:/etc/redis/redis.conf -d --net redis --ip 172.38.0.12 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf

[root@VM-16-14-centos conf]# docker run -p 6373:6379 -p 16373:16379 --name redis-3 -v /mydata/redis/node-3/data:/data -v /mydata/redis/node-3/conf/redis.conf:/etc/redis/redis.conf -d --net redis --ip 172.38.0.13 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf

[root@VM-16-14-centos conf]# docker run -p 6374:6379 -p 16374:16379 --name redis-4 -v /mydata/redis/node-4/data:/data -v /mydata/redis/node-4/conf/redis.conf:/etc/redis/redis.conf -d --net redis --ip 172.38.0.14 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf

[root@VM-16-14-centos conf]# docker run -p 6375:6379 -p 16375:16379 --name redis-5 -v /mydata/redis/node-5/data:/data -v /mydata/redis/node-5/conf/redis.conf:/etc/redis/redis.conf -d --net redis --ip 172.38.0.15 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf

[root@VM-16-14-centos conf]# docker run -p 6376:6379 -p 16376:16379 --name redis-6 -v /mydata/redis/node-6/data:/data -v /mydata/redis/node-6/conf/redis.conf:/etc/redis/redis.conf -d --net redis --ip 172.38.0.16 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf

# 查看docker容器，看是否开启6个服务
[root@VM-16-14-centos conf]# docker ps
CONTAINER ID   IMAGE                    COMMAND                  CREATED              STATUS              PORTS                                                                                      NAMES
19d7a8db6b74   redis:5.0.9-alpine3.11   "docker-entrypoint.s…"   6 seconds ago        Up 6 seconds        0.0.0.0:6376->6379/tcp, :::6376->6379/tcp, 0.0.0.0:16376->16379/tcp, :::16376->16379/tcp   redis-6
0d4cc2d63783   redis:5.0.9-alpine3.11   "docker-entrypoint.s…"   37 seconds ago       Up 36 seconds       0.0.0.0:6375->6379/tcp, :::6375->6379/tcp, 0.0.0.0:16375->16379/tcp, :::16375->16379/tcp   redis-5
043c6f41490d   redis:5.0.9-alpine3.11   "docker-entrypoint.s…"   About a minute ago   Up About a minute   0.0.0.0:6372->6379/tcp, :::6372->6379/tcp, 0.0.0.0:16372->16379/tcp, :::16372->16379/tcp   redis-2
311d486778ef   redis:5.0.9-alpine3.11   "docker-entrypoint.s…"   2 minutes ago        Up 2 minutes        0.0.0.0:6374->6379/tcp, :::6374->6379/tcp, 0.0.0.0:16374->16379/tcp, :::16374->16379/tcp   redis-4
3157d890947f   redis:5.0.9-alpine3.11   "docker-entrypoint.s…"   5 minutes ago        Up 5 minutes        0.0.0.0:6373->6379/tcp, :::6373->6379/tcp, 0.0.0.0:16373->16379/tcp, :::16373->16379/tcp   redis-3
b70a22388d80   redis:5.0.9-alpine3.11   "docker-entrypoint.s…"   27 minutes ago       Up 27 minutes       0.0.0.0:6371->6379/tcp, :::6371->6379/tcp, 0.0.0.0:16371->16379/tcp, :::16371->16379/tcp   redis-1

# 进入一个容器，开启集群
[root@VM-16-14-centos conf]# docker exec -it redis-1 /bin/sh
/data # ls
appendonly.aof  nodes.conf
# 开启集群
/data # redis-cli --cluster create 172.38.0.11:6379 172.38.0.12:6379 172.38.0.13:6379 172.38.0.14:6379 172.38.0.15:6379 172.38.0.16:6379 --cluster-replicas 1
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 172.38.0.15:6379 to 172.38.0.11:6379
Adding replica 172.38.0.16:6379 to 172.38.0.12:6379
Adding replica 172.38.0.14:6379 to 172.38.0.13:6379
M: af74e9f02e07fee3c91d81a9542be4d726403e7d 172.38.0.11:6379
   slots:[0-5460] (5461 slots) master
M: 57f26586465247f320fde59b1c89c7659126afe0 172.38.0.12:6379
   slots:[5461-10922] (5462 slots) master
M: 66181630051d3ab4256eb4ee6478ebaab61a1b05 172.38.0.13:6379
   slots:[10923-16383] (5461 slots) master
S: 0ffcdcf37af55fca6665a63f534d1a74747f7cff 172.38.0.14:6379
   replicates 66181630051d3ab4256eb4ee6478ebaab61a1b05
S: caf4d116501f032f79595abf2ff468404647a6f3 172.38.0.15:6379
   replicates af74e9f02e07fee3c91d81a9542be4d726403e7d
S: 0328ca87fdb7537a1f863890cfd8ca02991fd017 172.38.0.16:6379
   replicates 57f26586465247f320fde59b1c89c7659126afe0
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
..
>>> Performing Cluster Check (using node 172.38.0.11:6379)
M: af74e9f02e07fee3c91d81a9542be4d726403e7d 172.38.0.11:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: 57f26586465247f320fde59b1c89c7659126afe0 172.38.0.12:6379
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 0ffcdcf37af55fca6665a63f534d1a74747f7cff 172.38.0.14:6379
   slots: (0 slots) slave
   replicates 66181630051d3ab4256eb4ee6478ebaab61a1b05
M: 66181630051d3ab4256eb4ee6478ebaab61a1b05 172.38.0.13:6379
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: 0328ca87fdb7537a1f863890cfd8ca02991fd017 172.38.0.16:6379
   slots: (0 slots) slave
   replicates 57f26586465247f320fde59b1c89c7659126afe0
S: caf4d116501f032f79595abf2ff468404647a6f3 172.38.0.15:6379
   slots: (0 slots) slave
   replicates af74e9f02e07fee3c91d81a9542be4d726403e7d
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.

# 以集群方式进入redis，查看集群信息
# 以集群方式进入redis
/data # redis-cli -c

# 查看集群信息
127.0.0.1:6379> cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:6
cluster_my_epoch:1
cluster_stats_messages_ping_sent:235
cluster_stats_messages_pong_sent:255
cluster_stats_messages_sent:490
cluster_stats_messages_ping_received:250
cluster_stats_messages_pong_received:235
cluster_stats_messages_meet_received:5
cluster_stats_messages_received:490

# 向redis中存储一条数据
127.0.0.1:6379> set name kindred
# 返回哪台机器存储了这条信息
-> Redirected to slot [5798] located at 172.38.0.12:6379
OK

# 查看redis集群中的主从关系，master为主，slave为从，可以通过cluster nodes查看主从关系
172.38.0.12:6379> cluster nodes
57f26586465247f320fde59b1c89c7659126afe0 172.38.0.12:6379@16379 myself,master - 0 1640837484000 2 connected 5461-10922
66181630051d3ab4256eb4ee6478ebaab61a1b05 172.38.0.13:6379@16379 master - 0 1640837485000 3 connected 10923-16383
0328ca87fdb7537a1f863890cfd8ca02991fd017 172.38.0.16:6379@16379 slave 57f26586465247f320fde59b1c89c7659126afe0 0 1640837485000 2 connected
0ffcdcf37af55fca6665a63f534d1a74747f7cff 172.38.0.14:6379@16379 slave 66181630051d3ab4256eb4ee6478ebaab61a1b05 0 1640837485000 4 connected
af74e9f02e07fee3c91d81a9542be4d726403e7d 172.38.0.11:6379@16379 master - 0 1640837485000 1 connected 0-5460
caf4d116501f032f79595abf2ff468404647a6f3 172.38.0.15:6379@16379 slave af74e9f02e07fee3c91d81a9542be4d726403e7d 0 1640837485979 5 connected

# 关闭刚刚存储信息的redis容器，测试从机是否会顶替宕机的主机实现redis高可用
172.38.0.12:6379> get name
Could not connect to Redis at 172.38.0.12:6379: Host is unreachable
(32.53s)
not connected> get name
Could not connect to Redis at 172.38.0.12:6379: Host is unreachable
not connected> exit
/data # redis-cli -c
# 再次获取，成功从12的从机16获取到数据
127.0.0.1:6379> get name
-> Redirected to slot [5798] located at 172.38.0.16:6379
"kindred
```
