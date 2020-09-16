# blog
## springboot
### create bean

className | details
--- | ---
AbstractProcessor | 编译期注解
AnnotationMetaData | 获取指定类所有注解接口
ApplicationContextAware | 获取 application 容器(applicationContext对象)
ApplicationListener | 监听事件
BeanDefinitionRegistry | 将持有的类注册信息注册成 bean
BeanDefinitionRegistryPostProcessor | 获取、修改、注册、移除 bean 原数据
BeanFactory | IOC最基本的容器，负责生产和管理bean，它为其他具体的IOC容器提供了最基本的规范
BeanFactoryPostProcessor | 在容器实际实例化任何其它的bean之前读取配置元数据，并有可能修改它
BeanPostProcessor | bean 后置处理器，在 bean 实例化前后使用, BeanDefinition 做参数进行操作 
ClassPathBeanDefinitionScanner | 路径下扫描 bean 定义信息
FactoryBean | 注入工厂类，可以生产 bean
ImportBeanDefinitionRegistrar | 动态注册 bean 接口
ImportSelector | 导入外部配置的接口,收集需要导入的配置类,动态判断需要导入的类
ConditionOnClass | 判断类是否存在，主要判断是否引用某个包


1. setEnvironment (设置环境变量)
2. setResourceLoader
3. applyInitializers (ApplicationContextInitializer)
4. listenersPrepared (SpringApplicationRunListeners)
5. loadSource (Application)
6. listenersLoaded (SpringApplicationRunListeners)
7. init、validate propertySources
8. prepareBeanFactory
9. invokeBeanFactoryPostProcessors (BeanDefinitionRegistryPostProcessor, BeanFactoryPostProcessor)
10. registerBeanPostProcessors (BeanPostProcessor, InitializingBean, BeanPostProcessor)
11. initMessageSource (MessageSource)
12. registerListeners (ApplicationListener)

      Aware 接口在 BeanPostProcessor 的 postProcessBeforeInitialization 中调用
      
      ImportBeanDefinitionRegistrar 接口在 BeanDefinitionRegistryProcess 的 postProcessBeanDefinitionRegistry 调用

      ImportSelector 接口返回 ImportBeanDefinitionRegistrar 的类名
      
      @Import 放 ImportSelector （实际也是ImportBeanDefinitionRegistrar）或者 ImportBeanDefinitionRegistrar
    
--------------

spring-boot-configuration-processor.jar 使用 @ConfigurationProperties 注解产生元数据（-分割格式， 如 hasOne -> has-one)

Proxy.newProxyInstance(classLoader, interfaces[], invocationHandler) 

---------------

#### beanFactory 持有 bean 的容器
接口拥有获取 bean 的方法。
接口实现中拥有持有 bean 的容器。
对于实现 factoryBean 的接口，会获取它产生的 bean。

#### factoryBean 生产 bean 的工厂
该工厂本身也是个 bean。
factoryBean 的 bean 名称拥有特定的前缀 &。
isSingleton 为 true 时 beanFactory 会存储单例，不再创建新实例。
beanFactory.getBean（name）, 如果 name 不带该前缀，则返回 factoryBean 生产的 bean。
（为什么要用单例去生产单例？）

#### abstractAutoProxyCreator
实现 BeanPostProcessor，
对切面类做增强代理。
如 @transactional 注解所在类

    获取所有实现增强的 bean（TransactionInterceptor，实现了 Advice 接口）
    普通 bean 在装载时会扫描，判断是否需要增强（拥有@Transactional 注解的类会被 TransactionInterceptor 响应）
    使用动态代理进行增强
    
### network

className | details
--- |---
NetworkInterface | 获取本地网卡或网络接口信息
InetAddress | 通过本地 dns 信息过去本地 ip 和主机名
HandlerMethodArgumentResolver | 获取请求参数
HandlerInterceptor | 拦截请求
WebMvcConfigurer | 配置拦截器
ConstraintValidator | 使用注解校验参数

------------

### database
jdbc:mysql 链接可设置 mysql 编码集、时区

#### driver
创建连接的驱动

#### connection
与数据库的连接，事务执行的基本单位，由驱动创建

#### datasource
通过 driver 获取连接，能实现更多细节（如连接池）

#### SqlSessionFactory
创建 mybatis 会话
#### JdbcTemplate
封装了 connection 获取 statement 并从 statement 获取对象
#### sqlSession
缓存了 statement。
可以获取 mapper 实例
使用 executor 执行 sql
当前线程事务持有的 sqlSessionHolder 放在 TransactionSynchronizationManager.resources 中
    
    holder = new SqlSessionHolder(session, executorType, exceptionTranslator);
    TransactionSynchronizationManager.bindResource(sessionFactory, holder);
    
    
