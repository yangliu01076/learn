在依赖注入阶段，Spring会遍历所有的BeanPostProcessor执行
postProcessAfterInitialization方法。
1. 只要实现了 BeanPostProcessor：Spring 容器启动时，会检测到这个
类，并把它视为一个“后置处理器”。  
2. 对容器中所有的 bean：除了 Spring 内部的一些核心基础设施 Bean 外，
每一个 你定义的 @Component、@Service、@Bean 初始化完成后，都会经过这一步。
3. 完成初始化之后：也就是在属性填充完、@PostConstruct 注解的方法执行完、
InitializingBean 的 afterPropertiesSet 执行完之后。  
4. 执行 postProcessAfterInitialization：Spring 会拿着这个
Bean 去调用这个方法。  
入参：  
Object bean：就是这个 Bean 的对象引用（也就是实例本身）。  
String beanName：就是这个 Bean 在 Spring 容器里的名字（默认是类
名首字母小写，或者通过 @Service("xxx") 指定的名字）。