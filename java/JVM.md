## 类加载器与双亲委派
如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，
而是把这个请求委派给父类加载器去完成，每一个层次的类加载器都是如此。  
启动类加载器：无法被 Java 程序直接引用，负责加载 JVM 的核心类库（如 rt.jar）。  
扩展类加载器：负责加载 JAVA_HOME/lib/ext 目录下的类。  
应用程序类加载器：负责加载用户类路径（ClassPath）上所指定的类。  

### 作用
避免类的重复加载。 如果父加载器已经加载过了，子加载器就没必要再加
载一次，保证了 Java 类在 JVM 中的全局唯一性。

### 打破双亲委派
重写loadClass方法，不调用父类的loadClass方法，而是自己加载。

## JVM 内存区域
* 堆（Heap）：存放对象实例，GC 的主区域。分代：新生代（Eden + Survivor0 + Survivor1，
  比例 8:1:1）+ 老年代。所有线程共享。
* 方法区：存放类元信息、常量池（JDK 1.7 从方法区移到堆，1.8 用 Metaspace 替代永久代，
  使用本地内存）。所有线程共享。
* 虚拟机栈：线程私有。每个方法调用创建一个栈帧（局部变量表 + 操作数栈 + 动态链接 +
  返回地址）。-Xss 设置栈大小，默认 512KB~1MB。
* 本地方法栈：为 Native 方法服务，与虚拟机栈类似。
* 程序计数器（PC Register）：线程私有，记录当前执行的字节码行号。唯一不会 OOM 的区域。  

对象分配策略：  
* 优先分配到 Eden。
* 大对象直接进入老年代（-XX:PretenureSizeThreshold）。
* 长期存活的对象进入老年代（MaxTenuringThreshold，默认 15）。
* 动态年龄判断：Survivor 中相同年龄对象大小超过 Survivor 空间一半，该年龄及以上直接晋升。  

## GC 算法
* 标记-清除：标记存活对象，清除其余。缺点：产生内存碎片。
* 复制：将存活对象复制到另一半空间，清空原空间。新生代用此算法（Eden + Survivor）。
  缺点：浪费一半空间，但新生代 98% 对象朝生夕死，实际浪费不大。
* 标记-整理：标记存活对象，向一端移动，清除边界外。老年代用此算法。无碎片但慢。
* 分代收集：新生代用复制算法，老年代用标记-清除或标记-整理。  

## GC Roots
GC 从 GC Roots 开始可达性分析，不可达的对象即为垃圾。GC Roots 包括：  
* 虚拟机栈中引用的对象（局部变量）。
* 方法区中静态属性引用的对象。
* 方法区中常量引用的对象。
* 本地方法栈中 JNI 引用的对象。
* 被同步锁（synchronized）持有的对象。  

## GC 收集器
| 收集器 | 作用区域 | 算法 | 特点 |
|:---|:---|:---|:---|
| Serial / Serial Old | 新生代 / 老年代 | 复制 / 标记整理 | 单线程，STW，适合客户端 |
| ParNew / Parallel Scavenge | 新生代 | 复制 | 多线程并行，适合服务端 |
| Parallel Old | 老年代 | 标记整理 | 配合 Parallel Scavenge |
| CMS | 老年代 | 标记清除 | 低延迟，4 步：初始标记(STW) -> 并发标记 -> 重新标记(STW) -> 并发清除。缺点：碎片、浮动垃圾、Concurrent Mode Failure |
| G1 | 全堆 | 标记整理 + 复制 | Region 化（默认 2048 个），可预测停顿（-XX:MaxGCPauseMillis），混合回收。JDK 9 默认 |
| ZGC | 全堆 | 染色指针 + 读屏障 | 并发整理，停顿 < 10ms，适合大堆（TB 级）。JDK 11 引入 |

### CMS 的 Concurrent Mode Failure
CMS 在并发标记/清除阶段，老年代空间不足以分配新对象时，会退化为 Serial Old
做 Full GC（长时间 STW）。通过调高 -XX:CMSInitiatingOccupancyFraction 预留空间。  

### G1 的 Region
G1 将堆分为大小相等的 Region（1~32MB），每个 Region 可以是 Eden、Survivor、Old 或
Humongous。G1 维护每个 Region 的垃圾价值（回收空间/耗时），优先回收价值高的 Region
（Garbage First），从而在有限时间内回收最多垃圾。