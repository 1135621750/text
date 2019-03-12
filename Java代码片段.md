## Java代码片段

#### list操作，map转换

``` java
//提取对象中的某个字段变为数组
List<User> user = new ArrayList<>();
List<String> collect = user.stream().map(User::getUserName).collect(Collectors.toList());

//对象转换
List<User> user = new ArrayList<>();
List<UserNew> collect = user.stream.map(this::parse).collect(Collectors.toList());
public UserNew parse(User user){
    UserNew userNew = UserNew();
    userNew.setUserName(user.getUserName);
    return userNew;
}

//过滤对象
List<User> user = new ArrayList<>();
List<User> collect = user.stream().filter(u -> u.getAge().equals("12")).collect(Collectors.toList());

//转换类型
List<Integer> transform = Lists.transform(parm.getAge(), new Function<String, Integer>() {
    @Nullable
    @Override
    public Integer apply(@Nullable String input) {
        return Integer.valueOf(input);
    }
});

```

#### 计算1.8

``` java
List<User> list = Lists.newArrayList(
    new User(123,new BigDecimal(200).setScale(BigDecimal.ROUND_HALF_UP,2)),
    new User(456,new BigDecimal(100).setScale(BigDecimal.ROUND_HALF_UP,2))
);
//Comparator 对比值，最后获得的是整个对象Optional<User>
list.stream().max((u1, u2) -> u1.getHeight().compareTo(u2.getHeight()));
//1.8map方式获取的是BigDecimal的值，包括int，long double，long等算数类型都可以
list.stream().map(User::getHeight).reduce(BigDecimal.ZERO, BigDecimal::add);
list.stream().map(User::getId).reduce(Integer::sum);
//mapToInt方式，针对int，long double，long等算数类型都可以
list.stream().mapToInt(User::getId).sum();

```

#### 事件监听器

``` java
//spring事件监听器（默认同步也可异步）
private class MyEvent extends ApplicationEvent {//创建
    public MyEvent(Object source) {
        super(source);
    }
}
//@Async 添加此注解实现异步，默认为同步
@EventListener(MyEvent.class)
public void addDept(MyEvent myEvent) {
    Object object = myEvent.getSource();
	//处理业务逻辑
}
@Resource
private ApplicationEventPublisher eventPublisher;//调用方式
eventPublisher.publishEvent(new MyEvent(t));
```

#### 注解

``` java
//注解定义
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation {
    boolean check() default true;//默认参数
}
@Aspect
@Component
@Order(2)//切面顺序，越小执行越早
public class MyAnnotationAspect {

    @Around("@annotation(myAnnotation)")
    public Object around(ProceedingJoinPoint pjp, MyAnnotation myAnnotation) throws Throwable {
        //处理业务逻辑
        return pjp.proceed();
    }

}
```

#### 反射

``` java
//使用Hutool工具，反射对象中的方法，如果存在方法则操作
private void createDefaultParam(Object o){
    if(ObjectUtil.isNotNull(ReflectUtil.getMethod(o.getClass(), "setUserId"))){
        ReflectUtil.invoke(o,"setUserId","x-l-h-y-x-z");
    }
}
```

#### 切面

