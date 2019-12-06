---
title: Redis
date: 2019-12-06 17:02:36
categories:
    - 后端
tags: 
    - Redis
---

Redis is an open source (BSD licensed), in-memory data structure store, used as a database, cache and message broker.

## 缓存中间件 [Memcache](https://www.memcached.org/)和[Redis](https://redis.io/)的区别

- Memcache：代码层次类似Hash
  - 支持简单数据类型
  - 不支持数据持久化存储
  - 不支持主从
  - 不支持分片
- Redis：
  - 数据类型丰富
    - string（字符串），hash（哈希），list（列表），set（集合）及zset(sorted set：有序集合)。
  - 支持数据持久化存储
  - 支持主从
  - 支持分片

## 基本概念

### 基本命令

启动服务器：

```
redis-server.exe redis.windows.conf
```

连接服务器：

```
redis-cli.exe -h 127.0.0.1 -p 6379
```

键值对操作：

```
set myKey abc
get myKey
DEL myKey
```

### 配置文件

Redis 的配置文件位于 Redis 安装目录下，文件名为 redis.conf(Windows 名为 redis.windows.conf)。

#### 查看配置

Redis CONFIG 命令格式如下:`CONFIG GET CONFIG_SETTING_NAME`

```
CONFIG GET loglevel
```

#### 编辑配置

CONFIG SET 命令基本语法：`CONFIG SET CONFIG_SETTING_NAME NEW_CONFIG_VALUE`

```
CONFIG SET loglevel "notice"
```

### Q & A

#### Q: 从海量Key里查询出某一固定前缀的Key

- 摸清数据规模，问清楚边界

使用指令 `KEYS pattern` 可以查找所有符合给定模式pattern的key

```
KEYS pattern
KEYS tes*
```

缺点：
- KEYS指令一次性返回所有匹配的key可能会因为key数量过大使服务器卡顿

解决：使用SCAN指令无阻塞提取

```
SCAN cursor [MATCH pattern] [COUNT count]
```

- SCAN指令使用到了一个基于游标的迭代器，需要基于上一次的游标来延续之前的迭代过程
- 以0作为游标的开始来开启新的迭代，直到命令返回游标0完成一次遍历

缺点：一次返回的数量不可控，只能是大概率符合count参数。可能会返回重复的key，可以使用Set来接返回的数据

从0开始迭代匹配`tes*`的key，一次返回10条结果，把上一次返回的结果中的位置设置成下一次的cursor

```
SCAN 0 match tes* count 10
```

#### Q: 使用Redis实现分布式锁

特性：
- 互斥性：任意时刻只能有一个客户端获取锁
- 安全性：锁只能被持有该锁的客户端删除
- 死锁：若获取锁的客户端宕机则可能会导致死锁
- 容错：部分redis结点宕机时，客户端仍要能够获取锁和释放锁

可以使用指令SETNX key value: 如果key不存在，则创建并赋值

- 时间复杂度O(1)
- 返回值：1:成功, 0:失败

```
setnx locknx test
```

客户端可以先set locknx然后执行对应操作，此时如果有其它客户端想要获得锁会set不成功

缺点：存在长期有效问题，即不会过期

解决方法：使用有过期时间的指令：EXPIRE key seconds

```
setnx locknx test
expire locknx 2
```

缺点：若程序执行完setnx locknx就挂了，就不会设置超时时间也不会释放锁

解决方法：使用指令 SET key value [EX seconds] [PX milliseconds] [NX|XX]

- EX second: 设置键的过期时间为 second秒
- PX millisecond: 设置键的过期时间为 millisecond毫秒
- NX: 只在键不存在时，才对键进行设置操作
- XX: 只在键已经存在时，才对键进行设置操作
- SET操作完成时，返回OK，否则返回nil

```
set locktarget 12345 ex 10 nx
```

#### Q: 大量key同时过期的注意事项

大量key集中过期时，由于清除大量的key很耗时，会出现短暂卡顿现象

解决方案：
- 在设置key过期时间的时候加一个随机偏移时间

#### Q: 如何使用Redis做异步队列

使用List作为队列，RPUSH生产消息，LPOP消费消息

```
rpush testlist aaa
rpush testlist bbb
rpush testlist ccc
lpop testlist
lpop testlist
```

缺点：
- 没有等待队列里有值就直接消费

