# 安装
1. 环境: 安装有gcc的Linux系统, gcc版本不低于5
2. 安装
```shell
$ wget http://download.redis.io/releases/redis-6.0.6.tar.gz
$ tar xzf redis-6.0.6.tar.gz
$ cd redis-6.0.6
$ make
$ make install
```
3. 启动
	1. 前台启动  `redis-server`\
	2. 后台启动  
	将redis文件夹配置文件redis.conf中`deamonize no`改成`deamonize yes`, 执行`redis-server/[配置文件目录]` 启动redis

# 命令
## key
* `keys * ` 查看当前库所有key(匹配: keys*1)
* `exists key`判断某个key是否存在。
* `type key` 查看你的key是什么类型。
* `del key` 删除指定的key数据
* `unlink key` 根据value选择非阻塞删除·(仅将keys 从keyspace元数据中删除，真正的删除会在后续异步操作)
* `expire key 10` 10秒钟:为给定的key设置过期时间
* `ttl key` 查看还有多少秒过期，-1表示永不过期，-2表示已过期
* `select` 切换数据库
* `dbsize` 查看当前数据库key的数量
* `flushdb` 清空当前库
* `flushall` 清空全部库


# 基本数据类型

## 字符串String

基本数据类型, 存储的数据结构为简单动态字符串(SDS), 是可修改字符串, 类似java的ArrayList, 采用预分配冗余空间的方式来减少内存的频繁分配. 当容量小于1M时, 扩容为当前容量X2, 超过1M后每次增加1M. 一个Redis字符串value最多可以是512M.

* `set key value`
* `setnx key value` 只有在key不存在时设置键值
* `get key`
* `getrange key index1 index2` 获取指定key下标[index1,index2]的值
* `setrange key index value` 设置key从下标index开始的值 
* `mset k1 v1 k2 v2...` m...操作多个键值对
* `mget k1 k2...` 同上
* `msetnx k1 v1 k2 v2` 同上
* `append key valve` 在原来键值对的value后追加
* `strlen key` 获取key对应的value长度
* `incr/decr key` 将key中存储的数值+1/-1
* `incrby/decrby key 值` 增加/减少指定步长
*  `setex key 过期时间 value` 同时设置键值和过期时间
* `setget key value` 替换并获取旧值

## 列表List

