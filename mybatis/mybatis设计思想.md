## 缓存占位符
在执行sql的时候，会先在缓存里面塞个占位符，然后执行sql，执行完之后，再替换占位符。
防止循环依赖。  
具体来说，CachingExecutor 在查询二级缓存时，如果缓存未命中，会先放入一个占位符
（EXECUTION_PLACEHOLDER），再委托delegate执行实际查询。如果查询过程中又
触发了相同key的查询，命中占位符说明发生了循环依赖，直接跳过，避免无限递归。  

## 动态代理模式
MyBatis 的核心设计：Mapper 接口无需实现类，通过 JDK 动态代理生成代理对象。  
MapperProxy 实现 InvocationHandler，拦截接口方法调用，将其转化为对
SqlSession 的 CRUD 调用。  
优势：接口即配置，开发者只定义接口和 XML/注解，无需写实现类。  

## 模板方法模式
BaseExecutor 定义了查询和更新的骨架流程（一级缓存检查、缓存清除等），
将具体执行逻辑延迟到子类：  
* doQuery() / doUpdate() 由 SimpleExecutor、ReuseExecutor、BatchExecutor
  各自实现。  
子类只关注"怎么执行 SQL"，缓存管理等通用逻辑由父类统一控制。  

## 装饰器模式
CachingExecutor 装饰 BaseExecutor，在不改变原有执行逻辑的基础上增加二级缓存能力。  
CachingExecutor 持有一个 delegate（实际的 Executor），查询时先查二级缓存，
未命中再委托 delegate 查询，查到后回填二级缓存。  
同样，LruCache、BlockingCache 等缓存实现也是装饰器，层层包装 PerpetualCache。  

## 建造者模式
* SqlSessionFactoryBuilder -> XMLConfigBuilder -> Configuration，分步解析
  XML 配置并构建 Configuration 对象。
* MappedStatement.Builder、ResultMapping.Builder 等内部 Builder 类，
  分步设置参数构建不可变对象。  

## 责任链模式
InterceptorChain 管理所有插件（Interceptor）。pluginAll() 方法遍历所有 Interceptor，
对目标对象层层调用 Interceptor.wrap()，生成嵌套的代理对象。  
执行时形成洋葱模型：最外层插件先进入，最内层（真实方法）执行后，逐层返回。  
这也是 PageHelper、分表插件等的核心原理。  

## 工厂模式
* ObjectFactory：创建结果集映射的目标对象，默认 DefaultObjectFactory 通过
 反射调用无参构造器。可自定义 ObjectFactory 实现依赖注入或特殊实例化逻辑。
* SqlSessionFactory：工厂接口，生产 SqlSession。  
* MapperProxyFactory：为 Mapper 接口创建代理对象的工厂。  

## 组合模式
动态 SQL 的 SqlNode 体系：MixedSqlNode 持有 List<SqlNode>，IfSqlNode、
WhereSqlNode、ForEachSqlNode 等都是 SqlNode 的实现。  
apply() 方法递归调用子节点的 apply()，最终拼出完整的 SQL 字符串。  
XML 中的 `<if>`、`<where>`、`<foreach>` 嵌套结构天然适合组合模式。  