解决：
1. 通过在应用层引入Sleep机制去调用LPOP重试
2. 或使用指令BLPOP key [key...] timeout: 阻塞直到队列有消息或超时
  - 缺点：只能供一个消费者消费
    - 解决：使用pub/sub主题订阅者模式
      - 缺点：消息的发布是无状态的，无法保证可达

## 数据类型

Redis支持五种数据类型：string（字符串），hash（哈希），list（列表），set（集合）及zset(sorted set：有序集合)。

Redis底层数据类型基础：
- 简单动态字符串
- 链表
- 字典
- 跳跃表
- 数据集合
- 压缩列表
- 对象

### String

string 类型是二进制安全的(可以包含任何数据，比如jpg)。意思是 redis 的 string 可以包含任何数据。比如jpg图片或者序列化的对象。

string 类型是 Redis 最基本的数据类型，string 类型的值最大能存储 512MB。

```
SET chinese "中文"
get chinese
```

### Hash

Redis hash 是一个键值(key=>value)对集合。

Redis hash 是一个 string 类型的 field 和 value 的映射表，hash 特别适合用于存储对象。

HMSET 设置了两个 field=>value 对, HGET 获取对应 field 对应的 value。每个 hash 可以存储 232 -1 键值对（40多亿）。

```
HMSET test field1 "Hello" field2 "World"
HGET test field1
HGET test field2
```

### List

Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。列表最多可存储 232 - 1 元素 (4294967295, 每个列表可存储40多亿)

```
lpush testList redis
lpush testList mongodb
lpush testList rabitmq
lrange testList 0 10
```

### Set

集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。集合中最大的成员数为 232 - 1(4294967295, 每个集合可存储40多亿个成员)。

添加一个 string 元素到 key 对应的 set 集合中，成功返回1，如果元素已经在集合中返回 0，如果 key 对应的 set 不存在则返回错误。

```
sadd testSet redis
sadd testSet mongodb
sadd testSet rabitmq
sadd testSet rabitmq
smembers testSet
```

### zset(sorted set：有序集合)

不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。zset的成员是唯一的,但分数(score)却可以重复。

```
zadd testZset 0 redis
zadd testZset 0.3 mongodb
zadd testZset 0.1 rabitmq
zadd testZset 9.9 solace
ZRANGEBYSCORE testZset 0 1000
```

### 各数据结构的应用场景

<table class="reference">
<thead><tr>
<th>类型</th>
<th>简介</th>
<th>特性</th>
<th>场景</th>
</tr></thead>
<tbody>
<tr>
<td>String(字符串)</td>
<td>二进制安全</td>
<td>可以包含任何数据,比如jpg图片或者序列化的对象,一个键最大能存储512M</td>
<td>---</td>
</tr>
<tr>
<td>Hash(字典)</td>
<td>键值对集合,即编程语言中的Map类型</td>
<td>适合存储对象,并且可以像数据库中update一个属性一样只修改某一项属性值(Memcached中需要取出整个字符串反序列化成对象修改完再序列化存回去)</td>
<td>存储、读取、修改用户属性</td>
</tr>
<tr>
<td>List(列表)</td>
<td>链表(双向链表)</td>
<td>增删快,提供了操作某一段元素的API</td>
<td>1,最新消息排行等功能(比如朋友圈的时间线) 2,消息队列</td>
</tr>
<tr>
<td>Set(集合)</td>
<td>哈希表实现,元素不重复</td>
<td>1、添加、删除,查找的复杂度都是O(1) 2、为集合提供了求交集、并集、差集等操作</td>
<td>1、共同好友 2、利用唯一性,统计访问网站的所有独立ip 3、好友推荐时,根据tag求交集,大于某个阈值就可以推荐</td>
</tr>
<tr>
<td>Sorted Set(有序集合)</td>
<td>将Set中的元素增加一个权重参数score,元素按score有序排列</td>
<td>数据插入集合时,已经进行天然排序</td>
<td>1、排行榜 2、带权重的消息队列</td>
</tr>
</tbody>
</table>

## Redis 命令

启动 redis 客户端：

```
redis-cli
```

```
PING
```

连接到主机为 127.0.0.1，端口为 6379 ，密码为 mypass 的 redis 服务上。

```
redis-cli -h host -p port -a password
redis-cli -h 127.0.0.1 -p 6379 -a "mypass"
```

### Redis keys 命令

1. DEL key
  - 该命令用于在 key 存在时删除 key。
2. DUMP key 
  - 序列化给定 key ，并返回被序列化的值。
