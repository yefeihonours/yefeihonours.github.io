## Zookeeper 稳定版下载

[zookeeper-3.4.10.tar.gz](http://mirrors.hust.edu.cn/apache/zookeeper/zookeeper-3.4.10/zookeeper-3.4.10.tar.gz)

<!--more-->

项目下载后直接解压就可以使用
	tar -zxvf zookeeper-3.4.10.tar.gz

## Zookeeper 简介

Zookeeper 分布式服务框架是 Apache Hadoop 的一个子项目，它主要是用来解决分布式应用中经常遇到的一些数据管理问题，如：统一命名服务、状态同步服务、集群管理、分布式应用配置项的管理等.
Zookeeper 在开发和部署中一共有三种运行模式: 单机模式, 伪集群模式(单机多实例), 集群模式.

## 单机模式

### 创建zoo.cfg文件

```
# zookeeper config
cd zookeeper-3.4.10/conf/
cp zoo_sample.cfg zoo.cfg
```

### zookeeper 配置文件内容如下
```
# zookeeper-3.4.10/conf/zoo.cfg
#
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/tmp/zookeeper
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
#http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
```

### 启动与连接

```
# 启动服务
cd zookeeper-3.4.10/bin/
chmod a+x zkServer.sh zkCli.sh
./zkServer.sh start
#
# 连接zookeeper
./zkCli.sh -server 127.0.0.1:2181
#
# 其他命令
# ./zkServer.sh status 查看状态
# ./zkServer.sh stop   停止服务
```

java 连接配置

```
<dubbo:registry address="zookeeper://127.0.0.1:2181" />
```

## 伪集群模式(单机多实例)

Zookeeper 在使用的时候一般使用单机模式和集群模式, 但是在只有一台机器, 并且希望测试集群式部署时, 则会使用为集群模式, 即在一台机器上创建多个 zookeeper 实例.

### 创建目录结构

```bash
.
├── zkServer0
│   ├── data
│   ├── logs
│   └── zookeeper-3.4.10
├── zkServer1
│   ├── data
│   ├── logs
│   └── zookeeper-3.4.10
├── zkServer2
│   ├── data
│   ├── logs
│   └── zookeeper-3.4.10
```

zookeeper-3.4.10为下载解压的zookeeper文件

### 创建 myid 文件


```bash
echo 0 > zkServer0/data/myid
echo 1 > zkServer1/data/myid
echo 2 > zkServer2/data/myid
```

### 配置 zookeeper-3.4.10/conf/zoo.cfg
示例为 zkServer0 下面的配置文件, zkServer1和zkServer只需要clientPort不同

```bash
#
tickTime=2000
#
initLimit=10
#
syncLimit=5
#
dataDir=zookeeper/zkServer0/data
dataLogDir=zookeeper/zkServer0/logs
# 2181, 2182, 2183
clientPort=2181
#
server.0=127.0.0.1:7881:8881
server.1=127.0.0.1:7882:8882
server.2=127.0.0.1:7883:8883
```

### 启动三个实例, 在zookeeper最外层目录下 

```bash
./zkServer0/zookeeper-3.4.10/bin/zkServer.sh start
./zkServer1/zookeeper-3.4.10/bin/zkServer.sh start
./zkServer2/zookeeper-3.4.10/bin/zkServer.sh start
```

### java连接配置

```bash
<dubbo:registry address="zookeeper://127.0.0.1:2181?backup=127.0.0.1:2182,127.0.0.1:2183" />
```

## 集群部署
集群式部署需要多台服务器设备, 没有主机可以使用虚拟机实现.
假设现在有三台主机 , IP分别为: 192.168.0.101, 192.168.0.102, 192.168.0.103
	

### 创建目录结构
每一个主机上都需要有一个文件目录

```bash
. 
├── zkServer
│   ├── data
│   ├── logs
│   └── zookeeper-3.4.10
```
zookeeper-3.4.10为下载解压的zookeeper文件

### 创建 myid 文件, 每台主机写入不同的id

```bash
echo 0 > zkServer/data/myid
```

### 配置 zookeeper-3.4.10/conf/zoo.cfg

示例为 zkServer0 下面的配置文件, zkServer1和zkServer只需要clientPort不同


```bash
#
tickTime=2000
#
initLimit=10
#
syncLimit=5
#
dataDir=zookeeper/zkServer0/data
dataLogDir=zookeeper/zkServer0/logs
# 每台主机都设置相同端口
clientPort=2181
# 实际为不同的主机IP
server.0=192.168.0.101:7881:8881
server.1=192.168.0.102:7881:8881
server.2=192.168.0.103:7881:8881
```

### 启动三个实例, 在zookeeper最外层目录下 

```bash
./zkServer0/zookeeper-3.4.10/bin/zkServer.sh start
./zkServer1/zookeeper-3.4.10/bin/zkServer.sh start
./zkServer2/zookeeper-3.4.10/bin/zkServer.sh start
```

### java连接配置

```bash
<dubbo:registry address="zookeeper://192.168.0.101:2181?backup=192.168.0.101:2181,192.168.0.101:2181" />
```