

+++

title = "YuanyaCrontab"
description = "YuanyaCrontab"
tags = ["it", "go","project"]

+++



# Golang分布式任务调度



- 传统方案-linux下的crontab
  - 配置任务时,需要ssh登录脚本服务器进行操作
  - 服务器宕机，任务将终止调度，需要人工迁移
  - 排查问题低效，无法方便的查看任务状态与错误输出
- 分布式任务调度
  - 可视化web后台，方便进行任务管理
  - 分布式架构、集群化调度，不存在单点故障
  - 追踪任务执行状态，采集任务输出，可视化log查看
- 国内部分开源项目
  - elastic job：java写的
  - XXL-JOB：java写的
  - light-task-scheduler(LTS)：java写的
- 开源项目选型困难
  - 多是企业内部开源，维护精力有限，积累问题较多
  - 设计文档较少，对社区用户不友好，引入较为谨慎
- "自研”才是王道
- 从0构建的分布式架构
  - master-worker分布式架构：存在master、worker两种角色
  - etcd协调服务：依托于这个开源项目，它在golang中相当于java中的zookeeper
  - CAP理论：经典的分布式理论
  - Raft协议：基于该协议实现分布式日志同步
  - 任务分发
  - 事件广播
  - 分布式锁：防止某个任务被并发调度
  - 服务注册与发现
  - 多任务调度：一个woker节点同时调度多个任务
  - 并发设计：任务调度之后会并发执行
  - 异步日志：日志异步传输到mongodb
  - mongodb分布式存储
  - systemctl服务管理：主流linux下部署程序
  - nginx负载均衡：master是集群，通过nginx来LB

- 知识铺垫
  - 执行shelIl命令
  - cron表达式
  - etcd协调服务
  - mongodb分布式存储

- 实战项目
  - 分布式crontab架构
  - 实现master
  - 实现worker
  - 完善系统
  - 课程总结(含习题)

- 授课环境参数
  - 开发环境: windows10
  - 开发语言: Golang 1.10.1
  - 开发工具: Goland IDE 2018.1.1
  - 依赖存储: mongodb 4.0.0 + etcd v3.3.8
  - 生产部署环境: Centos7.1

- 技术储备要求
  - 语言:具备go语言语法基础，有一定练习或实践经验
  - 工具:了解linux shell、github、mysq|的简单用法



## Golang执行shell命令

- 任务：分2类。程序vs命令
  - 执行程序：通过一个绝对路径去指定。/usr/bin/python stats.py
  - 调用命令：cat nginx.log | grep "2018"
- 命令解释器：bash。
  - 2种工作模式
    - 交互模式：输入一些东西给你响应，如 ls -l
    - 非交互模式：如执行一个bash，/bin/bash -c "ls -l"。Golang来执行肯定是非交互模式
  - 命令也是程序：bash会找到命令背后对应的程序
    - /bin/bash-c "命令"：echo $PATH查看path，bash解释器会通过path这个环境变量中记录的路径去找到对应的二进制程序，如ls对应 /usr/bin/ls 程序，则该翻译过程就是由bash完成
  - bash也可以直接执行程序：/bin/bash -c "/usr/bin/ls -l"
  - 或者执行一个脚本： /bin/bash -c "/usr/bin/python xxx.py"
- Golang实现：用Golang程序去调用/bin/bash程序，把命令作为参数传入，即可实现执行任意shell命令
- 任务执行原理(底层)：
  - 如一个echo hello，打印一个hello。详见图
    - echo hello这个cmd给到golang程序：提交任务
    - golang程序创建一个管道pipe，pipe是linux操作系统的api，能提供类似于管道的东西，数据可以在其中传输
    - golang将fork一个子进程(Child Process)
    - 子进程重定向输出到pipe，其标准输出和标准错误输出都将输出到pipe中
    - 然后子进程来exec执行/bin/bash程序：/bin/bash -C "cmd"。并输出到管道pipe
    - pipe连接着父进程golang程序，golang程序可以取到pipe中的数据
    - 子进程退出，golang程序回收子进程，回收其相应的操作系统资源
  - 涉及的系统调用：都是c语言的api。写Golang用不上，只是加深理解
    - pipe()：创建2个文件描述符（类似于open文件），fd[0]可读，fd[1]可写。子进程从fd[1]写，父进程从fd[0]读
    - fork()：创建子进程，pipe的fd[1]被继承到子进程
    - dup2()：重定向子进程stdout/stderr到fd[1]
    - exec()：在当前进程内，加载并执行二进制程序
  - 在Golang中通过Command类执行任务
    - 定义在os/exec包：操作系统的包，都是基于操作系统的c语言的api实现的
    - 封装了所有上述底层细节：所以我们实际开发时用不到这些api的，只是加深理解

