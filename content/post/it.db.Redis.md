

+++

title = "Redis"
description = "Redis"
tags = ["it", "db"]

+++



# redis

https://redis.io/，http://www.redis.cn/

- Redis的特性
  - 速度快
    - 数据存在内存（主要原因）
    - c语言编写（50000line，单机的核心代码只有23000line）
    - 单线程：
  - 持久化：断电不丢数据。Redis所有数据保持在内存中,对数据的更新将异步地保存到磁盘上
    - RDB
    - AOF
  - 多种数据结构：key:value，value支持以下结构
    - 基本
      - Strings/Blobs/Bitmaps 
      - Hash Tables (objects!)
      - Linked Lists
      - Sets
      - Sorted Sets
    - 衍生
      - BitMaps：位图。本质是字符串实现
      - HyperLogLog：超小内存唯一值计数。本质是字符串实现
      - GEO：地理信息定位。本质是集合实现
  - 支持多种编辑语言
  - 功能丰富
    - 发布订阅
    - Lua脚本
    - 事务
    - pipeline
  - "简单"
    - 不依赖外部库(like libevent)
    - 单线程模型
  - 主从复制
  - 高可用、分布式
    - 高可用Redis Sentinel(v2.8)支持高可用
    - 分布式Re dis Cluster(v3.0)支持分布式
- 典型使用场景
  - 缓存系统
    - user - AppServer - cache - Stoage：如果缓存中有直接返回，没有则到Storage中取（并存入cache）。redis则充当cache的角色
  - 计数器
    - 如点赞转发评论数，可以在单线程下非常高效的进行计数
  - 消息队列系统
    - redis提供了发布订阅、阻塞队列来实现类似的模型，可用于实现一些对消息队列功能需求不是很强的系统时，可以直接使用redis，节省技术成本
  - 排行榜
    - 有序集合
  - 社交网络
    - 粉丝数、关注数、共同关注、时间轴列表，等都可以用redis实现
  - 实时系统
    - 如使用位图实现类似布隆过滤器这样的功能，实现对一些垃圾邮件过滤、实时系统处理等非常有帮助

# client

https://redis.io/clients ，redis客户端每种语言都有很多，自行选择
### go

#### redigo

```shell
$ go get github.com/gomodule/redigo
```

###### .go

```go
c, err := redis.Dial("tcp", "127.0.0.1:6379")
if err != nil {
    fmt.Println(err)
    return
}
defer c.Close()

/*set*/
V, err := c.Do("SET", "hello", "world")
if err!=nil{
    fmt.Println(err)
    return
}
fmt.Println(v)

/*get*/
v, err = redis.String(c.Do("GET", "hello"))
if err!=nil{
    fmt.Println(err)
    return
}

/*其它命令大都类似*/
```



### java

#### jedis

###### .pom

```xml
<!--redis-client：jedis-->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.2.0</version>
</dependency>
<!--还可以导入jackson-databind进行redis string序列化相关操作-->
```

##### Jedis直连

- jedis直连：TCP连接。适用于少量长期连接的场景。存在每次新建/关闭TCP开销，资源无法控制,存在连接泄露
  的可能，Jedis对象线程不安全
  1. 生成一个Jedis对象，这 个对象负责和指定Redis节点进行通信：Jedis jedis = new Jedis(" 127.0.0.1", 6379);
     - 构造函数很多，介绍一个：Jedis(String host, int port, int connectionTimeout, int soTimeout)
       - host：Redis节点的所在机器的IP
       - port：Redis节点的端口
       - connectionTimeout：客户端连接超时
       - soTimeout：客户端读写超时
  2. jedis执行set操作：jedis.set("hello", "world");
  3. jedis执行get操作，value= "world"：String value = jedis.get("hello' );
  4. 关闭连接：jedis.close();
  5. 所有 jedis 操作函数名都和 redis api 操作命令一致

##### jedis连接池

- jedis连接池：Jedis预先生成,降低开销使用，连接池的形式保护和控制资源的使用。相对于直连，使用相对麻烦,
  尤其在资源的管理上需要很多参数来保证, 一旦规划不合理也会出现问题。和一般的连接池一样，需要合理规划防止连接池爆满阻塞或者大量连接闲置等

  ```java
  //初始化Jedis连接池，通常来讲JedisPool是单例的。
  GenericObjectPoolConfig config = new GenericObjectPoolConfig();
  config.setMaxTotal(10);
  JedisPool jedisPool = new JedisPool(config, "127.0.0.1", 6379)
  
  Jedis jedis = null;
  try {
      // 1.从连接池获取jedis对象
      jedis = jedisPool.getResource();
      // 2.执行操作
      jedis.set("hello" ，"world" );
  } catch (Exception e) {
      e.printStackTrace();
  } finally {
      if (jedis != nulI) {
          //如果使用JedisPool,close操作不是关闭连接，代表归还到连接池
          jedis. close();
      }
  }
  ```

###### 配置优化

- commons-pool配置(1)-资源数控制
  - maxTotal：资源池最大连接数，缺省值8。
    - 建议：比较难确定的，举个例子 :1.命令平均执行时间0.1ms = 0.001S；2.业务需要50000 QPS；3.maxTotal理论值= 0.001 * 50000 = 50个。实际值要偏大一些。或者通过jmx来统计后设置
    - 业务希望Redis并发量
    - 客户端执行命令时间
    - Redis资源:例如nodes(例如应用个数) * maxTotal是不能超过redis的最大连接数。(config get maxclients)
  - maxIdle：资源池允许最大空闲连接数，缺省值8。后面讨论
    - 建议maxIdle = maxTotal，减少创建新连接的开销。
  - minIdle：资源池确保最少空闲连接数，缺省值0。后面讨论
    - 建议预热minIdle，减少第一次启动后的新连接开销。在连接池初始化的时候提前去getResource一下
  - jmxEnabled：是否开启jmx监控,可用于监控，缺省值true。建议开启
- 借还参数
  - blockWhenExhausted：当资源池用尽后,调用者是否要等待。只有当为true时，下面的maxWaitMillis才会生效。缺省值true。建议使用默认值
  - maxWaitMillis：当资源池连接用尽后,调用者的最大等待时间(单位为毫秒)。缺省值-1，表示永不超时。不建议使用默认值
  - testOnBorrow：向资源池借用连接时是否做连接有效性检测(ping)，无效连接会被移除。缺省值false。建议false
  - testOnReturn：向资源池归还连接时是否做连接有效性检测(ping)，无效连接会被移除。缺省值false。建议false

###### 常见问题

- redis.clients.jedis. exceptions. JedisConnectionException: Could not get a resource from the pool
  - 获取空闲连接超时：Caused by: java.util.NoSuchElementException: Timeout waiting for idle object。连接池没有空闲资源了
  - 池枯竭，资源耗尽：Caused by: java.util.NoSuchElementException: Pool exhausted。如maxIdle和minIdle不等时，
- 常见解决思路：
  - 慢查询阻塞：出现一个连接查询慢，其它连接阻塞等待。需要根据业务场景合理设置超时时间
  - 资源池参数不合理：例如QPS高、池子小。调大池子
  - 连接泄露(没有close())：此类问题比较难定位，可通过client list、netstat等，最重要的是代码
  - DNS异常等



### python

#### redis-py



# hello

###### redis安装

```shell
wget http://download.redis.io/releases/redis-3.0.7.tar.gz #下载
tar -xzf redis-3.0.7.tar.gz #解压
ln -s redis-3.0.7 redis #软连接，便于升级
cd redis #进入软连接目录
make #编译
make install # #安装
cd src/
ll | grep redis- #查看redis-开头的文件
```

- 可执行文件：src目录

  - redis-server：Redis服务器
  - redis-cli：Redis命令行客户端 
  - redis-benchmark：Redis性能测试工具
  - redis-check-aof：AOF文件修复工具
  - redis-check-dump：RDB文件检查工具
  - redis-sentinel：Sentinel服务器(2.8以后)

- 启动方法:

  - 最简启动：默认。直接执行redis-server

  - 动态参数启动：指定一些动态参数。redis-server --port 6380

  - 配置文件启动：推荐。redis-server <configPath>。生成环境选择配置启动，单机多实例配置文件可以用端口区分开

    ```shell
    cd redis #进入redis目录
    mkdir config #创建config目录，可能启动很多redis，为了统一管理
    cp redis.conf config/redis-6379.conf #拷贝默认配置文件到config，以对应端口号命名
    
    #删掉所有注释和空格并重定向到redis-6382.conf
    cat config/redis-6381.conf| grep -v "#" | grep -v "^$" > config/redis-6382.conf
    vim config/redis-6382.conf #编辑
    ```

    ```shell
    #暂时只需要这些参数，其它都删掉
    daemonize yes #是否以守护进程的方式启动，默认no，一般设置yes
    port 6382
    dir "/app/it/database/redis/data"
    Logfile "redis-6382.log"
    ```

    ```shell
    src/redis-server config/redis-6382.conf #启动
    ps -ef | grep redis-server | grep 6382 #检查
    cat data/redis-6382.log #查看日志
    ```

- 启动验证

  - 查看进程的方式：ps -ef | grep redis-server、ps -ef | grep redis-server | grep -v grep
  - 查看端口是否时lesten的状态：netstat -antpl| grep redis
  - 直接ping：redis-cli -h ip -p port ping

- Redis客户端连接

  ```shell
  redis-cli -h 10.10.79.150 -p 6380 #连接，缺省参数为127.0.0.1和6379
  ping #连接成功的话可以收到响应：PONG
  set hello world #响应：OK
  get hello #响应："world"
  ```

- 响应

  - 状态回复，如：ping→PONG
  - 错误恢复，如：hget hello field → (error) WRONGTYPE Operation against
  - 整数回复，如：incr hello → (integer) 1
  - 字符串回复，如：get hello → "world"
  - 多行字符串回复，如：mget hello foo → 1) "world" 2) "bar"

- 常用配置

  - daemonize：是否是守护进程(no|yes)
  - port：Redis对外端口号。缺省值6379，对应手机9宫格键盘的MERZ，取自意大利歌女Alessia Merz的名字
  - logfile：Redis系统日志，仅指定日志名
  - dir：Redis工作目录，日志、持久化文件的存储目录
  - RDB config、AOF config、slow Log config、maxMemory等等，后续章节介绍


###### docker安装

```shell
$ docker pull redis:5.0.9
$ docker run --name redis01 -d -p 6379:6379 redis:5.0.9 redis-server --appendonly yes #run并启用持久化
$ docker exec -it redis01 /bin/bash
$ redis-cli #使用客户端
$ set hello "world"
$ get hello
$ del hello
```



# api

- Redis API使用和理解
  - 通用命令
    - 通用命令
    - 数据结构和内部编码
    - 单线程架构
  - 字符串类型
  - 哈希类型
  - 列表类型
  - 集合类型
  - 有序集合类型

### 通用命令

###### info

info memory：查看redis内存使用信息

###### keys

O(n)

- 用于展示key。
  - keys命令一般不要在生产环境使用
    - 是一个O(n)的命令
    - redis是单线程，会阻塞其它命令
  - 怎么用
    - 热备从节点：在热备从节点上使用
    - scan

```shell
set hello world
set php good
set java best
keys * #遍历所有key
dbsize #计算key的总数
```

```shell
mset hello world hehe haha php good phe his #批量插入数据
keys he* #展示以he开头的key，hehe、hello
keys he[h-l]* #第三个字母是h到l的范围
keys ph? #一个?代表一位，即长度为3位的ph开头的key
```

###### dbsize

O(1)

- 可以在生产环境使用。redis内置有计数器，实时记录key的总数，O(1)

```shell
dbsize #计算key的总数
```

###### exists key

O(1)，一般来说可以随意使用

```shell
set a b
exists a #存在响应：(integer) 1
del a
exists a #不存在响应：(integer) 0
```

###### del key [key ..]

O(1)

```shell
set a b
get a #响应："b"
del a #删除。删除成功响应：(integer) 1
get a #响应：(nil)
del key [a k1 k2] #批量删除。删除失败响应：(integer) 0
```

###### expire、ttl、persist

O(1)

```shell
expire key 3 #key在3秒后过期
ttl key #查看key剩余的过期时间。-2表示key已经过期（已经不存在），-1表示key存在并且没有设置过期时间
persist key #去掉key的过期时间
```

###### type key

O(1)

```shell
type key #返回key的类型。string、hash、list、set、zset、none（不存在的key）
```



#### 数据结构和内部编码

- key：key的数据结构有 string、hash、list、set、zset
  - string：string内部编码有 raw、int、embstr
    - raw
    - int
    - embstr
  - hash：string内部编码有 hashtable、ziplist
    - hashtable：hash表
    - ziplist：压缩列表
  - list：string内部编码有 linkedlist、ziplist
    - linkedlist
    - ziplist
  - set：string内部编码有 hashtable、intset
    - hashtable
    - intset
  - zset：string内部编码有 skiplist、ziplist
    - skiplist
    - ziplist
- redisObject
  - 数据类型(type)：用户只需要知道数据结构即可，而不需要知道编码方式，类似于面向接口编程
  - 编码方式(encoding)
  - 数据指针(ptr)
  - 虚拟内存(vm)
  - 其他信息

#### 单线程

- 命令串行执行，类似于队列
- 单线程为什么这么快？
  - 纯内存：主要原因，其它都是辅助原因
  - 非阻塞IO：阻塞IO和非阻塞IO的区别 https://www.cnblogs.com/ynyhl/p/9792699.html
  - 避免线程切换和竞态消耗
- 使用
  - 一次只运行一条命令
  - 拒绝长(慢)命令：keys、flushall、flushdb、slow lua script、mutil/exec、operate big value(collection)
  - 其实不是单线程，如 fysnc file descriptor、close file descriptor 这样的操作会有独立的线程来做，但是整体模型大致是单线程的，只有极个别例外

### 字符串

结构和命令

- Up to 512MB：最大512MB，但是一般不要存太大（考虑网络开销、单线程等），所以一般100KB以内差不多了，一般也够用，最多几MB吧

- key:value

  - hello world：value可以是字符串的
  - counter 1：value可以是数字的，是可以内部数字、字符串转换的
  - bits 10111101：value是二进制的
  - 还有json、xml等序列化或者压缩的value，实际上本质上所有的key value的value都是二进制的

- 场景

  - 缓存
  - 计数器
  - 分布式锁
  - 等等

- 命令

  - set：O(1)。设置key-value。set hello "world"
  - get：O(1)。获取key对应的value。get hello
  - del：O(1)。删除key-value。del hello
  - incr：O(1)。key自增1，如果key不存在，自增后get(key)=1，即创建key并0开始计算并加1。incr count。因为redis天然单线程无竞争，不会因为大并发量记错数。所以也可以用这种方式实现自增的分布式id
  - decr：O(1)。key自减1，如果key不存在，自减后get(key)=-1，即创建key并0开始计算并减1。decr count
  - incrby：O(1)。key自增k，如果key不存在，自增后get(key)=k。incrby count 3
  - decrby：O(1)。key自减k，如果key不存在，自减后get(key)=-k。decrby count 3

- 实战

  - 记录网站每个用户个人主页的访问量

    - incr userid:pageview：如userid为1，pageview为index。则 incr 1:index
    - string实现用户信息
      - 一个key:value记录一个用户，如key是"user:1"，value是json串。使用时必须全部取出，且需要序列化开销
      - 多个key:value记录一个用户，一个key:value记录一个属性，key命名"user:1:age"，value是属性值，key较多，内存开销多一些，且key教分散的

  - 缓存视频的基本信息（数据源在MySQL中）伪代码

    - 如用户通过vedio_id获取视频信息，先到redis中找，找到则返回，找不到则到mysql中找，返回给用户同时也将视频信息存入redis

      ```java
      //伪代码
      public VideoInfo get(long id) {
          String redisKey = redisPrefix + id;
          VideoInfo videoInfo = redis.get(redisKey).desocialization(); //有反序列化的过程，因为redis中存的是二进制，转为VideoInfo对象需要反序列化
          if (videoInfo == nul) {
              videoInfo = mysql.get(id);
              if (videoInfo != nul) {
                  redis.set(redisKey, videoInfo.serialize()); //序列化并存入redis
              }
          }
          return videoInfo;
      }
      ```

- 补充：

  - 选择操作
    - set：set key value #不管key是否存在，都设置
    - setnx：setnx key value #key不存在，才设置。类似于新增操作
    - set xx：set key value xx #key存在，才设置。类似于更新操作
    - 无论setnx还是set xx本质上都是set命令，nx和xx类似于选项，如还有set ex，这是一个set和expire的组合命令，可以在set的同时设置过期时间，且是一个原子操作，这在分布式锁的时候非常有帮助
  - 批量
    - mget key1 key2 key3...：O(n)。批量获取key，原子操作。相比于 n次get = n次网络时间+n次命令时间，网络时间是巨大的开销，因为我们的程序服务器与redis可能在不同的机器、不同的机房、甚至不同的地区城市。而1次mget = 1次网络时间 + n次命令时间，节省网络时间，但要注意mget一次携带太多数据也会产生负面影响（数据量很大一般拆分开来，如1000个kv使用一次mget），且要注意时间复杂度是O(n)
    - mset key1 value1 key2 value2 key3 value3：O(n)。批量设置key-value
  - more
    - getset key newvalue：O(1)。set key newvalue并返回旧的value
    - append key value：O(1)。将value追加到旧的value
    - strlen key：O(1)。返回字符串的长度（注意中文，redis中utf8的一个中文是2字节）。之所以是O(1)，是因为在存储的字符串内部也实时记录了该字符串的长度，不需要遍历字符计算长度
    - incrbyfloat key 3.5：O(1)。增加key对应的值3.5
    - getrange key start end：O(1)。获取字符串指定下标所有的值
    - setrange key index value：O(1)。设置指定下标所有对应的值，即给指定下标设置新的值

### hash

- 一个key→多个field:value：类似于一个Map<String,Map<String,String>>

  - user:1:info
    - name:Ronaldo
    - age:40
    - Date:201
    - viewCounter:50

