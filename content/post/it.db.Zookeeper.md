

+++

title = "Zookeeper"
description = "Zookeeper"
tags = ["it","db","Zookeeper"]

+++



# Zookeeper

http://zookeeper.apache.org/

为分布式应用提供协调服务。

服务注册与发现，多个服务器节点注册，客户端通过zookeeper发现（访问）服务，具体访问哪个节点由zookeeper分发控制

- 工作机制：Zookeeper从设计模式角度来理解，是一个基于观察者模式设计的分布式服务管理框架，它负责存储和管理大家都关心的数据，然后接受观察者的注册，一旦这些数据的状态发生变化，Zookeeper就将负责通知已经在Zookeeper上注册的那些观察者做出相应的反应。

## 集群

- 集群
  - 一个Leader，多个Follower
  - 集群中只要有半数以上节点存活，Zookeeper集群 就能正常服务
  - 全局数据一致: 每个ZookeeperServer保存一份相同的数据副本，Client无论连接到哪一个ZookeeperServer，数据都是一致的
  - 更新请求顺序进行，来自同一个Client的更新请求按其发送顺序依次执行。
  - 数据更新原子性，一次数据更新要么成功，要么失败。
  - 实时性，在一 定时间范围内，Client能读到最新数据。
- 集群数据结构
  - ZooKeeper数据模型的结构与Unix文件系统很类似,整体上可以看作是一棵树,每个节点称做一个ZNode。每一个ZNode默认能够存储lMB的数据，每个ZNode都可以通过其路径唯一标识。
  - ​                                                /
  - ​                        ↓                                                ↓
  - ​                /znode1                                        /znode2
  - ​            ↓                    ↓                            ↓                    ↓
  - /znode1/leaf1    /znode2/leaf2    /znode1/leaf1    /znode2/leaf2

## 应用

- 应用：提供的服务包括：统一命名服务、 统一配置管理、统一集群管理、 服务器节点动态上下线、软负载均衡等
  - 统一命名服务：服务注册。在分布式环境下,经常需要对应用/服务进行统一命名,便于识别。例如: IP不容易记住,而域名容易记住。
  - 统一配置管理：配置中心。假如kafka集群由zookeeper管理配置信息，将配置信息写入某个znode，kafka集群各个服务器监听这个znode，一旦修改配置信息，由zookeeper广播通知使整个kafka集群配置同步
  - 统一集群管理：监控信息，健康管理。ZooKeeper可以实现实时监控节点状态变化
    - 可将节点信息写入ZooKeeper.上的一个ZNode.
    - 监听这个ZNode可获取它的实时状态变化。
  - 服务器节点动态上下线：在zookeeper注册的服务器，可以被客户端通过zookeeper得到服务器的上下线事件通知
  - 软负载均衡：在Zookeeper中记录每台服务器的访问数，让访问数最少的服务器去处理最新的客户端请求



## 安装配置

zookeeper-3.4.14.tar.gz

```shell
tar -zxvf zookeeper-3.4.14.tar.gz -C /opt/zookeeper #解压即可
```

conf/zoo.cfg

```shell
cp conf/zoo_simple.cfg conf/zoo.cfg #拷贝一份conf/zoo_simple.cfg为zoo.cfg
#修改zoo.cfg中数据文件保存路径配置dataDir，并在配置路径下建好文件夹
dataDir=/app/it/mq/zookeeper/zookeeper-x.x.x/zkData
#配置文件说明，以下均是默认值
tickTime=2000 #心跳
initLimit=10 #初始时通信最大延迟心跳数，主从之间如果超过initLimit次心跳没通信上，则认为挂了
syncLimit=5 #启动之后通信时间的最大延迟心跳数
clkientPort=2181 #客户端访问的端口号
```

bin/zkCli.sh 启动zk客户端连接zk服务器，bin/zkServer.sh 关闭zk服务器



## 选举

- 选举机制

  - 半数机制：集群中半数以上机器存活，集群可用。所以Zookeeper适合安装奇数台服务器。

  - Zookeeper虽然在配置文件中并没有指定Master和Slave。但是，Zookeeper.工作时，是有一个节点为Leader，其他则为Follower, Leader 是通过内部的选举机制临时产生的。

  - 流程：假如zookeeper集群机器数为5，

    - 一轮加一台，从小到大投，谁大都投谁，半数票以上成为主，成主之后不可改

      第一轮：大小对比指的是id值大小，从id最小1开始，上来先选自己，1票不成立，发出去的报文也没有其它机器可以响应，则保持LOOKING状态

      第二轮：2加入，2又选自己，1、2将交互，1看见2比自己id大，就选了2，但2票还不够

      第三轮：3加入，3选自己，1、2、3交互，1和2看见3最大，就都选了3，3票超过半数成为master

      第四轮：4加入，投自己，没用了，3已成主，局势已定

      第五轮：5加入，同4

- 节点类型

  - 持久(Persistent)：客户端和服务器端断开连接后，创建的节点不删除
    - 持久化目录节点：客户端与Zookeeper断开连接后，该节点依旧存在
    - 持久化顺序编号目录节点：客户端与Zookeeper断开连接后，该节点依旧存在，只是Zookeeper给该节点名称进行顺序编号
  - 短暂( Ephemeral)：客户端和服务器端断开连接后，创建的节点自己删除
    - 临时目录节点：客户端与Zookeeper断开连接后，该节点被删除
    - 临时顺序编号目录节点：客户端与Zookeeper断开连接后，该节点被删除，只是Zokeeper给该节点名称进行顺序编号。
  - 结构
    - ​                                                                    /
    - ​    |                                            |                                        |                                            |
    - /xnode1,persistent持久     /xnode2_001,persistent    /xnode3,ephemeral临时    /xnode4_001,ephemeral
    - ​    ↑                                            ↑                                    ↑                                        ↑
    - client1                                    client2                            client3                                client4

  - 说明：创建znode时 设置顺序标识，znode名称后会附加一个值，顺序号是一个单调递增的计数器，由父节点维护。注意：在分布式系统中，顺序号可以被用于为所有的事件进行全局排序，这样客户端可以通过顺序号推断事件的顺序

  

## 集群部署

集群规划：在hadoop101、hadoop102、hadoop103三个节点上部署Zookeeper

将已经安装配置好的zookeeper整个目录同步到 hadoop102，hadoop103（假设已经安装好的是 hadoop101）

```shell
[yuanya@hadoop101 zookeeper]$ xsync zookeeper-x.x.x/ #同步后，并根据配置的dataDir创建对应的目录zkData
[yuanya@hadoop101 zkData]$ touch myid #还需要在创建myid文件用于配置服务器编号
```

后续。。。上次竟然没看完，后面边看边整理吧







