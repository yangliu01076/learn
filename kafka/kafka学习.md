# 学习资源
Apache Kafka 官网：https://kafka.apache.org/documentation/

# 核心概念
* Broker：Kafka 节点，多个 Broker 组成集群。
* Topic：消息主题，逻辑分类。一个 Topic 分为多个 Partition。
* Partition：分区，每个 Partition 是一个有序的、不可变的消息日志。
  分区数决定并发度，越多吞吐越高，但也会增加开销。
* Replica：副本，每个 Partition 有多个副本（Leader + Follower）。读写都走 Leader，
  Follower 负责同步。
* ISR（In-Sync Replicas）：与 Leader 保持同步的副本集合。ISR 中的副本才被认为
  是可靠的，HW 由 ISR 中最小的 LEO 决定。
* Consumer Group：消费者组。组内消费者平分 Partition，一个 Partition 同一时刻只被
  组内一个消费者消费。组间广播，组内单播。
* Offset：消息在 Partition 中的位置编号，从 0 开始递增。  

# 生产者
## 消息发送流程
1. Producer 创建 ProducerRecord，指定 topic、partition（可选）、key（可选）、value。
2. 序列化：将 key 和 value 序列化为字节数组。
3. 分区器：如果指定了 partition 则直接用；否则如果 key 非空，按 key hash 分区；都没有则轮询。
4. 追加到 RecordAccumulator（消息累加器），按 batch 缓存。
5. Sender 线程将 batch 通过网络发送给 Leader Broker。
6. Broker 写入 Leader 后，根据 acks 策略决定是否等待 Follower 同步。  

## acks 参数
* acks=0：Producer 不等 Broker 确认，直接发下一条。吞吐最高，可能丢消息。
* acks=1（默认）：Leader 写入成功即返回确认。Leader 挂了但 Follower 还没同步时丢消息。
* acks=all / -1：ISR 中所有副本都写入成功才返回确认。最安全，但延迟最高。  

# 消费者
## 消费流程
1. Consumer 加入 Consumer Group，分配到若干 Partition。
2. 从上次提交的 offset 开始拉取消息（pull 模式，不是 push）。
3. 处理完消息后提交 offset。  

## Offset 管理
Kafka 0.9+ 将 offset 存储在内部 Topic `__consumer_offsets` 中。  
* 自动提交：enable.auto.commit=true，每隔 auto.commit.interval.ms 自动提交。
  缺点：可能重复消费（处理完但还没提交就挂了）或消息丢失（还没处理完就提交了）。
* 手动提交：处理完业务逻辑后手动 commitSync() / commitAsync()，精确控制。  

## Rebalance
当消费者加入/离开 Consumer Group，或 Partition 数量变化时，触发 Rebalance，
重新分配 Partition 与消费者的映射关系。  
Rebalance 期间所有消费者停止消费（Stop the World），是消费延迟的主要来源。  

# 高吞吐设计
* 顺序写磁盘：消息追加到日志文件末尾，磁盘顺序写速度接近内存随机写。
* 零拷贝（sendfile）：Broker 读取消息时，数据从 PageCache 直接通过 DMA 传到网卡，
  不经过用户态内存拷贝。
* 批量发送 + 压缩：Producer 端批量发送，支持 snappy/gzip/lz4/zstd 压缩，减少网络开销。
* PageCache：读写都利用操作系统页缓存，热数据在内存中直接命中。
* 分区并行：Topic 多 Partition，Consumer Group 内多消费者并行消费。  

# 消息可靠性
## ISR 与 HW
* LEO（Log End Offset）：每个副本下一条待写入消息的 offset。
* HW（High Watermark）：ISR 中所有副本 LEO 的最小值。Consumer 只能消费 HW 之前的消息。
* Leader 收到写入请求 -> 更新自身 LEO -> 等待 ISR 中 Follower 同步 -> 所有 Follower LEO
  达到 Leader LEO -> 更新 HW -> 返回 acks。  

## 消息语义
* At Most Once：先提交 offset 再处理消息，可能丢消息。
* At Least Once（默认）：先处理消息再提交 offset，可能重复消费（需业务幂等）。
* Exactly Once：通过幂等 Producer（enable.idempotence=true）+ 事务实现。