```go
/*代码规范建议
把变量都定义到头部，后面均使用=而不是:=，因为goto经常使用，使用:=就不能用goto了。代码可读性也比较高
err判断于代码写在一行，而不是单独拿出来判断
*/
func main() {
	var (
		cmd *exec.Cmd
		err error
	)
	cmd = exec.Command("E:\\it\\version-control\\Git\\bin\\bash.exe") //cmd实例
	if err = cmd.Run(); err != nil {                                  //执行cmd
		fmt.Println(err)
	}
}
```

```go
func main() {
	var (
		cmd    *exec.Cmd
		output []byte
		err    error
	)

	//cmd实例。sleep 1表示睡1秒，多个命令间用";"分开
	cmd = exec.Command("E:\\it\\version-control\\Git\\bin\\bash.exe", "-c", "sleep 1; ls -l; sleep 1; echo hello")

	//执行了命令，捕获了子进程的输出(pipe)
	if output, err = cmd.CombinedOutput(); err != nil {
		fmt.Println(err)
		return
	}

	//打印子进程的输出
	fmt.Println(string(output))
}
```

```go
/*杀死子进程
如果一个shell命令执行很久，比如下载一个文件，需要很长时间，我们如果想主动杀死该进程
执行一个cmd，让它在一个协程里去执行，执行2秒："sleep 2;"，在1秒的时候在主线程中去杀死它*/
func main() {
	var (
		ctx        context.Context    //context中有一个chan byte
		cancelFunc context.CancelFunc //cancelFunc用来关闭context的chan，close(chan byte)

		/*通过CommandContext获取cmd，可与传入一个context
		cmd实例将维护这个context，会有一个select{case <-ctx.Done():}来监听context中的chan是否被关闭了
		一旦调用cancelFunc关闭channel，则select命中，将调用操作系统的kill pid(进程id)，来杀死子进程*/
		cmd *exec.Cmd

		resultChan chan *result
		res        *result
	)
	resultChan = make(chan *result, 1000)                //创建一个结果队列
	ctx, cancelFunc = context.WithCancel(context.TODO()) //context.WithCancel获取context和对应cancelFunc

	go func() {
		var (
			output []byte
			err    error
		)
		cmd = exec.CommandContext(ctx, "E:\\it\\version-control\\Git\\bin\\bash.exe", "-c", "sleep 2;echo hello;")

		output, err = cmd.CombinedOutput()               //执行任务捕获输出
		resultChan <- &result{err: err, output: output,} //把任务输出结果传给main协程
	}()

	time.Sleep(1 * time.Second)
	fmt.Println("main")

	cancelFunc()       //取消上下文，命令就会被终止
	res = <-resultChan //在main协程里，等待子协程的退出，并打印任务执行结果
	fmt.Println(res.err, string(res.output))

	//结果与预期不符合。命令中断的err已经有了，err是exit status 1，但是这里还是收到hello了，命令实际并没有中断，有待解决
}

type result struct {
	err    error
	output []byte
}
```



## Cron表达式

- Cron基本格式
  -  \*             \*              \*              \*                  \*             Command
  -  分(0-59)  时(0-23)   日(1-31)   月(1-12)   星期(0-7)   shell命令\
- Cron常见用法
  - 每分钟执行一次：* * * * echo hello
  - 每隔5分钟执行1次：*/5 * * * * echo hello > /tmp/x.log
  - 第1-5分钟每分钟执行一次，共5次：1-5 * * * * /usr/bin/python/data/x.py
  - 每天10点、22点整执行1次：0 10,22 * * * echo bye | tail -1
