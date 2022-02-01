# reids

## 基本概念

sharding：分片

replication：复制

## 安装

```sh
$reidis_home/make MALLOC=libc
$make
```
### 单机
1. 启动
```sh
./redis-server ../redis.conf
```
2. 命令
```sh
#./redis-cli KEYS "SITE_ALARM_PERSON_PREFIX*"|xargs ./redis-cli DEL
#./redis-cli KEYS "*"
```
3. save

```sh
#redis 127.0.0.1:6379> SAVE 
会产生如下文件：/home/docker/dump.rdb
#redis 127.0.0.1:6379> BGSAVE
#>CONFIG GET maxclients
```
4. 恢复数据
如果需要恢复数据，只需将备份文件 (dump.rdb) 移动到 redis 安装目录并启动服务即可。获取 redis 目录可以使用 CONFIG 命令，如下所示：
```sh
#redis 127.0.0.1:6379> CONFIG GET dir
		1) "dir"
		2) "/usr/local/redis/bin"
$./redis-cli config get maxclients
$./redis-cli config get "*"
redis :6379 > shutdown
关闭服务# redis-cli shutdown 
```
###  配置存储路径
	$vi $redis_home/redis.conf
	dir /cloud/redis
### 配置日志
	dir /cloud/reids.log
### 刷写数据
	$./redis-cli save
### 清理数据

## 命令

### 基础命令

| 命令                             | 作用                                            | 举例                    |
| -------------------------------- | ----------------------------------------------- | ----------------------- |
| **select** index                 | 选择redis数据库                                 | select 1                |
| **dbsize**                       | 返回当前数据库的 key 的数量                     | dbsize                  |
| **del** key1 key2                | 删除指定key，返回删除数量                       | del key                 |
| **flushdb** [ASYNC]              | 清空当前数据库中的所有 key                      | flushdb                 |
| **flushall**[ASYNC]              | 清空整个 Redis 服务器的数据                     | flushall                |
| **keys** pattern                 | 查看当前数据库中的所有key                       | keys *                  |
| **scan** cursor match count type | 增量式迭代命令                                  | scan 0 match * count 20 |
| **exists** key1 key2             | 检查给定 key 是否存在。不存在:0                 | exists a b c            |
| **expire** key seconds           | 为给定 key 设置生存时间                         | expire a 5              |
| **ttl** key                      | 返回给定 key 的剩余生存时间                     | ttl a                   |
| **persist** key                  | 移除给定 key 的生存时间                         | persist a               |
| **type** key                     | 返回 key 所储存的值的类型                       | type a                  |
| **move** key db                  | 将当前数据库的 key 移动到给定的数据库 db 当中。 | move a 1                |

### String相关命令

**特点**

1. 值得大小不超过512m
2. 推荐key命名规则object：id：field，方便在redis桌面管理中查看

| 命令                             | 作用                                                | 举例             |
| -------------------------------- | --------------------------------------------------- | ---------------- |
| **set** key value [NX PX][NX XX] | 将字符串值 value 关联到 key 。                      | set a 1          |
| **mset** k1 v1 k2 v2             | 同时为多个键设置值。(原子)                          | mset a 1 b 2     |
| **msetnx** k1 v1 k2 v2           | 当且仅当所有给定键都不存在时， 为所有给定键设置值。 | msetnx a 1 b 2   |
| **setRange** offset key value    | 从offset开始，用value参数覆写key储存的字符串值。    | setrange a 1 abc |
| **get**                          | 返回与键 key 相关联的字符串值                       | get a            |
| **getset** key value             | 将键 key 的值设为新value,返回旧value，不存在返回nil | getset a 2       |
| **strlen** key                   | 返回键 key 储存的字符串值的长度                     | strlen a         |
| **getRange** key start end       | 返回键 key 储存的字符串值的指定部分                 | getrange a 0 -1  |
| **incr** key                     | 对应键值自增加+1                                    | incr a           |
| **incrby** key increment         | 对应键值自增加+increment                            | incr a 3         |
| **decr** key                     | 对应键值自减1                                       | decr a           |
| **decrby** key decrement         | 对应键值自减                                        | decrby a 1       |
| **append** key value             | 在key末尾添加value                                  | append a abc     |
| **rename** key newkey            | 将 key 改名为 newkey                                | rename a b       |