#### transaction
通过 datasource 获取 connection，或者通过构造器传入 connection（不同实现类有不同的实现）。
封装 connection 的操作。
SpringManagedTransaction 在获取连接时访问 

	  ConnectionHolder conHolder = (ConnectionHolder) TransactionSynchronizationManager.getResource(dataSource);

#### executor
构建 statementHandler 类执行 sql
baseExecutor 持有一个 transaction 实例
#### statementHandler
封装 statement 的执行


#### mapper
mapper 接口的方法会与 sql 模板绑定，使用动态代理，填充参数后使用 sqlSession 执行

-------------
    
    创建 sqlSessionFactory 的 bean；
    ClassPathMapperScanner 扫描 mapper 的目录，拿到 interface 列表
    使用单个 mapper 接口和 sqlSessionFactory 构造 MapperFactoryBean
    注册 mapperFactoryBean
    注入 mapper 时会获取 mapperFactoryBean 生产的 mapper 实例
    
    mapper 通过动态代理实现。
    获取路径：sqlSession.getMapper -> configuration.getMapper -> 
        mapperRegistry.getMapper -> mapperProxyFactory.newInstance -> 
        mapperProxy 动态代理实现
        
    mapper 的实现 mapperProxy 持有一个 sqlSession 的引用（代理实现）。每次查询都会代理创建一个新的 sqlSession，在事务中会不同 mapper 共用一个单独的 connection 执行。

### Transaction
#### steps

    1、获取连接 Connection con = DriverManager.getConnection()
    2、开启事务con.setAutoCommit(true/false);
    3、执行CRUD
    4、提交事务/回滚事务 con.commit() / con.rollback();
    5、关闭连接 conn.close();
    
#### PlatformTransactionManager
接口：平台事务管理
 DataSourceTransactionManager：jdbc事务管理，
 
    方法 doBegin 中
    if (txObject.isNewConnectionHolder()) {
	    TransactionSynchronizationManager.bindResource(obtainDataSource(), txObject.getConnectionHolder());
	}
	
	开始事务后，连接会存放在 threadLocal 中，mapper 会使用缓存的 connection 创建新的sqlSession （mapper 持有的 sqlSession 是动态代理，每次执行 sql 都会创建一个可执行的 sqlSession）并执行
#### TransactionDefinition
定义事务，Spring 默认实现：DefaultTransactionDefinition
#### TransactionStatus
事务的状态
#### TransactionTemplate
继承 transactionDefinition
编程式事务处理模板（AOP拦截方法实现提交回滚）
持有一个 platformTransactionManager
execute 方法执行传入的 transactionCallback

    执行：
    callbackPreferringPlatformTransactionManager.execute(transactionDefinition, transactionCallback) 
    或
    status = transactionManager.getTransaction(transactionDefinition) ;
    transactionCallback.doInTransaction(status) ;
    transactionManager.commit(status)
    异常时 rollback
#### TranscationInterceptor
声明式事务管理，事务拦截器, @Transactional 注解的方法会使用它处理
    
    transactionInterceptor.invokeWithinTransaction ->
    transactionAttribute.getTransactionAttribute ->
    transactionAttributeSource.getTransactionAttribute ->
    TransactionAnnotationParser.parseTransactionAnnotation 从@Transactional 注解中获取事务配置
#### TranscationProxyFactoryBean
声明式事务管理
#### TransactionCallback
事务的任务（具体实现）
一个 @FuncationInterface 标注的接口
方法：doInTransaction 传入事务的状态并执行
    
    有时候并不一定要管事务的状态，如 io.katharsis.spring.jpa.SpringTransactionRunner 中的代码
    
    public <T> T doInTransaction(final Callable<T> callable) {
        TransactionTemplate template = new TransactionTemplate(this.platformTransactionManager);
        return template.execute(new TransactionCallback<T>() {
            public T doInTransaction(TransactionStatus status) {
                try {
                    return callable.call();
                } catch (RuntimeException var3) {
                    throw var3;
                } catch (Exception var4) {
                    throw new IllegalStateException(var4);
                }
            }
        });
    }
    
    #### spring 启动注册 bean 时就会扫描 bean 中的 @Transactional 注解并使用代理

#### @transactional
有 @transactional 标注的类，会被动态代理增强
因为 transactionInterceptor 实现 advice 接口并注册成 bean，AbstractAutoProxyCreator 会有将其自动实现代理

### Statement
connection.createStatement() 静态 sql
connection.prepareStatement(sql) 支持占位符的 sql
connection.prepareCall(sql) 支持存储过程sql

