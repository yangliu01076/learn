InitializingBean 是在当前 Bean 自己的属性注入完成后，
对当前 Bean 自己执行初始化逻辑（afterPropertiesSet()）。（在这个逻辑里，你当然
可以去使用那些已经注入进来的 Bean，但这只是手段，不是目的）。

## 三种初始化方式对比
Spring 提供三种方式在 Bean 属性注入完成后执行初始化逻辑：  
* @PostConstruct（JSR-250 注解）：由 CommonAnnotationBeanPostProcessor 解析执行。
  非侵入，推荐使用。
* InitializingBean.afterPropertiesSet()：Spring 原生接口，侵入性强（耦合 Spring API）。
  框架内部组件常用，业务代码不推荐。
* init-method（XML / @Bean(initMethod=...)）：配置方式，最灵活，但不够直观。  

## 执行顺序
1. @PostConstruct
2. InitializingBean.afterPropertiesSet()
3. init-method  
这个顺序在 InitializeBean 的 invokeInitMethods 方法中写死。  

## 销毁顺序
与初始化对称：  
1. @PreDestroy
2. DisposableBean.destroy()
3. destroy-method  

## 使用建议
* 优先用 @PostConstruct，最简洁，不耦合 Spring。
* 如果初始化逻辑需要访问 Spring 基础设施（如 BeanFactory），可考虑 InitializingBean。
* init-method 适合无法修改源码的第三方 Bean（通过 XML 或 @Bean 配置）。