### List相关命令

**特点**

1. 底层实现为LinkedList链表
2. list 最多可以有 2^32 - 1 个元素

| 命令                                        | 作用                                                       | 举例                      |
| ------------------------------------------- | ---------------------------------------------------------- | ------------------------- |
| **lpush** key value…                        | 将一个或多个值 value 插入到列表 key 的表头                 | lpush alist a b c         |
| **lpushx** key value                        | 当列表存在，将值 value 插入到列表 key 的表头               | lpushx alist d            |
| **rpushx** key value                        | 当列表存在，将值 value 插入到列表 key 的表尾               | rpushx alist d            |
| **lpop** key                                | 移除并返回列表 key 的头元素                                | lpop alist                |
| **rpop** key                                | 移除并返回列表 key 的尾元素。                              | rpop alist                |
| **rpoplpush** source destination            | 将source尾部元素添加到destination头部，返回source尾部      | rpoplpush alist blist     |
| **lrem** key count value                    | 删除列表中与value值相同的元素，count表示删除个数和遍历位置 | lpush alist a b c         |
| **linsert** key [before\|after] pivot value | 在列表中pivot元素前后插入value值                           | linsert alist before b a  |
| **lset** key index value                    | 将列表index位置上的元素改为value值                         | lset alist 0 b            |
| **ltrim** key start stop                    | 将列表修剪为start-stop范围                                 | ltrim alist 0 10          |
| **blpop** key… timeout                      | 阻塞式弹出第一个不为空列表的表头                           | blpop alist blist 10      |
| **brpop** key… timeout                      | 阻塞式弹出第一个不为空列表的表尾                           | brpop alist blist 10      |
| **brpoplpush** source destination timeout   | 阻塞式弹出表头插入表尾                                     | brpoplpush alist blist 10 |

### SET相关命令

**特点**

1. 一个无序的字符串集合，其中元素不重复
2. set中最多可以有 2^32 - 1 个元素

| 命令                                | 作用                                            | 举例                       |
| ----------------------------------- | ----------------------------------------------- | -------------------------- |
| **sadd** key member…                | 将一个或多个 member 元素加入到集合 key 当中     | sadd aset a b c            |
| **smove** source destination member | 将member从source集合移动到destination集合       | smove aset bset d          |
| **spop** key count                  | 移除并返回集合中的count个随机元素               | spop aset 2                |
| **srem** key member…                | 移除集合 key 中的一个或多个 member 元素         | srem aset d j              |
| **srandmember** key count           | 随机返回count个元素                             | srandmember aset 2         |
| **smembers** key                    | 返回集合 key 中的所有成员                       | smembers aset              |
| **scard** key                       | 返回集合中元素的数量                            | scard aset                 |
| **sismember** key member            | 判断 member 元素是否集合 key 的成员             | sismember aset a           |
| **sdiff** key…                      | 返回所有给定集合之间的差集                      | sdiff aset bset            |
| **sdiffstore** destination key…     | 返回所有给定集合之间的差集并存储在destination中 | sdiffstore cset aset bset  |
| **sinter** key…                     | 返回所有给定集合的交集                          | sinter aset bset           |
| **sinterstore** destination key…    | 返回所有给定集合的交集并存储在destination中     | sinterstore cset aset bset |
| **sunion** key…                     | 返回所有给定集合的并集                          | sunion aset bset           |
| **sunionstore** destination key…    | 返回所有给定集合的并集并存储在destination中     | sunionstore cset aset bset |

### HASH相关命令

**特点**

1. hash中最多可以有 2^32 - 1 个key/value 键值对

