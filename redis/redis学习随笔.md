# 学习资源
电子书：https://huangz.works/redisbook/

# Pipeline 和 Transaction 的区别
Pipeline 是 Redis 提供的非原子性批量操作命令，可以一次性执行多个命令，
但是每个命令的执行是独立的，不保证原子性。一个命令失败不会影响其他命令的执行。  
Transaction 是 Redis 提供的原子性事务，
可以一次性执行多个命令, 并且保证这些命令是原子性的。  

# 网络延迟
Redis 的网络延迟大概是 mysql 的十分之一。

# 持久化机制
Redis 支持两种持久化机制：RDB 和 AOF。
## 1. RDB（Redis Database）持久化
特点：  
生成数据快照（snapshot）  
二进制格式，紧凑高效  
适合备份和灾难恢复

触发方式：
1. 配置文件自动触发（save指令）
save 900 1      # 900秒内至少有1个key被修改  
save 300 10     # 300秒内至少有10个key被修改  
save 60 10000   # 60秒内至少有10000个key被修改  

2. 手动执行命令
SAVE           # 同步保存，阻塞主线程  
BGSAVE         # 后台异步保存（fork子进程）

3. 关闭服务器时（如果开启了shutdown save）

优缺点：  
✅ 优点：文件小、恢复速度快、适合全量备份  
❌ 缺点：可能丢失较多数据（最后一次保存后的修改）

## 2. AOF（Append Only File）持久化
特点：  
记录每次写操作命令  
以Redis协议格式追加到文件末尾  

配置选项：  
appendonly yes                     # 开启AOF  
appendfilename "appendonly.aof"   # AOF文件名  

同步策略（从安全到性能排序）  
appendfsync always    # 每次写操作都同步，最安全但性能最差  
appendfsync everysec  # 每秒同步一次（推荐）  
appendfsync no        # 由操作系统决定  

AOF重写相关  
auto-aof-rewrite-percentage 100   # 增长比例达到100%  
auto-aof-rewrite-min-size 64mb    # AOF文件最小64MB  

优缺点：  
✅ 优点：数据更安全、可读性强、最多丢失1秒数据  
❌ 缺点：文件较大、恢复速度较慢  

## 3. 混合持久化
Redis 4.0 开始支持混合持久化，将 RDB 和 AOF 的优点结合起来。

## 重启恢复流程
启动Redis服务  
    ↓  
加载配置文件  
    ↓  
检查持久化文件  
│  
├── 如果只有RDB文件 → 加载RDB文件恢复数据  
│  
├── 如果只有AOF文件 → 加载AOF文件回放命令  
│  
└── 如果两者都有 → 优先使用AOF（AOF更完整）  
    ↓  
执行恢复过程中的命令  
    ↓  
服务准备就绪  

# RESP协议
RESP（Redis Serialization Protocol）是 Redis 客户端与服务端之间的通信协议。  
特点：文本协议，简单可读，易于解析。  

5 种数据类型：  
* 简单字符串（Simple String）：`+OK\r\n`，表示成功响应。
* 错误（Error）：`-ERR unknown command\r\n`，表示错误信息。
* 整数（Integer）：`:1000\r\n`，表示整数响应（如 INCR 的返回值）。
* 批量字符串（Bulk String）：`$6\r\nfoobar\r\n`，$ 后跟字节数，再跟实际内容。
  `$-1\r\n` 表示 null。
* 数组（Array）：`*2\r\n$3\r\nfoo\r\n$3\r\nbar\r\n`，* 后跟元素个数，再依次跟每个元素。  

客户端发送命令：以 Bulk String 数组形式发送，如 `SET name Tom` 会被编码为
`*3\r\n$3\r\nSET\r\n$4\r\nname\r\n$3\r\nTom\r\n`。  

RESP3（Redis 6.0 引入）：新增了 Map、Set、Push、Big Number、Verbatim String、
Typed Stream 等类型，支持客户端缓存、推送通知等新特性。  

# Redisson WatchDog 机制：
问题：分布式锁设置了过期时间，但业务执行时间超过了锁的过期时间，导致锁提前释放，
其他线程获取到锁，产生并发问题。  

机制：看门狗定时检查锁是否还被当前线程持有，如果是则自动续期。  
* 默认锁过期时间 30 秒（lockWatchdogTimeout）。
* 每隔 10 秒（lockWatchdogTimeout / 3）检查一次，如果锁还被当前线程持有，
  则将过期时间重置为 30 秒。
* 底层用 Netty 的 HashedWheelTimer（时间轮）调度定时任务，通过 Lua 脚本原子性
  地判断 + 续期。  

注意：  
* 只有在调用 `lock()` 不指定 leaseTime 时才会启动看门狗。如果调用
  `lock(10, TimeUnit.SECONDS)` 指定了过期时间，则不会续期，到时间自动释放。
* 如果持有锁的进程崩溃，看门狗也会停止，锁会在过期时间后自动释放，不会死锁。

