# 一、全局/通用命令（适用于所有 Key）
这些命令不区分数据类型，用于管理 Key 的生命周期。
## 命令	示例	说明
KEYS pattern	KEYS user:*	查找所有符合给定模式(pattern)的 Key。生产环境慎用，会阻塞。  
EXISTS key	EXISTS name	检查 Key 是否存在（返回 1 或 0）。  
TYPE key	TYPE mylist	查看 Key 对应的数据类型（string, list 等）。  
DEL key [key ...]	DEL name age	删除一个或多个 Key。   
EXPIRE key seconds	EXPIRE token 600	设置 Key 的过期时间（单位：秒）。  
TTL key	TTL session	查看 Key 还剩多少秒过期。-1 表示永不过期，-2 表示已不存在。  
RENAME key newkey	RENAME old_name new_name	重命名 Key。  

# 二、String（字符串）
最基础的类型，可以是普通字符串、数字、二进制数据。最大 512MB。
## 命令	示例	常用场景
SET key value	SET name "Duoyian"	设置值。  
SET key value EX seconds	SET code "1234" EX 300	设置值并指定过期时间（常用）。  
GET key	GET name	获取值。  
SETEX key seconds value	SETEX status 1 "ready"	等同于 SET + EXPIRE，原子操作。    
MGET key1 key2 ...	MGET name age	批量获取，减少网络耗时。  
MSET k1 v1 k2 v2	MSET k1 v1 k2 v2	批量设置。  
INCR key	INCR views	数字自增 +1（必须为整数）。用于计数器。  
INCRBY key increment	INCRBY age 2	增加指定数值。  
DECR key	DECR stock	数字自减 -1。用于秒杀扣库存。  
SETNX key value	SETNX lock "locked"	如果不存在才设置（If Not eXists）。用于实现分布式锁。  
STRLEN key	STRLEN name	获取字符串长度。  

# 三、Hash（哈希）
适合存储对象，比如用户信息（ID -> {name: "A", age: 18}）。Value 只能是 String。
## 命令	示例	说明
HSET key field value	HSET user:1 name "Tom"	设置 Hash 中的一个字段。  
HGET key field	HGET user:1 name	获取 Hash 中的一个字段。  
HMSET key f1 v1 f2 v2	HMSET user:2 name "Jack" age 20	批量设置（旧版，新版 HSET 也支持）。  
HMGET key field1 ...	HMGET user:1 name age	批量获取字段。  
HGETALL key	HGETALL user:1	获取该 Hash 中所有字段和值（慎用，如果数据量大会影响性能）。  
HDEL key field	HDEL user:1 age	删除 Hash 中的某个字段。  
HKEYS key	HKEYS user:1	只获取所有字段名。  
HVALS key	HVALS user:1	只获取所有值。  
HINCRBY key field n	HINCRBY user:1 score 10	给 Hash 中的数值字段增减 n。    

# 四、List（列表）
列表是有序的字符串集合，底层是链表。适合做消息队列或时间轴。
## 命令	示例	常用场景
LPUSH key value	LPUSH tasks "task1"	从列表左侧（头部）插入一个或多个值。  
RPUSH key value	RPUSH tasks "task2"	从列表右侧（尾部）插入。  
LPOP key	LPOP tasks	从左侧弹出一个值（取出并删除）。  
RPOP key	RPOP tasks	从右侧弹出一个值。  
LRANGE key start stop	LRANGE tasks 0 -1	获取列表指定范围内的元素。0 -1 表示获取全部。  
LLEN key	LLEN tasks	获取列表长度。  
LINDEX key index	LINDEX tasks 0	获取列表中索引为 index 的元素（类似数组下标）。  
LREM key count value	LREM tasks 1 "task1"	删除列表中指定数量的 value。   
RPOPLPUSH source dest	RPOPLPUSH list1 list2	安全队列操作：从一个列表右侧弹出，插入到另一个列表左侧。  

# 五、Set（集合）
无序、不重复的集合。适合做交集、并集运算。
## 命令	示例	常用场景
SADD key member	SADD tags "java" "redis"	向集合添加一个或多个成员。  
SMEMBERS key	SMEMBERS tags	获取集合中所有成员。  
SISMEMBER key member	SISMEMBER tags "java"	判断 member 是否在集合中。  
SREM key member	SREM tags "java"	移除集合中某成员。  
SCARD key	SCARD tags	获取集合的成员数（长度）。  
SINTER key1 key2	SINTER setA setB	求交集（例如：共同关注）。  
SUNION key1 key2	SUNION setA setB	求并集。  
SDIFF key1 key2	SDIFF setA setB	求差集（A 中有但 B 中没有的）。  
SRANDMEMBER key count	SRANDMEMBER tags 2	随机获取集合中的一个或多个成员（抽奖功能）。  

# 六、ZSet（Sorted Set - 有序集合）
最强大的类型之一。每个元素都会关联一个 double 类型的分数。根据分数排序，元素唯一。
## 命令	示例	常用场景
ZADD key score member	ZADD rank 100 "Tom"	添加成员并指定分数。  
ZRANGE key start stop	ZRANGE rank 0 -1	按分数从低到高获取成员。  
ZREVRANGE key start stop	ZREVRANGE rank 0 -1	按分数从高到低获取成员（排行榜常用）。  
ZRANGE key start stop WITHSCORES	ZRANGE rank 0 -1 WITHSCORES	获取成员的同时显示分数。  
ZSCORE key member	ZSCORE rank "Tom"	获取某个成员的分数。  
ZINCRBY key increment member	ZINCRBY rank 50 "Tom"	给指定成员的分数增加 increment。  
ZREM key member	ZREM rank "Tom"	移除成员。  
ZRANK key member	ZRANK rank "Tom"	返回成员的排名（从 0 开始，低分排 0）。  
ZREVRANK key member	ZREVRANK rank "Tom"	返回成员的排名（高分排 0）。    
ZCARD key	ZCARD rank	获取集合成员数量。  

# 七、高级控制命令（用于排查和运维）
## 命令	示例	说明
INFO	INFO	查看 Redis 服务器的运行状态、内存、连接数等。  
CLIENT LIST	CLIENT LIST	查看当前连接的所有客户端。  
FLUSHALL	FLUSHALL	清空所有数据库的所有数据（非常危险！）。  
FLUSHDB	FLUSHDB	清空当前数据库的所有数据（比较危险！）。  
SCAN cursor MATCH pattern	SCAN 0 MATCH user:*	遍历 Key（替代 KEYS，不阻塞服务器，适合生产环境）。  
SLOWLOG get 10	SLOWLOG get 10	获取最近 10 条慢查询命令（用于性能分析）。  