3. EXISTS key 
  - 检查给定 key 是否存在。
4. EXPIRE key seconds
  - 为给定 key 设置过期时间，以秒计。
5. EXPIREAT key timestamp 
  - EXPIREAT 的作用和 EXPIRE 类似，都用于为 key 设置过期时间。 不同在于 EXPIREAT 命令接受的时间参数是 UNIX 时间戳(unix timestamp)。
6. PEXPIRE key milliseconds 
  - 设置 key 的过期时间以毫秒计。
7. PEXPIREAT key milliseconds-timestamp 
  - 设置 key 过期时间的时间戳(unix timestamp) 以毫秒计
8. KEYS pattern 
  - 查找所有符合给定模式( pattern)的 key 。
9. MOVE key db 
  - 将当前数据库的 key 移动到给定的数据库 db 当中。
10.	PERSIST key 
  - 移除 key 的过期时间，key 将持久保持。
11.	PTTL key 
  - 以毫秒为单位返回 key 的剩余的过期时间。
12.	TTL key 
  - 以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live)。
13.	RANDOMKEY 
  - 从当前数据库中随机返回一个 key 。
14.	RENAME key newkey 
  - 修改 key 的名称
15.	RENAMENX key newkey 
  - 仅当 newkey 不存在时，将 key 改名为 newkey 。
16.	TYPE key 
  - 返回 key 所储存的值的类型。

### Redis 字符串命令

1.	SET key value 
  - 置指定 key 的值
2.	GET key 
  - 取指定 key 的值。
3.	GETRANGE key start end 
  - 回 key 中字符串值的子字符
4.	GETSET key value
  - 给定 key 的值设为 value ，并返回 key 的旧值(old value)。
5.	GETBIT key offset
  -  key 所储存的字符串值，获取指定偏移量上的位(bit)。
6.	MGET key1 [key2..]
  - 取所有(一个或多个)给定 key 的值。
7.	SETBIT key offset value
  -  key 所储存的字符串值，设置或清除指定偏移量上的位(bit)。
8.	SETEX key seconds value
  - 值 value 关联到 key ，并将 key 的过期时间设为 seconds (以秒为单位)。
9.	SETNX key value
  - 有在 key 不存在时设置 key 的值。
10.	SETRANGE key offset value
  -  value 参数覆写给定 key 所储存的字符串值，从偏移量 offset 开始。
11.	STRLEN key
  - 回 key 所储存的字符串值的长度。
12.	MSET key value [key value ...]
  - 时设置一个或多个 key-value 对。
13.	MSETNX key value [key value ...] 
  - 时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。
14.	PSETEX key milliseconds value
  - 个命令和 SETEX 命令相似，但它以毫秒为单位设置 key 的生存时间，而不是像 SETEX 命令那样，以秒为单位。
15.	INCR key
  -  key 中储存的数字值增一。
16.	INCRBY key increment
  -  key 所储存的值加上给定的增量值（increment） 。
17.	INCRBYFLOAT key increment
  -  key 所储存的值加上给定的浮点增量值（increment） 。
18.	DECR key
  -  key 中储存的数字值减一。
19.	DECRBY key decrement
  -  key 所储存的值减去给定的减量值（decrement） 。
20.	APPEND key value
  - 果 key 已经存在并且是一个字符串， APPEND 命令将指定的 value 追加到该 key 原来值（value）的末尾。

### Redis hash 命令

1.	HDEL key field1 [field2] 
  - 除一个或多个哈希表字段
2.	HEXISTS key field 
  - 看哈希表 key 中，指定的字段是否存在。
3.	HGET key field 
  - 取存储在哈希表中指定字段的值。
4.	HGETALL key 
  - 取在哈希表中指定 key 的所有字段和值
5.	HINCRBY key field increment 
  - 哈希表 key 中的指定字段的整数值加上增量 increment 。
6.	HINCRBYFLOAT key field increment 
  - 哈希表 key 中的指定字段的浮点数值加上增量 increment 。
7.	HKEYS key 
  - 取所有哈希表中的字段
8.	HLEN key 
  - 取哈希表中字段的数量
9.	HMGET key field1 [field2] 
  - 取所有给定字段的值
10.	HMSET key field1 value1 [field2 value2 ] 
  - 时将多个 field-value (域-值)对设置到哈希表 key 中。
11.	HSET key field value 
  - 哈希表 key 中的字段 field 的值设为 value 。