- Cron调度原理
  - 思考
    - 已知当前UNIX时间为1532659200，即2018/07/27 10:40:00
    - 已知道Cron表达式是30 * * *
    - 如何计算命令的下次调度时间?
  - 计算逻辑
    - 根据时间刻度线来计算，而不是配置crontab表达式的时间
      - 当前时间2018/07/27 10:40:00：40分   10时   27日。这是当前时间
      - Cron表达式30 * * *：                  00分   11时   27日。这是下次执行时间，离当前时间只有20分钟，因为是按时间刻度来的，这里是每30分钟执行一次就是[0 30]，如果是6 * * *则是[0 6 12 18 ... 54]
      - 枚举：                                         30分   11时   27日
        - 分钟：[30]
        - 小时：[0,1...23]
        - 日期：[1,2...31]
        - 月份：[1,2...12]
      - 下下次调度时间：2018/07/27 11:30:00
- 使用Cron解析库
  - 为什么不自己实现？
    - 时间计算容易写出BUG
    - 非关键技术环节，造轮子收益太低
  - 开源Cronexpr库
    - Parse()：解析与校验Cron表达式
    - Next()：根据当前时间，计算下次调度时间

```shell
go get github.com/gorhill/cronexpr #用于解析Cron表达式的库
```

```go
/*简单使用*/
func main() {
	var (
		expr     *cronexpr.Expression
		err      error
		now      time.Time
		nextTime time.Time
	)
	// linux crontab
	// 秒粒度, 年配置(2018-2099)
	// 哪一分钟（0-59），哪小时（0-23），哪天（1-31），哪月（1-12），星期几（0-6）

	/*//每分钟执行一次。MustParse不返回err，一旦出错程序将直接宕掉，一般使用Parse即可
	if expr, err = cronexpr.Parse("* * * * *"); err != nil {
		fmt.Println(err)
		return
	}*/

	//每隔5分钟执行一次
	if expr, err = cronexpr.Parse("*/5 * * * *"); err != nil {
		fmt.Println(err)
		return
	}

	//每隔5秒执行一次。
	//cronexpr支持的cron表达式比linux crontab更复杂，可以有7个粒度，多出了秒、年（2018-2099）两个粒度
	//因为是枚举的，只枚举到2099年
	if expr, err = cronexpr.Parse("*/5 * * * * * *"); err != nil {
		fmt.Println(err)
		return
	}

	now = time.Now()          //当前时间
	nextTime = expr.Next(now) //下次调度时间

	//调用一次
	time.AfterFunc(nextTime.Sub(now), func() {
		fmt.Println("被调度了", nextTime)
	})

	time.Sleep(6 * time.Second)

	fmt.Println(now, "\n", nextTime)
}
```