| 命令                                 | 作用                                                         | 举例                          |
| ------------------------------------ | ------------------------------------------------------------ | ----------------------------- |
| **hset** key field value             | 将哈希表 hash 中域 field 的值设置为 value                    | hset ahash name king          |
| **hsetnx** key field value           | 当且仅当域 field 尚未存在于哈希表的情况下， 将它的值设置为 value | hsetnx ahash name king        |
| **hmset** key (field value)…         | 同时将多个 field-value (域-值)对设置到哈希表 key 中          | hmset ahash name king age 10  |
| **hget** key field                   | 返回哈希表中给定域的值                                       | hget ahash name               |
| **hmget** key field…                 | 返回哈希表 key 中，一个或多个给定域的值                      | hmget ahash name age          |
| **hgetall** key                      | 返回哈希表 key 中，所有的域和值                              | hgetall ahash                 |
| **hdel** key field…                  | 删除指定key中的域值                                          | hdel ahash name age           |
| **hincrby** key field increment      | 为哈希表 key 中的域 field 的值加上增量 increment             | hincrby ahash age 2           |
| **hincrbyfloat** key field increment | 为哈希表 key 中的域 field 加上浮点数增量 increment           | hincrbyfloat ahash height 2.3 |
| **hkeys** key                        | 返回哈希表 key 中的所有域                                    | hkeys ahash                   |
| **hvals** key                        | 返回哈希表 key 中所有域的值。                                | hvals ahash                   |
| **hexists** key field                | 判断指定key的域是否存在                                      | hexists ahash name            |
| **hlen** key                         | 返回哈希表 key 中域的数量                                    | hlenahash age                 |
| **hstrlen** key field                | 返回哈希表 key 中， 与给定域 field 相关联的值的字符串长度    | hstrlen ahash height          |

#### Sorted Set相关命令

| 命令                                     | 作用                                                         | 举例                            |
| ---------------------------------------- | ------------------------------------------------------------ | ------------------------------- |
| **zadd** [NX\|XX] (score member)…        | 将一个或多个 member 元素及其 score 值加入到有序集 key 当中。 | zadd azset 1 name 2 king        |
| **zrem** key member…                     | 移除有序集 key 中的一个或多个成员，不存在的成员将被忽略。    | zrem azset name king            |
| **zrange** key start stop withscores     | 返回有序集 key 中，指定区间内的成员。                        | zrange azset 0 -1               |
| **zcard** key                            | 返回有序集 key 的基数                                        | zcard azset                     |
| **zcount** key min max                   | 返回有序集 key 中，score 值在 min 和 max 之间的成员的数量。  | zcount azset 0 2                |
| **zscore** key member                    | 返回有序集 key 中，成员 member 的 score 值                   | zscore azset name               |
| **zrank** key member                     | 返回有序集 key 中成员 member 的排名                          | zrank azset name                |
| **zrevrank** key member                  | 集合按照score降序排列，返回有序集 key 中成员 member 的排名   | zrevrank azset name             |
| **zincrby** key increment member         | 为有序集 key 的成员 member 的 score 值加上增量 increment     | zincrby azset 2 name            |
| **zinterstore** destination numkeys key… | 计算给定的一个或多个有序集的交集，存储到destination          | zinterstore czset 2 azset bzset |
| **zunionstore** destination numkeys key… | 计算给定的一个或多个有序集的并集，存储到destination          | zunionstore czset 2 azset bzset |

#### HyperLogLog相关命令

**特点**

1. 用于统计基数，牺牲准确性来节省空间

| 命令                           | 作用                                               | 举例                    |
| ------------------------------ | -------------------------------------------------- | ----------------------- |
| **pfadd** key element…         | 将任意数量的元素添加到指定的 HyperLogLog 里面。    | pfadd alog a b          |
| **pfcount** key…               | 返回所有给定 HyperLogLog 的并集的近似基数          | pfcount alog blog       |
| **pfmerge** destkey sourcekey… | 将多个 HyperLogLog 合并（merge）为一个 HyperLogLog | pfmerge clog blog ablog |

#### BitMap相关命令

**特点**

1. 定义在字符串类型上的面向位的操作的集合