``` java
@Aspect
@Component
@Order(1)
public class LogAspect {
    private Logger logger = LoggerFactory.getLogger(getClass().getName());

    @Pointcut("execution(public * com.*.*.controller.*.*(..))")
    public void webLog() {
    }

    @Before("webLog()")
    public void doBefore(JoinPoint joinPoint) throws Throwable {
		//在方法调用之前调用通知
    }

    @AfterReturning(returning = "ret", pointcut = "webLog()")
    public void doAfterReturning(Object ret) throws Throwable {
        // 处理完请求，返回内容
    }

    @AfterThrowing(throwing = "ex", pointcut = "webLog()")
    public void doAfterThrowing(JoinPoint jp, Exception ex) {
         //后置异常通知，方法返回的异常，必须是Controller标注的类中方法，且没有捕获异常
    }

    @After("webLog()")
    public void doAfter(JoinPoint jp) {
		//后置最终通知,final增强，不管是抛出异常或者正常退出都会执行
    }
    
    @Around("webLog()")
    public Object around(ProceedingJoinPoint pjp){
        //环绕通知，方法调用前调用后执行
        Object[] objects = pjp.getArgs();//取得方法入参
        return pjp.proceed();//放行
    }
}
//AspectJ指示器
//arg ()	限制连接点的指定参数为指定类型的执行方法
//@args ()	限制连接点匹配参数由指定注解标注的执行方法
//execution ()	用于匹配连接点的执行方法
//this ()	限制连接点匹配 AOP 代理的 Bean 引用为指定类型的类
//target ()	限制连接点匹配特定的执行对象，这些对象对应的类要具备指定类型注解
//within()	限制连接点匹配指定类型
//@within()	限制连接点匹配指定注释所标注的类型（当使用 Spring AOP 时，方法定义在由指定的注解所标注的类里）
//@annotation	限制匹配带有指定注释的连接点

```

#### 计时器

``` java
public class TimeInterval {//需要Hutool
	private long time;
	private boolean isNano;

	public TimeInterval() {
		this(false);
	}

	public TimeInterval(boolean isNano) {
		this.isNano = isNano;
		start();
	}

	/**
	 * @return 开始计时并返回当前时间
	 */
	public long start() {
		time = DateUtil.current(isNano);
		return time;
	}

	/**
	 * @return 重新计时并返回从开始到当前的持续时间
	 */
	public long intervalRestart() {
		long now = DateUtil.current(isNano);
		long d = now - time;
		time = now;
		return d;
	}
	
	/**
	 * 重新开始计算时间（重置开始时间）
	 * @return this
	 * @since 3.0.1
	 */
	public TimeInterval restart(){
		time = DateUtil.current(isNano);
		return this;
	}

	//----------------------------------------------------------- Interval
	/**
	 * 从开始到当前的间隔时间（毫秒数）<br>
	 * 如果使用纳秒计时，返回纳秒差，否则返回毫秒差
	 * @return 从开始到当前的间隔时间（毫秒数）
	 */
	public long interval() {
		return DateUtil.current(isNano) - time;
	}
	
	/**
	 * 从开始到当前的间隔时间（毫秒数）
	 * @return 从开始到当前的间隔时间（毫秒数）
	 */
	public long intervalMs() {
		return isNano ? interval() / 1000000L : interval();
	}
	
	/**
	 * 从开始到当前的间隔秒数，取绝对值
	 * @return 从开始到当前的间隔秒数，取绝对值
	 */
	public long intervalSecond(){
		return intervalMs() / DateUnit.SECOND.getMillis();
	}
	
	/**
	 * 从开始到当前的间隔分钟数，取绝对值
	 * @return 从开始到当前的间隔分钟数，取绝对值
	 */
	public long intervalMinute(){
		return intervalMs() / DateUnit.MINUTE.getMillis();
	}
	
	/**
	 * 从开始到当前的间隔小时数，取绝对值
	 * @return 从开始到当前的间隔小时数，取绝对值
	 */
	public long intervalHour(){
		return intervalMs() / DateUnit.HOUR.getMillis();
	}
	
	/**
	 * 从开始到当前的间隔天数，取绝对值
	 * @return 从开始到当前的间隔天数，取绝对值
	 */
	public long intervalDay(){
		return intervalMs() / DateUnit.DAY.getMillis();
	}
	
	/**
	 * 从开始到当前的间隔周数，取绝对值
	 * @return 从开始到当前的间隔周数，取绝对值
	 */
	public long intervalWeek(){
		return intervalMs() / DateUnit.WEEK.getMillis();
	}
/**
 * 日期时间单位，每个单位都是以毫秒为基数
 */
public enum DateUnit {
	/** 一毫秒 */
	MS(1), 
	/** 一秒的毫秒数 */
	SECOND(1000), 
	/**一分钟的毫秒数 */
	MINUTE(SECOND.getMillis() * 60),
	/**一小时的毫秒数 */
	HOUR(MINUTE.getMillis() * 60),
	/**一天的毫秒数 */
	DAY(HOUR.getMillis() * 24),
	/**一周的毫秒数 */
	WEEK(DAY.getMillis() * 7);
	
	private long millis;
	DateUnit(long millis){
		this.millis = millis;
	}
	
	/**
	 * @return 单位对应的毫秒数
	 */
	public long getMillis(){
		return this.millis;
	}
}
}
```

