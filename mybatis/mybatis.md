# 1.整体架构
## 接口层
这是 MyBatis 对外暴露的 API 层。  
主要作用是和数据库进行交互，核心接口是 SqlSession。通过 SqlSession，
开发者可以调用 CRUD 方法（如 selectOne, insert 等）或直接获取 Mapper 
接口的代理对象。
## 数据处理层
这是 MyBatis 的核心层，负责强大的 SQL 映射功能。
* 参数映射：将 Java 对象转换为 SQL 预编译语句的参数。
* SQL 解析与执行：根据传入的 Statement ID 执行对应的 SQL。
* 结果映射：将 JDBC 返回的 ResultSet 转换为 Java 对象（List, Map, POJO）。
## 框架支撑层
负责最基础的功能支撑，包括连接管理、事务管理、配置加载、缓存处理等。
核心组件：Configuration（全局配置）、Executor（执行器）、
StatementHandler（语句处理）、ParameterHandler（参数处理）、
ResultSetHandler（结果处理）。
## 引导层
负责启动 MyBatis，构建配置环境。
核心类：SqlSessionFactoryBuilder，它读取 XML 配置或注解配置，
构建出 SqlSessionFactory。

# 2.核心组件
## SqlSession
SqlSession 是 MyBatis 的核心接口，它代表了一个数据库会话。  
通过 SqlSession，开发者可以执行 SQL 语句、获取映射器（Mapper）、
管理事务等。
注意：SqlSession 不是线程安全的，因此它的最佳作用域是方法体或请求
作用域内，用完必须关闭。

## Executor
Executor 是 MyBatis 的执行器，负责执行 SQL 语句。  
它有多种实现，如 SimpleExecutor、ReuseExecutor、BatchExecutor 等。
* SimpleExecutor：默认执行器，每次执行都创建一个新的 Statement。
* ReuseExecutor：复用 Statement 对象（针对相同 SQL）。
* BatchExecutor：批量执行，专门用于批量更新。

## MappedStatement
它是 Mapper 中一个 SQL 节点（如 < select id="selectUser" >）的封装。
包含了 SQL 语句、输入参数映射、输出结果映射、缓存配置等信息。

## Mapper Proxy
MyBatis 通过 JDK 动态代理 为 Mapper 接口生成代理对象。当你调
用 mapper.selectUser(1) 时，实际上是调用了代理对象的 invoke 方法。

## Configuration
MyBatis 的所有配置信息都封装在这个对象中（类似于 Spring 的 
ApplicationContext）。  
它包含了：数据库连接池配置、Mapper 文件信息（MappedStatement）、
TypeHandler（类型处理器）、插件等。  
这是一个单例对象，应用启动时生成，随应用生命周期存在。  

## SqlSessionFactory
工厂接口，负责生产 SqlSession。  
通常是单例的，因为创建 SqlSession 的成本较高（涉及配置解析、环
境准备等），但 SqlSession 本身是线程不安全的，所以每次请求都需要
创建一个新的 SqlSession。

# 3.核心流程
## 初始化阶段
1. 加载 mybatis-config.xml。
2. SqlSessionFactoryBuilder 解析 XML，将配置信息解析为 Configuration 对象。
3. 解析 Mapper 文件中的每个 SQL 语句，生成对应的 MappedStatement 对象，
并存入 Configuration 的 Map 中。
4. 创建 SqlSessionFactory 对象，持有 Configuration 引用。 
## 代理创建阶段
1. 调用 sqlSession.getMapper(UserMapper.class)。
2. MyBatis 使用 MapperProxyFactory 工厂，利用 JDK 动态代理 
生成一个 MapperProxy 代理对象。
3. 这个代理对象实现了 UserMapper 接口。 
## 数据执行阶段
假设代码执行：userMapper.selectById(1);
1. 拦截调用：由于 userMapper 是代理对象，调用会进入 
MapperProxy.invoke() 方法。
2. 查找 MappedStatement：
MapperProxy 内部会封装成一个 MapperMethod 对象。  
根据接口名（com.example.UserMapper）和方法名（selectById） 
拼接成 key（com.example.UserMapper.selectById）。  
去 Configuration 对象中查找对应的 MappedStatement 对象。
3. 执行 SQL：
MapperMethod 调用 sqlSession.selectList(statement, parameter)。
sqlSession 将请求委托给 Executor。
4. JDBC 处理：
Executor 创建 StatementHandler。  
StatementHandler 利用 ParameterHandler 对 SQL 参数进行预
处理（设置 ? 占位符）。  
StatementHandler 调用 JDBC 的 Statement.execute() 执行 SQL。
5. 结果映射：
数据库返回 ResultSet。
StatementHandler 利用 ResultSetHandler 将 ResultSet
映射成 Java 对象（通过反射）。  
返回结果：结果层层返回，最终到达业务代码。

# 4.核心机制
## 缓存机制
MyBatis 包含两级缓存：  
* 一级缓存（SqlSession 级别）：
默认开启。  
基于 HashMap，存储在 BaseExecutor 中。  
只要 SqlSession 没有关闭，且中间没有执行增删改操作（会清空缓存），
* 相同的查询直接从内存返回。
* 二级缓存（Mapper/Namespace 级别）：
默认关闭，需要配置。  
跨 SqlSession 共享。  
如果一个 SqlSession 查询后提交了，另一个 SqlSession 查询
相同数据可以直接命中二级缓存。  
其底层通常使用 CachingExecutor 装饰器模式来包装真正的 Executor。