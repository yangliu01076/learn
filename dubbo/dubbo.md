# 学习资源
https://cn.dubbo.apache.org/zh-cn/overview/what/

# 核心架构
四个角色：  
* Provider：服务提供方，启动时将自己的服务信息（IP、端口、接口、版本）注册到注册中心。
* Consumer：服务消费方，启动时从注册中心订阅所需服务，获取 Provider 地址列表。
* Registry：注册中心（Nacos / Zookeeper），负责服务发现与地址推送。
* Monitor：监控中心，统计调用次数和耗时。  
调用流程：Consumer 启动 -> 从 Registry 拉取 Provider 地址列表并缓存本地 ->
Consumer 负载均衡选一台 Provider 发起直连调用 -> Registry 只参与地址发现，不参与调用。  

# SPI 机制
Dubbo SPI 是对 Java SPI 的增强。  
Java SPI（ServiceLoader）：遍历 META-INF/services 下所有实现类并全部实例化，无法按需加载，
不支持依赖注入。  
Dubbo SPI：  
* 按 key 加载：配置文件格式为 `key=实现类全限定名`，通过 ExtensionLoader.getExtension(key)
  精确加载指定实现。
* 自适应扩展（@Adaptive）：运行时根据参数（如 URL 中的 protocol）动态选择实现。
  编译生成 Adaptive 类，方法体内根据参数值查 ExtensionLoader。
* Wrapper（自动包装）：如果实现类构造器只有一个参数且为接口类型，Dubbo 会自动将其识别
  为 Wrapper，在加载扩展时层层包裹（类似 AOP）。
* IOC：扩展类的 setter 方法会被自动注入依赖（从 ExtensionLoader 查找对应扩展）。  

# 负载均衡策略
* Random（默认）：按权重随机，权重越高被选中概率越大。调用量越大越均匀。
* RoundRobin：加权轮询，按权重比例依次分配。
* LeastActive：最少活跃调用数优先，活跃数低的机器处理能力更强，分配更多请求。
* ConsistentHash：一致性哈希，相同参数的请求总是发到同一 Provider，适合有状态场景。
* ShortestResponse（3.0+）：响应时间最短优先。  

# 集群容错策略
| 策略 | 说明 | 适用场景 |
|:---|:---|:---|
| Failover（默认） | 失败自动重试，默认重试 2 次 | 幂等读操作 |
| Failfast | 快速失败，只调用一次 | 非幂等写操作（如新增） |
| Failsafe | 失败忽略，不抛异常 | 写日志、发通知等可容忍失败的场景 |
| Failback | 失败后台定时重试 | 实时性要求不高，最终一致的场景 |
| Forking | 并行调用多个，只要一个成功即返回 | 实时性要求高，浪费资源 |
| Broadcast | 逐个调用所有 Provider，任一失败即失败 | 刷新所有节点本地缓存 |

# 服务暴露与引用
## 服务暴露（Provider 端）
1. Spring 启动时扫描 @DubboService 注解的 Bean。
2. 通过 ServiceConfig 将服务信息注册到注册中心。
3. 启动 Netty Server 监听端口，等待 Consumer 调用。  
## 服务引用（Consumer 端）
1. Spring 启动时扫描 @DubboReference 注解的字段。
2. 通过 ReferenceConfig 从注册中心订阅 Provider 地址列表。
3. 为接口生成代理对象（默认 Javassist），代理对象内部通过负载均衡选择 Provider 发起调用。  

# 序列化
Dubbo 支持多种序列化协议：  
* hessian2（默认）：跨语言，性能较好。
* Kryo：性能高，但不跨语言。
* Protostuff：基于 Protobuf，性能高。  
序列化方式影响网络传输效率和兼容性，更换序列化方式时需确保上下游兼容。
