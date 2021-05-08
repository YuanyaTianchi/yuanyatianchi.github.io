

+++

title = "etcd"
description = "etcd"
tags = ["it", "db"]

+++



# etcd

## clients

### Golang

https://github.com/etcd-io/etcd/tree/master/clientv3

```shell
go get go.etcd.io/etcd/clientv3 #解决包导入版本问题：https://segmentfault.com/q/1010000021762281/
```

client中有KV、Lease、Watcher等对象，通过这些对象来操作kv对、租约、监听器等

#### kv

```go
func main() {
	var (
		err error

		config clientv3.Config
		client *clientv3.Client

		/*kv：用于操作kv*/
		kv clientv3.KV

		/*KeyValue：保存etcd响应kv信息的结构体
		Key：键
		Value：值
		CreateRevision：创建时的Revision
		ModRevision：修改时的ModRevision  
		Version：kv的版本，等于该kv的put次数*/
		kvPair *mvccpb.KeyValue

		/*PutResponse：保存put响应信息的结构体
		Header：响应头信息，包括档次操作的 ClusterId集群id、MemberId节点id、Revision、RaftTerm等
		PrevKv：kv被put后的上一个kv信息，如果是put的是新的kv则prevKv=nil*/
		putResp *clientv3.PutResponse

		/*GetResponse：保存get响应信息的结构体
		Header：响应头信息
		Count：kv数量
		Kvs：kv数组*/
		getResp *clientv3.GetResponse

		/*DeleteResponse：保存del响应信息的结构体
		Header：响应头信息
		Deleted：删除的kv数量
		PrevKvs：删除的所有kv信息，kv数组*/
		delResp *clientv3.DeleteResponse

		/*OP：对get、put、del等操作做了抽象，用同样的调用方式执行不同的操作，返回相同的响应结构体*/
		putOp  clientv3.Op
		getOp  clientv3.Op
		opResp clientv3.OpResponse
	)

	/*客户端配置*/
	config = clientv3.Config{
		Endpoints:   []string{"127.0.0.1:2379"}, //集群地址
		DialTimeout: 5 * time.Second,            //连接超时时间
	}

	/*建立连接*/
	if client, err = clientv3.New(config); err != nil {
		fmt.Println(err)
		return
	}
	defer client.Close()
	fmt.Println("连接etcd成功")

	/*KV：提供操作 etcd kv 的相关功能*/
	kv = clientv3.NewKV(client)

	/*Put
	WithPrevKV：响应将带上PrevKv*/
	if putResp, err = kv.Put(context.TODO(), "/cron/jobs/job1", "JOB1"); err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println("put Revision:", putResp.Header.Revision)
	if putResp, err = kv.Put(context.TODO(), "/cron/jobs/job2", "JOB2", clientv3.WithPrevKV()); err != nil {
		fmt.Println(err)
		return
	}
	if putResp.PrevKv != nil {
		fmt.Println("put PrevKv:", string(putResp.PrevKv.Key), string(putResp.PrevKv.Value))
	}

	/*Get
	WithCountOnly：仅返回符合条件的个数
	WithPrefix：根据前缀获取kv
	WithFromKey：从某个key开始。配合WithLimit(n)获取从某个key开始的n个kv*/
	if getResp, err = kv.Get(context.TODO(), "/cron/jobs/job1"); err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println("get:", getResp.Count, getResp.Kvs)
	if getResp, err = kv.Get(context.TODO(), "/cron/jobs/", clientv3.WithPrefix()); err != nil {
		fmt.Println(err)
	}
	fmt.Println("get:", getResp.Count, getResp.Kvs)

	/*del
	WithPrevKV：返回删除的所有kv
	WithPrefix：删除带有某前缀的所有kv
	WithFromKey：删除从某个key开始的所有kv。配合WithLimit(n)则删除key开始的n个kv*/
	if delResp, err = kv.Delete(context.TODO(), "/cron/jobs/job2", clientv3.WithPrevKV()); err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println("del:", delResp.Deleted, delResp.PrevKvs)
	if len(delResp.PrevKvs) != 0 {
		for _, kvPair = range delResp.PrevKvs {
			fmt.Println("del:", string(kvPair.Key), string(kvPair.Value), kvPair.ModRevision)
		}
	}

	/*OpPut*/
	putOp = clientv3.OpPut("/cron/jobs/job3", "JOB3")           //创建操作
	if opResp, err = kv.Do(context.TODO(), putOp); err != nil { //执行操作
		fmt.Println(err)
		return
	}
	fmt.Println(opResp.Put().Header.Revision)

	/*OpGet*/
	getOp = clientv3.OpGet("/cron/jobs/job8")
	if opResp, err = kv.Do(context.TODO(), getOp); err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println(opResp.Get().Kvs[0].ModRevision, string(opResp.Get().Kvs[0].Value)) //创建时CreateRevision=ModRevision
}
```

