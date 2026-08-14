# 目录
* 集合框架：HashMap、ConcurrentHashMap、ArrayList
* 并发包 (JUC)：ThreadPoolExecutor、ReentrantLock（AQS 原理）、ThreadLocal
* 基础：String 的不可变性
* 并发关键字：volatile、synchronized
* 其他：equals 判断方式、对象内存布局


## equals()方法的严格判断和宽松判断
equals()方法通常会有getClass()和instanceof两种判断方式，前者必须是类型一致
后者是类型兼容，比如A是B的父类，那么A instanceof B 返回true。JDK 官方类库
（如 java.util.Date 等）大多倾向于使用 instanceof，以便支持集合框架中的多
态操作。  

## volatile关键字和指令重排
volatile关键字可以保证可见性，禁止指令重排，但是不能保证原子性。
在单例模式中，双重检查锁的实现，需要使用volatile关键字，防止指令重排。主要是
防止指令重排，正常是先分配内存，再初始化，再赋值给引用，但是指令重排后，可能
先赋值给引用，再初始化，再分配内存，这样就导致其他线程获取到的是未初始化的对象。  

## synchronized关键字
原子性：确保同步代码块完整执行，不会被其他线程打断。  
可见性：确保同步代码块执行后，立即把修改的值刷新到主内存中。  
可重入性：同一个线程可以重复获取锁，也就是说同一个对象里面的
同步方法，可以相互调用。  
不可中断性：一旦线程在等待获取synchronized锁，那么其他线程就无法获取该锁，
除非等待锁的线程释放锁。和ReentrantLock不同，ReentrantLock可以中
断等待锁的线程。    

## HashMap，jdk1.8和jdk1.7的区别
jdk1.7是数组+链表，jdk1.8是数组+链表+红黑树，当链表长度大于8时，链表会
转换为红黑树。  
jdk1.7的头插法，在多线程环境下，可能会导致死循环，jdk1.8改为了尾插法。  
死循环过程：  
主要是在扩容的时候，多线程同时扩容，导致链表成环。  
头插法会导致链表逆序，扩容时，链表的顺序会颠倒，比如A->B->C，扩容后，
A->C->B，这样在扩容后，A->C->B，B->A，这样就形成了环。  
jdk1.8的尾插法，扩容后，链表的顺序不会颠倒，不会形成环。但是任然有数据覆盖的问题。  

## 对象内存布局
对象内存布局分为三部分：对象头、实例数据、对齐填充。  
对象头包括两部分：Mark Word和类型指针。Mark Word记录了对象的hashcode、锁状态、
GC分代年龄、线程持有的锁、偏向线程ID、偏向时间戳等。类型指针指向对象的类元数据，
用来确定该对象是哪个类的实例。   
因此，当对象使用锁时，不要轻易使用对象的hashCode 方法，因为这会破坏锁的级别。  

## ConcurrentHashMap
### JDK 1.7：分段锁
底层是 Segment[] + HashEntry[]，Segment 继承 ReentrantLock。
默认 16 个 Segment，并发度等于 Segment 数量，锁粒度是 Segment 级别。  
### JDK 1.8：CAS + synchronized
底层改为 Node[] + 链表/红黑树，与 HashMap 结构一致。
锁粒度降到桶（Node）级别：  
* 空桶：CAS 写入（无锁）。
* 非空桶：synchronized 锁住链表/红黑树的头节点。  
size 计算：用 CounterCell[] 分段计数（类似 LongAdder），减少 CAS 竞争。  
注意：ConcurrentHashMap 不允许 null key 和 null value，而 HashMap 允许。
因为 null 在并发场景下有歧义（get 返回 null 无法区分是"不存在"还是"值为 null"）。  

## ArrayList
底层是 Object[] 数组，默认初始容量 10（首次 add 时初始化）。  
扩容：新容量 = 旧容量 + 旧容量 >> 1（即 1.5 倍），通过 Arrays.copyOf 复制。  
* 随机访问 O(1)，尾部插入均摊 O(1)，中间插入/删除 O(n)。  
* vs LinkedList：LinkedList 底层是双向链表，理论上插入删除 O(1)，但实际需要先遍历到
  位置 O(n)，且每个节点有额外指针开销，缓存局部性差。实际开发中几乎总是用 ArrayList。  
* fail-fast：迭代过程中如果其他线程修改了列表结构（modCount 变化），会抛出
  ConcurrentModificationException。这是"尽力而为"的检测，不保证 100% 触发。  

## ThreadPoolExecutor
### 七个核心参数
* corePoolSize：核心线程数，即使空闲也不会回收（除非设置 allowCoreThreadTimeOut）。
* maximumPoolSize：最大线程数。
* keepAliveTime + unit：非核心线程的空闲存活时间。
* workQueue：任务队列，常用 LinkedBlockingQueue（无界）、ArrayBlockingQueue（有界）、
  SynchronousQueue（不存储，直接交接）。
