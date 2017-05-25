---
layout: post
title:  "spring-xml方式启动详解"
date:   2017-05-24 11:03:01 +0800
categories: spring
tag: spring 原创
sid: 1495559587
---

##  spring-xml方式启动详解
代码入口:
~~~java
ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath:bean.xml");
~~~
1.定位配置文件地址
~~~java
setConfigLocations(configLocations);
~~~
2.刷新整个容器
~~~java
public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
			prepareBeanFactory(beanFactory);

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

			catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}

				// Destroy already created singletons to avoid dangling resources.
				destroyBeans();

				// Reset 'active' flag.
				cancelRefresh(ex);

				// Propagate exception to caller.
				throw ex;
			}

			finally {
				// Reset common introspection caches in Spring's core, since we
				// might not ever need metadata for singleton beans anymore...
				resetCommonCaches();
			}
		}
	}
~~~
## 容器刷新流程
- prepareRefresh() 为刷新容器准备上下文环境
- ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory()  获取bean工厂
- prepareBeanFactory(beanFactory) 为bean工厂设置默认特性
- postProcessBeanFactory(beanFactory) 允许子类后处理bean工厂的特性
- invokeBeanFactoryPostProcessors(beanFactory) 调用后置处理bean的工厂
- registerBeanPostProcessors(beanFactory)  注册bean工厂的后置处理器
- initMessageSource() 初始化上下文的资源文件
- initApplicationEventMulticaster() 初始化时事件广播监听器
- onRefresh() 刷新
- registerListeners() 注册监听器
- finishBeanFactoryInitialization(beanFactory) 初始化一些遗留的单例bean
- finishRefresh() 完成容器刷新

## 容器刷新流程详解

### 容器类型

整个容器刷新流程是在AbstractApplicationContext中完成的,而AbstractApplicationContext继承了DefaultResourceLoader且实现了ConfigurableApplicationContext和DisposableBean接口
- 继承DefaultResourceLoader 表面可以从指定位置加载资源文件
- 实现DisposableBean 表面该容器可以销毁
- 实现ConfigurableApplicationContext(ApplicationContext,Lifecycle,Closeable) 表面该容器 可以(自动)关闭,具有生命周期
  - 继承EnvironmentCapable 该容器可以有多种环境
  - 继承ListableBeanFactory 该容器可以列举出所有bean
  - 继承HierarchicalBeanFactory 该容器具有层级关系
  - 继承MessageSource 该容器能够加载国际化资源文件
  - 继承ApplicationEventPublisher 该容器能发布事件
  - 继承ResourcePatternResolver 该容器能解析不同风格的资源文件地址形式

 ###  prepareRefresh()  为刷新容器做准备工作
 ~~~java
 protected void prepareRefresh() {
 		this.startupDate = System.currentTimeMillis();
 		this.closed.set(false);
 		this.active.set(true);

 		if (logger.isInfoEnabled()) {
 			logger.info("Refreshing " + this);
 		}

 		// Initialize any placeholder property sources in the context environment
 		initPropertySources();

 		// Validate that all properties marked as required are resolvable
 		// see ConfigurablePropertyResolver#setRequiredProperties
 		getEnvironment().validateRequiredProperties();

 		// Allow for the collection of early ApplicationEvents,
 		// to be published once the multicaster is available...
 		this.earlyApplicationEvents = new LinkedHashSet<ApplicationEvent>();
 	}
 ~~~
核心代码:
~~~java
#从web.xml中加载servlet的上线文参数servletContextInitParams
initPropertySources();
#校验当前容器环境所必须的属性参数(spring-core没有必须的属性参数)
getEnvironment().validateRequiredProperties();
#初始化早期发布事件的容器(LinkedHashSet)
this.earlyApplicationEvents = new LinkedHashSet<ApplicationEvent>();
~~~

###  obtainFreshBeanFactory() 获取指定类型的bean工厂
~~~java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
		refreshBeanFactory();
		ConfigurableListableBeanFactory beanFactory = getBeanFactory();
		if (logger.isDebugEnabled()) {
			logger.debug("Bean factory for " + getDisplayName() + ": " + beanFactory);
		}
		return beanFactory;
	}
~~~
核心代码：
~~~java
refreshBeanFactory();
~~~
告诉子类,需要重新刷新整个容器,内部代码如下：
~~~java
protected final void refreshBeanFactory() throws BeansException {
		if (hasBeanFactory()) {
			destroyBeans();
			closeBeanFactory();
		}
		try {
		   #return new DefaultListableBeanFactory(getInternalParentBeanFactory());创建一个DefaultListableBeanFactory
			DefaultListableBeanFactory beanFactory = createBeanFactory();
			#设置beanFactory的id
			beanFactory.setSerializationId(getId());
			#定制beanFactory,比如允许bean被覆写,允许循环引用
			customizeBeanFactory(beanFactory);
			#加载配置文件中的beans节点的内容
			loadBeanDefinitions(beanFactory);
			synchronized (this.beanFactoryMonitor) {
				this.beanFactory = beanFactory;
			}
		}
		catch (IOException ex) {
			throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
		}
	}
~~~

### prepareBeanFactory(beanFactory) 为bean工厂配置标准的上下文特征
~~~java
protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
		// 设置类加载器
		beanFactory.setBeanClassLoader(getClassLoader());
		//设置标准SpEL解析器
		beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
		//设置资源编辑器
		beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));
		// 为容器设置回调接口
		beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
		//忽略依赖的接口
		beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
		beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
		beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
		beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
		beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
		beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);

		// BeanFactory interface not registered as resolvable type in a plain factory.
		// MessageSource registered (and found for autowiring) as a bean.
		beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
		beanFactory.registerResolvableDependency(ResourceLoader.class, this);
		beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
		beanFactory.registerResolvableDependency(ApplicationContext.class, this);

		// Register early post-processor for detecting inner beans as ApplicationListeners.
		beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));

		// Detect a LoadTimeWeaver and prepare for weaving, if found.
		if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
			beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
			// Set a temporary ClassLoader for type matching.
			beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
		}

		// Register default environment beans.
		if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
			beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
		}
		if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
			beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
		}
		if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
			beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
		}
	}
~~~