| 命令                                                         | 作用                                                     | 举例                     |
| ------------------------------------------------------------ | -------------------------------------------------------- | ------------------------ |
| **setbit** key offset value                                  | 对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit) | setbit abit 0 1          |
| **getbit** key offset                                        | 对 key 所储存的字符串值，获取指定偏移量上的位(bit)       | getbit abit 0            |
| **bitcount** key start end                                   | 计算给定字符串中，被设置为 1 的比特位的数量。            | bitcount abit 0 1        |
| **bitpos** key bit start end                                 | 返回位图中第一个值为 bit 的二进制位的位置。              | bitpos abit 0 0 2        |
| **bitop** operation destkey key…                             | 对一个或多个 key 进行逻辑运算，并将结果保存到 destkey    | bitop and cbit abit bbit |
| **bitfield** key [get type offset][incrby type offset increment] | 接受一系列待执行的操作作为参数， 并返回一个数组作为回复  | bitfield abit get i8 100 |

#### 事务相关命令

| 命令           | 作用                                                         | 举例      |
| -------------- | ------------------------------------------------------------ | --------- |
| **multi**      | 标记一个事务块的开始。                                       | multi     |
| **exec**       | 执行所有事务块内的命令                                       | exec      |
| **discard**    | 取消事务，放弃执行事务块内的所有命令。                       | discard   |
| **watch** key… | 监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。 | watch a b |
| **unwatch**    | 取消 WATCH 命令对所有 key 的监视。                           | unwatch   |

#### 持久化相关命令

| 命令             | 作用                                                         | 举例         |
| ---------------- | ------------------------------------------------------------ | ------------ |
| **save**         | 将当前 Redis 实例的所有数据快照(snapshot)以 RDB 文件的形式保存到硬盘。 | save         |
| **bgsave**       | 在后台异步(Asynchronously)保存当前数据库的数据到磁盘。       | exec         |
| **bgrewriteaof** | 执行一个 AOF文件 重写操作。                                  | bgrewriteaof |
| **lastsave**     | 返回最近一次 Redis 成功将数据保存到磁盘上的时间              | lastsave     |



## 主从复制

### 作用

1. 数据冗余：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。

2. 故障恢复：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复；实际上是一种服务的冗余。

3. 负载均衡：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务（即写Redis数据时应用连接主节点，读Redis数据时应用连接从节点），分担服务器负载；尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高Redis服务器的并发量。

4. 读写分离：可以用于实现读写分离，主库写、从库读，读写分离不仅可以提高服务器的负载能力，同时可根据需求的变化，改变从库的数量；

5. 高可用基石：除了上述作用以外，主从复制还是哨兵和集群能够实施的基础，因此说主从复制是Redis高可用的基础。



## 切片

### 三种切片方式

#### 客户端实现数据分片 

即客户端自己计算数据的key应该在哪个机器上存储和查找，此方法的好处是降低了服务器集群的复杂度，客户端实现数据分片时，服务器是独立的，服务器之前没有任何关联。多数redis客户端库实现了此功能，也叫**sharding**,这种方式的缺点是客户端需要实时知道当前集群节点的联系信息，同时，当添加一个新的节点时，客户端要支持动态sharding.，多数客户端实现不支持此功能，需要重启redis。另一个弊端是redis的HA需要额外考虑。

 

#### 服务器实现数据分片 

其理论是，客户端随意与集群中的任何节点通信，服务器端负责计算某个key在哪个机器上，当客户端访问某台机器时，服务器计算对应的key应该存储在哪个机器，然后把结果返回给客户端，客户端再去对应的节点操作key，是一个重定向的过程，此方式是redis3.0正在实现，目前处于beta版本， Redis 3.0的集群同时支持HA功能，某个master节点挂了后，其slave会自动接管。

#### 通过代理服务器实现数据分片 

此方式是借助一个代理服务器实现数据分片，客户端直接与proxy联系，proxy计算集群节点信息，并把请求发送到对应的集群节点。降低了客户端的复杂度，需要proxy收集集群节点信息。Twemproxy是twitter开源的，实现这一功能的proxy。这个实现方式在客户端和服务器之间加了一个proxy，但这是在redis 3.0稳定版本出来之前官方推荐的方式。结合redis-sentinel的HA方案，是个不错的组合。