* threadFactory：线程工厂，用于自定义线程名、是否守护线程等。
* handler：拒绝策略。  
### 执行流程
1. 线程数 < corePoolSize：创建新核心线程执行。
2. 线程数 >= corePoolSize：任务入队列。
3. 队列满：线程数 < maximumPoolSize 时创建非核心线程。
4. 队列满且线程数 >= maximumPoolSize：触发拒绝策略。  
### 四种拒绝策略
* AbortPolicy（默认）：抛出 RejectedExecutionException。
* CallerRunsPolicy：由提交任务的线程自己执行（降级保护，不会丢任务）。
* DiscardPolicy：静默丢弃。
* DiscardOldestPolicy：丢弃队列头部最老的任务，重新提交当前任务。  
### 线程池状态
RUNNING -> SHUTDOWN（shutdown()，处理完队列任务） ->
STOP（shutdownNow()，中断所有任务） -> TIDYING（所有任务终止，线程数为 0） ->
TERMINATED（terminated() 钩子方法执行完毕）。  
### 为什么不推荐 Executors 工具类
* FixedThreadPool / SingleThreadExecutor：使用 LinkedBlockingQueue（无界），
  队列堆积会导致 OOM。
* CachedThreadPool：maximumPoolSize = Integer.MAX_VALUE，可能创建大量线程导致 OOM。
* 推荐：手动 new ThreadPoolExecutor，使用有界队列 + 明确的线程数上限。  

## ReentrantLock 与 AQS
### AQS（AbstractQueuedSynchronizer）
AQS 是 JUC 的基石，ReentrantLock、Semaphore、CountDownLatch、ReentrantReadWriteLock
都基于它实现。  
核心结构：  
* state（volatile int）：同步状态，不同实现赋予不同含义。独占锁中 0=未锁定，>0=锁定次数。
* CLH 队列变种：FIFO 双向链表，存放等待获取锁的线程节点。  
### 公平锁 vs 非公平锁
* 公平锁：新线程获取锁前，先检查队列中是否有前驱节点。严格 FIFO，无饥饿，但吞吐量低。
* 非公平锁（默认）：新线程直接 CAS 尝试抢锁，抢不到再入队。吞吐量高，但可能导致队列
  中的线程长时间拿不到锁。  
### 加锁流程（非公平）
1. CAS 将 state 从 0 改为 1，成功则获取锁，记录当前线程。
2. state != 0（已被占用），判断是否当前线程持有（可重入），是则 state + 1。
3. 不是当前线程持有，封装为 Node 加入 CLH 队列，park 线程。
4. 前驱节点释放锁后 unpark 当前线程，再次尝试 CAS。  
### Condition
Condition 由 AQS 的 ConditionObject 实现，维护一个独立的等待队列（单向链表）。
await() 将线程从同步队列移到条件队列并 park，signal() 将条件队列头部线程移回同步队列。  

## ThreadLocal
### 核心结构
每个 Thread 对象内部有一个 ThreadLocalMap。  
ThreadLocalMap 的 Entry 继承 WeakReference<ThreadLocal>，key 是 ThreadLocal 的弱引用，
value 是实际的值（强引用）。  
### 内存泄漏
当 ThreadLocal 对象没有外部强引用时，GC 会回收 key（弱引用），但 value 仍然被 Entry
强引用，无法回收。如果线程不销毁（如线程池中的线程），value 就会一直驻留。  
解决：使用完 ThreadLocal 后，在 finally 中调用 remove() 清除 Entry。  
### InheritableThreadLocal
父线程的值可以传递给子线程。原理是子线程创建时，会拷贝父线程的 InheritableThreadLocalMap。  
但线程池场景下线程复用，无法自动传递，需要用 TransmittableThreadLocal（阿里开源）。  

## String 的不可变性
String 被 final 修饰，内部 value 数组也是 final（JDK 1.8 是 char[]，1.9+ 是 byte[]）。  
不可变的好处：  
* 线程安全：不可变对象天然线程安全，无需同步。
* hashCode 缓存：hashCode 只需计算一次并缓存，适合作为 HashMap 的 key。
* 字符串常量池：不可变才能安全地复用常量池中的对象，减少内存。
* 安全性：类加载、网络连接、反射等场景中，String 作为参数不可被篡改。  
StringBuilder vs StringBuffer：  
* StringBuilder：非线程安全，性能好，单线程拼接首选。
* StringBuffer：用 synchronized 修饰，线程安全，性能较差。  
编译期优化：`"a" + "b"` 在编译期会被优化为 `"ab"`（常量折叠）。
循环拼接时 `s += "x"` 会被编译为 `s = new StringBuilder().append(s).append("x").toString()`，
每次循环都创建 StringBuilder 对象，应手动用 StringBuilder。