```go
/*同时调度多个cron表达式*/
// 需要有1个调度协程, 它定时检查所有的Cron任务, 谁过期了就执行谁
func main() {
	var (
		cronJob       *CronJob
		expr          *cronexpr.Expression
		now           time.Time
		scheduleTable map[string]*CronJob //调度表。key:任务的名字,
	)
	now = time.Now()
	scheduleTable = make(map[string]*CronJob) //初始化调度表

	//定义1个cronjob
	expr = cronexpr.MustParse("*/5 * * * * * *")
	cronJob = &CronJob{
		expr:     expr,
		nextTime: expr.Next(now),
	}
	scheduleTable["job1"] = cronJob // 任务注册到调度表

	//再定义1个cronjob
	expr = cronexpr.MustParse("*/5 * * * * * *")
	cronJob = &CronJob{
		expr:     expr,
		nextTime: expr.Next(now),
	}
	scheduleTable["job2"] = cronJob //任务注册到调度表

	//启动一个调度协程，用来检查调度表中cronjob的调度时间，如果达到了调度时间则执行任务
	go func() {
		var (
			jobName string
			cronJob *CronJob
			now     time.Time
		)

		//定时检查任务调度表
		for {
			now = time.Now()
			for jobName, cronJob = range scheduleTable {
				if cronJob.nextTime.Before(now) || cronJob.nextTime.Equal(now) {
					//启动一个协程执行任务。本来是发送cmd执行任务，这里只是模拟一下
					go func(jobName string) {
						fmt.Println("执行", jobName)
					}(jobName) //尽量不使用闭包

					//计算该任务再下一次的调度时间
					cronJob.nextTime = cronJob.expr.Next(now)
					fmt.Println(jobName, "下次执行时间：", cronJob.nextTime)
				}
			}

			//睡眠100毫秒，每100毫秒检查一次调度表
			select {
			//timer对象中有一个变量名为C的chan，timer向C中投递time对象，将在100毫秒内可读，返回
			case <-time.NewTimer(100 * time.Millisecond).C:
			}
		}
	}()

	time.Sleep(100 * time.Second) //避免主协程退出
}

//代表一个任务
type CronJob struct {
	expr     *cronexpr.Expression
	nextTime time.Time //expr.Next(now)，下次调度时间。通过判断这个时间来决定是否调度任务
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
  - etcd与Raft的关系：所以etcd只需要把kv存储在raft算法的日志里，即可实现强一致性同步 
    - Raft是强一致的集群日志同步算法
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



## mongodb

https://github.com/mongodb/mongo-go-driver

```go
func main() {
   var (
      err error

      client     *mongo.Client
      database   *mongo.Database
      collection *mongo.Collection

      insertOneResult *mongo.InsertOneResult
      record          *LogRecord
      docId           primitive.ObjectID

      findCond *FindByJobName
      cursor   *mongo.Cursor

      delCond   *DeleteCond
      delResult *mongo.DeleteResult
   )

   /*建立连接*/
   if client, err = mongo.Connect(context.TODO(),
      options.Client().ApplyURI("mongodb://127.0.0.1:27017"),
      options.Client().SetConnectTimeout(5*time.Second),
   ); err != nil {
      fmt.Println(err)
      return
   }
   database = client.Database("cron")      //数据库
   collection = database.Collection("log") //表

   /*insert：如果没有写入_id字段，则将自动生成全局唯一ID，为ObjectID类型，是一个12字节的二进制*/
   record = &LogRecord{ //数据
      JobName:   "job10",
      Command:   "echo hello",
      Err:       "",
      Content:   "hello",
      TimePoint: TimePoint{StartTime: time.Now().Unix(), EndTime: time.Now().Unix() + 10},
   }
   if insertOneResult, err = collection.InsertOne(context.TODO(), record); err != nil { //执行
      fmt.Println(err)
      return
   }
   docId = insertOneResult.InsertedID.(primitive.ObjectID) //转ObjectID
   fmt.Println("自增id：", docId.Hex())                       //转string

   /*find：通过options添加操作*/
   findCond = &FindByJobName{JobName: "job10"} //过滤条件：{"jobName": "job10"}
   if cursor, err = collection.Find(           //执行查询，返回游标，游标通过Next函数遍历
      context.TODO(),
      findCond,                   //过滤条件
      options.Find().SetSkip(0),  //偏移量0，等于limit 0 2
      options.Find().SetLimit(2), //2个
   ); err != nil {
      fmt.Println(err)
      return
   }
   defer cursor.Close(context.TODO()) //defer释放资源
   for cursor.Next(context.TODO()) {  //可以用context控制超时时间等
      record = &LogRecord{}
      if err = cursor.Decode(record); err != nil { //解码，反序列化
         fmt.Println(err)
         return
      }
      fmt.Println(*record)
   }

   /*delete
   删除开始时间早于当前时间的所有日志：delete({"timePoint.startTime": {"$lt": 当前时间}})。$lt表示小于，less than
   这里为了简便通过结构体来组织bson，也可以通过bson包的方法来设置，但是比较麻烦*/
   delCond = &DeleteCond{
      beforeCond: TimeBeforeCond{ //beforeCond被映射成timePoint.startTime
         Before: time.Now().Unix(), //Before被映射成$lt
      },
   }

   /*执行删除*/
   if delResult, err = collection.DeleteMany(context.TODO(), delCond); err != nil {
      fmt.Println(err)
      return
   }
   fmt.Println("删除的行数:", delResult.DeletedCount)
}

/*日志结构体*/
type LogRecord struct {
   JobName   string    `bson:"jobName"`   // 任务名
   Command   string    `bson:"command"`   // shell命令
   Err       string    `bson:"err"`       // 脚本错误
   Content   string    `bson:"content"`   // 脚本输出
   TimePoint TimePoint `bson:"timePoint"` // 执行时间点
}

