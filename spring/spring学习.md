Spring 只会操作和管理交给它托管的类，就算是实现了 SpringBoot
生命周期接口的类也不行。  

# IoC 与 DI
* IoC（控制反转）：对象的创建和依赖关系由 Spring 容器管理，而非开发者手动 new。
  核心思想是"好莱坞原则"——不要打电话给我们，我们会打电话给你。
* DI（依赖注入）：IoC 的具体实现方式。容器在运行时将依赖注入到对象中。
  注入方式：构造器注入（推荐）、Setter 注入、字段注入（@Autowired/@Resource）。  

# AOP
面向切面编程，将横切关注点（日志、事务、权限等）从业务逻辑中分离。  
核心概念：  
* JoinPoint（连接点）：可以被拦截的点，Spring 中仅方法执行。
* Pointcut（切点）：匹配连接点的表达式（如 execution、annotation）。
* Advice（通知）：在切点处执行的逻辑，分为 Before、After、AfterReturning、
  AfterThrowing、Around。
* Aspect（切面）：Pointcut + Advice 的组合。  
实现原理：  
* 接口有实现：JDK 动态代理（Proxy.newProxyInstance）。
* 接口无实现（类）：CGLIB 代理（生成子类，final 方法无法代理）。  
Spring Boot 2.0+ 默认使用 CGLIB（proxyTargetClass=true）。  

# 循环依赖
Spring 通过三级缓存解决单例 Bean 的循环依赖（A 依赖 B，B 依赖 A）：  
* singletonObjects（一级缓存）：存放完全初始化好的 Bean。
* earlySingletonObjects（二级缓存）：存放提前暴露的半成品 Bean（已实例化未初始化）。
* singletonFactories（三级缓存）：存放 ObjectFactory，用于生成半成品 Bean
  的早期引用（可能需要 AOP 代理）。  
流程：A 实例化后，将 ObjectFactory 放入三级缓存 -> A 注入 B -> B 实例化 ->
B 注入 A -> 从三级缓存获取 A 的 ObjectFactory，生成早期引用放入二级缓存 ->
B 完成初始化 -> A 注入 B 完成 -> A 完成初始化。  
注意：构造器循环依赖无法解决（实例化阶段就需要依赖，无法提前暴露）。  

# 事务传播行为
| 传播行为 | 说明 |
|:---|:---|
| REQUIRED（默认） | 有事务就加入，没有就新建 |
| REQUIRES_NEW | 总是新建事务，挂起当前事务 |
| NESTED | 有事务就创建嵌套事务（子事务回滚不影响父事务），没有就新建 |
| SUPPORTS | 有事务就加入，没有就非事务执行 |
| NOT_SUPPORTED | 非事务执行，挂起当前事务 |
| MANDATORY | 必须在事务中，否则抛异常 |
| NEVER | 必须非事务，否则抛异常 |