#### id生成

``` java
/**
 * 主键生成工具类，基于snowflake算法，结合实际情况，workId取IP地址后18bit位，自增序列号4bit.
 * 在测试中未发现id重复，是线程安全的
 */
@Slf4j
public class IdWorker {

    private final long workerId;
    private static final long TWEPOCH = 1483200000000L;
    private long sequence;
    private static final long workerIdBits = 18L;
    private static final long maxWorkerId = 262143L;
    private static final long sequenceBits = 4L;
    private static final long workerIdShift = 4L;
    private static final long timestampLeftShift = 22L;
    private static final long sequenceMask = 15L;
    private long lastTimestamp;
    private static IdWorker self = new IdWorker();



    public static IdWorker getInstance() {
        return self;
    }
    /**
     * 初始化
     */
    private IdWorker()
    {
        sequence = 0L;
        lastTimestamp = -1L;
        long workerId = -1L;
        try
        {
            workerId = generateWorkerId();
        }
        catch(Exception e)
        {
            log.error((new StringBuilder()).append("Occur an error when initializing the IdWorker because ")
                    .append(e.getMessage()).toString(), e);
        }
        if(workerId > 262143L || workerId < 0L)
        {
            throw new IllegalArgumentException(String.format("worker Id can't be greater than %d or less than 0", new Object[] {
                    Long.valueOf(262143L)
            }));
        } else
        {
            this.workerId = workerId;
            return;
        }
    }
    /**
     * 下个id 默认18位
     *
     */
    public synchronized Long nextId(){
        long timestamp = now();
        if (timestamp < lastTimestamp)
            try {
                throw new Exception(
                        String.format(
                                "Clock moved backwards. Refusing to generate id for %d milliseconds",
                                new Object[] { Long.valueOf(lastTimestamp
                                        - timestamp) }));
            } catch (Exception e) {
                log.error("ID获取异常", e);
            }
        if (lastTimestamp == timestamp) {
            sequence = sequence + 1L & 15L;
            if (sequence == 0L)
                timestamp = tilNextMillis(lastTimestamp);
        } else {
            sequence = 0L;
        }
        lastTimestamp = timestamp;
        long nextId = (timestamp - 1483200000000L & 2199023255551L) << 22//长度
                | workerId << 4 | sequence;
        return new Long(nextId);
    }

    private long tilNextMillis(long lastTimestamp) {
        long timestamp;
        for (timestamp = now(); timestamp <= lastTimestamp; timestamp = now())
            ;
        return timestamp;
    }

    private long now() {
        return System.currentTimeMillis();
    }

    private long generateWorkerId() throws Exception {
        String hostAddress = Inet4Address.getLocalHost().getHostAddress();
        byte ipBytes[] = textToNumericFormatV4(hostAddress);
        return (long) (((ipBytes[1] & 3) << 16) + ((ipBytes[2] & 255) << 8) + (ipBytes[3] & 255));
    }
    /**
     * 将ipv4字符串转为长度为4的byte数组
     * @param ip ipv4字符串
     * @return byte数组
     */
    @SuppressWarnings("restriction")
    public static byte[] textToNumericFormatV4(String ip)
    {
        if(ip == null || "".equals(ip))
            //参数IP不能为空
            throw new IllegalArgumentException("the argument ip can't be empty.");
        if(!IPAddressUtil.isIPv4LiteralAddress(ip))
            //IPv4地址格式错误
            throw new IllegalArgumentException("wrong format of ipv4 address.");
        else
        return IPAddressUtil.textToNumericFormatV4(ip);
    }
}
```