- 命令：

  - hget：O(1)。hget key field #获取hash key对应的field的value
    - hgetall #获取所有field和value
  - hset：O(1)。hset key field value #设置hash key对应field的value
  - hdel：O(1)。hdel key field #删除hash key对应field的value
  - hexists：O(1)。hexists key field #判断hash key是否有field
  - hlen：O(1)。hlen key #获取hash key field的数量
  - hmget：O(n)。hmget key field1 field2...fieldN #批量获取hash key的一批field对应的值

  - hmset：O(n)。hmset key field1 value1 field2 value2...fieldN valueN #批量设置hash key的一批field value、

- 实战

  - 记录网站每个用户个人主页的访问量

    - hash记录用户信息就简单的多了。key为"user:1:info"，多个field和对应value做属性名和值。直观，可以部分更新，且节省内存。而且使用ziplist编码可以更加节省内存。但是ttl不好控制，因为过期时间无法设置到个体fied，只能针对key，想要实现只能自己写管理或删除的逻辑

    ```shell
    hincrby user:1:info pageview count
    ```

  - 缓存视频的基本信息(数据源在mysqI中)

    ```java
    //伪代码，相比字符串，无须序列化和反序列化过程
    public VideoInfo get(long id) {
        String redisKey = redisPrefix + id;
        Map < String,String> hashMap = redis .hgetAll(redisKey);
        VideoInfo videoInfo = transferMap ToVideo(hashMap);
        if (videoInfo == nulI) {
            videoInfo = mysql.get(id);
            if (videoInfo != nul) {
                redis.hmset(redisKey, transferVideo ToMap(videoInfo));
            }
        }
    }
    ```

- 补充：

  - hgetall：O(n)。hgetall key #返回hash key对应所有的field和value。小心使用，牢记redis是单线程，如果field过多，获取过大数据量影响很大

  - hvals：O(n)。hvals key #返回hash key对应所有field的value
  - hkeys：O(n)。hkeys key #返回hash key对应所有field
  - hsetnx：O(1)。类似setnx，hsetnx key field value #设置hash key对应field的value(如field已经存在，则失败)
  - hincrby：O(1)。类似incrby。hincrby key field intCounter #hash key对应的field的value自增intCounter
  - hincrbyfloat：O(1)。hincrbyfloat key field floatCounter #hincrby浮点数版



### list

有序，可以重复

- 命令
  - rpush：O(1~n)。rpush key value1 value2...valueN #从列表 右 端插入值(1-N个)
  - lpush：O(1~n)。lpush key value1 value2...valueN #从列表 左 端插入值(1-N个)
  - linsert：O(n)。linsert key before|after value newValue #在list指定的值前|后插入newValue
  - lpop：O(1)。lpop key #从列表 左 侧弹出一个item
  - rpop：O(1)。rpop key #从列表 右 侧弹出一个item
  - Irem：O(n)。Irem key count value #根据count值，从列表中删除所有value相等的项
    - (1) count>0，从左到右，删除最多count个value相等的项
    - (2) count<0，从右到左，删除最多Math.abs(count)个value相等的项
    - (3) count=0，删除所有value相等的项
  - ltrim：O(n)。ltrim key start end #按照索弓|范围修剪列表，即保留start-end，其它都删掉（不含start和end）
  - lrange：O(n)。lrange key start end (包含end) #获取列表指定索引|范围所有item
    - list还有一个从右到左的索引，如从左到右索引是0~5，则从右到左则是-1~6，所以lrange key 1 5 与 lrange key 1 -1 是等价的
  - lindex：O(n)。lindex key index #获取列表指定索引|的item
  - llen：O(1)。llen key #获取列表长度
  - lset：O(n)。Iset key index newValue #设置列表指定索弓|值为newValue
- 实战
  - TimeLine：如微博，它会将你关注用户的动态按时间排序，你的timeline中存按序存放着微博动态的id（具体内容可以存为hash或者sting之类的用id进行外联即可），关注的人有新的动态就push进来
- 补充
  - blpop：O(1)。blpop key timeout #lpop阻塞版本，timeout是阻塞超时时间，timeout=0将一直阻塞，如对空的list执行lpop将立刻return，但是如果使用blpop它就会阻塞等待，直到list不为空，然后pop一个出来。对生产者消费者模式或者消息队列有帮助
  - brpop：O(1)。
- 数据结构实现：
  - LRUSH + LPOP = Stack
  - LPUSH + RPOP = Queue
  - LPUSH + LTRIM = Capped Collection
  - LPUSH + BRPOP = Message Queue

### set

- 元素：无序、不可重复
- 命令
  - sadd：O(1)。sadd key element #向集合key添加element(如果element已经存在添加失败)
  - srem：O(1)。srem key element #将集合key中的element移除掉
  - scard ber (1)。scard user:1:follow = 4 #计算集合大小
  - sismember ：O(1)。sismember user:1:follow it = 1(存在) #判断it是否在集合中

  - srandmem ：O(1)。srandmember user:1:follow count= his #从集合中随机挑count个元素
  - spop：spop user:1:follow = sports #从集合中随机弹出一个元素
  - smembers：O：O(1)。smembers user:1:follow = music his sports it #获取集合所有元素，返回结果无序，数据较多时谨慎使用
  - sscan：O(1)。使用游标获取集合中的值。即扫集合中的元素，根据游标
- 实战
  - 转发抽奖：将转发者存入集合，spop弹出
  - 赞、踩：将点赞者存入集合，scard获取数量，当然其它数据结构也可以做
  - 标签：给用户添加标签和给标签添加用户属于同一个事务
    - 给用户添加标签：sadd user:1:tags tag1 tag2 tag5，sadd user:2:tags tag2 tag3 tag5，sadd user:k:tags tag1 tag2 tag4
    - 给标签添加用户：sadd tag1:users user:1 user:3，sadd tag2:users user:1 user:2 user:3，sadd tagk:users user:1 user:2
- 补充：
  - sdiff：sdiff user:1:follow user:2:follow = music his #差集

  - sinter：sinter user:1:follow user:2:follow = it sports #交集，如共同关注
  - sunion：sunion user:1:follow user:2:follow = it music his sports news ent #并集
  - sdif | sinter | suion + store destkey ..  #将差集、交集、并集结果保存在destkey中
- TIPS
  - SADD = Tagging 
  - SPOP/SRANDMEMBER = Random item
  - SADD + SINTER = Social Graph

### zset

有序集合，通过打分score来实现有序，无重复element

- 一个key，多个score:element
- 命令
  - zadd：o(logN)。`zadd key score element`(可以是多对) #添加score和element
  - zrem：o(1)。`zrem key element`(可以是多个) #删除元素
  - zscore：o(1)。`zscore key element` #返回元素的分数
  - zincrby：o(1)。`zincrby key increScore element` #增加或减少(负数)元素的分数
  - zcard：o(1)。`zcard key` #返回元素的总个数
  - zrank：o(n)。`zrank key` #获取元素从低到高的排名
  - zrange：o(log(n)+m)，n指元素总数，m指要获取的范围个数。`zrange key start end [WITHSCORES]` #返回指定索引|范围内的升序元素[分值]，WITHSCORES可选，如果写上会带上分值
  - zrangebyscore：o(log(n)+m)。`zrangebyscore key minScore maxScore [WITHSCORES]` #返回指定分数范围内的升序元素[分值]
  - zcount：o(log(n)+m)。`zcount key minScore maxScore` #返回有序集合内在指定分数范围内的个数
  - zremrangebyrank：o(log(n)+m)。`zremrangebyrank key start end` #删除指定排名内的升序元素
  - zremrangebyscore：o(log(n)+m)。`zremrangebyscore key minScore maxScore` #删除指定分数内的升序元素
- 实战
  - 排行榜：score设定: timeStamp saleCount followCount
- 补充
  - zrevrank：zrevrank key #获取元素从低到高的排名，与zrank相反
  - zrevrange：从高到低，与zrange相反
  - zrevrangebyscore：从高到低，与zrangebyscore相反
  - zinterstore：计算交集并存储
  - zunionstore：计算并集并存储



## 更多功能

### 慢查询

- 一个查询的生命周期：client 发送命令→阻塞等待排队执行（redis单线程，命令执行类似于队列）→执行命令→返回结果
  - 慢查询发生在执行命令阶段，如hgetAll一个很大的hash
  - 客户端超时不一定是因为慢查询，但慢查询是客户端超时的一个可能因素，因为4个阶段任意一个阶段慢了都可能导致超时
- 一个列表：慢查询列表，一个redis list实现的队列
  - 如果一个命令执行时间超过了所配置的慢查询时间阈值，将作为慢查询被列入慢查询列表
  - 慢查询列表是固定长度：由slowlog-max-len配置
  - 慢查询列表保存在内存中
- 两个配置：修改配置文件（一般仅第一次启动时这样做），启动后建议动态配置即可
  - slowlog-max-len：配置慢查询列表大小
    - 查看：config get slowlog-max-len。缺省值128。
    - 动态配置：config set slowlog-max-len 1000
  - slowlog-log-slower-than：慢查询时间阈值。单位：微秒。为0时记录所有命令（开发调试可能用到吧，为了查看某个具体的命令临时设置一次）。
    - 查看：config get slowlog-log-slower-than。缺省值10000。
    - 动态配置：config set slowlog-log-slower-than 1000
- 三个命令
  - slowlog get [n]：获取慢查询队列，n指定条数
  - slowlog len：获取慢查询队列长度
  - slowlog reset：清空慢查询队列
- 运维经验
  - slowlog-max-len不要设置过小，通常设置1000左右
  - slowlog-log-slower-than不要设置过大，默认10ms，通常设置1ms。当然实际场景还是根据qps适当调整
  - 理解命令生命周期
  - 定期持久化慢查询：需要自己或者通过其它开源工具实现

### pipeline

- pipeline：流水线。用于打包一批命令，批量执行

  - 一次网络命令通信模型：客户端传输命令（网络）→服务器计算（执行命令）→返回结果（网络）。即1次时间= 1次网络时间（1、3）+ 1次命令时间（2）
  - 批量网络命令通信模型：n次时间= n次网络时间+ n次命令时间。
  - 流水线通信模型：1次pipeline(n条命令) = 1次网络时间+ n次命令时间。命令时间很快（redis命令时间是微秒级的），但是网络时间可能很大（跨地区跨城市），也许会想到mget、mset，但是如果是hash结构，不存在mhmset，包括也许还想要同时统一发送get、hget，流水线就是为此存在的，将一批命令批量打包一次发送，再在服务端批量计算，按顺序将结果批量打包一次性返回
  - redis命令时间是微秒级的；pipeline每次条数要控制（网络因素，一次性传递巨大的数据包是不太好的）

- 客户端实现

  ```java
  //hset 10000次。这样就只需要10次网络时间
  Jedis jedis = new Jedis("127.0.0.1" 6379);
  for (inti = 0;i < 10=; i++) {
      Pipeline pipeline = jedis. pipelined();
      for (intj= i* 1000;j < (i+ 1) * 1000;j++) {
          pipeline.hset("hashkey:" + j, "field" + j, "value" + j);
          pipeline.syncAndReturnAll();
      }
  }
  ```

- 对比原生操作（即m操作，如mset）

  - m命令：是原子的，是redis原生支持的命令，在操作队列中就是一个命令
  - pipeline命令：是非原子的，将根据其中打包的命令，被重新拆为一堆pipeline子命令，然后才挨个进入命令操作队列，中间是可以穿插其它操作命令的

- 使用建议

  - 注意每次pipeline携带数据量
  - pipeline每次只能作用在一个Redis节点 上

### 发布订阅

- 发布订阅：redis仅提供发布订阅的功能，但没有消息堆积的功能，新的订阅者无法从redis获取到历史消息

  - 角色：发布者publisher、订阅者subscriber、频道channel

  - API

    - publish：publish channel message

      ```shell
      redis> publish yuanya:channel "hello" #向"yuanya:channel"频道发布消息
      (integer) 3 #订阅者个数
      ```

    - subscribe：subscribe [channel] #一个或多个

      ```shell
      redis> subscribe yuanya:channel #从yuanya:channel订阅消息
      1) "subscribe"
      2) "yuanya:channel"
      3) (integer) 1
      1) "message"
      2) "yuanya:channel"
      3) "hello"
      ```

    - unsubscribe：unsubscribe [channel] #一个或多个

  - 补充

    - psubscribe [pattern..] #订阅模式。psubscribe v* #订阅v开头的频道
    - punsubscribe [pattern..] #退订指定的模式。
    - pubsub channels #列出至少有一个订阅者的频道。pubsub numsub [channel..] #列出给定频道的订阅者数量，pubsub numpat #列出被订阅模式的数量

- 消息队列：消费者是竞争关系，每一条消息只被一个消费者消费一次。可以使用list来实现

### Bitmap

- 位图：如 "big" 对应 ascii 码的二进制为：01100010 01101001 01100111，通过位图可以获取其每一个位的值

  ```shell
  set hello big  #OK
  getbit hello 0 #(integer) 0
  getbit hello 1 #(integer) 1
  ```

