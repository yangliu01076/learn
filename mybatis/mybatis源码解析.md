## 四大核心组件
Executor、StatementHandler、ParameterHandler、ResultSetHandler

### Executor（执行器）
MyBatis 的调度器，负责一级缓存管理、事务提交/回滚、并委托 StatementHandler 执行 SQL。  
四种实现：  
* SimpleExecutor（默认）：每次执行创建新 Statement，用完关闭。
* ReuseExecutor：缓存 Statement 对象（按 SQL 文本），相同 SQL 复用 Statement。
* BatchExecutor：批量执行，调用 addBatch() + executeBatch()，适合大量 INSERT/UPDATE。
* CachingExecutor（装饰器）：包装上述任意 Executor，在查询前先查二级缓存。  
执行入口：Executor.query() / Executor.update()。  
一级缓存在 BaseExecutor 中实现（PerpetualCache + HashMap），key 为
Statement ID + 参数 + 分页信息 + SQL 的组合。  

### StatementHandler（语句处理器）
负责创建 Statement 对象、设置参数、执行 SQL。  
* RoutingStatementHandler：路由入口，根据 MappedStatement 的 statementType
  选择具体 Handler。
* PreparedStatementHandler（最常用）：预编译 SQL，参数化查询，防止 SQL 注入。
* SimpleStatementHandler：普通 Statement，用于无参数的静态 SQL。
* CallableStatementHandler：调用存储过程。  
核心方法：prepare() 创建 Statement，parameterize() 委托 ParameterHandler 设参，
  query() / update() 执行 SQL。  

### ParameterHandler（参数处理器）
为 PreparedStatement 设置参数，完成 Java 类型到 JDBC 类型的转换。  
核心方法：setParameters(PreparedStatement ps)。  
遍历 ParameterMapping 列表，对每个参数：  
1. 从参数对象中根据属性路径（如 `user.name`）获取值。
2. 查找对应的 TypeHandler（类型处理器），将 Java 值设置到 PreparedStatement 的
   指定位置（setInt、setString 等）。  
TypeHandler 的映射关系在 TypeHandlerRegistry 中维护，也可自定义（如加密/脱敏）。  

### ResultSetHandler（结果集处理器）
将 JDBC ResultSet 映射为 Java 对象，是 MyBatis 最复杂的组件。  
核心方法：handleResultSets()。  
映射方式：  
* 自动映射：列名与属性名按驼峰规则对应（需开启 mapUnderscoreToCamelCase）。
* ResultMap：显式指定列与属性的映射关系，支持类型转换。
* 嵌套映射：association（一对一）、collection（一对多），通过嵌套查询或嵌套结果实现。  
处理流程：遍历 ResultSet 的每一行 -> 根据 ResultMap 找到目标类型 -> 创建对象
（ObjectFactory）-> 逐列映射属性（TypeHandler 转换）-> 处理嵌套关联 -> 
加入结果列表。  
懒加载：嵌套查询支持懒加载（aggressiveLazyLoading=false 时），通过动态代理
拦截 getter 方法，首次访问时才执行子查询。  