### SPI
Java SPI的约定
当服务的提供者，提供了服务接口的一种实现之后，在jar包的META-INF/services/目录里同时创建一个以服务接口命名的文件。
该文件里就是实现该服务接口的具体实现类。而当外部程序装配这个模块的时候，就能通过该jar包META-INF/services/里的配置文件找到具体的实现类名，并装载实例化，完成模块的注入。

### 配置
@ConfigurationProperties 可在 application.properties 配置参数
@EnableConfigurationProperties：使使用 @ConfigurationProperties 注解的类生效。
### 启动流程

![](media/15135894671726/15540842198666.png)


-------------
-------------

## THREAD
### lock
锁住当前lock() 到 unlock（）的代码
####ReentrantLock 可重入锁
####ReentrantReadWriteLock.ReadLock
####ReentrantReadWriteLock.WriteLock
#### condition
调用 await()方法后当前线程释放锁并在此等待，其他线程调用 Condition 对象的 signal() 方法后从 await() 方法返回，返回前获得锁

-----------
--------------

## vocabulary

simple name | full name | details
--- | --- | ---
LRU | least recently used 
LFU | least frequently used
FIFO | first in first out
POJO | plain Ordinary Java Object
EJB | Enterprise JavaBean
JWT | json web token
NIO | Non-blocking I/O
RPC | remote procedure call
SPI | service provider interface |  为某个接口寻求具体实现
JPA | Java Persistence API
CRUD | create retrieve update delete
FP | Functional Programming
RP | Reactive Programming
DDoS | Distributed Denial of Service 分布式拒绝服务，短时间大量请求消耗主机资源

--

### basic

byte ： 1字节（8位）有符号
char ： 2字节无符号
符号位 0 + 1 -
补码：负数： 符号位不变，其余位取反，末位加1

-------------
----------------

## mysql
### STRING fuctions

name | works
--- | ---
CANCAT(s1, s2...) | 拼接字符串（有一个为 null 则返回 null）
INSERT(str, x, y, instr) | 将 str 从 x 开始长度为 y 的子串替换为 instr
LOWER(str) | 小写
UPPER(str) | 大写
LEFT(str, x) | 返回 str 左边的 x 个字符
RIGHT(str, x) | 返回 str 右边的 x 个字符
LPAD(str, n, pad) | 使用 pad 填充 str 的左边 n 个字符, n 为总长度
RPAD(str, n, pad) | 使用 pad 填充 str 的左边 n 个字符
LTRIM(str) | 去除左边空格
RTRIM(str) | 去除右边空格
REPEAT(str, x) | 返回 str 重复 x 次的结果
REPLACE(str, a, b) | 用字符串 b 替换 str 中出现的所有字符串 a
STRCMP(s1, s2) | 比较字符串 s1, s2, 返回 -1，0，1
TRIM(str) | 去除空格
SUBSTRING(str, x, y) | 返回 str 从 x 起长度为 y 的子串
 
 ----------------
 
### NUMBER functions
 