12.	HSETNX key field value 
  - 有在字段 field 不存在时，设置哈希表字段的值。
13.	HVALS key 
  - 取哈希表中所有值
14.	HSCAN key cursor [MATCH pattern] [COUNT count] 
  - 代哈希表中的键值对。

### Redis 列表命令

1.	BLPOP key1 [key2 ] timeout 
  - 出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
2.	BRPOP key1 [key2 ] timeout 
  - 出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
3.	BRPOPLPUSH source destination timeout 
  - 列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它； 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
4.	LINDEX key index 
  - 过索引获取列表中的元素
5.	LINSERT key BEFORE|AFTER pivot value 
  - 列表的元素前或者后插入元素
6.	LLEN key 
  - 取列表长度
7.	LPOP key 
  - 出并获取列表的第一个元素
8.	LPUSH key value1 [value2] 
  - 一个或多个值插入到列表头部
9.	LPUSHX key value 
  - 一个值插入到已存在的列表头部
10.	LRANGE key start stop 
  - 取列表指定范围内的元素
11.	LREM key count value 
  - 除列表元素
12.	LSET key index value 
  - 过索引设置列表元素的值
13.	LTRIM key start stop 
  - 一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。
14.	RPOP key 
  - 除列表的最后一个元素，返回值为移除的元素。
15.	RPOPLPUSH source destination 
  - 除列表的最后一个元素，并将该元素添加到另一个列表并返回
16.	RPUSH key value1 [value2] 
  - 列表中添加一个或多个值
17.	RPUSHX key value 
  - 已存在的列表添加值

### Redis 集合命令

1.	SADD key member1 [member2] 
  - 集合添加一个或多个成员
2.	SCARD key 
  - 取集合的成员数
3.	SDIFF key1 [key2] 
  - 回给定所有集合的差集
4.	SDIFFSTORE destination key1 [key2] 
  - 回给定所有集合的差集并存储在 destination 中
5.	SINTER key1 [key2] 
  - 回给定所有集合的交集
6.	SINTERSTORE destination key1 [key2] 
  - 回给定所有集合的交集并存储在 destination 中
7.	SISMEMBER key member 
  - 断 member 元素是否是集合 key 的成员
8.	SMEMBERS key 
  - 回集合中的所有成员
9.	SMOVE source destination member 
  -  member 元素从 source 集合移动到 destination 集合
10.	SPOP key 
  - 除并返回集合中的一个随机元素
11.	SRANDMEMBER key [count] 
  - 回集合中一个或多个随机数
12.	SREM key member1 [member2] 
  - 除集合中一个或多个成员
13.	SUNION key1 [key2] 
  - 回所有给定集合的并集
14.	SUNIONSTORE destination key1 [key2] 
  - 有给定集合的并集存储在 destination 集合中
15.	SSCAN key cursor [MATCH pattern] [COUNT count] 
  - 代集合中的元素

### Redis 有序集合命令

1.	ZADD key score1 member1 [score2 member2] 
  - 有序集合添加一个或多个成员，或者更新已存在成员的分数
2.	ZCARD key 
  - 取有序集合的成员数
3.	ZCOUNT key min max 
  - 算在有序集合中指定区间分数的成员数
4.	ZINCRBY key increment member 
  - 序集合中对指定成员的分数加上增量 increment
5.	ZINTERSTORE destination numkeys key [key ...] 
  - 算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 key 中
6.	ZLEXCOUNT key min max 
  - 有序集合中计算指定字典区间内成员数量
7.	ZRANGE key start stop [WITHSCORES] 
  - 过索引区间返回有序集合指定区间内的成员
8.	ZRANGEBYLEX key min max [LIMIT offset count] 
  - 过字典区间返回有序集合的成员
9.	ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT] 
  - 过分数返回有序集合指定区间内的成员
10.	ZRANK key member 
  - 回有序集合中指定成员的索引
11.	ZREM key member [member ...] 
  - 除有序集合中的一个或多个成员
12.	ZREMRANGEBYLEX key min max 
  - 除有序集合中给定的字典区间的所有成员
13.	ZREMRANGEBYRANK key start stop 
  - 除有序集合中给定的排名区间的所有成员
14.	ZREMRANGEBYSCORE key min max 
  - 除有序集合中给定的分数区间的所有成员
15.	ZREVRANGE key start stop [WITHSCORES] 
  - 回有序集中指定区间内的成员，通过索引，分数从高到底