### redis cluster

![img](IRedis.assets/v2-d7623e8b34eea72c0651dc4158954b88_1440w.jpg)

Redis Cluster要求至少需要3个master才能组成一个集群，同时每个master至少需要有一个slave节点。各个节点之间保持TCP通信。当master发生了宕机， Redis Cluster自动会将对应的slave节点提拔为master，来重新对外提供服务。

Redis Cluster 功能 ： **负载均衡，故障切换**，**主从复制** 。

#### 安装



#### 数据切片和实例的对应分布关系

从 3.0 开始，官方提供了一个名为 Redis Cluster 的方案，用于实现切片集群。Redis Cluster 方案中就规定了数据和实例的对应规则。

#### hash slot

Redis Cluster 方案采用哈希槽（Hash Slot，接下来我会直接称之为  Slot），来处理数据和实例之间的映射关系。在 Redis Cluster 方案中，一个切片集群共有 16384  个哈希槽，这些哈希槽类似于数据分区，每个键值对都会根据它的 key，被映射到一个哈希槽中。

#### 步骤

1. 首先根据键值对的 key，按照CRC16 算法计算一个 16 bit 的值；然后，再用这个 16bit 值对 16384 取模，得到 0~16383 范围内的模数，每个模数代表一个相应编号的哈希槽。
2. 在部署 Redis Cluster 方案时，使用 cluster create 命令创建集群，此时，Redis  会自动把这些槽平均分布在集群实例上。例如，如果集群中有 N 个实例，那么，每个实例上的槽个数为 16384/N 个。当然， 我们也可以使用  cluster meet 命令手动建立实例间的连接，形成集群，再使用 cluster addslots 命令，指定每个实例上的哈希槽个数。**在手动分配哈希槽时，需要把 16384 个槽都分配完，否则 Redis 集群无法正常工作。**

#### 客户端如何定位数据？

一般来说，客户端和集群实例建立连接后，实例就会把哈希槽的分配信息发给客户端。但是，在集群刚刚创建的时候，每个实例只知道自己被分配了哪些哈希槽，是不知道其他实例拥有的哈希槽信息的。

那么，客户端为什么可以在访问任何一个实例时，都能获得所有的哈希槽信息呢？这是因为，Redis 实例会把自己的哈希槽信息发给和它相连接的其它实例，来完成哈希槽分配信息的扩散。当实例之间相互连接后，每个实例就有所有哈希槽的映射关系了。

客户端收到哈希槽信息后，会把哈希槽信息缓存在本地。当客户端请求键值对时，会先计算键所对应的哈希槽，然后就可以给相应的实例发送请求了。

在集群中，实例和哈希槽的对应关系并不是一成不变的，最常见的变化有两个：

1. 在集群中，实例有新增或删除，Redis 需要重新分配哈希槽；
2. 为了负载均衡，Redis 需要把哈希槽在所有实例上重新分布一遍。

此时，实例之间还可以通过相互传递消息，获得最新的哈希槽分配信息，但是，客户端是无法主动感知这些变化的。这就会导致，它缓存的分配信息和最新的分配信息就不一致了，那该怎么办呢？Redis Cluster  方案提供了一种重定向机制，所谓的“重定向”，就是指，当客户端把一个键值对的操作请求发给一个实例时，如果这个实例上并没有这个键值对映射的哈希槽，那么，这个实例就会给客户端返回**MOVED 命令**响应结果，这个结果中就包含了新实例的访问地址。

如果两个实例之间正在进行数据迁移，此时客户端请求读取数据，则会接收到一条ASK报错信息。**ASK命令**表示两层含义：第一，表明 Slot 数据还在迁移中；第二，ASK 命令把客户端所请求数据的最新实例地址返回给客户端，此时，客户端需要给新实例发送 **ASKING命令**，然后再发送操作命令。ASKING命令让这个实例允许执行客户端接下来发送的命令。

