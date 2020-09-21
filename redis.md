# Redis

## 管理 Redis
### 启动服务

修改 `redis.conf`
```
# 将`daemonize`由`no`改为`yes` 
daemonize yes

# 默认绑定的是回环地址，默认不能被其他机器访问 
# bind 127.0.0.1

# 是否开启保护模式，由yes该为no 
protected-mode no
```

启动服务
`./redis-server redis.conf`

### 关闭服务
`./redis-cli shutdown`

### 客户端连接
默认方式
`./redis-cli`

指定端口和参数
```bash
./redis-cli -h 127.0.0.1 -p 6379
```

### 其他命令
* `redis-benchmark` 性能测试的工具
* `redis-check-aof` aof 文件进行检查的工具
* `redis-check-dump` rdb 文件进行检查的工具
* `redis-sentinel` 启动哨兵监控服务

## 数据类型

### Redis 的 key 的设计
1. 用 : 分割
2. 把表名转化为 key 前缀，比如：`user:`
3. 第三段放主键值
4. 第四段放列名

比如：用户表 user
| userId | username | password | email |
| --- | --- | --- | --- |
| 9 | njin | 111111 | nj****ol@gmail.com |

转换为 redis 的 key-value 存储
* user:9:username -> njin
* user:9:password -> 111111
* user:9:email -> nj****ol@gmail.com

### String 字符串类型 
Redis 的 String 可以表达 3 种值：字符串、整数、浮点数。

常见操作：
* `set key value` 赋值
* `get key` 取值 
* `getset key value` 取值并赋值
* `setnx key value` 当 value 不存在时采用赋值
* `set key value nx px 3000` 原子操作，px 设置超时时间（毫秒）
* `append key value` 向尾部追加值
* `strlen key` 获取字符串长度
* `incr key` 递增数字
* `incrby key increment` 按指定的值递增数字
* `decr key` 递减
* `decrby key decrement` 按指定值递减

### List 列表类型
List 可以存储有序、可重复的元素，内部是双向链表，获取头或尾部元素是极快的。

常见操作：
* `lpush key v1 v2 v3 ...` 从左侧插入列表
* `lpop key` 从左侧取出
* `rpush key v1 v2 v3 ...` 从右侧插入列表
* `rpop key` 从右侧取出
* `lpushx key value` 将值插入到列表头部
* `rpushx key value` 将值插入到列表尾部
* `blpop key timeout` 从列表左侧取出，当列表为空时阻塞，可以设置最大阻塞时间（秒）
* `brpop key timeout` 从列表右侧取出，其他同上
* `llen key` 获得列表中元素个数
* `lindex key index` 获得列表中下标为 index 的元素，index 从 0 开始
* `lrange key start end` 返回列表中通过 start 和 end 指定的区间的元素
* `lrem key count value` 删除列表中与 value 相等的元素，当 count>0 时，lrem 会从列表左边开始删除；当 count<0 时，lrem 会从列表后边开始删除；当 count=0 时，lrem 删除所有值为 value 的元素
* `lset key index value` 将列表 index 位置的元素设置成 value 的值
* `ltrim key start end` 对列表进行修剪，值保留 start 到 end 区间的元素
* `rpoplpush key1 key2` 从 key1 列表右侧弹出并插入到 key2 列表左侧
* `brpoplpush key1 key2` 同上，但 key1 为空时会阻塞
* `linsert key before/after pivot value` 将 value 插入列表的 pivot 位置之前／之后

### Set 集合类型
无序、唯一元素。

常见操作：
* `sadd key mem1 mem2 ...` 为集合添加元素
* `srem key mem1 mem2 ...` 删除集合中指定的元素
* `smembers key` 获取集合中所有元素
* `spop key`　返回并删除集合中一个随机元素
* `srandmember key` 返回集合中一个随机元素
* `scard key` 获得集合中元素的数量
* `sismember key` 判断元素是否在集合内
* `sinter key1 key2 key3 ...` 求多个集合的交集
* `sdiff key1 key2 key3 ...` 求多个集合的差集
* `sunion key1 key2 key3 ...` 求多个集合的并集

### SortedSet 有序集合类型
有序集合 ZSet，元素本身是无序不重复的。每个元素关联一个分数（Score），可以按分数排序，分数可重复。