16.	ZREVRANGEBYSCORE key max min [WITHSCORES] 
  - 回有序集中指定分数区间内的成员，分数从高到低排序
17.	ZREVRANK key member 
  - 回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序
18.	ZSCORE key member 
  - 回有序集中，成员的分数值
19.	ZUNIONSTORE destination numkeys key [key ...] 
  - 算给定的一个或多个有序集的并集，并存储在新的 key 中
20.	ZSCAN key cursor [MATCH pattern] [COUNT count] 
  - 代有序集合中的元素（包括元素成员和元素分值）

### Redis HyperLogLog 命令

Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。

在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基 数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。

但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。

1.	PFADD key element [element ...] 
  - 加指定元素到 HyperLogLog 中。
2.	PFCOUNT key [key ...] 
  - 回给定 HyperLogLog 的基数估算值。
3.	PFMERGE destkey sourcekey [sourcekey ...] 
  - 多个 HyperLogLog 合并为一个 HyperLogLog

### Redis 发布订阅

1.	PSUBSCRIBE pattern [pattern ...] 
  - 阅一个或多个符合给定模式的频道。
2.	PUBSUB subcommand [argument [argument ...]] 
  - 看订阅与发布系统状态。
3.	PUBLISH channel message 
  - 信息发送到指定的频道。
4.	PUNSUBSCRIBE [pattern [pattern ...]] 
  - 订所有给定模式的频道。
5.	SUBSCRIBE channel [channel ...] 
  - 阅给定的一个或多个频道的信息。
6.	UNSUBSCRIBE [channel [channel ...]] 
  - 退订给定的频道。

### Redis 事务

- 批量操作在发送 EXEC 命令前被放入队列缓存。
- 收到 EXEC 命令后进入事务执行，事务中任意命令执行失败，其余的命令**依然被执行**。
- 在事务执行过程，其他客户端提交的命令请求不会插入到事务执行命令序列中。

1.	DISCARD 
  - 消事务，放弃执行事务块内的所有命令。
2.	EXEC 
  - 行所有事务块内的命令。
3.	MULTI 
  - 记一个事务块的开始。
4.	UNWATCH 
  - 消 WATCH 命令对所有 key 的监视。
5.	WATCH key [key ...] 
  - 视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。

### Redis 脚本命令

1.	EVAL script numkeys key [key ...] arg [arg ...] 
  - 行 Lua 脚本。
2.	EVALSHA sha1 numkeys key [key ...] arg [arg ...] 
  - 行 Lua 脚本。
3.	SCRIPT EXISTS script [script ...] 
  - 看指定的脚本是否已经被保存在缓存当中。
4.	SCRIPT FLUSH 
  - 脚本缓存中移除所有脚本。
5.	SCRIPT KILL 
  - 死当前正在运行的 Lua 脚本。
6.	SCRIPT LOAD script 
  - 脚本 script 添加到脚本缓存中，但并不立即执行这个脚本。

### Redis 连接命令

1.	AUTH password 
  - 证密码是否正确
2.	ECHO message 
  - 印字符串
3.	PING 
  - 看服务是否运行
4.	QUIT 
  - 闭当前连接
5.	SELECT index 
  - 换到指定的数据库

```
redis 127.0.0.1:6379> AUTH PASSWORD
(error) ERR Client sent AUTH, but no password is set
redis 127.0.0.1:6379> CONFIG SET requirepass "mypass"
OK
redis 127.0.0.1:6379> AUTH mypass
Ok
```

### Redis 服务器命令

1.	BGREWRITEAOF 
  - 步执行一个 AOF（AppendOnly File） 文件重写操作
2.	BGSAVE 
  - 后台异步保存当前数据库的数据到磁盘
3.	CLIENT KILL [ip:port] [ID client-id] 
  - 闭客户端连接
4.	CLIENT LIST 
  - 取连接到服务器的客户端连接列表
5.	CLIENT GETNAME 
  - 取连接的名称
6.	CLIENT PAUSE timeout 
  - 指定时间内终止运行来自客户端的命令
7.	CLIENT SETNAME connection-name 
  - 置当前连接的名称
8.	CLUSTER SLOTS 
  - 取集群节点的映射数组
9.	COMMAND 
  - 取 Redis 命令详情数组
10.	COMMAND COUNT 
  - 取 Redis 命令总数
11.	COMMAND GETKEYS 
  - 取给定命令的所有键
12.	TIME 
  - 回当前服务器时间