单键多值, Redis列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部(左边）或者尾部(右边)。它的底层实际是个双向链表，对两端的操作性能很高，通过索引下标的操作中间的节点性能会较差。
List的数据结构为快速链表quickList。在列表元素较少的情况下会使用一块连续的内存存储，这个结构是ziplist，也即是压缩列表。它将所有的元素紧挨着一起存储，分配的是一块连续的内存。当数据量比较多的时候才会改成quicklist。因为普通的链表需要的附加指针空间太大，会比较浪费空间。比如这个列表里存的只是int类型的数据，结构上还需要两个额外的指针prev和next。Redis将链表和ziplist结合起来组成 quicklist, 也就是将多个ziplist使用双向指针串起来使用, 既满足快速插入和删除性能, 又不会出现太多空间冗余.

* `lpush/rpush key v1 v2 v3` 从左/右往list中存放数据
* `lpop/rpop key` 从左/右弹出一个数据,  当值为空时list消亡
* `rpoplpush key` 从key的右边弹出一个值放到左边
* `lrange key index1 index2` 按索引从左到右获取元素
* `lindex key index` 获取指定下标元素
* `llen key` 获取列表长度
* `linser before/after value1 value2` 在value1前面/后面插入value2
* `lrem key n value` 从左边开始删除指定n个value值
* `lset key index value` 将列表下标为index的值替换成value

## 集合Set

 Set相当于一个去重列表. Set提供判断某个成员是否在集合的接口. 
Set是String类型你的无序集合, 底层是一个value为null的hash表.

* `sadd key v1 v2 v3` 存数据
* `smenbers key` 取出集合所有值
* `sismember key value` 判断集合中否包含value
* `scard key` 返回集合元素个数
* `srem key v1 v2` 删除集合中某个元素
* `spop key` 随机从集合中弹出一个元素
* `srandmember key n` 随即从集合中获取n个值, 不会删除
* `smove key1 key2 value` 从key1 取出value放到key2中
* `sinter key1 key2` 返回两个集合的交集
* `sunion key1 key2` 返回两个集合的并集
* `sdiff key1 key2` 返回两个集合差集

## 哈希Hash

是一个键值对集合, 是一个String类型的field和value的映射表, 类似java中的Map<String,Object>. 
Hash底层存储结构有两种实现方式。
第一种，当存储的数据量较少的时，hash 采用 ziplist 作为底层存储结构，此时要求符合以下两个条件：

- 哈希对象保存的所有键值对（键和值）的字符串长度总和小于 64 个字节。
- 哈希对象保存的键值对数量要小于 512 个。

当无法满足上述条件时，hash 就会采用第二种方式来存储数据，也就是 dict（字典结构），该结构类似于 Java 的 HashMap，是一个无序的字典，并采用了数组和链表相结合的方式存储数据。

* `hset key field value` 给key集合中的field键赋值value
* `hget key field`
* `hmset key f1 v1 f2 v2` 批量赋值
* `hexists key filed` 查看key中field是否存在
* `hkeys key` 查看key所有field
* `hvals key` 查看key所有value
* `hincrby key field 值` 给key中field的值+值
* `hsetnx key field value` 当且仅当key中field不存在时, 将key中field设置为value

## 有序集合Zset
相当于有序的Set,  

* `zadd key score1 value1 score2 value2... ` 将一个或多个score值加入到有序集合key中
* `zrange key index1 index2 [WITHSCORES]` 返回有续集key中下标index1-index2之间的元素, [WITHSCORES]选项同时显示score
* `zrangebyscore key score1 score2`获取key中score在[score1,score2]中的值
* `zrangebyscore key score1 score2` 同上, 顺序相反
* `zincrby key increment value` 为key中的value的score增加increment
* `zrem key value` 删除key中的value
* `zcount key min max` 统计score区间的元素个数
* `zrank key value` 返回该value在key中的排名

# 新数据类型

## 位图Bitmap

是由多个二进制位组成的数组，数组中的每个二进制位都有与之对应的偏移量（从 0 开始），通过这些偏移量可以对位图中指定的一个或多个二进制位进行操作。

位图并不是 Redis 提供的一种新的数据类型，它是字符串类型的扩展。所以位图的命令可以直接使用在字符串类型的键上，位图命令操作的键也可以被字符串类型命令操作。

应用场景:

* 用户行为记录器
  用用户 ID 作为偏移量，若用户做了某种行为则通过 SETBIT 将二进制位设置为 1，通过 GETBIT 判断用户是否做了某种行为，通过 BITCOUNT 可以知道有多少用户执行了行为。

* 用户上线统计
  可以使用 SETBIT 和 BITCOUNT 来实现，以用户 ID 作为 key ，假设今天是上线统计功能开放的第一天，ID 为 1 的用户上线后就通过 SETBIT 1 0 1。当要计算此用户的总共以来的上线次数时，使用 BITCOUNT 命令就可以得出的结果。



* `setbit key offset value` 设置key中偏移量为offset的位的值为value, 返回其旧值
* `getbit key offset` 
* `bitcount key [start end]`统计位图中指定下标内值为 1 的二进制位数量
* `bitpos key value [start end]` 在位图中查找第一个被设置为指定值的二进制位，返回其偏移量
* `bitop AND/OR/XOR/NOT destkey key [key ...]` 对一个或多个位图执行指定的二进制位运算，并将运算结果存储到指定的键中。

## HyperLogLog

Redis HyperLogLog 是用来做基数统计的算法，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基 数。HyperLogLog 只会根据输入元素来计算基数，不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。

* `pfadd key element [element ...]` 添加指定元素到 HyperLogLog 中。
* `pfcount key [key ...]` 返回给定 HyperLogLog 的基数估算值。
* `pfmerge destkey sourcekey [sourcekey ...]` 将多个 HyperLogLog 合并为一个 HyperLogLog

## Geo

GEO 主要用于存储地理位置信息，并对存储的信息进行操作.

- `GEOADD key longitude latitude member [longitude latitude member ...]`添加地理位置的坐标。
- `GEOPOS key member [member ...]`获取地理位置的坐标。
- `GEODIST key member1 member2 [m|km|ft|mi]`计算两个位置之间的距离。
- `GEORADIUS key longitude latitude radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] [ASC|DESC] [STORE key] [STOREDIST key] `根据用户给定的经纬度坐标来获取指定范围内的地理位置集合。
- `GEORADIUSBYMEMBER key member radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] [ASC|DESC] [STORE key] [STOREDIST key]` 根据储存在位置集合里面的某个地点获取指定范围内的地理位置集合。
  - WITHDIST: 在返回位置元素的同时， 将位置元素与中心之间的距离也一并返回。
  - WITHCOORD: 将位置元素的经度和纬度也一并返回。
  - WITHHASH: 以 52 位有符号整数的形式， 返回位置元素经过原始 geohash 编码的有序集合分值。 这个选项主要用于底层应用或者调试， 实际中的作用并不大。
  - COUNT 限定返回的记录数。
  - ASC: 查找结果根据距离从近到远排序。
  - DESC: 查找结果根据从远到近排序。
- `GEOHASH key member [member ...]`返回一个或多个位置对象的 geohash 值。

# 发布和订阅

Redis发布订阅是一种消息通信模式,  发布者(pub)向指定频道(channel)发送消息(message), 订阅者(sub)能接收所有已订阅频道中的消息.

发布订阅命令:

| 序号 | 命令及描述                                                   |
| :--- | :----------------------------------------------------------- |
| 1    | [PSUBSCRIBE pattern [pattern ...\]](https://www.runoob.com/redis/pub-sub-psubscribe.html) 订阅一个或多个符合给定模式的频道。 |
| 2    | [PUBSUB subcommand [argument [argument ...\]]](https://www.runoob.com/redis/pub-sub-pubsub.html) 查看订阅与发布系统状态。 |
| 3    | [PUBLISH channel message](https://www.runoob.com/redis/pub-sub-publish.html) 将信息发送到指定的频道。 |
| 4    | [PUNSUBSCRIBE [pattern [pattern ...\]]](https://www.runoob.com/redis/pub-sub-punsubscribe.html) 退订所有给定模式的频道。 |
| 5    | [SUBSCRIBE channel [channel ...\]](https://www.runoob.com/redis/pub-sub-subscribe.html) 订阅给定的一个或多个频道的信息。 |
| 6    | [UNSUBSCRIBE [channel [channel ...\]]](https://www.runoob.com/redis/pub-sub-unsubscribe.html) 指退订给定的频道。 |



# Jedis: Redis整合原生Java

maven依赖

```xml
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>4.1.0</version>
        </dependency>
```

```Java
public class RedisDemo {
    public static void main(String[] args){
        //创建一个jedis连接
        Jedis jedis = new Jedis("8.136.85.255",6379);
        //直接使用jedis.redis命令
        String res = jedis.ping();
        System.out.println(res);
    }
}

```

# Redis整合Spring boot

1. 依赖:

   ```xml
              <dependency>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-starter-data-redis</artifactId>
              </dependency>
              <dependency>
                  <groupId>org.apache.commons</groupId>
                  <artifactId>commons-pool2</artifactId>
                  <version>2.11.1</version>
              </dependency>
   ```

   

2. 配置文件

   ```yaml
   spring:
     redis:
       host: 8.136.85.255 # 服务器地址
       port: 6379 # 端口号
       database: 0 # 数据库索引
       connect-timeout: 20000 # 连接超时时间
       jedis:
         pool:
           max-active: 10 # 最大连接数
           max-wait: -1 # 最大阻塞等待时间
           max-idle: 5 # 最大空闲连接数
           min-idle: 0 #最小空闲连接数
   ```

3. 通过redis-starter提供的Template操纵redis
   ![opsFor](.\opsFor.png)    

    ```java
        @RestController
        public class RedisController {
            //redis-stater提供的模板类, 可直接操作redis
            @Autowired
            private RedisTemplate redisTemplate;
        
            @RequestMapping
            public Object testRedis(){
                //通过Template操纵redis
                return redisTemplate.opsForValue().get("k1");
            }
        }
    ```
   
    

# 事务

Redis 事务的本质是一组命令的集合。事务支持一次执行多个命令，一个事务中所有命令都会被序列化。在事务执行过程，会按照顺序串行化执行队列中的命令，其他客户端提交的命令请求不会插入到事务执行命令序列中。
总结说：redis事务就是一次性、顺序性、排他性的执行一个队列中的一系列命令。

Redis事务相关命令：
* MULTI ：开启事务，redis会将后续的命令逐个放入队列中，然后使用EXEC命令来原子化执行这个命令系列。
* EXEC：执行事务中的所有操作命令。
* DISCARD：取消事务，放弃执行事务块中的所有命令。
* WATCH：监视一个或多个key,如果事务在执行前，这个key(或多个key)被其他命令修改，则事务被中断，不会执行事务中的任何命令。
* UNWATCH：取消WATCH对所有key的监视。


# 持久化
## RDB

在指定的时间间隔内将内存中的数据集快照写入磁盘.

优势:

 * 适合大规模的数据恢复
 * 数据恢复快
 * 节省磁盘空间

劣势:

	* 无法做到实时持久化, 可能造成数据丢失. 
	* 不同版本Redis使用特定二进制格式保存rdb文件, 可能存在版本不兼容.

### 1. 自动触发

   在 redis.conf 配置文件中的 SNAPSHOTTING 下:
   **①、save：**这里是用来配置触发 Redis的 RDB 持久化条件，也就是什么时候将内存中的数据保存到硬盘。比如“save m n”。表示m秒内数据集存在n次修改时，自动触发bgsave（这个命令下面会介绍，手动触发RDB持久化的命令）

   　　默认如下配置：
   ```
   save 900 1：表示900 秒内如果至少有 1 个 key 的值变化，则保存
   save 300 10：表示300 秒内如果至少有 10 个 key 的值变化，则保存
   save 60 10000：表示60 秒内如果至少有 10000 个 key 的值变化，则保存
   ```

   　　　　如果你只是用Redis的缓存功能，不需要持久化，那么你可以注释掉所有的 save 行来停用保存功能。可以直接一个空字符串来实现停用：save ""

   　　**②、stop-writes-on-bgsave-error ：**默认值为yes。当启用了RDB且最后一次后台保存数据失败，Redis是否停止接收数据。这会让用户意识到数据没有正确持久化到磁盘上，否则没有人会注意到灾难（disaster）发生了。如果Redis重启了，那么又可以重新开始接收数据了
   　　**③、rdbcompression ；**默认值是yes。对于存储到磁盘中的快照，可以设置是否进行压缩存储。如果是的话，redis会采用LZF算法进行压缩。如果你不想消耗CPU来进行压缩的话，可以设置为关闭此功能，但是存储在磁盘上的快照会比较大。
   　　**④、rdbchecksum ：**默认值是yes。在存储快照后，我们还可以让redis使用CRC64算法来进行数据校验，但是这样做会增加大约10%的性能消耗，如果希望获取到最大的性能提升，可以关闭此功能。
   　　**⑤、dbfilename ：**设置快照的文件名，默认是 dump.rdb
   　　**⑥、dir：**设置快照文件的存放路径，这个配置项一定是个目录，而不能是文件名。默认是和当前配置文件保存在同一目录。

### 2. 手动触发

   * save
      　　该命令会阻塞当前Redis服务器，执行save命令期间，Redis不能处理其他命令，直到RDB过程完成为止。显然该命令对于内存比较大的实例会造成长时间阻塞，这是致命的缺陷，为了解决此问题，Redis提供了第二种方式。

   * bgsave
      　　执行该命令时，Redis会在后台异步进行快照操作，快照同时还可以响应客户端请求。具体操作是Redis进程执行fork操作创建子进程，RDB持久化过程由子进程负责，完成后自动结束。阻塞只发生在fork阶段，一般时间很短。

### 3. 数据恢复

 　将备份文件 (dump.rdb) 移动到 redis 安装目录并启动服务即可，redis就会自动加载文件数据至内存了。Redis 服务器在载入 RDB 文件期间，会一直处于阻塞状态，直到载入工作完成为止。

获取 redis 的安装目录可以使用 `config get dir` 命令

## AOF

以日志形式来记录每个写操作,  将Redis执行过程中所有的写指令记录下来, 只许追加文件不可以改写文件.

优点: 

*  提供多种同步频率, 数据丢失率低
* 采用命令追加构造持久化文件, 异常丢失易修正
* AOF文件格式可读性强,  可以手动修改AOF文件进行回滚操作.

缺点:

* AOF文件体积更大
* 同步频率高, 在Redis负载较高时性能较差
* EDB比AOF更健壮, AOF可能存在BUG

### 1. 触发

在 redis.conf 配置文件的 APPEND ONLY MODE 下：

​		①、**appendonly**：默认值为no，也就是说redis 默认使用的是rdb方式持久化，如果想要开启 AOF 持久化方式，需要将 appendonly 修改为 yes。
　　②、**appendfilename** ：aof文件名，默认是"appendonly.aof"
　　③、**appendfsync：**aof持久化策略的配置；
　　　　　　no表示不执行fsync，由操作系统保证数据同步到磁盘，速度最快，但是不太安全；
　　　　　　always表示每次写入都执行fsync，以保证数据同步到磁盘，效率很低；
　　　　　　everysec表示每秒执行一次fsync，可能会导致丢失这1s数据。通常选择 everysec ，兼顾安全性和效率。
　　④、**no-appendfsync-on-rewrite**：在aof重写或者写入rdb文件的时候，会执行大量IO，此时对于everysec和always的aof模式来说，执行fsync会造成阻塞过长时间，no-appendfsync-on-rewrite字段设置为默认设置为no。如果对延迟要求很高的应用，这个字段可以设置为yes，否则还是设置为no，这样对持久化特性来说这是更安全的选择。  设置为yes表示rewrite期间对新写操作不fsync,暂时存在内存中,等rewrite完成后再写入，默认为no，建议yes。Linux的默认fsync策略是30秒。可能丢失30秒数据。默认值为no。
　　⑤、**auto-aof-rewrite-percentage**：默认值为100。aof自动重写配置，当目前aof文件大小超过上一次重写的aof文件大小的百分之多少进行重写，即当aof文件增长到一定大小的时候，Redis能够调用bgrewriteaof对日志文件进行重写。当前AOF文件大小是上次日志重写得到AOF文件大小的二倍（设置为100）时，自动启动新的日志重写过程。
　　⑥、**auto-aof-rewrite-min-size**：64mb。设置允许重写的最小aof文件大小，避免了达到约定百分比但尺寸仍然很小的情况还要重写。
　　⑦、**aof-load-truncated**：aof文件可能在尾部是不完整的，当redis启动的时候，aof文件的数据被载入内存。重启可能发生在redis所在的主机操作系统宕机后，尤其在ext4文件系统没有加上data=ordered选项，出现这种现象 redis宕机或者异常终止不会造成尾部不完整现象，可以选择让redis退出，或者导入尽可能多的数据。如果选择的是yes，当截断的aof文件被导入的时候，会自动发布一个log给客户端然后load。如果是no，用户必须手动redis-check-aof修复AOF文件才可以。默认值为 yes。

### 2. 数据恢复

同RDB一致

### 3. 异常恢复

aof文件异常损坏, 可通过`/usr/local/bin/redis-check-aof --fix appendonly.aof`恢复

## RDB-AOF混合持久化

这里补充一个知识点，在Redis4.0之后又新增了RDB-AOF混合持久化方式。
　　这种方式结合了RDB和AOF的优点，既能快速加载又能避免丢失过多的数据。
　　具体配置为：
		`aof-use-rdb-preamble`设置为yes表示开启，设置为no表示禁用。
　　当开启混合持久化时，主进程先fork出子进程将现有内存副本全量以RDB方式写入aof文件中，然后将缓冲区中的增量命令以AOF方式写入aof文件中，写入完成后通知主进程更新相关信息，并将新的含有 RDB和AOF两种格式的aof文件替换旧的aof文件。
　　简单来说：混合持久化方式产生的文件一部分是RDB格式，一部分是AOF格式。
　　这种方式优点我们很好理解，缺点就是不能兼容Redis4.0之前版本的备份文件了。

# 主从复制

将一台Redis服务器的数据，复制到其他的Redis服务器。前者称为主节点(master/leader)，后者称为从节点(slave/follower) ; 数据的复制是单向的，只能由主节点到从节点。Master以写为主，Slave以读为主。

作用: 

* 读写分离
* 容灾快速恢复
* 负载均衡
* 高可用(集群、哨兵)基石

 ## 搭建

1. 启动多个redis服务
2. `slaveof [主机ip] [端口号] `将当前redis服务器设置为指定主机的从机

## 哨兵模式
哨兵模式是一种特殊的模式，首先Redis提供了哨兵的命令，哨兵是一个独立的进程，作为进程，它会独立运行。其原理是哨兵通过发送命令，等待Redis服务器响应，从而监控运行的多个Redis实例。

哨兵的两个作用

- 监控Redis(主服务器/从服务器)运行状态
- 当哨兵监测到master宕机，会自动将slave切换成master，然后通过**发布订阅模式**通知其他的从服务器，修改配置文件，让它们切换主机。

配置步骤:

1. 创建配置文件sentinel.conf

   ```
   sentinel monitor mymaster [主机ip] [端口号] [quorum]
   ```

   - **quorum** 是需要同意主节点不可用的Sentinels的数量

2. 启动哨兵

   ```shell
   redis-sentinel [配置文件目录]
   ```

   * 从服务器上线规则: 
     1. 优先级靠前的(slave-priority)
     2. 偏移量大的(数据复制完整度高的)
     3. runid(服务器运行id)最小的
   * 新服务器上线后原主服务器变成从服务器

# 集群

Redis 集群是 Redis 提供的分布式[数据库](https://cloud.tencent.com/solution/database?from=10680)方案，集群通过分片( sharding )来实现数据共享(水平扩容)，并提供复制和故障转移。

## 搭建

1. 修改配置文件

   ```
   port 8000 #端口号
   daemonize yes #后台模式
   cluster-enabled yes ##开启集群模式
   cluster-config-file nodes.conf #集群节点文件保存位置
   cluster-node-timeout 5000 #超时
   ```

2.  分别启动服务器

3.  创建集群

   在src目录下
   
   ```shell
   ./redis-cli --cluster create --cluster-replicas 1 [ip1:port1...]
   ```
   
   * --replicas 1 表示采用最简单的方式配置集群, 即一台主机, 一台从机



4. 连接

   采用集群策略连接

   ```
   redis-cli -p -c [端口号]
   ```

5. 分配原则

   尽量保证每个主库不在一个ip, 每个主库和从库不在一个ip上

## slot

Redis 集群中内置了 16384 个哈希槽(分配到每台主服务器上,类似于一致性Hash)，当需要在 Redis 集群中放置一个 key-value时，redis 先对 key（有效值）使用 crc16 算法算出一个结果，然后把结果对 16384 求余数，这样每个 key 都会对应一个编号在 0-16383 之间的哈希槽，redis 会根据节点数量大致均等的将哈希槽映射到不同的节点。

* 集群mset添加多值
  直接使用mset插入多值, 会导致每个key计算出不同的hash值,  解决办法是分组: 每个key后面加{组名}, 根据组名计算hash值得到槽位.

![slot-error](.\slot-error.png)

* 查看key的槽位 `cluster keyslot key`
* 查看槽位中key数量 `countkeysinsolt 槽位`
* redis.conf中的配置参数: cluster-require-full-coverage
  - yes:  某个服务器主从全部宕机, 整个集群挂掉
  - no:  某个服务器主从全部宕机, 仅该服务器全部槽无法使用



# 应用问题

## 1. 缓存穿透

大量请求穿过redis保护, 通过redis请求缓存中和数据库中都不存在的数据, 导致服务器负载急剧上升. 

解决:

* 设置黑名单, 拦截用户请求
* 用布隆过滤器判断请求数据是否存在
* 监控服务器指标, 当指标出现异常通知维护人员

## 2. 缓存击穿

redis中某个key过期, 仍有大量请求访问该key, 导致数据库负载上升.

解决:

* 预先设置热门数据(设置热门数据有效期为永久)
* 适时调整, 监控热词, 延长有效期
* 使用锁,  当缓存失效时只有拿到锁才可以去访问数据库
* 软过期, 由业务逻辑判断key是否失效

## 3. 缓存雪崩

在某个时间节点，大量的 key 失效，导致大量的请求从缓存中获取不到数据而去请求数据库。

解决:

* 构建多级缓存: nginx+redis+其他缓存
* 使用锁或者队列(不适合高并发场景)
* 设置过期标志更新缓存(设置过期提前量, 让其他线程更新即将过期的缓存)
* 分散缓存失效时间

## 4. 分布式锁

在分布式模型下, 需要使用分布式锁实现对某一资源的并发访问.

1. redis实现锁和设置锁的过期时间:

```
方式一: 
	setxn key value
	expire key [时间]
方式二:
	set key value ex ex [时间]
        ex : 将键的过期时间设置为 seconds 秒，与SETEX key seconds value效果等同
        px : 将键的过期时间设置为 milliseconds 秒，与PSETEX key milliseconds value效果等同
        nx : 只在键不存在时， 才对键进行设置操作，默认false
        px : 只在键已经存在时， 才对键进行设置操作，默认false

```

2. 使用uuid防止误删

   不同进程设置的分布式锁可能被其他进程误删, 导致锁异常失效. 

   解决: 使用uuid作为每个进程上的锁的value, 每次删除锁之前比较uuid是否一致, 才有权限释放锁.

3. 没有原子性操作造成的问题

   解决: 使用lus脚本实现原子性操作

# Redis6更新

1. ACL
2. IO多线程
3. Cluster工具支持