#### lease

```go
func main() {
   var (
      err     error
      config  clientv3.Config
      client  *clientv3.Client
      kv      clientv3.KV
      putResp *clientv3.PutResponse

      /*lease：租约*/
      lease                  clientv3.Lease
      leaseGrantResp         *clientv3.LeaseGrantResponse
      leaseKeepAliveResp     *clientv3.LeaseKeepAliveResponse
      leaseKeepAliveRespChan <-chan *clientv3.LeaseKeepAliveResponse
   )
   config = clientv3.Config{
      Endpoints:   []string{"127.0.0.1:2379"},
      DialTimeout: 5 * time.Second,
   }
   if client, err = clientv3.New(config); err != nil {
      fmt.Println(err)
      return
   }
   defer client.Close()

   /*创建租约*/
   lease = clientv3.NewLease(client)

   /*申请租约。单位：秒*/
   if leaseGrantResp, err = lease.Grant(context.TODO(), 10); err != nil {
      fmt.Println(err)
      return
   }

   /*KeepAlive：续租
   将启动一个协程自动延续心跳续租，默认每3秒续租一次，续租时间为申请租约时的时间，KeepAliveOnce则只续租一次
   续租不是由原来的祖约时间加新的租约时间，而是重新设置租约时间，从续租时开始计算
   返回一个只读chan，它将定时将续租的响应写入chan，我们可以读取出来查看
   通过context取消或者过期取消等，也可以使心跳中断，ctx, _ := context.WithTimeout(context, 5*time.Second)*/
   if leaseKeepAliveRespChan, err = lease.KeepAlive(context.TODO(), leaseGrantResp.ID); err != nil {
      fmt.Println(err)
      return
   }

   /*读取leaseKeepAliveRespChan*/
   go func() {
      for {
         select {
         case leaseKeepAliveResp = <-leaseKeepAliveRespChan:
            if leaseKeepAliveResp == nil { //说明租约已经终止，可能是故障导致客户端与服务器失联了，或者我们主动让其释放的
               fmt.Println("租约失效")
               goto END
            } else {
               fmt.Println("收到续租响应", leaseKeepAliveResp.ID)
            }
         }
      }
   END:
   }()

   //用lease实现分布式锁，如果程序正常运行，谁抢到了key就等于抢到了锁，在持有锁的时候锁不应该过期，则需要延续租期，直到主动释放
   //并且如果程序宕掉了，锁也不会永远留在etcd，没有程序续租自然就自动过期了

   /*关联租约：put的kv与lease关联，等于为kv设置了过期时间，通过WithLease(leaseId)实现*/
   kv = clientv3.NewKV(client)
   if putResp, err = kv.Put(context.TODO(), "/cron/lock/job1", "", clientv3.WithLease(leaseGrantResp.ID)); err != nil {
      fmt.Println(err)
      return
   }
   fmt.Println("写入成功", putResp.Header.Revision)
}
```

#### watcher

