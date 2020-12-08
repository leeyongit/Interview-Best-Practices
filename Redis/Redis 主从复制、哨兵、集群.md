# Redis 主从复制、哨兵、集群
开启 daemonize yes
* info replication
    - 打印主从复制的相关信息
* slaveof <ip> <port>
    - 成为某个实例的从服务器

## 复制原理
* 每次从机联通后，都会给主机发送sync命令
* 主机立刻进行存盘操作，发送RDB文件给从机
* 从机收到RDB文件后，进行全盘加载
* 之后每次主机的写操作，都会立刻发送给从机，从机执行相同的命令

## 薪火相传
* 上一个slave可以是下一个slave的master，slave同样可以接收其他slaves的连接和同步请求，那么该slave作为了链条中下一个的master，可以有效减轻master的写压力，去中心化降低风险。
* 用slaveof <ip> <port>
* 中途变更转向：会清除之前的数据，重新建立拷贝最新的
* 风险是一旦某个slave宕机，后面的slave都没发备份

## 哨兵模式(sentinel)

### 配置哨兵
* 调整为一主多从模式
* 新建sentinel.conf文件
* 在配置文件中填写内容：
    sentinel monitor mymaster 127.0.0.1 6379 1
* 其中mymaster为监控对象起的服务器名称，1为至少有多少个哨兵同意迁移的数量

### 启动哨兵
* 执行redis-sentinel /myredis/sentinel.conf

### 故障恢复
**新主登基**
1. 选择优先级靠前的
2. 选择偏移量最大的
3. 选择runid最小的从服务

    优先级在redis.conf中slave-priority 100
    偏移量是指获得原主数据最多的
    每个redis实例启动后都会随机生成一个40位的runid

**群扑俯首**
挑选出新的主服务之后，sentinel向原主服务的从服务发送slaveof新主服务的命令，复制新master

**旧主俯首**
当已下线的服务重新上线时，sentinel会向其发送slaveof命令，让其成为新主的从


## Redis 集群
解决问题：
* 容量不够，redis如何进行扩容？
* 并发写操作，redis如何分摊？

### 什么是集群
* Redis 集群实现了Redis的水平扩容，即启动N个redis节点，将整个数据库分布存储在这N个节点中，每个节点存储总数据的1/N。
* Redis 集群通过分区（partition) 来提供一定程度的可用性（availability)：即使集群中有一部分节点失效或者无法进行通信，集群也可以继续处理命令请求。

1. 安装ruby环境
```sh
yum install ruby
yum install rubygems
```
2. 拷贝redis-3.2.0.gem 到/opt目录下
3. 在opt 目录下执行 gem install --local redis-3.2.0.gem

### 安装redis cluster配置修改
* cluster-enabled yes 打开集群模式
* cluster-config-file nodes-6379.conf 设定节点配置文件名
* cluster-node-timeout 15000 设置节点失联时间，超过该时间（毫秒），集群自动进行主从切换

cluster nodes 查看集群信息

### 将六个节点合成一个集群
集群至少要有6个节点，3个master节点， 3个slave节点。
* 组合之前，请确保所有redis实例启动后，nodes-xxx.conf 文件都生成正常。
* 合体命令 (自动分配主从关系)
    * redis-trib.rb create --replicas 1 \
      192.168.1.11:6379 192.168.1.11:6380 \
      192.168.1.11:6381 192.168.1.11:6389 \
      192.168.1.11:6390 192.168.1.11:6391

**redis cluster 如何分配这六个节点？**
* 一个集群至少有三个主节点
* 选项 --replicas 1 代表我们希望为集群中的每个主节点创建一个从节点。
* 分配原则尽量保证每个主数据库运行在不同的ip地址，每个从库和主库不在同一个ip地址上

### 什么是slots
* 一个Redis集群包含16384个插槽（hash slot）, 数据库中的每个键都属于这16384个插槽的其中一个，集群使用公式 CRC16(key) % 16384 来计算键key属于哪个槽，其中CRC16(key) 语句用于计算键key的CRC16校验和。
* 集群中每个节点负责处理一部分插槽。举个例子，如果一个集群可以有主节点，其中：
    节点A负责处理0号至5500号插槽
    节点B负责处理5501号至11000号插槽
    节点C负责处理11001 号至 16383号插槽。

### 查询集群中的值
* cluster keyslot <key> 计算键key应该被放置在哪个槽上。
* cluster countkeysinslot <solt> 返回槽slot目前包含的键值对数量。
* cluster getkeysinslot <slot> <count> 返回count个slot槽中的键。
