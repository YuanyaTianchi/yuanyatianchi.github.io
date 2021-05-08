

+++

title = "面试大纲"
description = "面试大纲"
tags = ["it", "base"]

+++



# 面试大纲

- 算法
  - ascii：A-65、a-97
  - dp：base case、状态、选择、dp定义
  - dfs：backtrack(路径, 选择列表)、结束条件-决策树底层
  - bfs：要扩散的节点、已扩散的节点
  - math：平方根Sqrt、Pow   方、Abs绝对值Ceil向上取整、Floor向下取整
  - string：Trim去掉两端、子序列位置In-dex、Contains匹配子序列、Replace替换n次，Split拆分
- 设计模式
- 网络
  - tcp：可靠，字节流，3次握手、4次挥手。udp：不可靠，报文流，直接发送
  - rpc：远程过程调用，轻量，协议无关、序列化无关，消息头保存
  - http基于文本解析，http/2基于二进制解析，socket是对tcp等实现提供网络编程的方法
  - coocie保存再浏览器端，session保存在服务器端
- go
  - 类型：string(SRODATA)、array、slice(追加扩容growslice、对齐)、map(扩容)，结构、类型、初始化、访问、
  - 函数调用栈：一个函数调用另一个函数，编译器就会生成一个call指令，call指令做两件事，一是将下一条指令的地址入栈，就是返回地址，被调用函数结束后跳回到这里，二是跳转到被调用函数入口处执行
  - 闭包：带捕获变量的funcval，是否修改，参数、返回值作捕获变量
  - 方法、defer、panic和recover、类型系统、接口、断言、reflect、闭包（带捕获变量的funcval）
  - GPM：工作窃取、让出（sleep、channel（context）、io事件）、抢占（stackPreempt标识、信号、系统调用），gopark、goready ，监控线程
  - GC：标记清除、主体并发增量式回收、插入与删除的混合写屏障
- ​     mysql
  - 存储引擎：innodb、myisam、memory 的 存储、索引、锁、表空间、事务、缓存
  - 锁：表锁（读共享、写独占）、行锁（读共享、写排他、意向共享、意向排他）、锁算法（Record Lock、Gap Lock、Next-Key Lock）、乐观锁（cas、mvcc）、悲观锁（mutex）
  - 事务：ACID（原子一致隔离持久），脏读、不可重复读、幻读，READ UNCOMMITED、READ COMMITED、REPEATABLE READ、SERIALIZABLE
  - 三大范式：列的原子性（最小属性不可再分化）、行的唯一性（非主键列完全依赖主键列，不是部分依赖关键字）、非主键列直接依赖主键列
  - 慢查询：配置slow_query_log=1、mysqldumpslow工具
  - 索引：普通索引、唯一索引、复合索引、聚集索引、聚簇索引，执行计划 Explain（select_type、key、keylen、ref、Extra(temporary、index、where、join buffer)）
  - sql优化：全值匹配（联合索引中更多的列被使用）、最左前缀、范围条件最后、索引覆盖、慎用不等于，主要是避免索引失效变成全表扫，临时表                                 
- redis

  - API：string、hash、list、set、zset、slowlog、pipeline、pubsub、bitmap、HyperLogLog、GEO
  - 持久化：RDB（触发机制：save、bgsave、配置bgsave、全量复制）、AOF（刷新策略：severysec、always、everysecno，AOF重写）
  - 缓存更新策略：最大内存+LRU、过期时间超时剔除、业务上主动随数据库一起更新，高低一致性）
  - 缓存穿透：key对应的数据在缓存和数据源都不存在。缓存有过期时间的空对象；布隆过滤器拦截
  - 缓存击穿：key对应的数据存在，但在redis中过期。分布式锁防止并发访问db
  - 缓存雪崩：大量缓存集中失效。加锁；请求队列；设计时就要将缓存失效时间分散开尽量不要使大量数据的过期时间在同一个时间段
  - 无底洞：集群会使如mget这样的批操作非原子化，使更多的机器!=更高的性能。不要盲目加机器；多考虑命令优化、网络通信优化

  - 布隆过滤器：二进制向量01数组、多hash、多元素，本地、远程
  - 主从：全量复制、部分复制、偏移量、自动故障转移、读写分离。全量复制：需要尽量规避。第一次不可避免，
- etcd：KV（op）、Lease（KeepAlive）、Watcher（revition）、txn，key有序，内置grpc，raft         
- mongo：bson，database、collection、document、field
- kafka：topic、partition。消息模型、事务、丢失（三端）、重复（前置、唯一 幂等）、积压（两端   ），异步系统、异步网络io、序列化、协议、磁盘io、数据压缩，批处理，zookeeper

