```go
func main() {
   var (
      err     error
      config  clientv3.Config
      client  *clientv3.Client
      kv      clientv3.KV
      getResp *clientv3.GetResponse

      watchStartRevision int64
      /*watcher：监听器*/
      watcher       clientv3.Watcher
      watchResp     clientv3.WatchResponse
      watchRespChan <-chan clientv3.WatchResponse
      event         *clientv3.Event
   )
   config = clientv3.Config{
      Endpoints:   []string{"127.0.0.1:2379"},
      DialTimeout: 5 * time.Second,
   }
   if client, err = clientv3.New(config); err != nil {
      fmt.Println(err)
      return
   }
   defer client.Close()
   kv = clientv3.NewKV(client)

   /*模拟kv不断变化*/
   go func() {
      for {
         kv.Put(context.TODO(), "/cron/jobs/job7", "JOB7")
         kv.Delete(context.TODO(), "/cron/jobs/job7")
         time.Sleep(time.Second)
      }
   }()

   /*获取kv来得到开始监听的Revision：需要从当前Revision的下一个Revision开始监听*/
   if getResp, err = kv.Get(context.TODO(), "/cron/jobs/job7"); err != nil {
      fmt.Println(err)
      return
   }
   if getResp.Count != 0 {
      fmt.Println(getResp.Kvs[0].Value)
   }
   watchStartRevision = getResp.Header.Revision + 1 //需要从当前Revision的下一个Revision开始监听

   /*创建监听器*/
   watcher = clientv3.NewWatcher(client)
   fmt.Println("从该revision开始监听：", watchStartRevision)

   /*Watch：开启监听，通过WithRev(watchStartRevision)指定开始监听的版本
   返回一个只读chan，它将定时将监听的响应写入chan
   通过context可以取消监听*/
   ctx, cancel := context.WithCancel(context.TODO())
   time.AfterFunc(5*time.Second, cancel)
   watchRespChan = watcher.Watch(ctx, "/cron/jobs/job7", clientv3.WithRev(watchStartRevision))

   /*从watchRespChan读取监听响应*/
   for watchResp = range watchRespChan {
      for _, event = range watchResp.Events {
         switch event.Type {
         case mvccpb.PUT:
            fmt.Println("put:", event.Kv.CreateRevision, event.Kv.ModRevision)
         case mvccpb.DELETE:
            fmt.Println("del:", event.Kv.ModRevision)
         }
      }
   }
}
```

#### 分布式乐观锁

```go
/*分布式乐观锁：op操作、lease实现锁自动过期、txn事务：if else then
思路：上锁、处理业务、释放锁*/
func main() {
   var (
      err    error
      ctx    context.Context
      cancel context.CancelFunc

      config clientv3.Config
      client *clientv3.Client

      lease                  clientv3.Lease
      leaseGrantResp         *clientv3.LeaseGrantResponse
      leaseKeepAliveResp     *clientv3.LeaseKeepAliveResponse
      leaseKeepAliveRespChan <-chan *clientv3.LeaseKeepAliveResponse

      kv      clientv3.KV
      txn     clientv3.Txn
      txnResp *clientv3.TxnResponse
   )

   config = clientv3.Config{
      Endpoints:   []string{"127.0.0.1:2379"},
      DialTimeout: 5 * time.Second,
   }
   if client, err = clientv3.New(config); err != nil {
      fmt.Println(err)
      return
   }
   defer client.Close()

   /*1.上锁：创建租约，自动续租，拿租约去抢占一个key，谁第一个put上这个key，即抢到锁了*/

   /*租约：创建租约，自动续租*/
   lease = clientv3.NewLease(client)
   if leaseGrantResp, err = lease.Grant(context.TODO(), 10); err != nil {
      fmt.Println(err)
      return
   }
   defer lease.Revoke(context.TODO(), leaseGrantResp.ID) //确保函数退出时，立即释放租约

   ctx, cancel = context.WithCancel(context.TODO())
   defer cancel() //确保函数退出时，停止自动续租
   if leaseKeepAliveRespChan, err = lease.KeepAlive(ctx, leaseGrantResp.ID); err != nil {
      fmt.Println(err)
      return
   }
   go func() {
      for {
         select {
         case leaseKeepAliveResp = <-leaseKeepAliveRespChan:
            if leaseKeepAliveResp == nil {
               fmt.Println("租约失效")
               goto END
            } else {
               fmt.Println("收到续租响应", leaseKeepAliveResp.ID)
            }
         }
      }
   END:
   }()

   /*抢key：通过txn事务完成，if key不存在，then put这个key，else抢锁失败*/
   kv = clientv3.NewKV(client)
   txn = kv.Txn(context.TODO()) //创建事务
   txn.If(clientv3.Compare(clientv3.CreateRevision("/cron/lock/job9"), "=", 0)). //如果key不存在（key的CreateRevision=0）
      Then(clientv3.OpPut("/cron/lock/job9", "job9lock", clientv3.WithLease(leaseGrantResp.ID))). //执行putOp，可传多个op
      Else(clientv3.OpGet("/cron/lock/job9"))  //抢锁失败，取出来看看。。。同样是可以传入多个op
   if txnResp, err = txn.Commit(); err != nil { //提交事务
      fmt.Println(err)
      return
   }
   if !txnResp.Succeeded { //判断是否抢到了锁，如果没抢到，前面else是get操作，这里获取一下getResp(即rangeResp)
      fmt.Println("锁已被占用：", string(txnResp.Responses[0].GetResponseRange().Kvs[0].Value))
      return
   }

   /*2.处理业务
   已经在锁内了，很安全了*/
   fmt.Println("处理业务")
   time.Sleep(10 * time.Second)

   /*3.释放锁：取消自动续租，立即释放租约，释放租约同时该key也会被删掉，以实现立马释放锁
   前面defer做完了*/
}
```