type TimePoint struct {
   StartTime int64 `bson:"startTime"`
   EndTime   int64 `bson:"endTime"`
}

// jobName过滤条件
type FindByJobName struct {
   JobName string `bson:"jobName"` // JobName赋值为job10
}

/*{"timePoint.startTime": {"$lt": timestamp} }*/
type DeleteCond struct {
   beforeCond TimeBeforeCond `bson:"timePoint.startTime"`
}

// startTime小于某时间：{"$lt": timestamp}
type TimeBeforeCond struct {
   Before int64 `bson:"$lt"`
}
```



## Crontab

- 传统crontab痛点
  - 机器故障，任务停止调度，甚至crontab配置都找不回来
  - 任务数量多，单机的硬件资源耗尽，需要人工迁移到其他机器
  - 需要人工去机器上配置cron，任务执行状态不方便查看
- 分布式架构-核心要素
  - 调度器:需要高可用，确保不会因为单点故障停止调度
  - 执行器:需要扩展性，提供大量任务的并行处理能力
- 常见开源调度架构：见图
- 伪分布式设计
  - 分布式网络环境不可靠，RPC异常属于常态
  - Master下发任务RPC异常时，导致Master与Woker状态不一致：比如woker已经收到请求了执行工作了，但是因为网络问题没有响应到master
  - Worker上报任务RPC异常，导致Master状态信息落后
- 异常case举例
  - 状态不一致: Master' 下发任务给node1异常，实际node1收到并开始执行
  - 并发执行: Master重试下发任务给node2，结果node1与node2同时执行一个任务
  - 状态丢失: Master更新zookeeper中任务状态异常 ，此时Master宕机切换Standby,任务仍旧处于旧状态
- 分布式伪命题
  - 但凡需要经过网络的操作，都可能出现异常
  - 将应用状态放在存储中，必然会出现内存与存储状态不一致：这种方式最常用，所以使用cap理论
  - 应用直接利用raft管理状态，可以确保最终一致， 但是成本太高
- CAP理论(常用于分布式存储)：以mysql为例，mysql就是cp，但拥有一定的可用性倾斜
  - C：一致性，写入后立即读到新值。mysql就满足一致性，写入之后立马就可以读取了
  - A：可用性，通常保障最终一致。不会因为单点故障服务就不可用了，高可用一般就指避免单点故障，mysql就不满足可用性，单机的mysql宕掉就不能用了，但是可以通过主从同步来满足至少可以提供读的可用性，写会失效
  - P：分区容错性，分布式必须面对网络分区。两个节点通过网络通信，会因为网络被损坏，两个节点就网络分区了，所以一般P是**必须**实现的，不能说因为网络不通就不提供服务了，至少要保证在网络分区的情况下提供有限的服务，在网络恢复后继续服务
- BASE理论(常用于应用架构)
  - BA：基本可用(BasicallyAvailable)。损失部分可用性，保证整体可用性。比如熔断机制，一个服务宕机之后（一个服务多次调用失败），调用它的服务不要继续再调用它了，于是熔断这个请求，不能因为一个环节的故障，导致所有业务全部请求超时、不可用，那就不是基本可用了
  - S：软状态(Soft-state)。允许状态同步延迟，不会影响系统即可。
  - E：最终一致性(EventuallyConsistent)。经过一段时间后，系统能够达到一致性
- 架构师能做什么的?
  - 架构简化，减少有状态服务，秉承最终一致。强一致性本来很难做到，满足最终一致即可
  - 架构折衷，确保异常可以被程序自我治愈：不能因为1%的异常投入99%的努力，只需要确保异常后可以有备解决即可

### 整体架构

- web控制台
- 无状态Master集群
  - 保存任务到etcd；master可以从etcd查询任务
  - master可以从mongodb查询日志
- etcd任务存储
  - etcd同步任务到woker集群；为Master查询任务提供接口
  - etcd实现分布式锁
- mongodb
  - 保存woker执行任务的日志；为Master查询日志提供接口
- 可扩展woker集群
  - woker集群执行任务，每个worker独立调度全量任务，无需与master产生直接RPC（网络通信），通过抢占分布式乐观锁解决并发调度问题 。关于性能，像go语言这种编译型语言，快速排序一个1000万的数组，可能也只要1秒时间，所以做一个几千几万几十万的任务调度，不需要太担心性能
  - 执行任务的日志保存到mongodb。所有的woker都可以获得所有任务，所有的woker维护同样的任务列表

### master

- 功能
  - 任务管理HTTP接口：新建、修改、查看、删除任务
  - 任务日志HTTP接口：查看任务执行历史日志
  - 任务控制HTTP接口：提供强制结束任务的接口
  - 实现web管理界面：基于jquery+ bootstrap的Web控制台，前后端分离
- 架构
  - 任务管理HTTP接口：etcd，/cron/jobs/
    - 任务结构：/cron/jobs/任务名 -> {name:任务名, command:shell命令,cronExpr:cron表达式}
    - 保存到etcd的任务，会被实时同步到所有worker，worker监听etcd的变化来获取任务
  - 任务日志HTTP接口：mongodb
    - 日志结构：{jobName:任务名,command:shell命令,err:执行报错,output:执行输出,startTime:开始时间,endTime:结束时间}
    - 请求MongoDB,按任务名查看最近的执行日志
  - 任务控制HTTP接口：etcd，/cron/killer/
    - 结构：/cron/killer/任务名 -> ""value为空即可
    - worker监听/cron/ killer/目录下put修改操作，感知到对应key就去杀死对应任务即可
    - master将要结束的任务名put在/cron/killer/目录下，触发worker立即结束shell任务

### woker

- 功能
  - 任务同步:监听etcd中/cron/jobs/目 录变化
  - 任务调度:基于cron表达式计算，触发过期任务
  - 任务执行:协程池并发执行多任务，基于etcd分布式锁抢占
  - 日志保存:捕获任务执行输出，保存到MongoDB
- 架构
  - 监听协程：etcd
    - 利用watch API，监听/cron/jobs/和/cron/killer/ 目录的变化
    - 将变化事件通过channel推送给调度协程，更新内存中的任务信息
  - cron 调度协程：
    - 监听任务变更event，更新内存中维护的任务列表
    - 检查任务cron表达式，扫描到期任务，交给执行协程运行
    - 监听任务控制event,强制中断正在执行中的子进程
    - 监听任务执行result,更新内存中任务状态，投递执行日志
    - job：调用 job 任务执行协程池，并发执行 job 任务，并接收 job 任务执行结果，发送给日志协程
    - killer：执行 kill 任务，杀死 job 任务执行协程，kill 执行结果发送给日志协程
  - 任务执行协程
    - 在etcd中抢占分布式乐观锁：/cron/lock/任务名
    - 抢占成功则通过Command类执行shell任务
    - 捕获Command输出并等待子进程结束，将执行结果投递给调度协程
    - 并发的 woker 们通过 etcd 的 /cron/lock/ 抢占分布式锁，抢到锁的 woker 去执行任务，任务执行结果返回给cron 调度协程。
      - 因为woker是一个集群，而可能有每秒执行一次的任务，如果不做并发控制，那就会每个woker每秒都去执行一次这个任务，分布式锁即一种并发控制，谁抢到锁谁执行即可，但是这种调度依赖于各个woker节点的时间同步，如果各个节点校时不准确，一些机器总是可以先抢占到锁
      - 需要通过服务器校时来尽量保证各个woker时间一致
      - 还有一个小策略，比如抢锁之前，睡眠随机100ms（某个阈值）以内的时间
  - 日志协程：接收cron调度协程的任务执行结果，写入日志到mongodb
    - 监听调度发来的执行日志，放入一个batch中。因为日志很多，一条条插入可能插入不过来
    - 对新batch启动定时器（比如1秒），超时未满自动提交
    - 若batch被放满（比如1000个），那么立即提交，并取消自动提交定时器

## 项目实现

- crontab (github)
  - /master
    - 搭建go项目框架，配置文件，命令行参数，线程配置。。
    - 给web后台提供http API,用于管理job
    - 写一个web后台的前端页面，bootstrap+jquery, 前后端分离开发
  - /worker
    - 从etcd中把job同步到内存中
    - 实现调度模块，基于cron表达式调度N个job
    - 实现执行模块，并发的执行多个job
    - 对job的分布式锁，防止集群并发
    - 把执行日志保存到mongodb
  - /common