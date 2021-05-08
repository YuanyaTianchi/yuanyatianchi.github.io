

+++

title = "MongoDB"
description = "MongoDB"
tags = ["it", "db"]

+++



# MongoDB

- 文档数据库(schema free)，基于二进制JSON存储文档（BSON）。无需定义类型，field:value形式
- 高性能、高可用、直接加机器即可以解决扩展性问题
- 支持丰富的CRUD操作，例如:聚合统计、全文检索、坐标检索

# client

### go

https://github.com/mongodb/mongo-go-driver

```shell
$ go get go.mongodb.org/mongo-driver/mongo
```

###### .go

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





应用场景

传统的关系型数据库(如MySQL)， 在数据操作的"三高 ’需求以及应对Web2.0的网站需求面前，显得力不从心。
解释: "三高”需求:
●High performance -对数据库高并发读写的需求。
●Huge Storage -对海量数据的高效率存储和访问的需求。
●High Scalability && High Availability-对数据库的高可扩展性和高可用性的需求。
而MongoDB可应对“三高"需求。

具体的应用场景如:

- 社交场景，使用MongoDB存储存储用户信息,以及用户发表的朋友圈信息，通过地理位置索引实现附近的
  人、地点等功能。
- 游戏场景，高效率存储和访问。使用MongoDB存储游戏用户信息，用户的装备、积分等直接以内嵌文档的形式存储，方便查询
- 物流场景，使用MongoDB存储订单信息，订单状态在运送过程中会不断更新，以MongoDB内嵌数组的形式
  来存储，一次查询就能将订单所有的变更读取出来。
- 物联网场景，使用MongoDB存储所有接入的智能设备信息，以及设备汇报的日志信息，并对这些信息进行多
  维度的分析。
- 视频直播,使用MongoDB存储用户信息、点赞互动信息等。

这些应用场景中，数据操作方面的共同特点是:
(1) 数据量大
(2)写入操作频繁(读写都很频繁)
(3)价值较低的数据，对事务性要求不高
对于这样的数据，我们更适合使用MongoDB来实现数据的存储。

什么时候选择MongoDB ?
在架构选型上，除了上述的三个特点外，如果你还犹豫是否要选择它?可以考虑以下的一些问题:
应用不需要事务及复杂join支持
新应用，需求会变,数据模型无法确定,想快速迭代开发
应用需要2000-3000以上的读写QPS (更高也可以)
应用需要TB甚至PB级别数据存储
应用发展迅速，需要能快速水平扩展.
应用要求存储的数据不丢失
应用需要99.999%高可用
应用需要大量的地理位置查询、文本查询
如果上述有1个符合，可以考虑MongoDB, 2个及以上的符合，选择MongoDB绝不会后悔。

MongoDB简介
MongoDB是一个开源、 高性能、无模式的文档型数据库，当初的设计就是用于简化开发和方便扩展,是NoSQL数
据库产品中的一种。是最像关系型数据库I(MySQL)的非关系型数据库。
它支持的数据结构非常松散,是一种类似于JSON的格式叫BSON(二进制存储的json),所以它既可以存储比较复杂的数据类型，又相
当的灵活。
MongoDB中的记录是一个文档, 它是一个由字段和值对 (field:value) 组成的数据结构。MongoDB文档类似于
JSON对象，即一个文档认为就是一个对象。 字段的数据类型是字符型，它的值除了使用基本的一些类型外,还可
以包括其他文档、普通数组和文档数组。

| MongoDB术语/概念     | SQL术语/概念 | 解释/说明                                              |
| -------------------- | ------------ | ------------------------------------------------------ |
| database             | database     | 数据库                                                 |
| collection           | table        | 数据库表/集合                                          |
| document(bson格式)   | row          | 数据记录行/文档                                        |
| field                | column       | 数据字段/域                                            |
| index                | index        | 索引                                                   |
| 嵌入文档 $lookup     | table joins  | 跨表。MongoDB通过嵌入式文档来实现。mysql通过表连接实现 |
| _id                  | primary key  | 主键。主键MongoDB自动将id字段设置为主键                |
| aggregation pipeline | group by     | 聚合                                                   |

