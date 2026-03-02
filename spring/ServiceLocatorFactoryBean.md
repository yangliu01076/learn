# ServiceLocatorFactoryBean
## 1 用法
1. 创建一个 ServiceLocatorFactoryBean 对象，
   并设置 locator 属性为 ServiceLocator 接口的实现类。
   例如：
``` java
    @Bean("reportSettingHandlerFactory")
    public ServiceLocatorFactoryBean changeOrgParentTaskBusHandlerFactoryBean() {
        ServiceLocatorFactoryBean factoryBean = new ServiceLocatorFactoryBean();
        factoryBean.setServiceLocatorInterface(ReportSettingHandlerFactory.class);
        factoryBean.setServiceLocatorExceptionClass(ReportSettingHandlerFactory.ReportSettingHandlerFactoryException.class);
        factoryBean.setServiceMappings(createServiceMappings());
        return factoryBean;
    }
```
2. 将 ServiceLocatorFactoryBean 对象注入到需要使用
   ServiceLocator 接口的类中。
3. 在需要使用 ServiceLocator 接口的类中，
   使用 @Autowired 注解注入 ServiceLocatorFactoryBean 对象。
4. 使用 ServiceLocator 接口的任何普通接口方法获取 ServiceLocator
   接口的实现类。
## 2 原理
ServiceLocatorFactoryBean 是 Spring 框架中的一个工厂类，
用于创建 ServiceLocator 接口的实现类。  
他会创建接口的实现类，拦截所有接口中的方法，根据入参的toString()方法的值，到
serviceMappings 中查找对应的实现类的beanName，然后调用getBean()方法获取
对应的bean，返回给接口的方法的调用方。  
因此，主需要实现 createServiceMappings 方法 ，并且将对应beanName 的 bean
注入到 IoC容器中即可

## 3 运行流程详解
整个生命周期分为三个阶段：初始化阶段、代理生成阶段、方法调用阶段。

### 阶段一：属性注入与初始化
1. Spring 容器启动，实例化 ServiceLocatorFactoryBean。
2. 通过 setServiceLocatorInterface 传入你定义的接口（例如
   ServiceFactory.class）。
3. 通过 setBeanFactory 传入当前的 Spring 容器（ListableBeanFactory ）
   ，这里它把容器存了起来，为了以后找 Bean 用。
4. （可选）通过 setServiceMappings 设置 ID 映射关系（比如
   传 ID "a"，实际找名叫 "beanA" 的 Bean）。
### 阶段二：生成代理
这是核心逻辑，发生在 afterPropertiesSet 方法中：

``` java
public void afterPropertiesSet() {
// 1. 校验接口不能为空
if (this.serviceLocatorInterface == null) {
throw new IllegalArgumentException("Property 'serviceLocatorInterface' is required");
}

    // 2. 创建动态代理
    // 关键点：使用 JDK 动态代理
    this.proxy = Proxy.newProxyInstance(
            this.serviceLocatorInterface.getClassLoader(), // 类加载器
            new Class<?>[] {this.serviceLocatorInterface}, // 代理要实现的接口
            new ServiceLocatorInvocationHandler());        // 调用处理器（核心逻辑）
}
```
此时，ServiceLocatorFactoryBean 已经准备好了一个代理对象。

### 阶段三：业务调用
当你的业务代码调用接口方法时，比如 
myServiceFactory.getService("specialService")，
JDK 会拦截这个调用，转交给 ServiceLocatorInvocationHandler
的 invoke 方法。  

invoke 方法的处理逻辑：  

1. 检查常规方法：  
如果是 equals、hashCode、toString 等通用方法，直接处理，不走查找逻辑。  
执行查找逻辑 (invokeServiceLocatorMethod)：  
获取返回类型：通过反射拿到接口方法的返回值类型（例如 MyService.class），
这决定了要从容器里拿什么类型的 Bean。   
2. 获取 Bean 名称 (tryGetBeanName)：  
如果方法无参（getService()），Bean 名称为空。  
如果方法有参（getService(String id)），调用参数的 toString()
方法作为 Bean 名称。  
如果配置了 serviceMappings，会用拿到的名称去映射表里找真正的 BeanName。
3. 从容器获取：  
如果有 Bean 名称：调用 beanFactory.getBean(beanName, returnType)。  
如果没有 Bean 名称（无参方法）：调用 beanFactory.getBean(returnType)
（按类型查找）。  
4. 异常处理：
如果找不到 Bean，Spring 会抛出 BeansException。   
如果用户配置了自定义异常类（setServiceLocatorExceptionClass），
这里会通过反射将 Spring 异常包装成自定义异常抛出。  