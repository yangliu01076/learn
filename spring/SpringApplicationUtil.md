SpringApplicationUtil是一个 Spring 应用上下文工具类，
用于在非 Spring 托管的类中获取 Spring 管理的 Bean。  

```java
@Component
public class SpringApplicationUtil implements ApplicationContextAware {
	private static ApplicationContext APPLICATION_CONTEXT;

	@Override
	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
		APPLICATION_CONTEXT = applicationContext;
	}

	public static <T> T findBean(Class<T> beanClazz) {
		return APPLICATION_CONTEXT.getBean(beanClazz);
	}

	public static <T> T findBean(String beanName, Class<T> beanClazz) {
		return APPLICATION_CONTEXT.getBean(beanName, beanClazz);
	}
}
```
implements ApplicationContextAware：实现 Spring 的回调接口，
当 Bean 初始化后，Spring 会自动注入 ApplicationContext。  