常见操作：
* `zadd key score1 memeber1 score2 memeber2 ...` 为有序集合添加新成员
* `zrem key mem1 mem2 ...` 删除有序集合中指定成员
* `zcard key` 获取有序集合中元素的个数
* `zcount key min max` 返回集合中的 score 值在 [min, max] 之间的元素数量
* `zincrby key incrment member` 在集合的 member 分值上加 increment
* `zscore key member` 获得集合中 member 的分值
* `zrank key member` 获得集合中 member 的排名（分值从小到大）
* `zrevrank key member` 同上，但分值从大到小
* `zrange key start end` 获得集合中指定区间成员，按分值递增排序
* `zrevrange key start end` 同上，但按分值递减排序

### Hash 散列表
Redis Hash 是一个 String 类型的 field 和 Value 的映射表，它提供了字段和字段值的映射。

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1giya9is3ytj310k0l0gnz.jpg)

常见操作：
* `hset key field value` 赋值，不区别新增或修改
* `hmset key field1 value1 field2 value2` 批量赋值
* `hsetnx key field value` 赋值，如果 field 存在则不操作
* `hexists key field` 查看某个 field 是否存在
* `hget key field` 获取一个字段值
* `hmget key field1 field2 ...` 获取多个字段值
* `hgetall key` 获取全部字段和值
* `hdel key field1 field2 ...` 删除指定字段
* `hincrby key field incrment` 指定字段自增 increment
* `hlen key` 获得字段数量

### Bitmap 位图类型
Bitmap 是进行位操作，通过一个 bit 位来表示某个元素对应的值或者状态。Bitmap 本身会极大的节省存储空间。

常见操作：
* `setbit key offset value` 设置 key 在 offset 处的 bit 值（只能是 0 或 1）
* `getbit key offset` 获得 key 在 offset 处的 bit 值
* `bitcount key` 获得 key 的 bit 为 1 的个数
* `bitpos key value` 返回 key 第一个被设置为 value 的值的索引值
* `bitop and[or/xor/not] destkey key [key...]` 对多个 key 进行逻辑运算之后，结果存入 destkey

### Geo 地理位置类型
Redis 3.2 中正式开始使用，利用了 z 阶曲线、Base32编码和 geohash 算法。

常见操作：
* `geoadd key 经度1 维度1 成员名称1 经度2 维度2 成员名称2 ...` 添加地理坐标
* `geohash key 成员名称1 成员名称2 ...` 返回标准的 geohash 串
* `geopos key 成员名称1 成员名称2 ...` 返回成员经纬度
* `geodist key 成员1 成员2 单位（km）...` 计算成员间距离
* `georadiusbymember key 成员 值单位 count 数量 asc[desc]` 根据成员地理位置查找附近的成员

### Stream 数据流类型
Redis 5.0 后新增的数据结构，用于可持久化的消息队列。几乎满足了消息队列具备的全部内容，包括：
* 消息 ID 的序列化生成
* 消息的遍历
* 消息的阻塞和非阻塞读取
* 消息的分组消费
* 未完成消息的处理
* 消息队列监控

常见操作：

* `xadd key id <*> field1 value1 ...` 将制定消息数据追加到制定队列（key）中，* 表示最新生成的ID（当前时间+序列号）
* `xread [COUNT count] [BLOCK milliseconds] STREAMS key [key ...] ID [ID ...]` 从消息队列中读取，COUNT：读取条数，BLOCK：阻塞读（默认不阻塞） key：队列名称 ID：消息ID
* `xrange key start end [COUNT]` 读取队列中给定 ID 范围的消息，COUNT：返回消息数（消息 ID 从小到大）
* `xrevrange key start en [COUNT]` 同上，消息 ID 从大到小
* `xdel key id` 删除队列消息
* `xgroup create key groupname id` 创建一个新的消息组
* `xgroup destory key groupname` 删除指定的消费组
* `xgroup delconsumer key groupname cname` 删除指定消费组中的某个消费者
* `xgroup setid key id` 修改指定消息的最大ID
* `xreadgroup group groupname consumer COUNT stream key` 从队列中的消费组创建消费者，并消费数据（consumer不存在则创建）


### 其他常用命令

* `keys pattern` 返回满足给定 pattern 的所有 key
* `del key` 删除 key
* `exists key` 确定一个 key 是否存在
* `expire key seconds` 设置 key 的生存时间（秒），超时则自动删除
* `ttl key` 查看 key 的生存时间
* `persist key` 清除生存时间
* `pexpire key milliseconds` 生存时间设置单位为：毫秒
* `rename oldkey newkey` 重命名
* `type key` 显示指定 key 的数据类型