13.	COMMAND INFO command-name [command-name ...] 
  - 取指定 Redis 命令描述的数组
14.	CONFIG GET parameter 
  - 取指定配置参数的值
15.	CONFIG REWRITE 
  - 启动 Redis 服务器时所指定的 redis.conf 配置文件进行改写
16.	CONFIG SET parameter value 
  - 改 redis 配置参数，无需重启
17.	CONFIG RESETSTAT 
  - 置 INFO 命令中的某些统计数据
18.	DBSIZE 
  - 回当前数据库的 key 的数量
19.	DEBUG OBJECT key 
  - 取 key 的调试信息
20.	DEBUG SEGFAULT 
  -  Redis 服务崩溃
21.	FLUSHALL 
  - 除所有数据库的所有key
22.	FLUSHDB 
  - 除当前数据库的所有key
23.	INFO [section] 
  - 取 Redis 服务器的各种信息和统计数值
24.	LASTSAVE 
  - 回最近一次 Redis 成功将数据保存到磁盘上的时间，以 UNIX 时间戳格式表示
25.	MONITOR 
  - 时打印出 Redis 服务器接收到的命令，调试用
26.	ROLE 
  - 回主从实例所属的角色
27.	SAVE 
  - 步保存数据到硬盘
28.	SHUTDOWN [NOSAVE] [SAVE] 
  - 步保存数据到硬盘，并关闭服务器
29.	SLAVEOF host port 
  - 当前服务器转变为指定服务器的从属服务器(slave server)
30.	SLOWLOG subcommand [argument] 
  - 理 redis 的慢日志
31.	SYNC 
  - 于复制功能(replication)的内部命令

### Redis 数据备份与恢复

在 redis 安装目录中创建dump.rdb文件:

```
SAVE
```

如果需要恢复数据，只需将备份文件 (dump.rdb) 移动到 redis 安装目录并启动服务即可。获取 redis 目录可以使用 CONFIG 命令:

```
CONFIG GET dir
```

### Redis 安全

查看是否设置了密码验证：

```
CONFIG get requirepass
```

默认情况下 requirepass 参数是空的，这就意味着你无需通过密码验证就可以连接到 redis 服务。

你可以通过以下命令来修改该参数：

```
CONFIG set requirepass "Tuesday2"
```

登录：

```
AUTH "runoob"
```

## 性能调优

Redis可以达到100000+QPS(Query Per Second, 每秒查询次数)

原理：
- 完全基于内存，绝大部分请求是纯粹的内存操作，执行效率高
- 数据结构简单，对数据操作简单
- 采用单线程，单线程也能处理高并发请求，避免频繁创建与销毁线程占用资源，避免了频繁的上下文切换和锁竞争
- 使用多路I/O复用模型，非阻塞IO

Redis采用的I/O多路复用函数：epol/kqueue/evport/select

优先选择时间复杂度O(1)的I/O多路复用函数作为底层实现，若当前编译环境无其它更优的函数则默认调用select

基于react设计模式监听I/O事件

### 多路I/O复用模型

FD(File Descriptor): 文件描述符

一个打开的文件通过唯一的描述符进行引用，该描述符是打开文件的元数据到文件本身的映射。


#### 传统I/O复用模型

<img src="IO&#32;model.png"></img>

## Redis 持久化

1. RDB(快照)持久化：保存某个时间点的全量数据快照
2. AOF(Append-Only-File)持久化：保存写状态

### RDB(快照)持久化：保存某个时间点的全量数据快照

RDB持久化是指在指定的时间间隔内将内存中的数据集快照写入磁盘，实际操作过程是fork一个子进程，先将数据集写入临时文件，写入成功后，再替换之前的文件，用二进制压缩存储。RDB是Redis默认的持久化方式，会在对应的目录下生产一个dump.rdb文件，重启会通过加载dump.rdb文件恢复数据。

#### 优点：

- 只有一个文件dump.rdb，方便持久化；
- 容灾性好，一个文件可以保存到安全的磁盘；
- 性能最大化，fork子进程来完成写操作，让主进程继续处理命令，所以是IO最大化（使用- 单独子进程来进行持久化，主进程不会进行任何IO操作，保证了redis的高性能) ；
- 如果数据集偏大，RDB的启动效率会比AOF更高。

#### 缺点：

- 数据安全性低。
- 如果当数据集较大时，可能会导致整个服务器停止服务几百毫秒，甚至是1秒钟。