name | works
--- | ---
ABS(x) | 绝对值
CEIL(x) | 大于 x 的最小整数
FLOOR(x) | 小于 x 的最大整数
MOD(x, y) | x % y
RAND() | 0~1 内随机数(16位？）
ROUND(x, y) | 对 x 四舍五入保留 y 位小数
TRUNCATE(x, y) | 对 x 用去尾法保留 y 位小数

----------

### DATE TIME function

name | works
--- | ---
CURDATE() | 当前日期
CURTIME() | 当前时间
NOW() | 当前时间日期
UNIX_TIMESTAMP(date) | date 日期的 unix 时间戳 
FROM_UNIXTIME(number) | number 的 unix 时间戳的日期时间值
WEEK(date) | date 在一年中的第几周
YEAR(date) | date 年份
HOUR(time) | 小时值(24)
MINUTE(time) | 分钟值(当前小时）
MONTHNAME(date) | 月份名, date不能为时间(使用 now() 报错)，如 March
DATE_FORMAT(date, fmt) | 格式化日期 
DATE_ADD(date, INTERVAL expr type) | 加减时间间隔 : DATE_ADD(NOW(), INTERVAL -3 MONTH)
DATEDIFF(expr, expr2) | expr - expr2 的天数 
TIMESTAMPDIFF(SECOND..., date/time, date/time) | 两个时间的间隔

------

### FLEW function

name | works
--- | ---
IF(value, t, f) | return if (value) t else f (-1 也为 true)
IFNULL(v1, v2) | return v1 ?: v2 
CASE WHEN[v1] THEN[r1]...ELSE[default]END | if ... else if ... ... else 
CASE[expr] WHEN[v1] THEN...ELSE[default]END | switch case default

---------

### OTHER function

name | works
--- | ---
DATABASE() | 当前数据库名
VERSION() | 当前版本
USER() | 当前登录用户名
INET_ACTION(ip) /INET6_ACTION(ip)| ip 地址的数字表示
INET_NTOA(num) / INET6_NTOA(ip) | 数字代表的 ip 地址
PASSWORD(str) | 加密 str
MD5(str) | 返回 str 的 md5 值


## linux

命令 | work
--- | ---
nslookup | 查看 host ip


## 设计模式

模式 | 描述
--- | ---
 原型模式 |这种模式是实现了一个原型接口，该接口用于创建当前对象的克隆。当直接创建对象的代价比较大时，则采用这种模式。<br/>例如，一个对象需要在一个高代价的数据库操作之后被创建。我们可以缓存该对象，在下一个请求时返回它的克隆，在需要的时候更新数据库，以此来减少数据库调用。
 模板模式 | 抽象类定义执行步骤，子类重写具体实现。
 工厂模式 | 创建对象时不会对客户端暴露创建逻辑，并且是通过使用一个共同的接口来指向新创建的对象。
抽象工厂模式 | 绕一个超级工厂创建其他工厂。该超级工厂又称为其他工厂的工厂。
 单例模式 | 这种模式涉及到一个单一的类，该类负责创建自己的对象，同时确保只有单个对象被创建。这个类提供了一种访问其唯一的对象的方式，可以直接访问，不需要实例化该类的对象。
 建造者模式 | 使用多个简单的对象一步一步构建成一个复杂的对象。如 lombok 的 @Builder
 适配器模式 | 作为两个不兼容的接口之间的桥梁。适配器模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。
桥接模式 | 用于把抽象化与实现化解耦，使得二者可以独立变化。抽象实现分离。
 享元模式 | 尝试重用现有同类对象，如果未找到匹配的对象，则创建新对象。
 责任链模式 | 通常每个接收者都包含对另一个接收者的引用。如果一个对象不能处理该请求，那么它会把相同的请求传给下一个接收者，依此类推。
 命令模式 | 请求以命令的形式包裹在对象中，并传给调用对象。调用对象寻找可以处理该命令的合适的对象，并把该命令传给相应的对象，该对象执行命令。
 解释器模式 | 这种模式实现了一个表达式接口，该接口解释一个特定的上下文。这种模式被用在 SQL 解析、符号处理引擎等。
 中介者模式 | 用来降低多个对象和类之间的通信复杂性。这种模式提供了一个中介类，该类通常处理不同类之间的通信，并支持松耦合，使代码易于维护。
备忘录模式 | 保存一个对象的某个状态，以便在适当的时候恢复对象。
 观察者模式 | 定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。
 状态模式 | 允许对象在内部状态发生改变时改变它的行为，对象看起来好像修改了它的类。将各种具体的状态类抽象出来。
 空对象模式 | 一个空对象取代 NULL 对象实例的检查。Null 对象不是检查空值，而是反应一个不做任何动作的关系。这样的 Null 对象也可以在数据不可用的时候提供默认的行为。
 策略模式 | 一个类的行为或其算法可以在运行时更改。实现同一个接口。定义一系列的算法,把它们一个个封装起来, 并且使它们可相互替换。
访问者模式 | 使用了一个访问者类，它改变了元素类的执行算法。通过这种方式，元素的执行算法可以随着访问者改变而改变。<br/>根据模式，元素对象已接受访问者对象，这样访问者对象就可以处理元素对象上的操作。在数据基础类里面有一个方法接受访问者，将自身引用传入访问者。

### COOUTINES

function | details
--- | ---
GlobalScope | 后台启动一个全局协程（非阻塞）launch 返回 job, async 返回 deferred
delay | 挂起协程（不阻塞线程）
runBlocking | 阻塞主线程直到里面代码运行结束
Job.join | 等待job（无结果）协程结果，阻塞主线程
coroutineScope | 申明协程作用域并不阻塞当前线程
Channel | 管道（类似阻塞列队）
Deferred | is Job, 有结果
async | 启动单独一个协程，返回 Deferred
laungh | 启动协程，返回 Job

### TOOLs

class | details
--- | ---
HttpAsyncClients | NIO 异步请求


### KOTLIN

关键词或方法 | 细节
--- | ---
lazy | 线程安全的延迟加载（可以用于构建单例）
object | 创建内部类时（object : A() 或者 a = object : A()），如果不申明对象，则为静态内部类