## etcd协调服务

- 功能介绍
  - 核心特性
    - 将数据存储在集群中的高可用K-V存储
    - 允许应用实时监听存储中的K-V的变化
    - 能够容忍单点故障，能够应对网络分区（网络分区：分布式系统一般都有多个节点，如果把某几个节点与另几个节点之间的网络切断，系统仍能够短暂容忍这种问题提供服务）
  - 重量级用户：
    - kubernetes：Google开源容器调度平台
      - etcd支撑：服务发现、集群状态存储、配置同步
    - Cloud Founder：Vmware开源容器调度平台
      - etcd支撑：集群状态存储、配置同步、分布式锁
  - 传统存储模型
    - 单点存储：单点故障
    - 主从复制：master宕机，slave选为master需要时间，可以读，但无master不可写；主从之间数据同步延迟，master宕机可能损失数据，有些场景下这是不可容忍的
- 原理介绍
  - 抽屉理论：大多数理论
    - 一个班级61人
    - 有一个秘密，告知给班里的31个人
    - 那么随便挑选31个人，一定有1个人知道秘密
  - etcd与Raft的关系：所以etcd只需要把kv存储在raft算法的日志（预写日志）里，即可实现强一致性同步
    - Raft是强一致的集群日志同步算法（一致性hash）
    - etcd是一个分布式KV存储
    - etcd利用raft算法在集群中同步key-value
  - quorum模型：大多数模型，是一个二阶段模型
    - 例如集群需要2n+1个节点，假设是1个leader，2n个follower
      - 第一阶段：调用者将kv传给leader，但leader不会立刻响应调用者，而是日志实时复制( Replication)，leader会给n个follower做日志拷贝
      - 第二阶段：本地提交（写入kv存储），返回客户端。算上leader，就是n+1个节点保存了这个日志，刚好大多数了，就可以本地提交，返回客户端。并且会异步通知folloer完成提交
    - 日志实时复制：写入性能差，官方数据是约1000次/秒
  - 日志格式
    - index按序递增，其中装的kv对
- Raft
  - Raft日志概念
    - replication:日志在leader生成，向follower复制， 达到各个节点的日志序列最终一致
    - term：任期，重新选举产生的leader，其term单调递增
    - log index:日志行在日志序列的下标
  - Raft异常场景：详见图。可能没来得及复制就宕掉了，选举来去，比较乱，
  - Raft异常安全
    - 选举leader需要半数以上节点参与
    - 节点commit日志最多的允许选举为leader
    - commit日志同样多,则term、 index越大的允许选举为leader
  - Raft工作示例：
    - leader如果被网络分区了，那么它收到一个a=1的kv写入请求时，会不断重试复制日志，但是不断失败。这称为悬而未决
    - 而分区另一端的follower们，失去leader时则，会选举新的leader，假如加收新的a=2，并复制给大多数
    - 旧的leader重连上集群时，称为follower，丢弃a=1，同步a=2，因为现在a=2才是大多数
  - Raft保证
    - 提交成功的请求，一定不会丢
    - 各个节点的数据将最终一致
- 交互协议：etcp支持的协议
  - 通用的HTTP+JSON协议，性能低效
  - SDK内置GRPC协议（google的），性能高效：基于IDL、HTTP/2、Protobuffer 3
- 重要特性
  - 底层存储是按key有序排列的，可以顺序遍历
  - 因为key有序，所以etcd天然支持按目录结构高效遍历
  - 支持复杂事务，提供类似if...then .. else ... 的事务能力
  - 基于租约机制实现key的TTL过期