#### RDB(快照)持久化策略通过redis.conf来配置

```
save 900 1 # 900秒内如果有1条写入指令就触发一次快照
save 300 10 # 300秒内如果有10条写入指令就触发一次快照，若10>变动数>0，则会等到900秒后才备份
save 60 10000

stop-writes-on-bgsave-error yes # 当备份进程出错则主进程停止写入
rdbcompression yes # 在备份时压缩rdb文件再保存
```

#### rdb文件操作


手动持久化：
- SAVE：阻塞Redis的服务器进程，直到RDB文件被创建完毕
- BGSAVE：Fork出一个子进程来创建RDB文件，不阻塞服务器进程
  - 使用到了系统调用fork()：创建进程，实现了Copy-on-Write
  - Copy-on-Write(写时复制)：如果有多个调用者同时要求相同资源，他们会共同获取相同的指针指向相同的资源，直到某个调用者试图修改资源的内容时，系统才会真正复制一份专用副本给该调用者，而其他调用者所见到的最初的资源仍保持不变

```
SAVE
LASTSAVE
BGSAVE
LASTSAVE
```

自动持久化：
- 根据redis.conf配置里的 `SAVE m n`定时触发(用的是BGSAVE)
- 主从复制时，主节点自动触发
- 执行Debug Reload
- 执行Shutdown且没有开启AOF持久化

缺点：
- 内存数据全部同步，数据量大时会因为I/O影响性能
- 可能会因为Redis挂掉而丢失当前至最近一次快照期间的数据

### AOF(Append-Only-File)持久化：保存写状态

AOF持久化是以日志的形式记录服务器所处理的每一个写、删除操作，查询操作不会记录，以文本的方式记录，文件中可以看到详细的操作记录。她的出现是为了弥补RDB的不足（数据的不一致性），所以它采用日志的形式来记录每个写操作，并追加到文件中。Redis 重启的会根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作。与快照持久化相比，AOF 持久化的实时性更好，因此已成为主流的持久化方案。 默认情况下 Redis 没有开启 AOF持久化

- 记录下除了查询以外的所有变更数据库状态的指令
- 以append的形式追加保存到AOF文件中(增量)
  
优点：
- 数据安全性更高，AOF持久化可以配置appendfsync属性
- 通过append模式写文件，即使中途服务器宕机，可以通过redis-check-aof工具解决数据一致性问题。
- AOF机制的rewrite模式。

缺点：
- AOF文件不断增大
  - 日志重写：
    1. 调用fork()创建子进程
    2. 子进程把新的AOF写到一个临时文件里，不依赖原来的AOF文件
    3. 主进程持续把新的变动同时写到内存和原来的AOF里
    4. 主进程获取子进程重写AOF完成的信号，往新的AOF同步增量变动
    5. 使用新的AOF文件替换老AOF文件
- 根据同步策略的不同，AOF在运行效率上往往会慢于RDB。

#### AOF持久化策略通过redis.conf来配置

默认情况下 Redis 没有开启 AOF持久化，可以通过设置 appendonly 参数开启：

```
appendonly yes
```

- appendfsync always：每次有数据修改发生时都会写入AOF文件
- appendfsync everysec：每秒钟同步一次，将多个写命令同步到硬盘
- appendfsync no：让操作系统决定何时进行同步

### AOF vs RDB

RDB：
- 优点：全量数据快照，文件小，恢复快
- 缺点：无法保存最近一次快照之后的数据

AOF：
- 优点：可读性高，适合保存增量数据，数据不易丢失
- 缺点：文件体积大，恢复时间长

### RDB-AOF混合持久化方式(Redis默认)

使用BGSAVE做镜像全量持久化，AOF做增量持久化

#### 通过redis.conf来配置

```
appendonly yes
appendfilename "appendonly.aof"
appendfsync everysec # 配置文件写入方式：always一旦缓存区发生变化就及时写入, everysec每隔一秒写入, no由操作系统来觉得写入时间，一般是缓存区满时写入
```

## Pipeline

- Pipeline和Linux的管道机制类似
- 基于请求/响应模型，单个请求处理需要一一应答
- Pipeline批量执行指令，节省多次IO往返时间
- 有顺序依赖的指令需要分批发送

## Redis同步机制

### 主从同步

