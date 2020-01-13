# Redis哨兵（Sentinel）模式
主从切换技术的方法是：当主服务器宕机后，需要手动把一台从服务器切换为主服务器，这就需要人工干预，费事费力，还会造成一段时间内服务不可用。这不是一种推荐的方式，更多时候，我们优先考虑哨兵模式。

哨兵模式是一种特殊的模式，首先Redis提供了哨兵的命令，哨兵是一个独立的进程，作为进程，它会独立运行。其原理是哨兵通过发送命令，等待Redis服务器响应，从而监控运行的多个Redis实例。

当哨兵监测到master宕机，会自动将slave切换成master，然后通过发布订阅模式通知其他的从服务器，修改配置文件，让它们切换主机

# 搭建哨兵（Sentinel）模式

## 先搭建master和slave：一主二从
先准备三个redis配置文件：redis_6380.conf、redis_6381.conf、redis_6382.conf

redis_6380.conf的配置文件
```
#修改的端口
port 6380  

#密码 三台redis服务密码都保持一致
requirepass "123456"
#后台运行
daemonize yes

supervised no

#修改pid 每个配置文件都修改一下，不然出现问题
pidfile "/var/run/redis_6380.pid"

```
redis_6381.conf、redis_6382.conf的配置修改一下port 端口 pidfile

配置slave：
第一种方法在redis_6381.conf、redis_6382.conf配置中添加：

```
# 指定主服务器，注意：有关slaveof的配置只是配置从服务器，主服务器不需要配置
slaveof 192.168.11.128 6379
# 主服务器密码，注意：有关slaveof的配置只是配置从服务器，主服务器不需要配置
masterauth 123456
```

第二种方法：SLAVEOF 127.0.0.1 6380

启动三redis：

```
./src/redis-server  redis_6380.conf 
./src/redis-server  redis_6381.conf 
./src/redis-server  redis_6382.conf 
```



```
127.0.0.1:6380> INFO replication
# Replication
role:master
connected_slaves:2
slave0:ip=127.0.0.1,port=6381,state=online,offset=33640851,lag=1
slave1:ip=127.0.0.1,port=6382,state=online,offset=33640984,lag=0
master_replid:1f0533224ffd2a0bd14c7559076b270a0cc8ada9
master_replid2:bf28cef1d992f824d2a3314935d729e7c1e8f7e8
master_repl_offset:33640984
second_repl_offset:16670
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:32592409
repl_backlog_histlen:1048576

```


# 配置Sentinel哨兵模式

准备三个sentinel_26380.conf、 sentinel_26381.conf 、 sentinel_26382.conf
```
-rw-r--r-- 1 root root 61874 Jan 11 16:59 redis_6380.conf
-rw-r--r-- 1 root root 61874 Jan 11 16:56 redis_6381.conf
-rw-r--r-- 1 root root 61819 Jan 11 16:56 redis_6382.conf
-rw-r--r-- 1 root root 10155 Jan 11 16:56 sentinel_26380.conf
-rw-r--r-- 1 root root 10061 Jan 11 16:56 sentinel_26381.conf
-rw-r--r-- 1 root root 10061 Jan 11 16:56 sentinel_26382.conf
```

修改配置文件： 三个配置只需要修改端口和pidfile就可以，其他一样
```
port 26380

pidfile "/var/run/redis-sentinel_23680.pid"

#启动后在后台运行
daemonize yes

#这儿ip三个配置文件都是一样的，填master的ip， slave是自发现
sentinel monitor mymaster 127.0.0.1 6380 2

sentinel auth-pass mymaster 123456
```

```
port 26381

pidfile "/var/run/redis-sentinel_23681.pid"

#启动后在后台运行
daemonize yes

#这儿ip三个配置文件都是一样的，填master的ip， slave是自发现
sentinel monitor mymaster 127.0.0.1 6380 2

sentinel auth-pass mymaster 123456
```

