# SpringBoot
## 思路
将流程结构化，如主方法：
    
    仅为实例，与 spring 无关
    fun main() {
        initializeFramework()
        beforeInitialize()
        initialize()
        afterInitialize()
        doFunction()
        beforeShutdown()
        shutdown()
    }

在各个步骤，spring 都有申明接口，用户可以通过实现接口参与到进程中

    fun beforeInitialize() {
        val handlers = getHandlers(Handler::class)
        handlers.forEach(Handler::doHandle)
    }
    
spring 的核心是 BeanFactory, 所以 spring 将 BeanFactory 的初始化拆分成各个步骤。
实现方式为反射

## SpringApplication()


        public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
		this.resourceLoader = resourceLoader;
		Assert.notNull(primarySources, "PrimarySources must not be null");
		this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
		this.webApplicationType = deduceWebApplicationType();
		setInitializers((Collection) getSpringFactoriesInstances(
				ApplicationContextInitializer.class)); // 注册实现了 ApplicationContextInitializer 的类
		setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class)); // 注册实现了 ApplicationListener 的类
		this.mainApplicationClass = deduceMainApplicationClass();
	}
## SpringApplication.run
    
        
        StopWatch stopWatch = new StopWatch(); // 记录启动用时
		stopWatch.start();
		ConfigurableApplicationContext context = null;
		Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
		configureHeadlessProperty();
		SpringApplicationRunListeners listeners = getRunListeners(args); // 注册 SpringApplicationRunListener
		listeners.starting();
		try {
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(
					args);
			ConfigurableEnvironment environment = prepareEnvironment(listeners,
					applicationArguments);
			configureIgnoreBeanInfo(environment);
			Banner printedBanner = printBanner(environment);
			context = createApplicationContext();
			exceptionReporters = getSpringFactoriesInstances(
					SpringBootExceptionReporter.class,
					new Class[] { ConfigurableApplicationContext.class }, context);
			prepareContext(context, environment, listeners, applicationArguments,
					printedBanner); // 设置环境变量；注册 BeanNameGenerator 和 ResourceLoader; 调用 ApplicationContextInitializer 初始化 applictionContext；SpringApplicationRunListener 监听applicationContext；加载BeanDefinitionLoader；
			refreshContext(context); // 最终调用 applicationContext.refresh
			afterRefresh(context, applicationArguments);
			stopWatch.stop();
			if (this.logStartupInfo) {
				new StartupInfoLogger(this.mainApplicationClass)
						.logStarted(getApplicationLog(), stopWatch);
			}
			listeners.started(context);
			callRunners(context, applicationArguments);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, listeners);
			throw new IllegalStateException(ex);
		}

		try {
			listeners.running(context);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, null);
			throw new IllegalStateException(ex);
		}
		return context;


## AbstractApplicationContext.refresh

        prepareRefresh(); // 初始化属性

			// Tell the subclass to refresh the internal bean factory.
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
			prepareBeanFactory(beanFactory); // 注册几个特殊的BeanPostProcessor(ApplicationContextAwareProcessor, ApplicationListenerDetector，LoadTimeWeaverAwareProcessor),忽略几个Aware接口的自动注入（EnvironmentAware, EmbeddedValueResolverAware,ResourceLoaderAware,ApplicationEventPublisherAware,MessageSourceAware,ApplicationContextAware, aware需要依赖的对象初始化完成才能初始化)；将自身注册为 BeanFactory, ResourceLoader， ApplicationEventPublisher，ApplicationContext

			try {
				// Allows post-processing of the bean factory in context subclasses.
				postProcessBeanFactory(beanFactory);

				// Invoke factory processors registered as beans in the context.
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
				registerBeanPostProcessors(beanFactory);

				// Initialize message source for this context.
				initMessageSource();

				// Initialize event multicaster for this context.
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
				onRefresh();

				// Check for listener beans and register them.
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
				finishRefresh();
			}