全同步过程：
1. Slave发送sync命令到Master
2. Master启动一个后台进程，将Redis中的数据快照保存到文件中
3. Master将保存数据快照期间接受到的写命令缓存起来
4. Master完成写文件操作后，将文件发送给Slave
5. 使用新的RDB文件替换掉就的RDB文件
6. Master将这期间收集的增量写命令发送给Slave端

增量同步过程：
1. Master接收到用户的操作指令，判断是否需要传播到Slave
2. 将操作记录追加到AOF文件
3. 将操作传播到其它Slave：
  1. 对齐主从库
  2. 往响应缓存写入指令
4. 将缓存中的数据发送给Slave

主从模式缺点：当Master挂掉之后将无法写入数据

### Redis Sentinel 哨兵模式

用于解决主从同步Master宕机后的主从切换问题：
- 监控：检查主从服务器是否运行正常
- 提醒：通过API向管理员或其它应用程序发送故障通知
- 自动故障迁移：主从切换，自动选举一个Slave为Master

### 流言协议Gossip

- 每个节点都随机地与对方通信，最终所有节点的状态达成一致
- 种子节点定期随机向其它节点发送节点列表以及需要传播的消息
- 不保证信息一定会传递给所有节点，但是最终会趋于一致

## Redis集群

如何从海量数据里快速找到所需？

分片：按照某种规则去划分数据，分散存储在多个节点上

缺点：常规的按照哈希划分无法实现节点的动态增减

### 原理

#### 一致性哈希算法：对2^32取模，将哈希值空间组织成虚拟的圆环


---

## 集成到系统

### 集成到 Java

可以使用 jedis包

### 集成到数据库

#### Redis 设置过期时间

Redis可以对存储在缓存中的数据设置过期时间。作为一个缓存数据库，这是非常实用的功能。之前写过一篇前后端交互的文章讲过，Token 或者一些登录信息，尤其是短信验证码都是有时间限制的，按照传统的数据库处理方式，一般都是自己判断过期，这样无疑会严重影响项目性能。而有一个好的方案其实就是将这些验证信息存入Redis设置过期时间，如果设置了存活时间30分钟，那么半小时之后这些数据就会从Redis中进行删除。那说到删除，Redis是如果做到对这些数据进行删除的呢：

- 定期删除：Redis 默认是每隔 100ms 就随机抽取一些设置了过期时间的 Key，检查其是否过期，如果过期就删除。为什么是随机抽取而不是检查所有key？因为你如果设置的key成千上万，每100毫秒都将所有存在的key检查一遍，会给CPU带来比较大的压力。
- 惰性删除 ：定期删除可能会导致很多过期 Key 到了时间并没有被删除掉。用户在获取key的时候，redis会检查一下，这个key如果设置过期时间那么是否过期了，如果过期就删除这个key。

但是只是使用定期删除 + 惰性删除的删除机制还是存在一个致命问题：如果定期删除漏掉了很多过期 Key，而且用户长时间也没有使用到这些过期key，就会导致这些过期key堆积在内存里，导致Redis内存块被消耗殆尽。所以有了Redis内存淘汰机制的诞生。

---

## 可能出现的问题

### 缓存雪崩

缓存处理过程：接收到请求请求，先从缓存中取数据，取到直接返回结果，取不到时从数据库中取，数据库取到更新缓存，并返回结果，数据库也没取到，那直接返回空结果。

缓存雪崩：缓存雪崩是指缓存中数据大批量到过期时间，而查询数据量巨大，引起数据库压力过大甚至down机。

解决办法：
- 缓存数据的过期时间设置随机，防止同一时间大量数据过期现象发生。
- 如果缓存数据库是分布式部署，将热点数据均匀分布在不同搞得缓存数据库中。
- 设置热点数据永远不过期。

### 缓存穿透

简介：缓存穿透是指缓存和数据库中都没有的数据，而用户不断发起请求，如发起为id为“-1”的数据或id为特别大不存在的数据。这时的用户很可能是攻击者，攻击会导致数据库压力过大。

解决办法：
- 接口层增加校验，如用户鉴权校验，id做基础校验，id<=0的直接拦截；
- 从缓存取不到的数据，在数据库中也没有取到，这时也可以将key-value对写为key-null，缓存有效时间可以设置30秒

### 缓存击穿

简介： 缓存击穿是指缓存中没有但数据库中有的数据，这时由于并发用户特别多，同时读缓存没读到数据，又同时去数据库去取数据，引起数据库压力瞬间增大

解决方法：
- 设置热点数据永远不过期。
- 加互斥锁