- key有序存储
  - 存储引擎是按Key有序排列的，如下3个字符串key，是前缀有序的。可以用来模拟目录结构，获取子目录内容，只需要seek到不大于/feature-flags/的第一个key，开始向后scan即可
    - /config/database
    - /feature-flags/verbost-logging
    - /feature-flags/redesign
- MVCC多版本控制。提交版本(revision)在etcd中单调递增；同key维护多个历史版本，用于实现watch机制；历史版本过多，可以通过执行compact命令完成删减
  - Put /key1 value1-> revision=1
  - Put /key2 value2-> revision=2
  - Put /key1 value3-> revision=3
- 监听KV变化：通过watch机制，可以监听某个key，或者某个目录(key前缀)的连续变化。常用于分布式系统的配置分发、状态同步。即那些app就可以监听到配置的变化
  - watch工作原理：假如有多版本存储 a=1 rev=1、b=2 rev=2、a=3 rev=3。见图
    - SDK发出请求 watch key=a from rev=1，即想要监听key为a的kv对，从revision=1开始监听
    - etcd就会建立一个watcher监听者，扫描历史revision，发现要找的key，即有a=1 rev=1、a=3 rev=3都是key=a的，并且从rev=1开始监听
    - etcd则将推送a=1 rev=1、a=3 rev=3这两个记录到我们的SDK。如果是from rev=3，则只有一条记录
- lease租约
  - 如SDK通过Grant创建10秒租约，即租约生命期只有10秒
  - etcd返回一个租约id，如Lease ID=5
  - SDK可以put a=1 with lease=5，即写入kv对时带上租约id到存储引擎，存储引擎与租约建立关联
  - 当租约过期则将delete a，删除过期数据
  - SDK可以定期向租约keepAlive续租，a就可以持续保存
- 实践任务
  - 搭建单机etcd，熟悉命令行操作
  - golang调用etcd的put/get/delete/lease/watch方法
  - 使用txn事务功能，实现分布式乐观锁

### 部署

https://github.com/etcd-io/etcd

安装：https://github.com/etcd-io/etcd/releases/download/v3.3.21/etcd-v3.3.21-linux-amd64.tar.gz，在linux上解压即可

```shell
tar -zxvf  etcd-v3.3.21-linux-amd64.tar.gz
cd  etcd-v3.3.21-linux-amd64
nohup ./etcd --listen-client-urls 'http://0.0.0.0:2379' --advertise-client-urls 'http://0.0.0.0:2379' & #启动时指定0.0.0.0，否则只有本地
less nohup.out #查看nohup.out，可以在最底部看到服务启动了，并且可以在前面看到选举信息，只有一个节点，自己投给自己称为了leader
./etcdctl #查看etcd的控制命令
ETCDCTL_API=3 ./etcdctl #根据./etcdctl查看到的警告需要在./etcdctl前加环境变量ETCDCTL_API=3才能使用v3版本的API
ETCDCTL_API=3 ./etcdctl put "name" "yuanya" #put存入一个kv对
ETCDCTL_API=3 ./etcdctl get "name" #获取
ETCDCTL_API=3 ./etcdctl del "name" #删除
```

### API

打开一个命令窗口，路径形式的key

```shell
ETCDCTL_API=3 ./etcdctl put "/cron/jobs/job1" "{一段json...job1}" #因为etcd是按key有序的，则我们保存kv时，key都以路径的形式来保存则非常方便
ETCDCTL_API=3 ./etcdctl put "/cron/jobs/job2" "{一段json...job2}"
ETCDCTL_API=3 ./etcdctl get "/cron/jobs/job1" #可以获取某一个job
ETCDCTL_API=3 ./etcdctl get -h #查看get帮助，可以发现有--prefix前缀查询、有--from-key从某个key开始查询等
ETCDCTL_API=3 ./etcdctl get "/cron/jobs/" --prefix #返回了job1和job2
```

打开另一个命令窗口，试试监听

```shell
ETCDCTL_API=3 ./etcdctl watch -h #watch，监听某个key或者某些key
ETCDCTL_API=3 ./etcdctl get "/cron/jobs/" --prefix #监听以某个字符串为前缀的key
```

在第一个命令窗口修改、删除 job2，可以发现监听的窗口中输出了我们的操作内容

```shell
ETCDCTL_API=3 ./etcdctl put "/cron/jobs/job2" "{...job2}"
ETCDCTL_API=3 ./etcdctl del "/cron/jobs/job2"
```