- 命令

  - setbit：setbit key offset value #给位图指定索弓|设置值（值只能是0或1）。返回结果是之前该位置对应的值（如果没有值就是0），offset 跳过的中间未设置值的位将补0，所以最大offset不要突然变大，否则补0影响性能

    ```shell
    setbit key 0 1 #(integer) 0。当前位图：1
    setbit key 7 1 #(integer) 0。当前位图：10000001
    setbit key 9 1 #(integer) 0。当前位图：1000000101
    ```

  - bitcount：bitcount key [start end]，获取位图指定范围（start到end，单位为字节，如果不指定就是获取全部）位值为1的个数

  - bitop：bitop op destkey key [key....，做多个Bitmap的and(交集)、or(并集)、 not(非)、 xor(异或) 操作并将结果保存在destkey中

  - bitpos：bitpos key targetBit [start] [end]，算位图指定范围(start到end，单位为字节，如果不指定就是获取全部)第一个偏移量对应的值等于targetBit的位置

- 实战

  - 独立用户统计：总共有1亿用户

    - 假如每天5千万独立访问用户，使用set和Bitmap对比

      | 数据类型 | 每个userid占用空间                                   | 需要存储的用户量 | 全部内存量（一天）         | 一个月 | 一年 |
      | -------- | ---------------------------------------------------- | ---------------- | -------------------------- | ------ | ---- |
      | set      | 32位(假设userid用的是整型，实际很多网站用的是长整型) | 50,000,000       | 32位 * 50,000,000= 200MB   | 6G     | 72G  |
      | Bitmap   | 1位                                                  | 100,000,000      | 1位 * 100,000,000 = 12.5MB | 375M   | 4.5G |

    - 假如每天只有10万独立用户，合适的场景使用合适的技术

      | 数据类型 | 每个userid占用空间                                   | 需要存储的用户量 | 全部内存量（一天）         |
      | -------- | ---------------------------------------------------- | ---------------- | -------------------------- |
      | set      | 32位(假设userid用的是整型，实际很多网站用的是长整型) | 1,000,000        | 32位 * 1,000,000= 4MB      |
      | Bitmap   | 1位                                                  | 100,000,000      | 1位 * 100,000,000 = 12.5MB |

- 经验

  - 要注意的是位图type=string，所以最大是512M，大部分情况都可以满足一天的统计量，无法满足时，拆分为多个位图即可
  - 注意setbit时的偏移量,可能有较大耗时
  - 位图不是绝对好



### HyperLogLog

- 基于HyperLogLog算法：极小空间完成独立数量统计。本质还是字符串

- 命令

  - pfadd：pfadd key element [element ..  #向hyperloglog添加元素
  - pfcount：pfcount key [key ..  #计算hyperloglog的独立总数
  - pfmerge：pfmerge destkey sourcekey [sourcekey ...]  #合并多个hyperloglog

  ```shell
  redis> pfadd 2017_03_06:unique:ids "uuid-1" "uuid-2" "uuid-3" "uuid-4"
  (integer) 1
  redis> pfcount 2017_03._06:unique:ids
  (integer) 4
  redis> pfadd 2017_ 03_06:unique:ids "uuid-1" "uuid-2" "uuid-3" "uuid-90"
  (integer) 1
  redis> pfcount 2017_03_06:unique:ids
  (integer) 5
  ```
  - 内存消耗(百万独立用户)：1天 15KB

- 经验

  - 是否能容忍错误? (官方给出错误率: 0.81%)
  - 是否需要单条数据?不能单独取出



### GEO

- GEO(地理信息定位)：存储经纬度,计算两地距离,范围计算等
- 应用场景：实现如微信摇一摇这种类似功能、计算附近一定距离的酒店
  - 5个城市经纬度
    - 北京，116.28，39.55，beijing
    - 天津，117.12，39.08，tianjin
    - 石家庄，114.29，38.02，shijiazhuang
    - 唐山，118.01，39.38，tangshan
    - 保定，115.29，38.51，baoding
- api
  - geo：`geo key longitude latitude member [longitude latitude member...]` #增加地理位置信息
  - geopos：`geopos key member [member...]` #获取地理位置信息
  - geodist：`geodist key member1 member2 [unit]` #获取两个地理位置的距离
    - unit：m(米)、 km(千米)、mi(英里)、ft(尺)
  - georadius：georadius key longitude latitude radiusm|km|ft|mi [withcoord] [withdist] [withhash] [COUNT count] [asc|desc] [store key] [storedist key]
  - georadiusbymember：georadiusbymember key member radiusm|km|ft|mi [withcoord] [withdist] [withhash] [COUNT count] [asc|desc] [store key] [storedist key] #获取指定位置范围内的地理位置信息集合
    - withcoord：返回结果中包含经纬度。
    - withdist：返回结果中包含距离中心节点位置。
    - withhash：返回结果中包含geohash
    - COUNT count：指定返回结果的数量。
    - asc|desc：返回结果按照距离中心节点的距离做升序或者降序。
    - store key：将返回结果的地理位置信息保存到指定键。
    - storedist key：将返回结果距离中心节点的距离保存到指定键



# 持久化

- 持久化：redis所有数据保持在内存中，对数据的更新将异步地保存到磁盘上
  - 快照：将数据备份。如 MySQL Dump、Redis RDB
  - 日志：记录所有数据更新操作到日志，数据丢失时，完整重走一边日志记录的操作即可恢复。如 MySQL Binlog、Hbase HLog、Redis AOF

##### RDB

在硬盘创建RDB文件（二进制）备份数据，redis启动的时候载入RDB文件。主从复制中也可以用RDB文件做复制媒介

###### 触发机制

触发机制-主要三种方式

- save：该命令是同步的。主动创建RDB文件，执行时将阻塞redis。不会消耗额外内存

  - 文件策略：如存在老的RDB文件，将替换
  - 复杂度：O(n)

- bgsave：异步。主动创建RDB文件，将调用fork()（fork非常快，会消耗一定内存，使用不当还是可能发生阻塞）开一个子进程来执行，除了fork本身，之后不会阻塞redis客户端命令

  - 文件策略：如存在老的RDB文件，将替换
  - 复杂度：O(n)

- 自动：达到某些条件时自动触发RDB文件生成（bgsave方式），save配置。自动生成的文件为./dump.rdb

  - 以下是redis默认的3条save配置，满足任意一个即可触发

    | 配置 | seconds | changes | 说明                                                         |
    | ---- | ------- | ------- | ------------------------------------------------------------ |
    | save | 900     | 1       | 900秒改变（增删改等）了1条数据，生成rdb文件。即使每900秒只改变1条，也会每900秒生成rdb文件 |
    | save | 300     | 10      | 300秒改变了10条数据，生成rdb文件                             |
    | save | 60      | 10000   | 60秒改变了10000条数据，生成rdb文件                           |

  - 无法控制生成rdb的频率，写操作大的情况可能会很高

  - 默认配置

    ```shell
    save 900 1
    save 300 10 
    save 60 10000
    dbfilename dump.rdb             #rdb文件文件名
    dir ./                          #rdb文件保存目录
    stop-writes-on-bgsave-error yes #如果bgsave发生error，是否停止写入
    rdbcompression yes              #是否采用压缩格式
    rdbchecksum yes                 #是否进行校验和检验
    ```

  - 推荐配置

    ```shell
    #删掉默认save配置
    dbfilename dump-${port}.rdb #因为一台机器可能有多个redis
    dir /bigdiskpath #选择一个较大的磁盘路径，甚至可能根据不同端口上的redis分盘
    stop-writes-on-bgsave-error yes
    rdbcompression yes
    rdbchecksum yes
    ```

- 触发机制补充-不容忽略方式

  - 全量复制：如主从复制时，主会自动生成RDB文件
  - debug reload：debug级别的重启，不清空内存的重新加载，这也会触发生成RDB文件
  - shutdown：执行shutdown时会执行一个shutdownsave，也会生成RDB文件

- 试验

  - save阻塞：执行save同时在另一个窗口get一下，可以发现在save完成之前是阻塞的

  - bgsave fork：执行bgsave时立马查看进程，可以发现有多出一个子进程（执行完后消失），文件夹多出一个临时rdb文件（执行完后用替换掉原来的dump-6379.rdb）

    ```shell
    data ps -ef | grep redis- | grep -V "redis-cli" | grep -V "grep"
    501 36775     1 0 10:22下午 ?? 0:17.91 redis-server *:6379
    501 36954 36775 0 10:28下午 ?? 0:02.81 redis-rdb-bgsave *:6379 #子进程
    > data ll
    total 1126064
    -rw-r--r-- 1 carlosfu staff 1.9K 10 6 22:29 6379.10g
    -rw-r--r-- 1 carlosfu staff 459M 10 6 22:28 dump-6379.rdb
    -rw-r--r-- 1 carlosfu staff  90M 10 6 22:29 temp-36985.rdb #临时文件
    ```

  - 真的自动?是的

  - RDB长啥样?二进制文件，看不懂

- 

###### 总结与问题

- 总结
- RDB是Redis内存到硬盘的快照,用于持久化。
  - save通常会阻塞Redis。
  - bgsave不会阻塞Redis ,但是会fork新进程。
  - save自动配置满足任一就会被执行。
  - 有些触发机制不容忽视
- 问题

  - 耗时、耗性能
    - O(n)数据:耗时
    - fork() :消耗内存，copy-on-write策略
    - DiskI/O : IO性能
  - 不可控、丢失数据
    - T1：执行多个写命令
    - T2：满足RDB自动创建的条件
    - T3：再次执行多个写命令
    - T4：宕机，T3的命令将丢失



##### AOF

- AOF文件：记录所有写命令到AOF文件。

  - 如客户端发送set hello world命令→rerdis保存set hello world命令到AOF文件。
  - 如果宕机了，重启redis时载入aof文件进行恢复

###### 三种策略

三种策略：aof并不是直接写入磁盘，是写在磁盘的缓冲区中，缓冲区根据策略刷新到磁盘（为了提高写入）

- always：每条命令都fsync到硬盘。不会丢失数据，io开销大，一般sata盘只有几百tps
- everysec：默认策略。每秒把缓冲区fsync到硬盘。有可能丢失最后1秒的命令。一般就使用everysec
- no：操作系统os决定fsync。不用管，不可控。一般不使用

###### AOF重写

AOF重写：对redis当前在内存中的数据进行一次回溯，回溯成一个新aof文件，并覆盖旧aof文件

- 因为要记录每条写命令，长期下来aof文件会变得很大，恢复和写入都有一定性能影响。进行重写优化，来减少硬盘占用量、加速恢复速度

- 例子：例子只是作用效果举例，不是真的对原aof文件重写，实际是内存数据回溯

  - 假如有set hello world、set hello java、set hello hehe是aof中的记录，实际上只有最后一条是有意义的，AOF重写就会优化为只记录最后一条命令set hello hehe
  - 如incr counter、incr counter优化为set counter 2（假如incr了1e次，将优化了太多）
  - 如rpush mylist a、rpush mylist b、rpush mylist c，优化为rpush mylista bc
  - 设置过期时间并已经过期的数据，其相关操作直接优化为无了

- 实现

  - bgrewriteaof：异步，fork一个子进程执行
  - AOF重写配置：aof重写自动触发相关配置
    - 配置
      - auto-aof-rewrite-min-size：AOF文件重写需要的尺寸
      - auto-aof-rewrite-percentage：AOF文件增长率
    - 统计
      - aof_current_size：AOF当前尺寸(单位:字节)
      - aof_base_size：AOF_上次启动和重写的尺寸(单位:字节)
    - 触发时机：以下条件需要同时满足
      - 当前尺寸>AOF文件重写需要的尺寸：aof_current_size > auto-aof-rewrite-min-size
      - 当前增长率>AOF文件增长率：aof_current_size - aof_base_size / aof_base_size > auto-aof-rewrite-percentage

- 流程：bgrewriteaof命令发送给redis父进程。父进程fork一个子进程，去执行回溯；同时父进程任然会将新收到的写命令写入缓冲aof_buf并同步到旧aof文件；同时还会将新收到的写命令写入另一个缓冲当中aof_rewrite_buf，这个buf会在新aof文件生成完毕之后补充进去。最后新aof文件覆盖旧aof文件

- 配置

  ```shell
  appendonly yes #默认值是no。开启以使用aof重写相关功能
  appendfilename "appendonly-${port}.aof" #命名
  appendtsync everysec #同步策略
  dir /bigdiskpath #选一个大磁盘目录，甚至在比较严格情况情况下分盘
  no-appendtsync-on-rewrite yes #在aof重写的时候，是否不执行旧aof文件的append同步操作，yes即不做这个操作。如果是完全不允许丢失数据的场景，就设置no，因为万一aof重写时宕机，这段时间的数据可能就丢失了
  auto-aof-rewrite-percentage 100
  auto-aof-rewrite-min-size 64mb
  aof-load-truncated yes #重启加载aof文件时，如果出现错误是否忽略，因为可能重写时只刷入一半缓存时宕机，造成文件内容不完整
  ```
  
- aof文件内容：

  ```shell
  data> head appendonly.aof
  *2   # *指定接下来的一个命令是几个参数。下面这个命令有2个参数
  $6   # $指定接下来的一个参数是几个字节。下面这个参数有6个字节
  SELECT #确实是6个字节
  $1
  0    #select 0即选中了0号数据库
  *3
  $3
  set
  $5
  hello
  $5
  world
  ```

  

#### 取舍和选择

- 对比

  | 命令                                   | RDB                          | AOF                      |
  | -------------------------------------- | ---------------------------- | ------------------------ |
  | 启动优先级（如果都配置了，优先启动谁） | 低                           | 高                       |
  | 体积                                   | 小                           | 大                       |
  | 恢复速度                               | 快                           | 慢                       |
  | 数据安全性                             | 丢数据                       | 根据策略决定             |
  | 轻重                                   | 重（每次从内存全部写入硬盘） | 轻（每次只追加缓冲内的） |

- RDB最佳策略

  - "关"，但是启动时主从复制时全量复制还是会用到RDB，所以是无法彻底关闭的
  - 集中管理：用于备份数据是不错的选择
  - 主从，从开?：有时候需要在从节点开启RDB，以保存历史的RDB文件，但要控制自动生成的力度，不要太频繁，因为redis一般都是混合部署，单机多部署，而RDB是一个重操作

- AOF最佳策略

  - "开"：缓存和存储。因为一般是每秒追加缓存同步，如果丢失数据，从数据源加载即可，redis毕竟大多只作缓存，不是数据源。如果对数据源压力不大，redis也只是起到一定缓存作用，关掉也无妨，因为AOF每秒追加缓存到磁盘确实是有一定开销的
  - AOF重写集中管理：一般分配百分之六七十内存给redis，剩下的要留给做类似fork这样的操作，因为单机多部署，可能AOF重写集中发生，产生大量fork，就可能内存爆满等情况
  - everysec：建议使用everysec策略

- 最佳策略

  - 小分片（没听懂，对fork不熟）：使用max memory（最大内存）对redis进行规划，如每个redis的max memory只设置4g，这样fork、rdb复制传输、等操作都只会产生较小的开销。但也有缺点，分布式下，产生更多的redis进程可能对cpu占用更多
  - 缓存、存储：根据存储和缓存的特性来决定是否使用那种的策略
  - 监控(硬盘、内存、负载、网络)
  - 足够的（冗余）内存



## 开发运维常见问题

（很多没搞懂，对fork和linux不熟）

- fork操作：fork操作本身，不包括fork后产生的子进程
  - fork操作本身，是一个在主进程中完成的同步操作，只是做一个内存页的拷贝而不是完全做内存的拷贝，所以大部分情况下速度是非常快的。但如果fork操作本身较慢，则会阻塞redis主进程
  - 与内存量息息相关：内存越大，耗时越长(与机器类型有关)
  - info: latest fork_ usec：查询上一次fork操作消耗的微秒数，如果对此要关注，可以做一些监控或相对应的告警
  - 改善fork
    - 优先使用物理机或者高效支持fork操作的虚拟化技术
    - 控制Redis实例最大可用内存: maxmemory
    - 合理配置Linux内存分配策略: vm.overcommit _memory= 1。默认值0，如果没有足够内存做内存分配的时候（内存较低时）就不去分配，这将造成fork阻塞
    - 降低fork频率：例如放宽AOF重写自动触发时机，不必要的全量复制
- 进程外开销：子进程开销和优化
  - CPU：
    - 开销：RDB和AOF文件生成，属于CPU密集型（写入是一个集中的过程）
    - 优化：不做CPU绑定（如果绑定，会和主进程集中消耗cpu，可能对主进程造成很大资源影响），不和CPU密集型应用部署在一起
  - 内存：
    - 开销: fork内存开销，copy-on-write。因为子进程是通过fork来产生的，理论上占用内存是等于父进程，但是linux有一个显式复制的机制copy-on-write，父子进程共享相同的物理内存页，当父进程有写请求的时候，会创建一个副本，相当于这是才会消耗内存，而在整个期间子进程会共享fork时父进程的内存的快照，即在做如aof重写fork产生子进程过程中，如果父进程有大量内存写入，就证明子进程的内存会开销比较大，因为它会做一个副本，如果父进程没什么写入，实际上子进程也就开销不了多少内存
    - 优化: echo never > /sys/kernel/mm/transparent hugepage/enabled
  - 硬盘
    - 开销: AOF和RDB文件写入，可以结合iostat、iotop等工具分析
    - 优化
      - 不要和高硬盘负载服务部署一 起:存储服务、消息队列等
      - no-appendfsync-on-rewrite = yes
      - 根据写入量决定磁盘类型:例如ssd
      - 单机多实例持久化文件目录可以考虑分盘
- AOF追加阻塞
  - everysec流程：
    - 主线程将命令写入aof缓冲区；主线程还会负责对比上次fsync时间，如果距离上次fsync时间小于2秒，主线程返回继续其它操作，否则阻塞直到同步完成，这是为了保证aof文件安全性的策略
    - 同时还有一个AOF同步线程，用来每秒将缓冲区内容同步到硬盘，并且会记录最近一次fsync时间
  - 问题：
    - 主线程阻塞问题
    - aof丢失的数据，最大可达2秒
  - AOF阻塞定位
    - Redis日志：Asynchronous AOF fsync is taking too long (disk is busy?). Writing the AOF buffer without waiting for fsync to complete, this may slow down Redis
    - info persistence：记录上述过程发生的数量，aof_ delayed fsync: 100。没发生一次+1
    - 查看硬盘：top命令，查看是否有发生io资源紧张的时间
  - 硬盘优化策略，在上面已经有了，根据实际情况考量即可
- 单机多实例部署

# Redis缓存

- 缓存的使用与设计
  - 缓存的受益与成本
  - 缓存更新策略
  - 缓存粒度控制
  - 缓存穿透优化
  - 无底洞问题优化
  - 缓存雪崩优化
  - 热点key重建优化



## 缓存的受益与成本

- 收益
  - 加速读写
    - 通过缓存加速读写速度：如CPU L1/L2/L3 Cache、Linux page Cache加速硬盘读写、浏览器缓存、Ehcache缓存数据库结果。
  - 降低后端负载
    - 后端服务器通过前端缓存降低负载：业务端使用Redis降低后端MySQL负载等
- 成本
  - 数据不一致：缓存层和数据层有时间窗口不一致，和更新策略有关。如redis做缓存层，mysql做数据源，需要将数据源的数据放到缓存层进行缓存，而数据源更新时缓存需要如何更新？是立刻发出通知还是，按时间间隔轮询自动更新，还是懒加载用到时才更新。策略要根据所能容忍的时间窗口或范围来决定
  - 代码维护成本：多了一层缓存逻辑，缓存的读写、缓存与数据库的沟通等
  - 运维成本：例如Redis Cluster
  - 经济成本：实体机器成本或者云计算机器成本
- 使用场景
  - 降低后端负载
    - 对高消耗的SQL：join结果集/分组统计结果缓存。比如做排行榜计算，涉及很多表，做一个实时计算，很复杂，我们不需要每次都实时计算，只需要计算某个时间点的结果进行缓存即可
  - 加速请求响应:
    - 利用Redis/Memcache优化IO响应时间
  - 大量写合并为批量写
    - 如计数器：如果真的需要将计数写入DB，我们不能每次计数都写DB，先Redis累加再批量写DB



## 缓存更新策略

缓存的数据通常都有生命周期，需要做定期更新或删除以保证空间在可控范围内，且保证数据的定期更新。但是缓存中的数据和真实的数据可能存在不一致，需要一个合理的更新策略来保证数据的不一致在可容忍范围内

- LRU/LFU/FIFO算法剔除：例如maxmemory-policy，最大内存对应的策略，在达到最大内存时执行的策略

  - LRU：把最近没有使用的key删除，保证不超过maxmemory，尽可能保证了数据安全。还有all keys lru，即在所有的键去执行LRU
  - 使用场景：需要控制配置maxmemory

- 超时剔除：例如expire。设置过期时间，时间内访问到缓存中读来保证性能，如果时间内用户更行了一些重要的信息，expire就不太好了，所以对于不重要的信息可以expire，比如一个视频信息的说明修改了一个标点符号，可能时间内信息不一致是可以容忍的，但如果是涉及钱的金融方面的信息，肯定就不能用这样的策略了

- 主动更新：开发控制生命周期。用户信息在存储层发生了变化，如果对应的缓存层能通过业务代码或者开发一些工具能知道存储层的变化，比如订阅存储层一些消息的变化，比如更新存储层时会发布一条消息，缓存层收到消息则主动更新或者做一次重建，到数据层重新取一次数据然后缓存，来实现数据的一致性，虽然仍然不是完全一致的强一致性，而是需要一个最终一致性，最终实现一致性的时间也是比较短的，即由该更新机制保证的

  | 策略             | 一致性 | 维护成本 |
  | ---------------- | ------ | -------- |
  | LRU/LIRS算法剔除 | 最差   | 低       |
  | 超时剔除         | 较差   | 低       |
  | 主动更新         | 强     | 高       |

建议：

- 低一致性环境：最大内存+淘汰策略。随意往缓存里扔就行了，达到最大内存就淘汰即可，淘汰什么也无所谓，没有的话下次再到存储层读取重新构建缓存即可
- 高一致性情境：超时剔除、主动更新结合，超时策略为主动更新兜底，最大内存和淘汰策略做最终兜底。主动更新的代码是开发人员自己维护的，万一出了问题，没有将真正的数据删除，后期我们无法发现这样的问题，那么我们就给数据设置一个较长的过期时间，如果出了问题，最终数据也会被超时剔除，当然这里的数据指真正有生命周期的，确实要在一定时间后过期的，如果有数据真的是不会过期，就不能设置过期时间。最后是最大内存和淘汰策略兜底，因为无法保证那天监控不到位内存就上去了，保证高可用性，而不会直接爆内存导致缓存不可用



## 缓存粒度控制

- 从MySQL获取用户信息：select * from user where id= {id}
- 设置用户信息缓存：set user:{id} 'select * from user where id= {id}'。缓存数据来源于存储层，是需要做一个两者间映射的，key需要id值，value是select * from user where id= {id}的结果值，可能需要很多操作，如果对它做一个压缩，或者不使用字符串类型，使用hash类型，将每一个属性变成hash的field和value
- 缓存粒度：到底是缓存select *得到的所有字段还是仅仅缓存需要的字段
  - 全部属性：set user:{id} 'select * from user where id={id}
  - 部分重要属性：set user:{id} select importantColumn1, ...importantColumnK from user where id={id}'
- 缓存粒度控制-三个角度
  - 通用性：全量属性更好
- 占用空间：部分属性更好
- 代码维护：表面上全量属性更好。不会出现有新的业务需求时，需要用到的字段没有，还要重新到存储层单独取字段又更新到缓存。但是大部分时候，需求和业务固定了，我们只需要缓存我们需要的属性，而不需要过多考虑扩展性，虽然考虑扩展性是一个很好的习惯，但是在使用缓存的时候要更多考虑空间占用和性能问题，因为内存空间珍贵，而缓存也更是为了解决性能问题而存在的。所以还是要综合考虑



## 缓存穿透优化

- 缓存穿透：key对应的数据在数据源并不存在，每次针对此key的请求从缓存获取不到，请求都会到数据源，从而可能压垮数据源。比如用一个不存在的用户id获取用户信息，不论缓存还是数据库都没有，若黑客利用此漏洞进行攻击可能压垮数据库。

- 缓存穿透问题-大量请求不命中：request打到cache上，如果miss未命中，就会把流量往下引到storage层，正常情况存储层拿到对应的结果回写cache并返回response，当下次再有同样的数据请求时就可以直接命中cache，不需要到存储层取了。但是如果cache中miss时，storage也miss，将响应一个空的业务，当下一次再进行同样的请求时，他仍然会先访问cache再导到storage，并任然全部miss响应空，所有的流量都会打到存储层，这就是缓存穿透。缓存穿透使得缓存失去了意义，因为本来缓存就是为了保护存储层的，如果有大量这样的请求穿透缓存直击存储层，会给存储层带来很大的隐患

- 原因

  - 业务代码自身问题：比如从mysql拿了数据，结果存的时候用了空的或错的变量，导致存入了mysql原本根本不存在内容，并且还将key响应了出去供用户使用
  - 恶意攻击、爬虫等等：我们知道虽然一般视频网站的url做了很多加密，来防止别人猜到视频的id规则，但别人仍然可以强制访问不存在的id，就可能会发生缓存穿透

- 发现

  - 业务的响应时间：一般都会有监控系统，平时缓存扛了很大量，并且响应速度会很快，是可预期的，如果出现缓存穿透，必然会在响应时间上有所体现
  - 业务本身问题
  - 相关指标：总调用数、缓存层命中数、存储层命中数，比如采集每分钟的变化，可以知道有没有这样的问题

- 解决方法

  - 缓存空对象：如果缓存穿透，存储层返回了null，我们仍然将null当作结果，配合请求的id值将其回写到缓存层。

    - 使用：当然请求的数据可能是真的不可用，也可能比如存储层对外提供了接口，该节点暂时不可用，等等原因，那么就可以将null结果的cache设置过期时间，就可以在过期时间内暂缓缓存穿透对存储层带来的影响
    - 两个问题：
      - 需要更多的键：会占用额外的空间，为了不使key越堆越多，设置过期时间就是一个较好的方法
      - 缓存层和存储层数据"短期"不一致。在null缓存过期时间内都是不一致的，可以订阅消息，尽量使存储层恢复后能马上同步回写数据刷新null缓存的这个key，甚至可以使用消息队列单独定位到某个key或者某个业务，来解决不一致问题。当然仍然不是强一致的，永远会存在短期的不一致
    - 总的来说是一个不错的解决方式

    ```java
    //java伪代码。具体的过期时间还有逻辑要根据实际开发需求进行变动
    public String getPassThrough(String key) {
        String cacheValue = cache.get(key);
        if (StringUtils.isBlank(cacheValue)) {
            String storageValue = storage.get(key);
            cache.set(key, storageValue);
            //如果存储数据为空，需要设置一个过期时间(300秒)
            if (StringUtils.isBlank(storageValue)) {
                cache.expire(key, 60 * 5);
            }
            return storageValue;
        } else {
            return cacheValue;
        }
    }
    ```

  - 布隆过滤器拦截：通过很小的内存来实现对数据的过滤。比如巨大的电话本，10E行，判断一个电话是否在这个电话本里。电话本如此巨大不可能放在内存中，bloom filter就是解决类似问题的，可以通过一些算法将电话本这样的大数据放在bloom filter预热一遍，当下次访问需要判断一个电话是否在这个电话本里的时候，可以用很小的内存来解决这个问题。bloom filter后面说明

    - bloom filter在cache前再做一次拦截，如果在bloom filter这里被过滤了，就不认为请求的数据是有效的，没被过滤才能到cache层去。问题就在于bloom filter如何去生成？离线去生成或者如何去做？对于固定的数据很好解决，对于频繁更新的数据要如何解决就有很多问题了。比如一个非实时的推荐服务，根据用户前一天的日志给出第二天的结果，一般就可以在夜间去计算，存储在如hbase上，对于第二天来说这就是一个比较固定的数据，有每个用户的行为，可以把每个用户的key做成布隆过滤器，就会是一个相对可信的布隆过滤器，而如果是现在大多数的实时推荐，需要实时更新布隆过滤器，就相对不可信，可能会不完全符合实时。所以说布隆过滤器是有一定局限性的，需要特殊使用场景，布隆过滤器本身也需要单独维护代码，实现过滤也是需要额外的空间来完成的

  - 

## 缓存击穿

缓存击穿：key对应的数据存在，但在redis中过期，此时若有大量并发请求过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端DB压垮。业界比较常用的做法，是使用mutex。简单地来说，就是在缓存失效的时候（判断拿出来的值为空），不是立即去load db，而是先使用缓存工具的某些带成功操作返回值的操作（比如Redis的SETNX或者Memcache的ADD）去set一个mutex key，当操作返回成功时，再进行load db的操作并回设缓存；否则，就重试整个get缓存的方法

- 解决：通过redis的setnx（不存在才set）设置key做互斥锁，设置成功的才去数据库加载数据



## 缓存雪崩优化

- 缓存雪崩：当缓存服务器重启或者大量缓存集中在某一个时间段失效，这样在失效的时候，也会给后端系统(比如DB)带来很大压力。加锁或者请求队列，还有尽量不要使大量数据的过期时间在同一个时间段
- 缓存失效时的雪崩效应对底层系统的冲击非常可怕！大多数系统设计者考虑用加锁或者队列的方式保证来保证不会有大量的线程对数据库一次性进行读写，从而避免失效时大量的并发请求落到底层存储系统上。还有一个简单方案就是，将缓存失效时间分散开，比如我们可以在原有的失效时间基础上增加一个随机值，比如1-5分钟随机，这样每一个缓存的过期时间的重复率就会降低，就很难引发集体失效的事件

https://zhuanlan.zhihu.com/p/75588064




## 无底洞问题优化

- 问题描述
  - 2010年, Facebook有了3000个Memcache节点。
  - 发现问题："加"机器性能没能提升，反而下降。节点过多了
- 关键点：批量操作的变化，如mget，单机是一个原子操作O(1)，集群则不是，前面提到过了
  - 更多的机器!=更高的性能
  - 批量接口需求(mget,mset等)
  - 数据增长与水平扩展需求：服务端水平扩容无非就是加节点、加机器，而客户端又需要更高的性能
- 优化思路：如redis这种命令本身很快的，就更多考虑优化网络时间，但是如果是mysql这种可能命令会执行极慢的关系型数据库，可能就需要更多考虑优化sql本身。不过一般情况下mysql本来也不需要像redis那么高的性能。要学会根据不同类型数据库调整不同的优化思路
  - 命令本身优化：例如慢查询keys、hgetall bigkey
  - 减少网络通信次数
  - 降低接入成本：例如客户端长连接/连接池、NIO等
- 优化方法：跟前面一样，不再赘述
  - 串行mget
  - 串行IO
  - 并行IO
  - hash_tag



- 



## 热点key重建优化

- 缓存重建：到cache中获取数据，miss则到数据源获取，这个过程就是缓存重建

- 问题：比如一个微博大V在重要的时间节点发布了一个重要消息会落在一个重要的key上，成为一个有极大访问量的热点key，在重建完成之前，有巨大的多线程的访问打过来，那么每个线程都miss，都都要去数据源取，即很多线程都参与重建，重建可能很慢，比如是一个复杂的sql、一个很慢的api，这就产生了问题，每个线程都要执行一遍重建过程，会很浪费时间，会对数据源造成巨大压力，

- 三个目标

  - 减少重缓存的次数
  - 数据尽可能一致
  - 减少潜在危险：比如死锁，线程池被阻塞等

- 解决

  - 互斥锁(mutex key)将重建的过程上互斥锁，其它线程则阻塞等待，重建好之后，其它线程则可以直接获取缓存。个人理解：为了防止大量线程都阻塞等待，可以直接响应一些提示内容

    - 问题：重建过程中需要其它线程都阻塞等待

    ```java
    //java伪代码
    String get(String key){
        String value = redis.get(key);
        if (value == nul) {
            String mutexKey = " mutex:key." + key;
            if (redis.set(mutexKey, "1", "ex 180", "nx")) { //设置一个互斥锁，只有1个线程可以进去执行，为了保证这个锁会被删掉，设置一个过期时间180秒。ex和nx是一个组合命令，set exnx，保证命令的原子性
                value = db.get(key);
                redis.set(key, value);
                redis.delete(mutexKey);
            } else {
                //其他线程休息50毫秒后重试
                Thread.sleep(50);
                get(key);
            }
            return value;
        }
    }
    ```

  - 永远不过期：

    - 缓存层面：不设置过期时间(没有用expire)。
    - 功能层面：为每个value添加逻辑过期时间，但发现超过逻辑过期时间后，会使用单独的线程去构建缓存。
    - 问题：存在数据不一致，虽然数据永远不会过期，任何时刻到来的线程都可以无需等待获取数据，但某一时刻一个线程A发现数据逻辑时间到期，则单独开启一个线程去完成重建并设置新的过期时间，而重建过程中，线程A和之后时间的线程获取到的数据等于是旧的数据

    ```java
    String get(final String key){
        V v = redis.get(key);
        String value = v.getValue();
        long logicTimeout = v.getLogicTimeout();
        if (logicTimeout >= System.currentTimeMillis()) {
            String mutexKey = " mutex:key:" + key;
            if (redis set(mutexKey, "1", "ex 180", "nx")) {
                //异步更新后台异常执行
                threadPool.execute(new Runnable() {
                    public void run() {
                        String dbValue = db.get(key);
                        redis.set(key, (dbValue,newLogicTimeout));
                        redis.delete(keyMutex);
                    }
                });
            }
        }
        return value;
    }
    ```

  | 方案       | 优点                         | 缺点                                                |
  | ---------- | ---------------------------- | --------------------------------------------------- |
  | 互斥锁     | 思路简单<br/>保证一致性      | 代码复杂度增加<br/>存在死锁的风险                   |
  | 永远不过期 | 基本杜绝热点key重建问题<br/> | 不保证一致性<br/>逻辑过期时间增加维护成本和内存成本 |

  

## 总结

- 缓存收益：加速读写、降低后端存储负载。
- 缓存成本：缓存和存储数据不一致性、代码维护成本、运维成本。
- 推荐结合剔除、超时、主动更新三种方案共同完成。
- 穿透问题：使用缓存空对象和布隆过滤器来解决，注意它们各自的使用场景和局限性。
- 无底洞问题:分布式缓存中,有更多的机器不保证有更高的性能。有四种批量操作方式:串行命令、串行IO、并行IO、hash_tag。
- 雪崩问题:缓存层高可用、客户端降级、提前演练是解决雪崩问题的重要方法。
- 热点key问题:互斥锁、“永远不过期”能够在一-定程度 上解决热点key问题，开发人员在使用时要了解它们各自的使用成本。

# RedisBloomFilter

基于Redis的分布式布隆过滤器

- 引出布隆过滤器
- 布隆过滤器原理
- 布隆过滤器误差率
- 本地布隆过滤器
- Redis单机布隆过滤器
- Redis分布式布隆过滤器



## 引出布隆过滤器

- 问题：现有50亿个电话号码，现有10万个电话号码，要 **快速、准确** 判断这些电话号码是否已经存在?
  - 通过数据库查询：实现快速有点难。假设在mysql中，写个in？肯定跑飞了；循环？也肯定快不起来
  - 数据预放在集合中：50亿*8字节(long型)≈40GB，内存浪费或不够。比如java集合？jvm一般也不可能开这么大。hbase？可能会开一些比较大的堆栈，但为了这么一个小功能也太过于浪费
  - hyperloglog：准确有点难。很好，但是违背准确性
- 类似问题很多
  - 垃圾邮件过滤
  - 文字处理软件(例如word )错误单词检测
  - 网络爬虫重复ur|检测
  - Hbase行过滤
- 布隆过滤器：1970年伯顿.布隆提出,用很小的空间,解决上述类似问题
  - 实现原理：一个很长的二进制向量和若干个哈希函数
    - 内容
      - 二进制向量：比如000000000000000000000000000000000000000000000
      - 哈希函数：如F1、F2、...Fn
      - 过滤的内容：如 p=185xxxxxxxx。
    - 构建过程
      - p通过F1进行hash，假设落在第3位，则将第3位变成1，001000000000000000000000000000000000000000000
      - p通过F2进行hash，假设落在第9位，则将第9位变成1，001000001000000000000000000000000000000000000
      - ...一直到Fn全部执行完
  - 



## 布隆过滤器原理

- 构建
  - 参数
    - m个二进制向量：00000...0，m个初始的0
    - k个hash函数：一般8个即可
    - n个预备数据：比如那5E个电话号码
  - 过程
    -  n个预备数据全部走一遍上面过程
       - 如果m值较小，比如只有10个0，而n是50，就算只有少量hash函数，这个二进制向量也可能直接全部变为1了，所以二进制向量的1的密度跟m和n比例是很有关系的
       - 当然和hash函数的个数也有关系，不过hash函数个数越多对判断的提高是有帮助的
- 判断元素存在：让元素走一遍同样的过程：如果得到的位置在二进制向量上都是1，则在表示存在，反之表示不存在



## 布隆过滤器误差率

- 误差率：肯定存在误差，对的肯定是对的，错的可能是对的，错的也可能恰好都命中了

  - 只管因素：m/n的比率，hash函数的个数

  - 实际误差率公式

    - 1个元素，1个hash函数，任意一个比特为1的概率为1/m，依然为0的概率为1 - 1/m

    - k个函数， 任意一个比特依然为0的概率为 
      $$
      (1-1/m)^k
      $$
      n个元素，任意一个比特依然为0的概率为(1-M)nk
      $$
      (1-1/m)^{kn}
      $$

    - 任意一个比特位被设置为1的概率
      $$
      1-(1-1/m)^{kn}
      $$

    - 新元素全中的概率为
      $$
      (1-(1-1/m)^{kn})^k≈(1-e^{-kn/m})k
      $$

  - m/n与误差率成反比，k与误差率成反比

  - 



## 本地布隆过滤器

- 现有库：guava，https://github.com/google/guava
- 本地布隆过滤器的问题
  - 容量受限制：受限于容器，比如jvm，或者tomcat（也是jvm）等web容器。在n的体量较大、或者期望的误差率较低，还是会受限于容量，单机也会成为限制
  - 多个应用存在多个布隆过滤器，构建同步复杂：多个应用都需要在本地构建布隆过滤器，它们之间会产生布隆过滤器同步问题，而前端发来的请求可能通过一些负载均衡是落在不同的应用上的，布隆过滤器的范围类似于session，无法跨应用（container），需要去实现同步，比如session集中存储，就采用一个独立的session，让所有的container去访问它，或者笨一点让该用户的请求一定要打在之前的容器上，但这违背了负载均衡
  - 使用redis来实现布隆过滤器的集中存储



## Redis单机布隆过滤器

- 基于位图：布隆过滤器的二进制向量天然吻合redis位图，redis位图提供了setbit、getbit等功能
- 实现
  - 定义布隆过滤器构造参数：m、n、k、误差概率
  - 定义布隆过滤器操作函数：add和contain
  - 封装Redis位图操作
  - 开发测试样例

###### BloomFilter

```java
package com.yuanya.bloomFilter;

import java.util.List;
import java.util.Map;

/**
 * 布隆过滤器接口
 *
 * @param <T>
 * @author yuanya
 * 2017年12月23日 下午10:03:12
 */
public interface BloomFilter<T> {
    /**
     * 添加
     *
     * @param object
     * @return 是否添加成功
     */
    boolean add(T object);

    /**
     * 批量添加
     *
     * @param objectList
     * @return
     */
    Map<T, Boolean> batchAdd(List<T> objectList);

    /**
     * 是否包含
     *
     * @param object
     */
    boolean contains(T object);

    /**
     * 批量是否包含
     *
     * @param object
     */
    Map<T, Boolean> batchContains(T object);

    /**
     * 预期插入数量
     */
    long getExpectedInsertions();

    /**
     * 预期错误概率
     */
    double getFalseProbability();

    /**
     * 布隆过滤器总长度
     */
    long getSize();

    /**
     * hash函数迭代次数
     */
    int getHashIterations();

    /**
     * 获取子布隆过滤器个数
     */
    int getChildNum();

}
```

###### RedisBloomFilter

```java
/**
 * 布隆过滤器接口
 * 
 * @param <T>
 * @author yuanya
 * 2019年4月15日 下午10:06:54
 */
public class RedisBloomFilter<T> implements BloomFilter<T> {
    private Logger logger = LoggerFactory.getLogger(RedisBloomFilter.class);
    private BloomFilterBuilder config;

    public RedisBloomFilter(BloomFilterBuilder bloomFilterBuilder) {
        this.config = bloomFilterBuilder;
    }

    public boolean add(T object) {
        if (object == null) {
            return false;
        }

        //偏移量列表
        List<Integer> offsetList = this.hash(object);
        if (offsetList == null || offsetList.isEmpty()) {
            return false;
        }

        //设置偏移量到二进制向量位图
        for (Integer offset : offsetList) {
            Jedis jedis = null; 
            try {
                jedis = getJedisPool().getResource();
                jedis.setbit(getName(), offset, true);
            } catch (Exception e) {
                logger.error(e.getMessage(), e);
                return false;
            } finally {
                if (jedis != null) {
                    jedis.close();
                }
            }
        }
        return true;
    }

    public Map<T, Boolean> batchAdd(List<T> objectList) {
        if (objectList == null || objectList.isEmpty()) {
            return Collections.emptyMap();
        }

        Map<T, Boolean> resultMap = new HashMap<T, Boolean>();
        for (T object : objectList) {
            boolean result = this.add(object);
            resultMap.put(object, result);
        }
        return resultMap;
    }

    public boolean contains(T object) {
        if (object == null) {
            return false;
        }
        //偏移量列表
        List<Integer> offsetList = hash(object);
        if (offsetList == null || offsetList.isEmpty()) {
            return false;
        }
        for (int offset : offsetList) {
            Jedis jedis = null;
            try {
                jedis = getJedisPool().getResource();
                boolean result = jedis.getbit(getName(), offset);
                if (!result) { //如果是0，即没有在列表中
                    return false;
                }
            } catch (Exception e) {
                logger.error(e.getMessage(), e);
            } finally {
                if (jedis != null) {
                    jedis.close();
                }
            }
        }
        return true; //所有位都是1则返回true
    }

    public Map<T, Boolean> batchContains(T object) {
        return null;
    }

    public BloomFilterBuilder getConfig() {
        return config;
    }

    public long getExpectedInsertions() {
        return getConfig().getExpectedInsertions();
    }

    public double getFalseProbability() {
        return getConfig().getFalseProbability();
    }

    public long getSize() {
        return getConfig().getTotalSize();
    }

    public int getHashIterations() {
        return getConfig().getHashIterations();
    }

    public int getChildNum() {
        return getConfig().getChildNum();
    }

    public String getName() {
        return getConfig().getName();
    }

    public JedisPool getJedisPool() {
        return getConfig().getJedisPool();

    }

    public HashFunction getHashFunction() {
        return getConfig().getHashFunction();
    }

    public List<Integer> hash(Object object) {
        byte[] bytes = object.toString().getBytes();
        return getHashFunction().hash(bytes, (int) getSize(), getConfig().getHashIterations());
    }
}

```

###### BloomFilterBuilder

```java
/**
 * 布隆过滤器构造器
 *
 * @author yuanya
 * 2019年4月15日 下午10:05:57
 */
public class BloomFilterBuilder {
    private Logger logger = LoggerFactory.getLogger(RedisBloomFilter.class);
    private static final long MAX_SIZE = Integer.MAX_VALUE * 100L;
    
    /**
     * 需要的JedisPool
     */
    private JedisPool jedisPool;

    /**
     * 需要的JedisCluster
     */

    /**
     * 布隆过滤器名(位图的key)
     */
    private String name;

    /**
     * 位图总长度
     */
    private long totalSize;

    /**
     * hash函数循环次数
     */
    private int hashIterations;

    /**
     * 预期插入条数
     */
    private long expectedInsertions;

    /**
     * 预期错误概率
     */
    private double falseProbability;

    /**
     * 子布隆过滤器个数
     */
    private int childNum;
    /**
     * 子布隆过滤器m。根据需求设置即可
     */
    private int childMaxSize = 1000000000;


    /**
     * hash函数:默认murmur3
     * 自己随便找一个hash也都行
     */
    private HashFunction hashFunction = new MurMur3HashFunction();

    /**
     * 是否完成
     */
    private boolean done = false;

    public BloomFilterBuilder(JedisPool jedisPool, String name, long expectedInsertions, double falseProbability) {
        this.jedisPool = jedisPool;
        this.name = name;
        this.expectedInsertions = expectedInsertions;
        this.falseProbability = falseProbability;
    }

    public BloomFilterBuilder setHashFunction(HashFunction hashFunction) {
        this.hashFunction = hashFunction;
        return this;
    }

    public <T> RedisBloomFilter<T> build() {
        checkBloomFilterParam();
        return new RedisBloomFilter<T>(this);
    }

    /**
     * 检查布隆过滤器参数
     */
    private void checkBloomFilterParam() {
        if (done) {
            return;
        }
        if (name == null || "".equals(name.trim())) {
            throw new
                    IllegalArgumentException("Bloom filter name is empty");
        }
        if (expectedInsertions < 0 || expectedInsertions > MAX_SIZE) {
            throw new
                    IllegalArgumentException(
                    "Bloom filter expectedInsertions can't be greater than " + MAX_SIZE + " or smaller than 0");
        }
        if (falseProbability > 1) {
            throw new IllegalArgumentException("Bloom filter false probability can't be greater than 1");
        }
        if (falseProbability < 0) {
            throw new IllegalArgumentException("Bloom filter false probability can't be negative");
        }

        //计算布隆过滤器(位图)长度
        totalSize = optimalNumOfBits();
        logger.info("{} optimalNumOfBits is {}", name, totalSize);
        if (totalSize == 0) {
            throw new IllegalArgumentException(
                    "Bloom filter calculated totalSize is " + totalSize);
        }
        if (totalSize > MAX_SIZE) {
            throw new IllegalArgumentException("Bloom filter totalSize can't be greater than " + MAX_SIZE
                    + "But calculated totalSize is" + totalSize);
        }

        // hash函数迭代次数
        hashIterations = optimalNumOfHashFunctions();
        logger.info("{} hashIterations is {}", name, hashIterations);

        childNum = (int) (totalSize / childMaxSize + 1);


        done = true;
    }

    /**
     * 根据预期插入条数和概率计算布隆过滤器(位图)长度
     */
    private long optimalNumOfBits() {
        if (falseProbability == 0) {
            falseProbability = Double.MIN_VALUE;
        }
        return (long) (-expectedInsertions * Math.log(falseProbability) / (Math.log(2) * Math.log(2)));
    }

    /**
     * 根据布隆过滤器长度与预期插入长度之比，计算hash函数个数
     */
    private int optimalNumOfHashFunctions() {
        return Math.max(1, (int) Math.round((double) totalSize / expectedInsertions * Math.log(2)));
    }

    public Logger getLogger() {
        return logger;
    }

    public static long getMaxSize() {
        return MAX_SIZE;
    }

    public JedisPool getJedisPool() {
        return jedisPool;
    }

    public String getName() {
        return name;
    }

    public long getTotalSize() {
        return totalSize;
    }

    public int getHashIterations() {
        return hashIterations;
    }

    public long getExpectedInsertions() {
        return expectedInsertions;
    }

    public double getFalseProbability() {
        return falseProbability;
    }

    public int getChildNum() {
        return childNum;
    }

    public int getChildMaxSize() {
        return childMaxSize;
    }

    public HashFunction getHashFunction() {
        return hashFunction;
    }

    public boolean isDone() {
        return done;
    }

}
```



### 存在的问题

- 速度慢：比本地慢，输在网络
  - 解决：单独部署，与应用同机房甚至机架部署
- 容量受限：Redis最大字符串为512MB，通过拆分成多个子串解决，但仍然收Redis单机容量限制
  - 解决：基于Redis Cluster实现



## Redis分布式布隆过滤器

- 实现方法：基于单机改改即可
  - 个布隆过滤器：二次路由。输入参数不变，m比如100E，预定好多少个布隆过滤器，比如一个布隆过滤器最大是1E，即需要100份，设计的时候对key做一个二次路由即可
  - 基于pipeline提高效率：不然分布式布隆过滤器性能下降还是会比较厉害

# 主从

- 单机存在的问题：机器故障（分布式问题）、容量瓶颈（分布式问题）、QPS瓶颈（高可用问题）

- 主从复制：一个master可以有多个slave。主复制到从，类似于数据备份的作用，多副本，高可用，读写分离，读分流负载均衡

  - 一个master可以有多个slave
  - 一个slave只能有一-个master
  - 数据流向是单向的, master到slave
  - slave要保证只读：因为主从复制只能从主到从，从如果写入数据，主不会知道。且主节点宕机，由某个从节点补上，要保证数据一致
  - 作用
    - 数据副本；做rdb也可以在从节点做，减轻master负载
    - 扩展读性能

- 配置

  - 命令：无需重启，但不便于管理

    - slaveofs：在从节点执行 slaveof 主节点ip port。是异步的
    - slaveof no one：不成为任何节点的从节点，即辞掉从节点地位，与主节点断开连接。如果再次成为其它节点的slave，主节点会清除掉从节点所有旧的数据

  - 配置：需要重启，但可以统一配置管理

    ```shell
    slaveof ip port #在从节点配置的，指向主节点的ip和port
    slave-read-only yes #从节点只读
    ```

- 操作

  - 查看节点状态，在成为从节点之前每个节点都是master

    ```shell
    sys> redis-cli info #查看所有信息
    sys> redis-cli -p 6380 info replication #在客户端外查看
    127.0.0.1:6379> info replication #客户端内查看
    ```


- 全量复制和部分复制

  - run_id：每个redis启动都会有一个run_id作为标识。例子，如果从节点发现主节点run_id发生了变化（如主节点重启，或者从节点第一次连接主节点，即第一次获得主节点的run_id标识），这种变化可以引起从节点全量复制主节点数据

    ```shell
    redis-cli -p 6379 info server | grep run #查看run_id
    ```

  - 偏移量：是主从之间实时同步的一个指标，如果主从之间偏移量差值过大，可能是主从同步复制发生了问题

    ```shell
    redis-cli -p 6379 info replication #其中master_repl_offset:1865
    ```

  - 全量复制：master将当前状态RDB文件同步给slave，在同步期间产生的新数据也将被记录，通过对比偏移量，再同步给slave

  - 流程

    1. 从节点发送命令 psync ? 1 给主节点：psync（2.8之前是sync）可以完成全量复制和部分复制的功能。参数1是run_id，参数2是偏移量，首次复制还不知道主节点的run_id和自己的偏移量是多少，run_id以?占位，偏移量以-1
    2. master收到命令，从参数可见从节点都不知道run_id和偏移量，肯定就全量复制了，并会返回run_id和offset
    3. slave保存mster的信息 run_id、offset
    4. mster执行bgsave，主从复制期间，master中新增的命令被保存在repl_back_buffer（用于记录最新写入的命令）中
    5. master向slave send RDB
    6. master向slave send buffer
    7. 从节点flush old data（删除原来的数据）
    8. 从节点加载RDB和buffer

  - 开销

    - bgsave时间
    - RDB文件网络传输时间
    - 从节点清空数据时间
    - 从节点加载RDB的时间
    - 可能的AOF重写时间：如果开启了AOF将进行一次AOF重写，保证AOF是最新的状态

  - 问题：如果网络主从之间网络抖动，从节点无法及时同步数据：2.8以前，会直接再进行一次全量复制，会消耗较大资源；2.8之后

    1. master在会写命令到复制缓冲区repl_back_buffer（默认1mb）
    2. 当slave再次连上master时，slave向mster发送pysnc {offset} {runId}
    3. mster对比偏移量，如果发现slave错过的数据还在缓冲区记录范围内
    4. master发送continue表示继续部分复制
    5. 然后send partial data

- 故障处理

  - 自动故障转移：当一个机器挂掉了，另一台机器自动顶上，之后再对故障机器或服务进行处理，保证高可用
  - 主从结构故障转移：假设一主（读写）二从（只读）
    - slave宕掉：宕一个，如果另一个从节点有足够性能冗余，将宕掉从节点的客户端改到另一个上即可
    - master宕掉：从节点仍可以正常服务，然后发送命令slaveof no one让其中一个成为master，让另一个成为新mster的从节点slaveof new master
    - 但是都没有实现自动故障转移，等待redis-sentinel出场吧

- 常见问题

  - 读写分离：读流量分摊到从节点
    - 可能遇到问题：
      - 复制数据延迟
      - 读到过期数据：删除过期数据有两种策略
        - lazy：去操作key的时候才判断该key是否过期，过期则返回空
        - 定时任务：每次去采样一些key，看它是否过期，但是如果采样速率低于数据产生速率，可能会发生很多过期数据没有删除，然后在主从复制中，slave没有删除数据的资格，如果mster没有及时将删除命令同步给slave时，slave就可能读到脏（过期）数据，但是redis3.2已经解决了这个问题
      - 从节点故障，需要迁移其客户端到其它从节点进行数据读取，如果很多应用使用这个节点，迁移成本就很高了
  - 主从配置不一致
    - 例如maxmemory不一致：丢失数据。比如mster配置4g、slave配置2g，主从复制可以正常进行，mster传输RDB，slave加载RDB，但这就可能RDB过大，触发从节点maxmemory的触发策略，将数据进行淘汰（这个过程有可能产生OOM），如果触发策略是以剔除过期数据优先的策略，就会剔除过期数据，对于数据这就已经没有真正实现副本的功能了，也不会报任何错误，如果主节点再宕机，从节点顶替为主节点，从节点无法挽回那些剔除的数据，数据丢失。比较好的办法是使用一些标准工具安装上对其做监控
    - 例如数据结构优化参数：内存不一致。例如主节点设置hash-max-ziplist-entries优化值，从节点没优化，就会造成主从节点内存不一致
  - 规避全量复制
    - 第一次全量复制：第一次不可避免
      - 小主节点：数据分片，maxmemory不要设置太大，这样bgsave、传输、加载、fork等速度都会更快，开销相对更小
      - 低峰：在访问量较低的时候（如夜间）进行
    - 节点运行ID不匹配：主节点重启run_id改变，主从节点记录的master run_id不一致，会认为数据不安全，将全量复制
      - redis4.0中提供了psyncto的规则可以有效解决这类问题
      - 故障转移，例如哨兵或集群，让从节点顶替成为新的master
    - 复制积压缓冲区（repl_back_buffer，实际上是一个队列，默认大小1mb）不足：网络中断，部分复制可能无法满足
      - 增大复制缓冲区配置rel_ backlog_ size，根据实际情况设置，假如网络故障一般是几分钟，就根据统计每分钟传输字节数*分钟数来计算大小
      - 网络"增强"
  - 规避复制风暴：感觉最好的办法还是直接让slave顶替为master这种高可用的方式，后面讲，这里讲重启方式
    - 单主节点复制风暴，如果一个mster上挂载了很多slave，如果master宕掉重启，所有slave都要做主从复制，虽然redis有优化，master只生成一次rdb，但要做多次传输，对master开销极大
      - 更换复制拓扑：树形。不是所有slave都挂在master上，可以master上挂一个slave1，这个slave上再挂载其它几个salve，这样就将传输压力分担到了一个slave上。但是这会产生新的问题，如果slave1故障了...如何处理故障或者如何故障转移
    - 单机器（机器上很多mster）复制风暴：机器宕机后，大量全量复制
      - 主节点分散多机器



# RedisSentinel

- 主从复制高可用？：主从复制存在问题

  - 手动故障转移：虽然也可以用脚本监控master，出现问题就让slave顶替上来，但这就很复杂，比如怎么判定master是有问题的，怎么通知客户端更改为服务端为新的master，还有保证事务，所以有了Redis Sentinel这样高可用的实现服务 来完成这些事情
  - 写能力和存储能力受限

- 架构说明

  - client从sentinel获取redis信息，即
  - sentinel挡在redis集群前面，监控其后每一个mster和slave。sentinel也是多个的，且一套sentinel可以同时监控多套mster及其slave，每套mster及其slave将以master-name这个配置作为标识

- 故障自动转移：即sentinel将作为redis客户端来自动进行操作

  - 多个sentinel发现并确认master有问题
  - 选举出一个sentinel作为领导
  - 选出一个slave作为master
  - 通知其余slave成为新的master的slave
  - 通知客户端主从变化
  - 等待老的master复活成为新master的slave：依然会对老的mster进行监控（死的时候就已经配置为slave了，复活直接复制master的数据）

- 安装配置

  - 配置开启主从节点

    ```conf
    port 7002
    daemonize yes
    pidfile /var/run/redis-7002.pid
    logfile "7002. log"
    dir "/opt/soft/redis/redis/data/"
    slaveof 127 .0.0.1 7000 #主节点不用配置
    ```

  - 配置开启sentinel监控主节点。(redis-sentinel是特殊的redis，不存储数据，支持的redis命令非常有限，主要功能就完成监控、故障转移、通知)

    ```shell
    daemonize yes #以守护进程方式启动
    port ${port} #默认26379
    dir "/opt/soft/redis/data/"
    logfile "${port}.log'
    sentinel monitor mymaster 127.0.0.1 7000 2 #监控的 主节点名字、ip、端口，2表示至少几个sentinel发现该出master出现问题才会进行故障转移 
    sentinel down-after-milliseconds mymaster 30000 #故障判定时间，30000毫秒无法连通则判定master为故障
    sentinel parallel-syncs mymaster 1 #故障转移后其它slave会对新master进行复制，这个配置指定同时复制的个数，1表示每次只复制一个，减轻master压力
    sentinel failover-timeout mymaster 180000 #故障转移超时时间
    #可以发现只有master的配置，因为sentinel通过对master执行info并解析就可以获取对应从节点信息并自动添加配置，以及会去掉一些与默认配置相同的配置以及添加一些配置
    ```

  - 多个sentinel实际应该多机器，演示仅一台机器

    ```shell
    redis-sentinel redis-sentinel-26379.conf #启动
    redis-cli -p 26379 #连接
    ping
    info #命令和redis一样，只是支持的命令更少
    
    sed "s/26379/ 26380/g" redis-sentinel-26379.conf > redis-sentinel-26380.conf #sed命令偷懒copy出26380和26381即可
    ```

- 客户端连接：现在要从直连redis换为连接sentinel，这样才能达到客户端也高可用 

  - 过程：client会获取到Sentinel节点集合 + masterName，遍历Sentinel节点集合，获取一个可用的Sentinel节点，将发送命令sentinel get-master-addr-by-name masterName，然后sentinel返回mster节点信息，获取到mster执行role 或者role replication以进行验证，然后master会返回节点角色信息

  - 如何通知：类似于发布订阅模式，客户端订阅某一个频道，假如该频道master变化，客户端将收到通知，与新的master进行连接

  - 接入流程：Sentinel地址集合，masterName，不是代理模式而是发布订阅模式

    ```java
    //不管jedis还是其它实现模式都是一样的，只是封装程度不同
    String masterName =" mymaster" ;
    Set<String> sentinels = new HashSet <String>();
    sentinels. add("127.0.0.1:26379");
    sentinels. add("127.0.0.1:26380");
    sentinels. add("127.0.0.1:26381");
    JedisSentinelPool sentinelPool = new JedisSentinelPool(masterName, SentinelSet, poolConfig, timeout); //本质还是连接master的
    Jedis jedis = null;
    try{
        jedis = redisSentinelPool.getResource();
        //jejedis command
    } catch (Exception e) {
        logger.error(e.getMessage(, e);
    } finally {
        if (jedis != null){
            jedis.close();
        }
    }4
    ```

- 实现原理：

  - 故障转移演练：在java程序中循环执行jedis命令，然后将master shutdown
    - 客户端高可用观察：mster宕掉以后，程序开始报错，但是一会儿之后，恢复了正常
    - 服务端日志分析：数据节点和sentinel节点
      - 主节点7000：因为直接kill掉了，日志停止在kill掉的瞬间
      - 从节点7001日志：首先与master失联（不断尝试连接master并失败），收到user request（可以肯定是来自sentinel）希望让它自己成为master，并进行了配置重启，然后发现7002要复制它的数据（说明已经被选为master了），然后开始bgsave，然后去完成复制的过程
      - 从节点7002日志首：首先和7001一样与master失联，然后收到user request希望它slave of 7001，然后复制7001数据的过程
      - sentinel-26379：（日志量很大，只是大概过程）先有sdown master mymaster 127.0.0.1 7000表示该master下线了（一个人的意见），后有odown...表示sentinel达成条件（多个人的意见，具体是多少，根据前面的配置）认为它该做下线了。然后是sentinel的选举 +vote-for-leader 尝试去做领导者，然后记录了26380、26381都投票给它，然后选择7001成为master，选择7001成为slave，然后还是用odown...对7000做下线标识，然后正式标识新的master和slave（应该是会自动更变配置内容）

- 三个定时任务：为了对redis做失败判定、故障转移，redis sentinel有3个定时任务来作为实现这些过程的基础

  - 每10秒每个sentinel对master和slave执行info
    - 发现slave节点
    - 确认主从关系
  - 每2秒每个sentinel通过master节点的channel交换信息(pub/sub)
    - 通过sentinel_ :hello频道交互，类似于发布订阅，准确一点大概是一个广播网络
    - 交互对节点的"看法”和自身信息
  - 每1秒每个sentine|对其他sentinel和redis执行ping
  
- 主观下线和客观下线

  ```shell
  #通过命令配置，前面配置文件已经见过了
  sentinel monitor <masterName> <ip> <port> <quorum>
  sentinel monitor myMaster 127.0.0.1 6379 2 #一般配置ceil(n/2)，n是sentinel节点总数（集群一般是单数）
  sentinel down-after-milliseconds < masterName> < timeout>
  sentinel down-after-milliseconds mymaster 30000
  ```

  - 主观下线：每个sentinel节点对Redis节点失败的"偏见"。slave被主观下线可以直接让其下线，如果是对master主观下线，则对其它所有节点发出命令sentinel is-master-down-by-addr
  - 客观下线：所有sentinel节点对Redis节点失败"达成共识”( 超过quorum个统一)，达成共识的交互手段就是通过命令sentinel is-master-down-by-addr，然后选举出leader控制mster进行客观下线

- leader选举

  - 原因：只一个sentinel节点来完成故障转移
  - 选举：通过sentinel is-master-down-by-addr命令，是的，这个命令也具有自荐为leader的作用
    - 每个做主观下线的Sentinel节点向其他Sentinel节点发送命令，要求将它设置为领导者
    - 收到命令的Sentinel节点如果没有同意通过其他Sentinel节点发送的命令，那么将同意该请求，否则拒绝
    - 如果该Sentinel节点发现自己的票数已经超过Sentinel集合半数且超过quorum，那么它将成为领导者。
    - 如果此过程有多个Sentinel节点成为了领导者，那么将等待一段时间重 新进行选举
    - 虽然可能某个短时间内几乎同时有多个节点发出命令，但是同意总有先后

- 故障转移：前提是sentinel leader节点选举完成

  1. 从slave节点中选出一一个“合适的”节点作为新的master节点
     1. 选择slave-priority(slave节点优先级，一般没配置)最高的slave节点，如果存在则返回,不存在则继续
     2. 选择复制偏移量最大的slave节点(复制的最完整) ,如果存在则返回，不存在则继续
     3. 选择runId最小的slave节点
  2. 对上面的slave节点执行slaveof no one命令让其成为master节点。
  3. 向剩余的slave节点发送命令, 让它们成为新master节点的slave节点,复制规则和parallel-syncs参数有关。
  4. 更新对原来master节点配置为slave ,并保持着对其"关注"，当其恢复后命令它去复制新的master节点。

常见开发运维问题

- 节点运维
  - 节点下线
    - 原因
      - 机器下线：例如过保等情况
      - 机器性能不足：例如CPU、内存、硬盘、网络等
      - 节点自身故障：例如服务不稳定等
    - 主节点：sentinel failover <masterName>，手动在某个sentinel上执行命令让该sentinel执行指定master的故障转移
    - 从节点：考虑临时下线还是永久下线，例如是否做一些清理工作（配置、日志、数据、RDB文件等）。还要考虑读写分离的情况，如转移连接该节点的客户端到其它节点等。
  - 节点上线
    - 主节点：sentinel failover <masterName> 进行替换。
    - 从节点：slaveof即可, sentinel节点可以感知。
    - sentinel节点：参考其他sentinel节点启动即可
- 高可用读写分离
  - JedisSentinelPool的实现：客户端高可用
    - 从构造方法进入，先初始化sentinels，初始化和前面讲过的客户端原理一致，循环sentinel集合，对sentinel发出命令让其根据mastername获取master地址来检验连接，直到找到一个可用的sentinel则跳出循环。
    - 然后去订阅频道，MasterListener（继承了Thread）来监听，连接每一个sentinel，订阅master更变的通道，如果mster更变将重新初始化连接池，完成客户端高可用
  - 从节点的作用
    - 副本:高可用的基础
    - 扩展:读能力
  - 三个"消息"：可以把所有slave看作一个池子，客户端通过监听三个"消息（通道）"，监听到变化则重新初始化redis连接池，完成高可用读写分离
    + switch-master：切换主节点(从节点晋升主节点)
    + convert-to-slave：切换从节点(原主节点降为从节点)
    + sdown：主观下线
  - 高可用读写分离相对复杂，一般来说真正需要高扩展性的高可用redis集群，都通过redis-cluster（redis集群版本）来完成

##### 总结

Redis Sentinel是Redis的高可用实现方案：故障发现、故障自动转移、配置中心、客户端通知

Redis Sentinel从Redis2.8版本开始才正式生产可用，之前版本生产不可用

尽可能在不同物理机上部署Redis Sentinel所有节点，建议放在一个网络中，减少误差

Redis Sentinel中的Sentinel节点个数应该为大于等于3，且最好为奇数。

Redis Sentinel中的数据节点与普通数据节点没有区别

客户端初始化时连接的是Sentinel节点集合，不再是具体的Redis节点，但Sentinel只是配置中心不是代理。

Redis Sentinel通过三个定时任务实现了Sentinel节点对于主节点、从节点、%其余Sentinel节点的监控。

Redis Sentinel在对节点做失败判定时分为主观下线和客观下线

看懂Redis Sentinel故障转移日志对于Redis Sentinel以及问题排查非常有帮助

Redis Sentinel实现读写分离高可用可以依赖Sentinel节点的消息通知，获取Redis数据节点的状态变化

# RedisCluster

- 呼唤集群
- 数据分布
- 搭建集群
- 集群伸缩
- 客户端路由
- 集群原理
- 开发运维常见问题



## 呼唤集群

- 为什么呼唤集群
  - 1.并发量。redis宣称：10万/每秒。业务需要100万/每秒呢?
  - 2.数据量。机器内存: 16~256G。业务需要500G呢?
  - 3.网络流量。网卡：千兆网卡。业务需要万兆呢
- 解决方法：
  - 配置"强悍”的机器:超大内存、牛x CPU等，但单机上限远远比不上集群
  - 分布式集群，才是正确的解决方法
- 集群：规模化需求
  - 并发量: OPS
  - 数据量: "大数据
  - 网络流量

## 数据分布

- 分布式数据库-数据分区：全量数据→分区规则→子集-1、子集-2...子集-n

  | 分布方式 | 特点                                                         | 典型产品                                              |
  | -------- | ------------------------------------------------------------ | ----------------------------------------------------- |
  | 哈希分布 | 数据分散度高<br/>键值分布业务无关<br/>无法顺序访问<br/>支持批量操作 | 一致性哈希Memcache<br/>Redis Cluster<br/>其他缓存产品 |
  | 顺序分布 | 数据分散度易倾斜<br/>键值业务相关<br/>可顺序访问<br/>支持批量操作 | BigTable<br/>HBase                                    |

  - 分序分布（区）：1~100→1~33、34~ 66、67~ 100
  - 哈希分布（例如节点取模）：1~100→hash(key)%3→(3,6...99 )、(1,4...100)、( 2,5.. .98)
    - 节点取余：hash(key)%nodes。nodes只节点数。
      - 扩容：扩展节点时对所有数据重新计算，并根据结果将其迁移到应该去的机器（如果结果不属于原来的机器的话）。好像是。。。。。：数据迁移并不是从一个节点直接到另一个节点，而是在计算以后如果应该到新的节点上取，发现没有数据，则会到数据源取，然后再放入新节点，这样就完成了"迁移"，其实是一个访问数据源回写的过程。
        - 比如从3个节点到4个节点，会发现数据的迁移率（从原来的节点迁移的另一个节点的数据的比例）较高（计算为80%）。
        - 多倍扩容：3个节点变为6个节点，则只有50%迁移率
      - 建议
        - 客户端分片：哈希+取余，简单、原始，迁移率较大会产生庞大的数据源访问和数据回写到内存，不建议使用
        - 节点伸缩：数据节点关系变化,导致数据迁移
        - 迁移数量和添加节点数量有关：建议翻倍扩容
    - 一致性哈希：见图
      - 客户端分片：哈希+顺时针(优化取余)
      - 节点伸缩：只影响邻近节点，但是还是有数据"迁移"
      - 翻倍伸缩：保证最小迁移数据和负载均衡（因为访问量大的可能一直只是某些段，比如老用户访问少且id较小，新用户访问多而id都再较大数字的段）
    - 虚拟槽：Redis Cluster就是这种分区方式。不同的操由不同的节点管理，计算key得到value将知道属于哪个槽，然后任意分配给集群的一个节点，如果不属于该节点负责的槽，则由该节点返回目标节点给客户端（因为edis cluster中节点之间都有槽信息共享），让客户端去到正确的节点，但这种方式性能不高，节点较多时利用率较低，所以需要让客户端知道所有槽-节点。每个节点负责的槽是固定的，新节点将负责新的槽，不会有数据迁移
      - 预设虚拟槽:每个槽映射一个数据子集, 一般比节点数大
      - 良好的哈希函数:例如CRC16
      - 服务端管理节点、槽、数据:例如Redis Cluster

## 搭建集群

- 基本架构

  - 单机架构：主从复制，只有主可以写
  - 分布式架构：都可以读写

- Redis Cluster架构

  - 节点：配置cluster-enabled:yes，是否是集群模式启动
  - meet：一个命令。节点之间完成通信的基础。A meet B、A meet C，就可以使B和C找到对方，这样就能完成所有节点的相互通信
  - 指派槽：redis cluster指定有16384个slot，
  - 复制：也有主从复制，每个主节点都有从节点，集群中有很多主节点。且无需redis sentinel
  - 高可用
  - 分片

- 安装

  - 原生命令安装：就是按上面架构流程。无需记住，仅用来理解架构，实际生产基本还是使用安装工具安装

    - 配置开启节点

      ```conf
      port ${port}
      daemonize yes
      dir "/opt/redis/redis/data/"
      dbfilename "dump-${port}.rdb"
      logfile "${port}.log"
      cluster-enabled yes #集群模式启动
      #下面是集群主要配置
      cluster-enabled yes
      cluster-node-timeout 1 5000
      cluster-config-file "nodes.conf"
      cluster-require-full-coverage yes #是否需要集群内所有节点都能提供服务才能认为该集群是正确的，即有一个节点出现问题集群就不可用，默认yes，肯定要改为no
      ```

      ```shell
      redis-server redis-7000.conf
      redis-server redis-7001.conf
      redis-server redis-7002.conf
      redis-server redis-7003.conf
      redis-server redis-7004.conf
      redis-server redis-7005.conf
      ```

    - meet：cluster meet ip port

      ```shell
      redis-cli -h 127.0.0.1 -p 7000 cluster meet 127.0.0.1 7001
      redis-cli -h 127.0.0.1 -p 7000 cluster meet 127.0.0.1 7002
      redis-cli -h 127.0.0.1 -p 7000 cluster meet 127.0.0.1 7003
      redis-cli -h 127.0.0.1 -p 7000 cluster meet 127.0.0.1 7004
      redis-cli -h 127.0.0.1 -p 7000 cluster meet 127.0.0.1 7005
      ```

    - 指派槽：cluster addslots slot [slot ..]

      ```shell
      redis-cli -h 127.0.0.1 -p 7000 cluster addslots {0...5461}
      redis-cli -h 127.0.0.1 -p 7001 cluster addslots {5462...10922}
      redis-cli -h 127.0.0.1 -p 7002 cluster addslots {10923...16383}
      ```

    - 主从：cluster replicate node-id

      ```shell
      redis-cli -h 127.0.0.1 -p 7003 cluster replicate ${node-id-7000}
      redis-cli -h 127.0.0.1 -p 7004 cluster replicate ${node-id-7001}
      redis-cli -h 127.0.0.1 -p 7005 cluster replicate ${node-id-7002}
      ```

    - 操作

      配置文件

      ```conf
      port 7000
      daemonize yes
      dir "/opt/soft/redis/data"
      logfile "7000. log"
      dbfilename "dump-7000.rdb"
      cluster-enabled yes
      cluster-config-file nodes-7000.conf
      cluster-require-full-coverage no
      ```

      快速sed配置文件

      ```shell
      sed 's/7000/7001/g' redis-7000.conf > redis-7001.conf
      sed 's/7000/7002/g' redis-7000.conf > redis-7002.conf
      sed 's/7000/7003/g' redis-7000.conf > redis-7003.conf
      sed 's/7000/7004/g' redis-7000.conf > redis-7004.conf
      sed 's/7000/7005/g' redis-7000.conf > redis-7005.conf
      ```

      启动和查看

      ```shell
      redis-server redis-7000.conf
      redis-server redis-7001.conf
      redis-server redis-7002.conf
      redis-server redis-7003.conf
      redis-server redis-7004.conf
      redis-server redis-7005.conf
      ps -ef | grep redis #查看服务
      redis-cli -p 7000 cluster nodes #查看节点信息，这种查看和redis-cli -p 7000客户端先连接，后再执行cluster nodes是一个效果
      redis-cli -p 7000 cluster info #查看集群信息
      ```

      meet

      ```shell
      redis-cli -p 7000 cluster meet 127.0.0.1:7001
      redis-cli -p 7000 cluster meet 127.0.0.1:7002
      redis-cli -p 7000 cluster meet 127.0.0.1:7003
      redis-cli -p 7000 cluster meet 127.0.0.1:7004
      redis-cli -p 7000 cluster meet 127.0.0.1:7005
      redis-cli -p 7005 cluster info
      ```

      分配槽：脚本

      ```shell
      vim addslots.sh #vim可以自动创建
      
      #内容
      start=$1
      end=$2
      port=$3
      for slot in `seq ${start} ${end}`
      do
          echo "slot:${slot}" 
      	redis-cli -p ${port} cluster addslots ${slot}
      done
      
      #执行，3个主节点（现在还不是）
      sh addslots.sh 0 5461 7000
      sh addslots.sh 5461 10922 7001
      sh addslots.sh 10922 16383 7002
      ```

      主从

      ```shell
      redis-cli -p 7000 cluster nodes #查看
      redis-cli -p 7003 cluster replicate 560598cc5f6b13663ee7aaa9ff403e799e7d4918 #主节点id
      redis-cli -p 7004 cluster replicate xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
      redis-cli -p 7005 cluster replicate xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
      redis-cli -p 7000 cluster nodes #查看
      redis-cli -p 7000 cluster slots #查看槽分配信息
      redis-cli -c -p 7000 #客户端连接redis cluster，可以api操作了！
      ```

      主从可以这样设计：左主右从，错开设计，减少机器

      10.0.0.1:7000 10.0.0.2:7003
      10.0.0.2:7001 10.0.0.3:7004
      10.0.0.3:7002 10.0.0.1:7005

      

  - 官方工具安装：安装工具简单、高效、准确，但是如果真的是几百上千的超大集群，最好还是需要专门的平台来管理

    - 官方工具安装提供了Ruby安装脚本，需要准备Ruby环境

      - 下载、编译、安装Ruby

        ```shell
        wget https://cache.ruby-lang.org/pub/ruby/2.3/ruby-2.3.1.tar.gz
        tar -xvf ruby-2.3.1.tar.gz
        ./configure -prefix=/usr/ocal/ruby
        make
        make install 
        cd /usr/local/ruby
        cp bin/ruby/usr/local/bin
        ```

      - 安装rubygem redis：ruby的一个客户端

        ```shell
        wget http://rubygems.org/ downloads/redis-3.3.0.gem
        gem install -l redis-3.3.0.gem
        gem list --check redis gem
        ```

    - 安装redis-trib.rb：官方工具安装

      ```shell
      #配置开启Redis
      redis-server redis-8000.conf
      redis-server redis-8001.conf
      redis-server redis-8002.conf
      redis-server redis-8003.conf
      redis-server redis-8004.conf
      redis-server redis-8005.conf
      
      #一键开启，前3个是主节点，后3个是按序对应的从节点，如果每个主节点需要2个从节点，写6个即可，不符合标准的话会给你返回错误的
      ./redis-trib.rb create --replicas 1 127.0.0.1:8000 127.0.0.1:8001 \127.0.0.1:8002127.0.0.1:8003 127.0.0.1:8004 127.0.0.1:8005
      
      
      cp ${REDIS_ HOME}/src/redis-trib.rb /usr/local/bin
      ```

  - 可视化部署：开发专门的管理平台来进行



## 集群伸缩

- 伸缩原理：集群伸缩=槽和数据在节点之间的移动

- 扩容集群：主要作用是：为它迁移槽和数据实现扩容，作为从节点负责故障转移

  - 手动操作

    - 准备新节点

      - 集群模式

      - 配置和其它节点统一

      - 启动后是孤儿节点

        ```shell
        redis-server conf/redis-6385.conf
        redis-server conf/redis-6386.conf
        ```

    - 加入集群：让集群中的节点去meet这些孤立节点

      ```shell
      127.0.0.1:6379> cluster meet 127.0.0.1 6385
      127.0.0.1:6379> cluster meet 127.0.0.1 6386
      ```

    - 迁移槽和数据

      - 槽迁移计划：一般使平分16383个槽即可

      - 迁移数据：非常复杂

        - 对目标节点发送：cluster setslot {slot} importing {sourceNodeId}命令，让目标节点准备导入槽的数据。目标节点准备导入槽{slot}
        - 对源节点发送：cluster setslot {slot} migrating {targetNodeId}命令，让源节点准备迁出槽的数据。通知{slot}被目标节点负责
        - 源节点循环执行cluster getkeysinslot {slot} {count}命令，每次获取count个属于槽的健。批量迁移相关键的数据
        - 在源节点上执行migrate {targetIp} {targetPort} key 0 {timeout}命令把指定key迁移。0是对应的数据库，不过redis cluster中只有db 0 没有其它db。获取slot下{count}个健
        - 重复执行步骤3~4直到槽下所有的键数据迁移到目标节点。5:循环迁移键
        - 向集群内所有主节点发送cluster setslot {slot} node {targetNodeId}命令，通知槽分配给目标节点。原节点准备导出槽{slot}

        ```python
        #python伪代码
        def move_slot(source , target, slot):
            #目标节 点准备导入槽slot
            target.cluster("setslot",slot,"importing",source.nodeID) ;
            #目标节点准备全出槽slot
            source. cluster(" setslot",slot,"migrating",target.nodeId);
            while true :
                #批量从源节点获取键
                keys = source. cluster("getkeysinslot",slot,pipeline_size);
                if keys .length == 0:
                    #键列表为空时，退出循环
                    break;
            #批量迁移键到目标节点
            source.call( "migrate",target.host,target.port,"",0,timeout,"keys",keys]);
            #向集群所有主节点通知槽slot被分配给目标节点
            for node in nodes:
                if node.flag == "slave":
                    continue;
                node. cluster("setslot",slot,"node",target.nodeId);
        ```

      - 添加从节点

  - 官方工具redis-trib.rb操作。官方工具操作时会检测你要加入的节点是否是新节点，能够避免新节点已经加入了其他集群，造成故障。所以一般用官方工具比较好，因为如果将两个集群混合在一起了，可能造成严重后果

    - redis-trib.rb add-node new_host.new_ port existing_ host:existing. port --slave --master-id <arg>

    - redis-trib.rb add-node 127.0.0.1:6385 127.0.0.1:6379

    - 迁移数据

      ```shell
      redis-trib.rb reshard 127.0.0.1: 7000 #迁移数据，7000是要找个主节点，执行过程中会提示要求更多参数，按照提示填即可
      #假如数据源选择了all，则将从7000、7001、7002中各区同等数量的一部分，最终效果则是4个主节点基本平分16383个槽
      ```

- 缩容集群：与扩容类似

  - 下线迁移槽：迁移槽操作与扩容一致

    ```shell
    redis-trib.rb reshard --from 172d689c3ed7b3721afa71a1ca20450ad0147ebb --to 97ea959862c796988810c2a4f13ee245d318942b --slots 1366 127.0.0.1: 7006 #from 7006的id to 7000的id，迁移1366个槽
    redis-trib.rb reshard --from 172d689c3ed7b3721afa71a1ca20450ad0147ebb --to xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx --slots 1365 127.0.0.1: 7006 #from 7006的id to 7001的id，迁移1366个槽
    redis-trib.rb reshard --from 172d689c3ed7b3721afa71a1ca20450ad0147ebb --to xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx --slots 1365 127.0.0.1: 7006 #from 7006的id to 7002的id，迁移1365个槽
    ```

  - 忘记节点：让其它节点忘记它。redis-cli> cluster forget {downNodeId}，60s有效，也就是要在60内让所有节点都忘记它，否则有一个节点还保留有它的信息，会让所有节点都记起它

  - 关闭节点

    ```shell
    #用工具忘记和关闭一起。先下从节点，再下主节点，7007是从，7006是主
    redis-trib.rb del-node 127.0.0.1:7000 c94d20dedf3d5c365286b733ef5d6bd23c3c33eb #7007的id
    redis-trib.rb del-node 127.0.0.1:7000 xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx #7006的id
    ```




## 客户端路由

- moved重定向：moved表示已经确定不在本节点了，完成式，已经迁移完毕了

  - 客户端发送键命令（如set php best）给任意节点
  - 节点根据key计算槽和对应节点（cluster keyslot php可以计算该hash值），如果指向自身就执行命令，否则回复moved异常给客户端
  - 客户端重定向发送命令给目标节点（客户端不会自己发送，需要自己去写这个逻辑）

  ```shell
  redis-cli -c -p 7000 #-c集群模式
  127.0.0.1:7000> cluster keyslot hello
  (integer) 866
  127.0.0.1:7000> set hello world
  OK
  127.0.0.1:7000> cluster keyslot php
  (integer) 9244
  127.0.0.1:7000> set php best
  -> Redirected to slot [9244] located at 127.0.0.1:7001 #集群模式下会帮我们自动重定向
  OK
  127.0.0.1:7001> get php
  "best"
  redis-cli -p 7000 #非集群模式连接
  127. 0.0.1:7000> cluster keyslot php
  (integer) 9244
  127.0.0.1:7000> set php best
  (error) MOVED 9244 127.0.0.1:7001 #非集群模式下仅返回moved异常
  ```

- ask重定向：redis正在从源节点到目标节点迁移slot的情况，因为槽迁移是遍历槽中的key逐步执行migrate的过程，这很慢。ask则表示在槽迁移过程中

  - 源节点正在与目标节点执行槽迁移
  - 客户端已经从其它节点得到槽属于当前这个源节点，然后发送命令
  - 源节点发现该槽其实已经迁移给目标节点了，然后回复ask转向异常给客户端，意思是这个槽确实本属于源节点，但是已经迁移到目标节点去了
  - 客户端执行asking命令，然后发送键命令给目标节点，也是一个在客户端重定向的过程
  - 目标节点返回

- smart客户端

  - smart客户端原理：目标是追求性能。所以尽量不使用代理的模式（在集群前加一层代理，由代理梳理节点、槽、键的关系，客户端连接代理来找到目标节点，每次都要moved或者akd后进行重定向，非常损耗性能。而直连的性能就要高的多，所以需要实现客户端直连目标槽、节点，当然万一碰到moved和ask异常也还是得兼容

    - 从集群中选一个可运行节点,使用cluster slots初始化槽和节点映射
    - 将cluster slots的结果映射到本地（比如一个map），为每个节点创建JedisPool.
    - 准备执行命令。key-> slot-> node得关系是知道得，所以可以JedisCluster直接计算然后连接目标节点
      - 如果出现连接出错，就随机发送命令给一个活跃节点，然后回归到moved或者ask那种代理的过程
      - 返回moved或ask（或者直接命中也有可能，但集群中这种可能较小）后获得响应，然后要记得重新初始化slot→node的缓存（更新map）
      - 如果连接出错的过程反复超过了5次，会有Too many cluster redirection!的错误

  - JedisCluster源码：

    - 比如随便选一个set方法：返回了一个new JedisClusterCommand{}.run(key)
    - run方法：判断key是否为null，是则返回一个异常，否则执行runWithRetries方法
    - runWithRetries方法：参数attempts是一个初始化尝试的次数，默认为5次，attempts小于0就会异常；tryRandomNode表示是否尝试随机一个节点，前面能看到传入的是false；asking表示是否是ask，也是false。判断asking状态和tryRandomNode。都是false，然后这样connection = connectidnHandler. getConnecti onF romSlot(Jedi sClusterCRC16. getSlot(key));拿到连接，connectidnHandler就是在jedis初始化的时候梳理节点和槽的关系，通过key计算槽拿到对应节点连接
    - 然后excute方法执行命令：执行过程忽略，找到异常的处理。
      - 捕获到没有节点可达的异常则直接抛出异常
      - 捕获连接异常，则释放当前连接，attempts<=1时才会connectidnHandler.renewSlotCache来刷新缓存（因为不要轻易刷新缓存，保证效率），然后attempts--并重新调用runWithRetries
      - JedisRedirectionException重定向异常，然后分别判断moved和ask异常
        - 如果是moved异常会直接connectidnHandler.renewSlotCache来刷新缓存
        - 释放jedis连接
        - 如果ask异常，会设置asking为true，然后重新拿到目标节点连接
        - 调用runWithRetries重试

  - smart客户端使用: JedisCluster

    - JedisCluster基本使用

      ```java
      Set<HostAndPort> nodeList = new HashSet <HostAndPort>();
      nodeList.add(new HostAndPort(HOST1, PORT1));
      nodeList.add(new HostAndPort(HOST2, PORT2));
      nodeList.add(new HostAndPort(HOST3, PORT3));
      nodeList.add(new HostAndPort(HOST4, PORT4));
      nodeList.add(new HostAndPort(HOST5, PORT5));
      nodeList.add(new HostAndPort(HOST6, PORT6));
      JedisCluster redisCluster = new JedisCluster(nodeList, timeout, poolConfig);redisCluster.command... //执行命令就好了，归还连接都不需要，jediscluster帮我们做了
      ```

      - 使用技巧：
        - 保证单例：内置了所有节点的连接池，从节点也有连接，保证故障转移后也有完整连接池。单例保证资源唯一性，并且不至于资源浪费
        - 无需手动借还连接池
        - 合理设置commons- pool

    - 整合spring

      ```java
      public class JedisClusterFactory {
          private JedisCluster jedisCluster;
          private List<String> hostPortList;
          private int timeout; //单位是毫秒
          private Logger logger = LoggerFactory.getLogger(JedisClusterFactory.class);
          
          //初始化方法
          public void init() {
              JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();//用于设置相关参数
              
              Set <HostAndPort> nodeSet = new HashSet<HostAndPort>();
              for(String hostPort : hostPortList) {
                  String[] arr = hostPort.split(":");
                  if (arr.length != 2) {
                      continue;
                  }
                  nodeSet.add(new HostAndPort(arr[0], Integer .parseInt(arr[1])));
              }
              try {
                  jedisCluster = new JedisCluster(nodeSet, timeout, jedisPoolConfig) ;
              } catch (Exception e) {
                  logger.error(e.getMessage(), e);
              }
          }
          
          //销毁方法
          public void destroy() {
              if (jedisCluster != null) {
                  try {
                      jedisCluster.close();
                  } catch (I0Exception e) {
                      logger.error(e.getMessage(), e);
                  }
              }
          }
          
          public JedisCluster getJedisClusterO) {
              return jedisCluster;
          }
          public void setHostPortList(List<String> hostPortList) {
              this. hostPortList = hostPortList;
          }
          public void setTimeout(int timeout) {
              this. timeout = timeout;
          }
      }
      ```

      ```xml
      <bean id= "jedisClusterFactory" class= "com.carlosfu.redis.factory.JedisClusterFactory" init-method= "init" destroy-method="destroy">
          <property name= "hostPortList ">
              <list>
                  <value>127.0.0.1:7000</value>
                  <value>127.0.0.1:7001</value>
                  <value>127.0.0.1:7002</value>
                  <value>127.0.0.1:7003</value>
                  <value>127.0.0.1:7004</value>
                  <value>127.0.0.1:7005</value>
              </list>
          </property>
          <property name= "timeout" value= "1000" />
      </bean>
      <bean id= "jedisCluster" factory-bean= "jedisClusterFactory" factory-method= "getJedisCluster"/></bean>
      <bean id= "redisClusterService" class= "xxx.RedisClusterServiceImpl"></bean>
      ```

      ```java
      public class RedisClusterServiceImpl implements RedisService { //RedisService省略
          private Jedi sCluster jedi sCluster;
          public String set(String key, String value) {
              return jedisCluster . set(key, value);
          }
          public String get(String key) {
              return jedisCluster . get(key);
          }
          public void setJedisCluster(Jedi sCluster jedisCluster) {
              this. jedisCluster = jedisCluster;
          }
      }
      ```

      ```java
      //测试类
      public class Jedi sClusterSpringTest extends BaseTest {
          @Resource(name = " redi sClusterService" )
          private Redi sService redi sClusterService;
          @Test
          public void testNotNullO {
              assertNotNul I(redi sClusterService);
              redi sClusterService . set("hello", "world");
              assertTrue("world".equals(redisClusterServicel.get("hello")));
          }
      }
      ```

      非常简单，无需去用spring data jedis的库

    - 多节点命令实现

      ```java
      //获取所有节点的JeidsPool
      //获取主节点即可，获取到所有主节点连接就可以进行多节点操作了
      //一般来说是try catch处理，这里仅简单演示，写的时候注意用try catch即可
      Map < String, JedisPool> jedisPoolMap = jedisCluster.getClusterNodes0;
      for (Entry<String, JedisPool> entry : jedisPoolMap.entrySet() {
          //获取每个节点的Jedis连接
          Jedis jedis = entry.getValue().getResource();
          //只删除主节点数据
          if (!isMaster(jedis)) { //判断主节点的方法，只需执行一个row（听起来是这个，具体拼写不知道）命令，会返回master还是sleve。如果客户端不支持这个命令，infoReplication中row的字段来判断
              continue;
          }
          // finally close
      }
      ```

    - 批量命令实现：mget mset必须在一个槽，这样的条件很苛刻，将用以下4中方法解决

      - 串行mget：for循环遍历所有key，分别操作。n次网络操作，n次网络时间，效率很差
      - 串行IO：做一个聚合，将CRC16hash计算槽，根据槽、节点对应关系，将属于相同节点的key放入同一子集合来mget，多个节点的mget操作可以并行，节点数次网络操作，节点数次网络时间
      - 并行IO：做一个聚合，将CRC16hash计算槽，根据槽、节点对应关系，将属于相同节点的key放入同一子集合来mget，多个节点的mget操作可以多线程并行，使用节点数的网络操作次数，1次网络时间
      - hash_tag：将key进行hash_tag的包装，{tag}key1、{tag}key2...{tag}keyN，让所有key都落到一个redis节点上，每次mget就可以只去一个节点取即可

      | 方案     | 优点                                  | 缺点                                          | 网络IO            |
      | -------- | ------------------------------------- | --------------------------------------------- | ----------------- |
      | 串行mget | 编程简单<br/>少量keys满足需求         | 大量keys请求延迟严重O(keys)                   | O(keys)           |
      | 串行IO   | 编程简单<br/>少量节点满足需求         | 大量node延迟严重                              | O(nodes)          |
      | 并行IO   | 利用并行特性<br/>延迟取决于最慢的节点 | (多线程)编程复杂<br/>(多线程)超时定位问题难   | O(max_slow(node)) |
      | hash_tag | 性能最高                              | 读写增加tag维护成本<br/>tag分布易出现数据倾斜 | O(1)              |



## 故障转移

### 故障发现

- 通过ping/pong消息实现故障发现:不需要sentinel
  - 
- 主观下线：定义:某个节点认为另一一个节点不可用，"偏见"。节点A定期发送ping给节点B，成功节点B回复PONG，节点A记录与节点B的最后通信时间，如果ping失败了则通信异常断开连接，在下一次执行ping时，如果与节点B的最后通信时间超过了node-timeout则表示节点B为pfail状态，即主观下线
- 客观下线：定义:当半数以上持有槽的主节点都标记某节点主观下线，只有主节点才有读写和槽维护的工作，从节点只做复制的作用。节点B接收到其它节点发来的ping消息，它包含了pfail消息，会将该主观下线的信息添加到其自身维护的一个故障链表中，该故障链表中包含了当前节点收到的每个节点对其它节点的信息，能知道每个节点对每个节点的看法，链表是有周期的，周期为cluster-node-timeout*2，保证很久之前的故障消息不再生效，保证客观下线的公平性和有效性
  - 尝试客观下线：计算有效下线报告数量，大于持有槽的主节点总数的一般，则更新为客观下线，向集群广播下线的节点的fail消息；否则退出尝试下线方法
  - 下线通知：通知集群内所有节点标记故障节点为客观下线；通知故障节点的从节点触发故障转移流程

### 故障恢复

- 资格检查：对多个从节点进行资格检查，在资格审查范围内的从节点才有资格去做故障恢复的工作
  - 每个从节点检查与故障主节点的断线时间。
  - 超过cluster-node-timeout * cluster-slave-validity-factor取消资格。cluster-node-timeout默认15000毫秒，cluster-slave-validity-factor默认10
- 准备选举时间：为了使偏移量值最大的从节点具备优先成为主节点的条件，因为偏移量最大，保证与原主节点数据一致性最高
  - offset较大会优先去进行选举，比如offset最大的节点rank=0，第二rank=1，第三rank=2，那么rank=0的节点拥有最小的准备选举时间，延迟1秒就去进行选举，rank=1的节点延迟2秒才去选举，rank=2的节点延迟3秒才去选举
- 选举投票：让其它非客观下线的主节点进行投票，选票大于主节点一半数即该从节点选举为新的主节点，是否投票的机制与前面主从故障转移是一样的
- 替换主节点：
  - 1.当前从节点取消复制变为主节点。(slaveof no one)。
  - 2.执行clusterDelSlot撤销故障主节点负责的槽，并执行clusterAddSlot把这些槽分配给自己。
  - 3.向集群广播自己的pong消息,表明已经替换了故障从节点

### 故障转移演练

1.执行kill -9节点模拟宕机

2.观察客户端故障恢复时间：大概不到20秒

3.观察各个节点的日志

- 7000：被kill后没有任何日志，重启后日志记录连接到了新的master
- 7003：7000原来的从节点
  - 与原来的master7000失联了，很多连接失败的信息
  - 接收到FAIL消息，客观下线
  - 打印offset，
  - 成为新的master
  - 清除7000主节点的一些信息，如FAIL信息
  - 与其它从节点同步
- 7002：清除7000主节点的一些信息，如FAIL信息；等等，不再多看，有兴趣自己看



## 开发运维常见问题

### 集群完整性

- cluster-require-full-coverage，默认为yes
  - 集群中16384个槽全部可用：保证集群完整性
  - 如果节点故障或者正在故障转移，将(error) CLUSTERDOWN The cluster is down，整个集群不可用
  - 大多数业务无法容忍，cluster-require-full-coverage建议设置为no

### 带宽消耗

- 问题
  - 官方建议: 1000个节点
  - PING/PONG消息
  - 不容忽视的带宽消耗
- 三方面：
  - 消息发送频率:节点发现与其它节点最后通信时间超过cluster-node-timeout/2时会直接发送ping消息
  - 消息数据量: slots槽数组(2KB空间)和整个集群1/10的状态数据(10个节点状态数据约1KB)
  - 节点部署的机器规模:集群分布的机器越多且每台机器划分的节点数越均匀,则集群内整体的可用带宽越高。
- 一个例子：规模，节点200个、20台物理机(每台10个节点)
  - cluster-node-timeout = 15000，ping/pong带宽为25Mb
  - cluster-node-timeout = 20000，ping/pong带宽低于15Mb
- 优化
  - 避免"大"集群：避免多业务使用一个集群，一个大业务也可以多集群（比如一个大的推荐服务，根据不同推荐类型使用不同集群，因为不同推荐类型之间没有相关性）
  - cluster-node-timeout：带宽和故障转移速度的均衡尽量均匀分配到多机器上:保证高可用和带宽
  - 尽量均匀分配到多机器上：保证高可用和带宽。可以自己去开发一套分配规则来达到负载的均衡（使用工具）

### Pub/Sub广播

- 问题：publish在集群每个节点广播，加重带宽。对任意节点执行publish发布消息，publish将广播到整个集群
- 解决：单独"走"一套Redis Sentinel。因为发布订阅的功能本身比较独立

### 集群倾斜

- 集群倾斜
  - 数据倾斜:内存不均
    - 节点和槽分配不均，导致数据倾斜
      - redis-trib.rb info ip:port查看节点、槽、键值分布
      - redis-trib.rb rebalance ip:port进行自动均衡节点和槽：redis内部自己的算法。谨慎使用，建议自己做计划手动迁移，而不要直接rebalance
    - 不同槽对应键值数量差异较大
      - CRC16正常情况下比较均匀。
      - 可能存在hash_tag：通常是主要原因
      - cluster countkeysinslot {slot}获取槽对应键值个数
    - 包含bigkey：
      - 因为数据只能以key为单位存储在某个节点上，比如一个hash、set等非常大，有几百万元素，就会造成数据倾斜
      - 从节点: redis-cli --bigkeys，发现bigkey，建议在从节点执行
      - 优化：优化数据结构。拆分该大个数据，二次hash拆分等
    - 内存相关配置不一致：我们知道如hash、list、zset等都有一些内存优化的配置参数，比如整数集合的优化等对于redis来说会节省一些内存，假如项目中大量使用了集合、hash的时候，我们做了一些优化，但没有在所有节点做优化，就会出现倾斜；还有假如某个节点的客户端缓冲区比较高，比如节点被执行了moniter命令，也会出现数据倾斜的情况；还有我们知道redis的键值对是存在一个hash表中的，如果key较多，该表也会扩容，扩容如果刚好触发到某一个数值，需要做rehash，hash表也可能会占用很大内存
      - hash-max- ziplist-value、set-max-intset-entries等
      - 优化：定期"检查"配置一致性
  - 请求倾斜：热点
    - 热点key：重要的key或者bigkey。比如redis某个节点有一个重要的key，比如一个微博大V在重要的时间节点发布了一个重要消息会落在一个重要的key上，就会存在热点问题
    - 优化：后面会讲缓存设计与优化，这里不详细讲
      - 避免bigkey
      - 热键不要用hash_tag
      - 当一致性不高时，可以用本地缓存 + MQ。比如java的本地缓存，没有网络时间，肯定比redis效率高，但就需要考虑gc的问题，还需要达到一致性，就可以使用mq，比如redis中做了更新就去更新本地缓存达到一致性

### 读写分离

- 只读连接：集群模式的从节点不接受任何读写请求
  - 如果对从节点请求，它将重定向到负责槽的主节点
  - readonly命令可以使从节点只读：连接级别命令，即连接断掉以后，需要重新对该从节点执行readonly来连接
- 读写分离：更加复杂
  - 与单机同样的问题：复制延迟、读取过期数据、从节点故障
  - 需要修改客户端：redis-cluster提供了cluster slaves {nodeId}命令，可以根据主节点id获取其从节点，所以需要实现自己的客户端，很复杂，要维护一个slave的池子，要知道从节点和槽的关系
- 建议：redis集群使用读写分离成本很高，尽量选择扩展集群规模，而不是利用从节点来实现读写分离。实在没有集群扩展成本，也可以考虑自己实现客户端来实现读写分离

### 数据迁移

- 官方迁移工具: redis-trib.rb import

  ```shell
  ./redis-trib.rb import --from 127 .0.0.1:6388 --copy 127.0.0.1:7000 #这里copy，也有replace参数。
  #
  ```

  - 只能从单机迁移到集群
  - 不支持在线迁移：source需要停写，否则源节点在迁移期间的写入的数据可能不会被记录。redis-trib.rb使用的是一个scan的模式，所有被scan到的数据都会被copy，scan期间源节点有新的数据，可能也会正好被scan到，有的数据加入时可能scan的游标已经到更后面了，就不会被scan到
  - 不支持断点续传：迁移过程中断不会记录之前迁移了的过程，只能重新重头迁移
  - 单线程迁移：影响速度。数据较大可能迁移比较慢

- 在线迁移工具：都是在GitHub开源的

  - 唯品会：redis-migrate-tool
  - 豌豆荚：redis-port

### 集群vs单机

- 集群限制
  - key批量操作支持有限：例如mget、mset必须在一 个slot
  - Key事务和Lua支持有限：如果使用事务或者Lua，操作的key必须在一个节点，虽然可以使用hash_tag使它们落在一个节点上，但本质上还是受限的
  - key是数据分区的最小粒度：不支持bigkey分区
  - 不支持多个数据库：集群模式下只有一个db 0。当然在一般的单机下也不建议使用多db模式（最多16）
  - 复制只支持一层：不支持树形复制结构
- 思考：分布式Redis不一定好
  - Redis Cluster：满足容量和性能的扩展性，很多业务”不需要"
    - 大多数时客户端性能会"降低"：比如批量操作再怎么优化也不如单节点原子操作
    - 命令无法跨节点使用：mget、keys、 scan、flush、sinter等
    - Lua和事务无法跨节点使用。
    - 客户端维护更复杂: SDK和应用本身消耗(例如更多的连接池)。
  - 很多场景Redis Sentinel已经足够好

## 总结

- Redis cluster数据分区规则采用虚拟槽方式(16384个槽) ,每个节点负责一部分槽和相关数据,实现数据和请求的负载均衡。
- 搭建集群划分四个步骤:准备节点、节点握手、分配槽、复制。redis-trib.rb工具用于快速搭建集群。
- 集群伸缩通过在节点之间移动槽和相关数据实现。
  - 扩容时根据槽迁移计划把槽从源节点迁移到新节点。
  - 收缩时如果下线的节点有负责的槽需要迁移到其它节点,再通过cluster forget命令让集群内所有节点忘记被下线节点。
- 使用smart客户端操作集群达到通信效率最大化,客户端内部负责计算维护键->槽->节点的映射,用于快速定位到目标节点。
- 集群自动故障转移过程分为故障发现和节点恢复。节点下线分为主观下线和客观下线,当超过半数主节点认为故障节点为主观下线时标记它为客观下线状态。从节点负责对客观下线的主节点触发故障恢复流程,保证集群的可用性。
- 开发运维常见问题包括：超大规模集群带宽消耗，pub/sub广播问题，集群倾斜问题，单机和集群对比等

# 开发规范



## 键值设计

- key设计

  - 可读性和可管理性：以业务名(或数据库名)为前缀(防止key冲突) , 用冒号分割，比如业务名:表名:id ,如: ugc:video:1、database:table:1

  - 简洁性：保证语义的前提下，控制key的长度，当key较多时，内存占用也不容忽视( redis3：39字节embstr，这里的意思是redis3版本的embstr编码可以很大程度节省内存) , 如 user:{uid}:friends:messages:{mid}简化为 u:{uid}:f:m :{mid}

    ```shell
    type #对外可能都是string类型
    object encoding key #查看key的编码。内部编码各不相同：int、raw、embstr等，大于39个字节的字符串是raw，embstr则是用于存储小于等于39个字节的字符串。int会有int的优化，embstr也会有一些优化，redis在分配内存的时候，除了给redis对象分配内存，还会给其存储的字符串内容分配内存，embstr则是将对象和其字符串内容进行一次性分配，并且是连续的内存，可以节省分配内存的次数，也能节省一定的空间。在redis4版本之后对redis内部的sds实现做了一些优化，embstr的阈值更变为44个字节了
    ```

    Redis3 embstr测试

    | key-value个数 | 39字节  | 40字节  |
    | ------------- | ------- | ------- |
    | 10万          | 15.69M  | 18.75M  |
    | 100万         | 146.29M | 176.81M |
    | 1000万        | 1.47G   | 1.77G   |
    | 1亿           | 14.6G   | 17.7G   |

  - 不要包含特殊字符。反例:包含空格、换行、单双引号以及其他转义字符

- value设计

  - 拒绝bigkey

    - 阈值（并非绝对，只是参考标准）

      - string类型控制在10KB以内
      - hash、list、set、zset元素 个数不要超过5000

    - 反例：一个包含几百万个元素的list、hash等， 一个巨大的json字符串

    - 危害：

      - 网络阻塞：比如千兆网，最大流量为128m，一个热点key的qps为10w，那么key为10kb，也能差不多瞬间将网络吃满
      - Redis阻塞：慢查询，hgetall、lrange、 zrange(例如几十万)等全量操作，因为redis单线程，所以会阻塞其它命令
      - 集群节点数据不均衡
      - 频繁序列化：应用服务器CPU消耗

    - bigkey发现

      - 应用异常：报错，如一个bigkey操作，其它命令将阻塞然后time out

      - 官方工具：redis-cli --bigkeys

      - 自己实现：scan + debug object

        ```shell
        debug object key #获取序列化长度。如果bigkey很大，debug object也可能阻塞redis，可以使用zcat、hlen、strlen等，比如长度大于多少来判定是否为bigkey
        ```

      - 主动报警：网络流量监控、客户端监控

      - 改造redis源码内核：内核热点key问题优化，在redis中维护一个堆来记录redis对key大小的变化

    - bigkey删除：删除也可能会阻塞redis

      - 阻塞:注意隐性删除(过期、rename等)：也是删除，可能会阻塞redis，且过期、删除的命令不会记录在redis的慢查询当中，redis慢查询只记录客户端的行为，过期、删除只会记录在redis的license（一个记录延迟事件的记录）中，但是这些命令会同步给slave，可以在从节点找到对应慢查询

      - Redis 4.0 : lazy delete (unlink命令)，后台删除，单独开启线程去执行删除，不会阻塞redis了

      - 通过scan删除

        ```java
        public void delBigHash(String host, int port, String password, String bigHashKey) {
            Jedis jedis = new Jedis(host, port);
            if (password != null && !"" .equals(password)) {
                jedis.auth(password);
            }
            ScanParams scanParams = new ScanParams().count(100);
            String cursor = "0";
            do {
                ScanResult<Entry<String, String>> scanResult = jedis.hscan(bigHashKey,cursor, scan); //不全，不知道对不对
                List<Entry<String, String>> entryList = scanResult. getResult();
                if (entryList != null && !entryList. isEmpty()) {
                    for (Entry<String, String> entry : entryList) {
                        jedis.hdel(bigHashKey, entry . getKey());
                    }
                }
                cursor = scanResult . getStringCursor();
            } while (!"0" .equals(cursor));
            
            jedis.del(bigHashKey);//删除bigkey
        }
        ```

    - bigkey预防

      - 优化数据结构：例如二级拆分
      - 物理隔离或者万兆网卡：治标不治本
      - 命令优化：例如hgetall-> hmget、hscan
      - 报警和定期优化

    - bigkey总结

      - 牢记Redis单线程特性
      - 选择合理的数据结构和命令
      - 清楚自身OPS
      - 了解bigkey的危害



## 选择合适的数据结构

- 实体类型(数据结构内存优化:例如ziplist ,注意内存和性能的平衡)

- 反例
  set user:1:name tom;
  set user:1:age 19;
  set user:1:favor football;

- 正例：使用hash，前面有讲好处
  hmset user:1 name tom age 19 favor football

- 一个例子、三种方案：需求: picId= > userId (100万)

  - 全部string：set picId userId。常规方法。使用内存116m

    - 优点：编程简单
    - 缺点：浪费内存；全量获取复杂

    | Key         | Value        |
    | ----------- | ------------ |
    | pic:1       | user:1       |
    | ......      | ......       |
    | pic:1000000 | user:1000000 |

  - 一个hash：hset allPics picId userId。形成一个bigkey，不合理的设计。使用内存129m

    - 优点：无
    - 缺点：浪费内存；形成bigkey

    | Key     | Field                           | Value                             |
    | ------- | ------------------------------- | --------------------------------- |
    | allPics | pic:1<br/>.....<br/>pic:1000000 | user:1<br/>.....<br/>user:1000000 |

  - 若干个小hash：hset picId/100 picId%100 userId。使用内存26m

    - 优点：节省内存
    - 缺点：编程复杂；超时问题；ziplist性能问题

    | Key           | Field                       | Value                                  |
    | ------------- | --------------------------- | -------------------------------------- |
    | segment:0     | pic:1<br/>.....<br/>pic:100 | user:1<br/>.....<br/>user:100          |
    | ......        | ......                      |                                        |
    | segment:99999 | pic:1<br/>.....<br/>pic:100 | user:999901<br/>.....<br/>user:1000000 |

- 内存

  - 配置(支持动态修改)：当元素个数少于512、元素value小于64字节时，redis默认使用zplist来存储hash结构
    - hash-max-ziplist-entries 512
    - hash-max-ziplist-value 64
  - ziplist：有好有坏，需要衡量，所以才配置为元素个数较少、元素value较小时使用
    - 连续内存：节省内存
    - 读写有指针位移，最坏O(n2)，影响读写效率
    - 新增删除有内存重分配，影响增删效率



## 过期设计

- 键值生命周期：Redis不是垃圾桶，不要什么东西都扔给redis，还不控制缓存时间

  - 周期数据需要设置过期时间，object idle time可以找到超出指定闲置时间的垃圾kv，即多久没有使用的kv
  - 过期时间不宜集中：缓存穿透和雪崩等问题，比如每三小时集体过期，并通过数据库重建缓存

  

## 命令优化

- [推荐] O(N)以上命令关注N的数量：例如，hgetall、Irange、 smembers、zrange、 sinter等并非不能使用,但是需要明确N的值。有遍历的需求可以使用hscan、sscan、 zscan代替。即主要考虑n的大小
- [推荐]禁用命令：禁止线上使用keys、flushall、 flushdb等，通过redis的rename机制禁掉命令，或者使用scan的方式渐进式处理。
- [推荐]合理使用select：选择redis数据库，可以用redis的db对一些数据进行隔离或测试环境区分等。但redis的多数据库较弱，使用数字进行区分（db 0~15），很多客户端对redis数据库的操作支持较差；注意同时多业务用多数据库实际还是单线程处理，还是会有干扰，频繁切换redis数据库也有一定消耗
- [推荐] Redis事务功能较弱，不建议过多使用：Redis的事务功能较弱(不支持回滚)，需要时借助外部来实现，比如java的事务实现等；而且集群版本(自研和官方)要求一次事务操作的key必须在一个slot上(可以使用hashtag功能解决，但也可能造成数据不均匀问题)。使用时不要太过信赖，只是一个辅助功能而已
- [推荐] Redis集群版本在使用Lua上有特殊要求：不要过度使用，虽然可以实现很多功能。或者单独开一个redis来做对应的功能，避免一些集群问题
  - 所有key，必须在1个slot上,否则直接返回error，同样通过hashtag可以解决
  - '-ERR eval/evalsha command keys must in same slot\r\n"
- [建议] 必要情况下使用monitor命令时，要注意不要长时间使用。monitor可以查看redis监控redis执行的命令，但是如果并发量过高，会有大流量打在monitor-client（monitor也是一个redis客户端）上，会造成monitor无法实时消费大量的流量，会导致输出缓冲区暴增，会占用redis内存，撑爆redis内存



## 客户端优化

Java客户端优化

- [推荐] 避免多个应用使用一个Redis实例

  - 问题
    - 业务之间key冲突，可以通过select redis数据库来解决
    - 单线程redis，业务A的命令会影响到业务B
  - 解决
    - 不相干的业务拆分：不要使用一个redis实列，可以在一个机器上启动多个redis实例合理利用多核机器
    - 公共数据做服务化：有公共数据，比如视频、音频等，不论底层使用redis还是mysql，对外只暴露服务接口即可，如http接口，让其作为单独的微服务应用对外提供服务

- [推荐] 使用连接池，标准使用方式

  ```java
  Jedis jedis = null;
  try {
      jedis = jedisPool,getResource();
      //具体的命令
      jedis. executeCommand()
  } catch (Exception e) {
      logger.error("op key {} error: " + e.getMessage(), key, e);
  } finally {
      //注意这里不是关闭连接，在JedisPool模式下，Jedis会被归还给资源池。
      if (jedis != null)
          jedis. close();
  }
  
  ```




# 开发运维常见坑

# Redis内存管理



# RedisCacheCloud

https://github.com/sohutv/cachecloud

Redis云平台CacheCloud。老师自己在原来公司参与开发的

- Redis规模化运维
- 快速构建
- 机器部署
- 应用接入
- 用户功能
- 运维功能



## Redis规模化运维

- 遇到的问题
  - 发布构建繁琐，私搭乱盖
  - 节点&机器等运维成本
  - 监控报警初级
- CacheCloud
  - 一键开启Redis。 (Standalone、 Sentinel、 Cluster)
  - 机器、应用、实例监控和报警。
  - 客户端:透明使用、性能上报。
  - 可视化运维:配置、扩容、Failover、机器/应用/实例上下线。
  - 已存在Redis直接接入和数据迁移。
- 使用规模
  - 300+亿commands/day
  - 3TB Memory Total
  - 1300+ Instances Total
  - 200+ Machines Total
- 使用场景
  - 全量视频缓存(视频播放API) :跨机房高可用
  - 消息队列同步(RedisMQ中间件)
  - 分布式布隆过滤器(百万QPS)
  - 计数系统:计数(播放数)
  - 其他:排行榜、社交(直播)、实时计算(反作弊)等。



## 快速构建

https://github.com/sohutv/cachecloud/wiki

## 机器部署

## 应用接入

## 用户功能

## 运维功能