```
port 26382

pidfile "/var/run/redis-sentinel_23682.pid"

#启动后在后台运行
daemonize yes

#这儿ip三个配置文件都是一样的，填master的ip， slave是自发现
sentinel monitor mymaster 127.0.0.1 6380 2

sentinel auth-pass mymaster 123456
```


解释一下：

```
sentinel monitor <master-name> <ip> <redis-port> <quorum>
告诉sentinel去监听地址为ip:port的一个master，这里的master-name可以自定义，quorum是一个数字，指明当有多少个sentinel认为一个master失效时，master才算真正失效

sentinel auth-pass <master-name> <password>
设置连接master和slave时的密码，注意的是sentinel不能分别为master和slave设置不同的密码，因此master和slave的密码应该设置相同。

sentinel down-after-milliseconds <master-name> <milliseconds> 
这个配置项指定了需要多少失效时间，一个master才会被这个sentinel主观地认为是不可用的。 单位是毫秒，默认为30秒

sentinel parallel-syncs <master-name> <numslaves> 
这个配置项指定了在发生failover主备切换时最多可以有多少个slave同时对新的master进行 同步，这个数字越小，完成failover所需的时间就越长，但是如果这个数字越大，就意味着越 多的slave因为replication而不可用。可以通过将这个值设为 1 来保证每次只有一个slave 处于不能处理命令请求的状态。

sentinel failover-timeout <master-name> <milliseconds>
failover-timeout 可以用在以下这些方面：     
1. 同一个sentinel对同一个master两次failover之间的间隔时间。   
2. 当一个slave从一个错误的master那里同步数据开始计算时间。直到slave被纠正为向正确的master那里同步数据时。    
3.当想要取消一个正在进行的failover所需要的时间。    
4.当进行failover时，配置所有slaves指向新的master所需的最大时间。不过，即使过了这个超时，slaves依然会被正确配置为指向master，但是就不按parallel-syncs所配置的规则来了。

```

## 启动哨兵模式：

```
./src/redis-cli sentinel/sentinel_26380.conf
./src/redis-cli sentinel/sentinel_26381.conf
./src/redis-cli sentinel/sentinel_26382.conf

```

# 运行分析故障转移


```
[root@chen 16:32 ~/redis-5.0.7]$ps -ef|grep redis
root      11334  52676  0 16:20 pts/0    00:00:00 ./src/redis-cli -p 6382
root      12209 119392  0 16:32 pts/1    00:00:00 grep --color=auto redis
root      27328      1  0 Jan11 ?        00:07:05 ./src/redis-server 127.0.0.1:6381
root      27333      1  0 Jan11 ?        00:07:44 ./src/redis-server 127.0.0.1:6382
root      27707      1  0 Jan11 ?        00:11:10 ./src/redis-sentinel *:26380 [sentinel]
root      27712      1  0 Jan11 ?        00:10:11 ./src/redis-sentinel *:26381 [sentinel]
root      27717      1  0 Jan11 ?        00:10:11 ./src/redis-sentinel *:26382 [sentinel]
root      27762      1  0 Jan11 ?        00:07:01 ./src/redis-server 127.0.0.1:6380
```

自己尝**试kill -9** 杀死master的redis服务， 然后**redis-cli** 连接上去查看那台服务会成为master


```
[root@chen 16:20 ~/redis-5.0.7]$./src/redis-cli -p 6382
127.0.0.1:6382> INFO replication
# Replication
role:master
connected_slaves:2
slave0:ip=127.0.0.1,port=6381,state=online,offset=33640851,lag=1
slave1:ip=127.0.0.1,port=6380,state=online,offset=33640984,lag=0
master_replid:1f0533224ffd2a0bd14c7559076b270a0cc8ada9
master_replid2:bf28cef1d992f824d2a3314935d729e7c1e8f7e8
master_repl_offset:33640984
second_repl_offset:16670
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:32592409
repl_backlog_histlen:104857
```