BSON数据类型参考列表，基本和JSON一致

| 数据类型      | 描述                                                         | 举例                                                 |
| ------------- | ------------------------------------------------------------ | ---------------------------------------------------- |
| 字符串        | UTF-8字符串都可表示为字符串类型的数据                        | {"x" : "foobar"}                                     |
| 对象id        | 对象id是文档的12字节的唯一ID                                 | {"X" :objectld() }                                   |
| 布尔值        | 真或者假: true或者false                                      | {"x":true}+                                          |
| 数组          | 值的集合或者列表可以表示成数组                               | {"x”: ["a", "b","c"]}                                |
| 32位整数      | 类型不可用。JavaScript仅支持64位浮点数， 所以32位整数会被自动转换。 | shell是不支持该类型的，shell中默认会转换成64位浮点数 |
| 64位整数      | 不支持这个类型。shell会使用一 个特殊的内嵌文档来显示64位整数 | shell是不支持该类型的，shell中默认会转换成64位浮点数 |
| 64位浮点      | shell中的数字就是这一种类型                                  | {"x": 3.14159, "y": 3}                               |
| null          | 表示空值或者未定义的对象                                     | {"x"null}                                            |
| undefined     | 文档中也可以使用未定义类型                                   | {"x":undefined}                                      |
| 符号          | shell不支持，shell会将数据库中的符号类型的数据自动转换成字符串 |                                                      |
| 正则表达      | 文档中可以包含正则表达式，采用JavaScript的正则表达式语法     | {"x" : /foobar/}                                     |
| 代码          | 文档中还可以包含JavaScript代码                               | {"x" : function0{/* ... */}}                         |
| 二进制数据    | 二进制数据可以由任意字节的串组成,不过shell中无法使用         |                                                      |
| 最大值/最小值 | BSON包括一个特殊类型，表示可能的最大值。shell中没有这个类型。 |                                                      |

提示:
shell默认使用64位浮点型数值。{"x": 3.14}或{"x": 3}。对于整型值，可以使用NumberInt (4字节符号整数)或
NumberLong (8字节符号整数) , {"X":Number1It*"3"}{"x":NumberLngl"}

MongoDB的特点
MongoDB主要有如下特点:
(1)高性能:
MongoDB提供高性能的数据持久性。特别是，对嵌入式数据模型的支持减少了数据库系统上的I/O活动。
索引支持更快的查询，并粗可以包含来自嵌入式文档和数组的键。(文本索弓 |解决搜索的需求、TTL索引解决历史
数据自动过期的需求、地理位置索引可用于构建各种020应用)
mmapv1、wiredtiger. mongorocks (rocksdb) 、in-memory 等多引擎支持满足各种场景需求。
Gridfs解决文件存储的需求。
(2)高可用性:
MongoDB的复制工具称为副本集(replicaset) ，它可提供自动故障转移和数据冗余。
(3)高扩展性:
MongoDB提供了水平可扩展性作为其核心功能的一-部分。
分片将数据分布在一组集群的机器上。 (海 量数据存储，服务能力水平扩展)
从3.4开始，MongoDB支持基于片键创建数据区域。在-个平衡的集群中，MongoDB将一个区域所覆 盖的读写只
定向到该区域内的那些片。
(4)丰富的查询支持: 
MongoDB支持丰高的查询语言，支持读和写操作(CRUD),比如数据聚合、文本搜索和地理空间查询等。
(5)其他特点:如无模式(动态模式)、灵活的文档模型、

## 安装

### windows

- 下载并解压：https://www.mongodb.com/download-center/community

- 数据存储目录：手动创建目录，如data/db，

- 启动MongoDB：mongod命令。启动过后该cmd窗口不能关，否则服务就关闭了

  - 指定参数启动
    - --dbpath：指定数据存储目录
    - --port：指定端口，缺省27017

    ```cmd
    mongod --dbpath=../data
    ```

  - 指定配置文件启动：启动时可以指定配置文件，config目录下新建.conf文件如mongod.conf

    ```cmd
    mongod -f ../config/mongod.conf
    #或者
    mongod --config ../config/mongod.conf
    ```

    配置文件内容参考

    详细配置项内容可以参考官方文档: https://ocs.mongodb.com/manual/reference/configuration-options/
    [注意]
    1)配置文件中如果使用双引号，比如路径地址，自动会将双引号的内容转义。如果不转义,则会报错:
    error -parsing-yam1-config-file-yaml-cpp-error -at-1ine-3-column-15-unknown-escape-character-d
    解决:
    a.把 \ 换成 / 或 \ \
    b.如果路径中没有空格,则无需加引号。

    ```conf
    #配置文件，遵循yaml文件的格式
    storage:
      dbPath: E:\it\database\mongodb\data
    ```

  - 链接数据库：

    - mongo命令

    ```cmd
    mongo #连接
    mongo --host=127.0.0.1 --port=27017 #连接with参数。这里均为缺省值
    >show databases #显示数据库，databases可简写为dbs
    >exit #退出
    ```

    - MongoDBCompass：图形化界面
      - 下载并解压：https://www.mongodb.com/download-center/compass
      - 找到MongoDBCompass.exe即可打开

### Linux

(2) - 上传压缩包到Linux中,解压到当前目录:
tar -xvf mongodb-1inux-x86_ 64-4.0.10.tgz
(3)移动解压后的文件夹到指定的目录中:
mv mongodb-1inux-x86_ 64-4.0.10 /usr /loca1/mongodb
(4)新建几个目录,分别用来存储数据和日志:
擞数据存储目录
mkdir -p /mongodb/sing1e/data/db
#日志存储目录
mkdir -p /mongodb/sing1e/log
(5)新建并修改配置文件
vi /mongodb/s ing1e/mongod. conf

启动

```shell
bin/mogond --dbpath=./data #监听到所有网络上，一般当然不能这么做
```

```cmd
/usr/yuanya/it/database/mongodb/bin/mongod -f /mongodb/single/mongod.conf #启动with配置文件
ps -ef | grep mongod #检测是否启动
kill -2 27017 #关闭服务，数据可能出错
#标准关闭方法，数据不容易出错
mongo --port 27017
use admin
db.shutdownServer()
```



## 配置

```conf
systemLog:
  destination: file #MongoDB发送所有日志输出的目标指定为文件
  path: "/mongodb/sing1e/log/mongod.log" #mongod或imongos应向其发送所有诊断日志记录信息的日志文件的路径
  logAppend: true #当mongos或mongod实例重新启动时，mongos或mongod会将新条目附加到现有日志文件的末尾。
storage:
  dbPath: "/mongodb/sing1e/data/db' #mongod实例存储其数据的目录。storage.dbPath设置仅适用于mongod
  journal:
    enabled: true #启用或禁用持久性日志以确保数据文件保持有效和可恢复。缺省为true
processManagement:
  fork: true #启用在后台运行mongos或imongod进程的守护进程模式。启用后即可关闭控制台
net:
  bindIp: 192.168.0.2 #服务实例绑定的IP，默认只监听localhost，默认其它机器无法访问
  port: 27017 #绑定端口，默认是27017
```

## 命令

- db.help()：帮助
  
- 基本库
  
  - admin：类似于mysql的root库，有关mongodb的用户和权限等
  - local：部署集群时库之间数据会相互复制，但是local中的数据不会复制，可以用来存储限于本地单台服务器的任意集合
  - config：当mongo用于分片设置时，cofig数据库在内部使用，用于保存分片的相关信息
  
- database：库操作

  - show databases：显示库
  - use 数据库名称：有则进入库，无则创建并进入库。在内存中创建库，当库非空时才会持久化到磁盘

  - db：显示当前数据库
  - db.dropDatabase()：删除当前库。其实是js代码

- collection：集合操作

  - show collections：显示当前库的集合
  - db.createCollection("集合名")：显式创建集合。隐式创建只创建文档时如果没有指定的集合则将顺便创建集合
  - db.集合名.drop()：删

- document：文档操作

  - db.collection.insert(文档, {写关心, 排序})：插入文档。参数1为文档数据，参数2为可选

    如db.collection.insert({"username": "username1", "password": "password1"})插入单个文档

    如db.collection.insert( [{"username": "username1"}, {"username":"username2"}] )插入多个文档

    - 参数列表

      | Parameter    | Type              | Description                                                  |
      | ------------ | ----------------- | ------------------------------------------------------------ |
      | document     | document or array | 要插入到集合中的文档或文档数组。数据格式为bson(基本与json一致) |
      | writeConcern | document          | Optional. A document expressing the write concern. Omit to use the default write concern. See Write Concern.Do not explicitly set the write concern for the operation if run in a transaction. To use write concern with transactions, see Transactions and Write Concern. <br />翻译：可选。表示写关心的文档。忽略使用默认的写关注点。看到写问题。如果在事务中运行，不要显式地设置操作的写关注点。要在事务中使用写关注，请参阅事务并写关注。<br />简单理解为：插入时选择的性能和可靠性的级别。 |
      | ordered      | boolean           | 可选。如果为真,则按顺序插入数组中的文档，如果其中一个文档出现错误，MongoDB将返回而不处理数组中的其余文档。如果为假，则执行无序插入，如果其中一个文档出现错误，则继续处理数组中的主文档。在版本2.6+中默认为true |

  - db.集合名.find()：查询指定集合所有文档

    db.集合名.find({"username": "username1"})：条件查询

    db.集合名.findOne({"username": "username1"})：返回查询到的数据中的第一个，类似于limit 0,1

    db.集合名.findOne({"username": "username1"}, {username:1})：投影查询，只显示指定字段，这里将只显示username字段和\_id字段，\_id字段是mongodb中自动添加的主键，在不指定排除的情况下都会显示

    db.集合名.findOne({username,password:1,\_id:0})：显示username和password字段(可以分开写)，排除\_id字段

  - 

- 





# MongoDB

## 功能

- 增：db.my_collection.insertOne({uid: 10000, name: "xiaoming ", likes: ["football", "game"]})：插入一条json
  - 任意嵌套层级的BSON (二 进制的JSON)
  - 文档ID是自动生成的，通常无需自己指定
- 查：db,my_collection.find( { likes: 'football'; name: {$in: [ 'xiaoming', 'libai' ] } } ).sort({uid: 1})：$in表示只要在满足是一组名字 [ 'xiaoming', 'libai' ] 中的一个即可，.sort({uid: 1}) 表示按uid升序排列，正数即表示升序。
  - 可以基于任意BSON层级过滤
  - 支持的功能与MYSQL相当
- 改：db.my_collection.updateMany( { likes: 'football' }, { $set: { name: 'libai' } } )：更新喜欢足球的所有文档，设置其name为libai
  - 第一个参数是过滤条件
  - 第二个参数是更新操作
- 删：db.my_ collection.deleteMany({name: 'xiaoming'})：删除name为xiaoming的文档
  
- 参数是过滤条件
  
- 创建索引：db.my collection.createlndex({uid: 1, name: -1})：uid和name都创建索引，联合索引，

  - 可以指定建立索引时的正反顺序，正数表示正序，负数表示反序，非常灵活，这将影响到排序的性能，mysql指定索引不能指定正反顺序，

- 聚合操作：db.my_collection.aggregate( [ { $unwind: '$likes' }, { $group: { id: { likes: '$likes' } }, { $project: { id: 0, like: "$_ id.likes", total: { $sum:1 } } } )。pipeline流水线模式，将前一个阶段的输出，作为后一个阶段的输入，[ ]内即pipeline

  - { $unwind: '$likes' }：unwind，解开，会把 likes: ["football", "game"] 的内容拆分开，拆成 喜欢football 是一行，喜欢game 是另一行，将两行输入给下一阶段{ $group: { id: { likes: '$likes' } }
  - { $group: { id: { likes: '$likes' } }：聚合（即分组），将把 喜欢 football 的聚合在一起，喜欢game 的聚合在一起，然后进入下一阶段
  - { $project: { id: 0, like: "$_id.likes", total: { $sum:1 } } }：project操作符可以取出每一个桶类，按group去分桶，每个桶类是相同喜好的文档，like: "$\_id.likes"将把对应的likes提取出来
  - 输出：{"Iike": "game", "total": 1} {"Ilike": "football", "total": 1}。pipeline流式计算，功能复杂，使用时多查手册

  | MongoDB  | MYSQL    | 说明 |
  | -------- | -------- | ---- |
  | $match   | where    |      |
  | $group   | group by |      |
  | $match   | having   |      |
  | $project | select   |      |
  | $sort    | order by |      |
  | $limit   | limit    |      |
  | $sum     | sum      |      |
  | $count   | count    |      |



## 原理

- 整体存储架构：类似于单机->主从->多主从的分布式
  - Mongod：单机版数据库。没错就是叫"Mongod"不是"MongoDB"
  - Replica Set：复制集，由多个Mongod组成的高可用存储单元。
    - 有主从关系，数据写入主节点，异步复制数据到从节点。
    - 客户端默认读取主节点，可以要求客户端取读从节点，但从节点读取的数据可能存在延迟
    - 主节点宕机，复制集内会重新选举
    - 至少3个节点组成，其中1个可以只充当arbiter
    - 主从基于oplog复制同步(类比mysql binlog)
  - Sharding：分布式集群，由多 个Replica Set组成的可扩展集群
    - 每个Replica Set称为一个shard（分片），对外提供服务时通过一个或多个router（程序叫mongos）来代理转发客户端请求到正确的分片上
    - 存在一个ConfigServers，它也是一个Replica Set，专门用来存储数据元信息，即存放分片与数据的对应信息，记录哪些key存放在那个分片上的
    - Collection分片：Collection自动分裂成多个chunk，每个chunk被自动负载均衡到不同的shard，每个shard可以保证其上的chunk高可用。通俗点说就是一个表的数据是被分为多份（比如8份），均匀的分摊在多个分片（比如2个分片，每个分片上有4份该Collection的数据）上的
      - 按range拆分chunk：与mysql分库分表是类似的，假如表有一个x字段，按x拆分，该字段为shard key。假设x的取键是 负无穷~正无穷，可能就按 负无穷~-25、-75~25、25~175、175~正无穷 区分为4个chunk。具体按什么拆分，要根据数据特征来定，假如x是时间，可能过了175这个时间以后，所有数据都写到一个chunk里面，造成写入热点。
        - Shard key可以是单索引|字段或者联合索引字段：即必须要上索引
        - 超过16MB的chunk被一分为二
        - 一个collection的所有chunk首尾相连，构成整个表
      - 按hash拆分chunk
        - Shard key必须是哈希索引字段：mongodb支持一种hash索引，会先把这个字段hash成一个整型，用这个整型来建立索引。因为可能出现hash碰撞，无法保证key的唯一性，hash索引的字段是不能建立唯一键的
        - Shard key经过hash函数打散，避免写热点
        - 支持预分配chunk，提前分配较多chunk，可以避免运行时分裂影响写入
    - shard用法示例
      - 为DB激活特性：sh.enableSharding( 'my_db' ) 
      - 配置hash分片: sh.shardCollection(" my_db.my_collection", { _id: "hashed" }, false, { numInitialChunks: 10240} )。让my_db的表my_collection根据\_id做hash分片，预分配10240个chunk
    - 注意：如果按非shard key查询，比如如果我们按x字段分片，即x是shard key，但是查询的时候我们通过uid查询，请求将被**扇出**给所有shard，性能会差很多
- Mongod架构：默认采用WiredTiger高性能存储引擎，
  - 架构
    - Client：客户端。客户端发送请求到查询层
    - MongoDB Query Language：查询解释层，解析请求
    - MongoDB Data Model：存储层的抽象，支持多种底层的存储引擎
      - WiredTiger：默认的，非常高性能
      - MMAPv1：早期使用的引擎
      - In Memory
      - Encrypted
      - 3rd Party Engine
  - 基于journaling log宕机恢复(类比mysql的redo log)：写入东西的时候先去写log，将 "操作内容" 顺序追加成log文件，而数据会暂时修改到内存里面，它将定期把内存中的数据写入到磁盘，如果写入磁盘之前宕机了，就会通过log把数据恢复出来









