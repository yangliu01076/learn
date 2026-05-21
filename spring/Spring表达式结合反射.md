## Spring表达式结合反射
可以通过反射获取方法的参数名，然后通过Spring表达式获取参数的值，然后可以用注解
实现方法加锁。

```java
@Slf4j
@Order(Ordered.HIGHEST_PRECEDENCE+10000)
@Aspect
@Component
public class RateLimitAspect {

	@Resource
	@Qualifier("commonBatLockFactory")
	private BatLockFactory commonBatLockFactory;

	private final static String LOCK_KEY_TEMPLATE = "express-provider:%s:%s";

	private final ExpressionParser parser = new SpelExpressionParser();

	private final LocalVariableTableParameterNameDiscoverer discoverer =
			new LocalVariableTableParameterNameDiscoverer();


	@Pointcut("@annotation(RateLimit)")
	public void rateLimitPointCut() {
	}


	@Around("rateLimitPointCut() && @annotation(rateLimit)")
	public Object around(ProceedingJoinPoint pjp, RateLimit rateLimit) throws Throwable {
		log.info("[RateLimitCheck]start check... rateLimit=[{}]", JSON.toJSONString(rateLimit));
		if (rateLimit == null || StringUtils.isBlank(rateLimit.fetchBizIdSplExpress())
				|| StringUtils.isBlank(rateLimit.rateLimitConfigEnum().getBizType()) ||
				rateLimit.rateLimitConfigEnum().getCoolTimeOfSeconds() == null) {
			return pjp.proceed();
		}
		Signature signature = pjp.getSignature();
		MethodSignature methodSignature = (MethodSignature) signature;
		Method method = methodSignature.getMethod();
		Object[] args = pjp.getArgs();
		String bizId = parseBizId(rateLimit, method, args);
		if (StringUtils.isBlank(bizId)) {
			return pjp.getSignature();
		}
		String key = String.format(LOCK_KEY_TEMPLATE, rateLimit.rateLimitConfigEnum().getBizType(), bizId);
		log.info("[RateLimitCheck]doCheck... key=[{}]", key);
		final BatLock batLock = commonBatLockFactory.build(key);
		try {
			boolean lockSuccess = batLock.tryLock(rateLimit.rateLimitConfigEnum().getCoolTimeOfSeconds(),
					TimeUnit.SECONDS);
			if (!lockSuccess) {
				log.warn("[RateLimitCheck] 操作太频繁了!");
				throw new ExpressProviderException(Errors.OPERATE_FREQUENTLY);
			}
			return pjp.proceed();
		} finally {
			batLock.unlock();
		}
	}

	@Nullable
	public String parseBizId(RateLimit rateLimit, Method method, Object[] args) {
		String expressionString = rateLimit.fetchBizIdSplExpress();
		if (StringUtils.isBlank(expressionString)) {
			return null;
		}
		String[] paraNameArr = discoverer.getParameterNames(method);
		if (paraNameArr == null || paraNameArr.length == 0) {
			return null;
		}
		EvaluationContext context = new StandardEvaluationContext();
		for (int i = 0; i < paraNameArr.length; i++) {
			context.setVariable(paraNameArr[i], args[i]);
		}
		Expression expression = parser.parseExpression(expressionString);
		return expression.getValue(context, String.class);
	}


}
```