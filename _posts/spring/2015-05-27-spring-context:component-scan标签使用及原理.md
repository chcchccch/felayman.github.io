---
layout: post
title:  "context:component-scan标签 源码分"
date:   2017-05-25 11:03:01 +0800
categories: spring
tag: spring 原创
sid: 1495854290
---

## context:component-scan的使用

~~~java
<context:component-scan base-package="com.vcg.community"
                            annotation-config="false"
                            name-generator="com.vcg.community.spring.core.config.SimpleAnnotationBeanNameGenerator"
                            resource-pattern="**/*.class"
                            scope-resolver="org.springframework.context.annotation.AnnotationScopeMetadataResolver"
                            use-default-filters="true"
>
<context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
<context:exclude-filter type="annotation" expression="org.springframework.stereotype.Repository"/>
~~~



## context:component-scan源码分析

-   xsd文件定义

context:component-scan是spring2.5引入的一个标签,沿用至今。我们先看看xsd文件是如何定义的:
~~~xml
<xsd:element name="component-scan">
		<xsd:annotation>
			<xsd:documentation><![CDATA[
	Scans the classpath for annotated components that will be auto-registered as
	Spring beans. By default, the Spring-provided @Component, @Repository, @Service,
	@Controller, @RestController, @ControllerAdvice, and @Configuration stereotypes
	will be detected.

	Note: This tag implies the effects of the 'annotation-config' tag, activating @Required,
	@Autowired, @PostConstruct, @PreDestroy, @Resource, @PersistenceContext and @PersistenceUnit
	annotations in the component classes, which is usually desired for autodetected components
	(without external configuration). Turn off the 'annotation-config' attribute to deactivate
	this default behavior, for example in order to use custom BeanPostProcessor definitions
	for handling those annotations.

	Note: You may use placeholders in package paths, but only resolved against system
	properties (analogous to resource paths). A component scan results in new bean definitions
	being registered; Spring's PropertySourcesPlaceholderConfigurer will apply to those bean
	definitions just like to regular bean definitions, but it won't apply to the component
	scan settings themselves.

	See javadoc for org.springframework.context.annotation.ComponentScan for information
	on code-based alternatives to bootstrapping component-scanning.
			]]></xsd:documentation>
		</xsd:annotation>
		<xsd:complexType>
			<xsd:sequence>
				<xsd:element name="include-filter" type="filterType"
					minOccurs="0" maxOccurs="unbounded">
					<xsd:annotation>
						<xsd:documentation><![CDATA[
	Controls which eligible types to include for component scanning.
	Note that these filters will be applied in addition to the default filters, if specified.
	Any type under the specified base packages which matches a given filter will be included,
	even if it does not match the default filters (i.e. is not annotated with @Component).
							]]></xsd:documentation>
					</xsd:annotation>
				</xsd:element>
				<xsd:element name="exclude-filter" type="filterType"
					minOccurs="0" maxOccurs="unbounded">
					<xsd:annotation>
						<xsd:documentation><![CDATA[
	Controls which eligible types to exclude for component scanning.
						]]></xsd:documentation>
					</xsd:annotation>
				</xsd:element>
			</xsd:sequence>
			<xsd:attribute name="base-package" type="xsd:string"
				use="required">
				<xsd:annotation>
					<xsd:documentation><![CDATA[
	The comma/semicolon/space/tab/linefeed-separated list of packages to scan for annotated components.
					]]></xsd:documentation>
				</xsd:annotation>
			</xsd:attribute>
			<xsd:attribute name="resource-pattern" type="xsd:string">
				<xsd:annotation>
					<xsd:documentation><![CDATA[
	Controls the class files eligible for component detection. Defaults to "**/*.class", the recommended value.
	Consider use of the include-filter and exclude-filter elements for a more fine-grained approach.
					]]></xsd:documentation>
				</xsd:annotation>
			</xsd:attribute>
			<xsd:attribute name="use-default-filters" type="xsd:boolean"
				default="true">
				<xsd:annotation>
					<xsd:documentation><![CDATA[
	Indicates whether automatic detection of classes annotated with @Component, @Repository, @Service,
	or @Controller should be enabled. Default is "true".
					]]></xsd:documentation>
				</xsd:annotation>
			</xsd:attribute>
			<xsd:attribute name="annotation-config" type="xsd:boolean"
				default="true">
				<xsd:annotation>
					<xsd:documentation><![CDATA[
	Indicates whether the implicit annotation post-processors should be enabled. Default is "true".
					]]></xsd:documentation>
				</xsd:annotation>
			</xsd:attribute>
			<xsd:attribute name="name-generator" type="xsd:string">
				<xsd:annotation>
					<xsd:documentation><![CDATA[
	The fully-qualified class name of the BeanNameGenerator to be used for naming detected components.
					]]></xsd:documentation>
					<xsd:appinfo>
						<tool:annotation>
							<tool:expected-type type="java.lang.Class"/>
							<tool:assignable-to type="org.springframework.beans.factory.support.BeanNameGenerator"/>
						</tool:annotation>
					</xsd:appinfo>
				</xsd:annotation>
			</xsd:attribute>
			<xsd:attribute name="scope-resolver" type="xsd:string">
				<xsd:annotation>
					<xsd:documentation><![CDATA[
	The fully-qualified class name of the ScopeMetadataResolver to be used for resolving the scope of
	detected components.
					]]></xsd:documentation>
					<xsd:appinfo>
						<tool:annotation>
							<tool:expected-type type="java.lang.Class"/>
							<tool:assignable-to type="org.springframework.context.annotation.ScopeMetadataResolver"/>
						</tool:annotation>
					</xsd:appinfo>
				</xsd:annotation>
			</xsd:attribute>
			<xsd:attribute name="scoped-proxy">
				<xsd:annotation>
					<xsd:documentation><![CDATA[
	Indicates whether proxies should be generated for detected components, which may be necessary
	when using scopes in a proxy-style fashion. Default is to generate no such proxies.
					]]></xsd:documentation>
				</xsd:annotation>
				<xsd:simpleType>
					<xsd:restriction base="xsd:string">
						<xsd:enumeration value="no"/>
						<xsd:enumeration value="interfaces"/>
						<xsd:enumeration value="targetClass"/>
					</xsd:restriction>
				</xsd:simpleType>
			</xsd:attribute>
		</xsd:complexType>
	</xsd:element>
~~~

其核心作用如同在文件中描述的一样

      Scans the classpath for annotated components that will be auto-registered as
      Spring beans. By default, the Spring-provided @Component, @Repository, @Service,
      @Controller, @RestController, @ControllerAdvice, and @Configuration stereotypes
      will be detected.

**翻译如下**

    扫描类路径下所有被标注为注解组件的类,并注册为sping的bean.默认的,所有被标注为spring提供的注解,如
    @Component,@Repository,@Service,@Controller,@RestController,@ControllerAdvice.其中@Configuration
    并不推荐使用.

-  工作原理

在BeanDefinitionParserDelegate中的parseCustomElement()方法中完成此对context:component-scan标签的解析工作,方法内容如下：

~~~java
	String namespaceUri = getNamespaceURI(ele);
		NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);
		if (handler == null) {
			error("Unable to locate Spring NamespaceHandler for XML schema namespace [" + namespaceUri + "]", ele);
			return null;
		}
		return handler.parse(ele, new ParserContext(this.readerContext, this, containingBd));
~~~

所有在xml文件中定义的标签都有一个对应的NamespaceHandlerResolver来进行解析,即在xml文件最上面定义的命名空间文件,这些命名空间所对应
的解析器都通过java提供的SPI机制来引入,这些解析器在spring的不同组件被分别引入,比如(并非全部):

- http\://www.springframework.org/schema/aop=org.springframework.aop.config.AopNamespaceHandler
- http\://www.springframework.org/schema/c=org.springframework.beans.factory.xml.SimpleConstructorNamespaceHandler
- http\://www.springframework.org/schema/p=org.springframework.beans.factory.xml.SimplePropertyNamespaceHandler
- http\://www.springframework.org/schema/util=org.springframework.beans.factory.xml.UtilNamespaceHandler
- http\://www.springframework.org/schema/context=org.springframework.context.config.ContextNamespaceHandler
- http\://www.springframework.org/schema/jee=org.springframework.ejb.config.JeeNamespaceHandler
- http\://www.springframework.org/schema/lang=org.springframework.scripting.config.LangNamespaceHandler
- http\://www.springframework.org/schema/task=org.springframework.scheduling.config.TaskNamespaceHandler
- http\://www.springframework.org/schema/cache=org.springframework.cache.config.CacheNamespaceHandler
- http\://www.springframework.org/schema/jdbc=org.springframework.jdbc.config.JdbcNamespaceHandler
- http\://www.springframework.org/schema/mvc=org.springframework.web.servlet.config.MvcNamespaceHandler
- http\://www.springframework.org/schema/tx=org.springframework.transaction.config.TxNamespaceHandler
- http\://www.springframework.org/schema/data/repository=org.springframework.data.repository.config.RepositoryNameSpaceHandler
- http\://www.springframework.org/schema/data/keyvalue=org.springframework.data.keyvalue.repository.config.KeyValueRepositoryNameSpaceHandler

在spring中用来存储上述命名空间名称以及对应的处理器的结构为:
~~~java
/** Stores the mappings from namespace URI to NamespaceHandler class name / instance */
	private volatile Map<String, Object> handlerMappings;
~~~

在spring启动的时候,会将上述的述命名空间名称以及对应的处理器缓存在handlerMappings中,在使用的时候直接通过命名空间
的名称来获取对应的处理器实例.而解析component-scan标签的处理器为org.springframework.context.config.ContextNamespaceHandler,
而component-scan对应的命名空间为context:
获取的具体源码如下:
~~~java
public NamespaceHandler resolve(String namespaceUri) {
		Map<String, Object> handlerMappings = getHandlerMappings();
		Object handlerOrClassName = handlerMappings.get(namespaceUri);
		if (handlerOrClassName == null) {
			return null;
		}
		else if (handlerOrClassName instanceof NamespaceHandler) {
			return (NamespaceHandler) handlerOrClassName;
		}
		else {
			String className = (String) handlerOrClassName;
			try {
				Class<?> handlerClass = ClassUtils.forName(className, this.classLoader);
				if (!NamespaceHandler.class.isAssignableFrom(handlerClass)) {
					throw new FatalBeanException("Class [" + className + "] for namespace [" + namespaceUri +
							"] does not implement the [" + NamespaceHandler.class.getName() + "] interface");
				}
				NamespaceHandler namespaceHandler = (NamespaceHandler) BeanUtils.instantiateClass(handlerClass);
				namespaceHandler.init();
				handlerMappings.put(namespaceUri, namespaceHandler);
				return namespaceHandler;
			}
			catch (ClassNotFoundException ex) {
				throw new FatalBeanException("NamespaceHandler class [" + className + "] for namespace [" +
						namespaceUri + "] not found", ex);
			}
			catch (LinkageError err) {
				throw new FatalBeanException("Invalid NamespaceHandler class [" + className + "] for namespace [" +
						namespaceUri + "]: problem with handler class file or dependent class", err);
			}
		}
	}
~~~

核心代码如下：
~~~java
NamespaceHandler namespaceHandler = (NamespaceHandler) BeanUtils.instantiateClass(handlerClass);
namespaceHandler.init();
handlerMappings.put(namespaceUri, namespaceHandler);
~~~

在获取对应的NamespaceHandler后,首先会进行初始化操作,调用init()方法,源码如下:
~~~java
registerBeanDefinitionParser("property-placeholder", new PropertyPlaceholderBeanDefinitionParser());
registerBeanDefinitionParser("property-override", new PropertyOverrideBeanDefinitionParser());
registerBeanDefinitionParser("annotation-config", new AnnotationConfigBeanDefinitionParser());
registerBeanDefinitionParser("component-scan", new ComponentScanBeanDefinitionParser());
registerBeanDefinitionParser("load-time-weaver", new LoadTimeWeaverBeanDefinitionParser());
registerBeanDefinitionParser("spring-configured", new SpringConfiguredBeanDefinitionParser());
registerBeanDefinitionParser("mbean-export", new MBeanExportBeanDefinitionParser());
registerBeanDefinitionParser("mbean-server", new MBeanServerBeanDefinitionParser());
~~~
其中registerBeanDefinitionParser("component-scan", new ComponentScanBeanDefinitionParser())则表明,若想处理
component-scan标签,则需要用到ComponentScanBeanDefinitionParser来进行处理.在获得具体的解析器后,会调用如下方法:

~~~java
public BeanDefinition parse(Element element, ParserContext parserContext) {
		String basePackage = element.getAttribute(BASE_PACKAGE_ATTRIBUTE);
		basePackage = parserContext.getReaderContext().getEnvironment().resolvePlaceholders(basePackage);
		String[] basePackages = StringUtils.tokenizeToStringArray(basePackage,
				ConfigurableApplicationContext.CONFIG_LOCATION_DELIMITERS);

		// Actually scan for bean definitions and register them.
		ClassPathBeanDefinitionScanner scanner = configureScanner(parserContext, element);
		Set<BeanDefinitionHolder> beanDefinitions = scanner.doScan(basePackages);
		registerComponents(parserContext.getReaderContext(), beanDefinitions, element);

		return null;
	}
~~~

整个解析流程如下：

1. 首先会读取context:component-scan的属性节点,所有节点属性或子节点列表如下:
    ~~~java
    private static final String BASE_PACKAGE_ATTRIBUTE = "base-package";
    private static final String RESOURCE_PATTERN_ATTRIBUTE = "resource-pattern";
    private static final String USE_DEFAULT_FILTERS_ATTRIBUTE = "use-default-filters";
    private static final String ANNOTATION_CONFIG_ATTRIBUTE = "annotation-config";
    private static final String NAME_GENERATOR_ATTRIBUTE = "name-generator";
    private static final String SCOPE_RESOLVER_ATTRIBUTE = "scope-resolver";
    private static final String SCOPED_PROXY_ATTRIBUTE = "scoped-proxy";
    private static final String EXCLUDE_FILTER_ELEMENT = "exclude-filter";
    private static final String INCLUDE_FILTER_ELEMENT = "include-filter";
    private static final String FILTER_TYPE_ATTRIBUTE = "type";
    private static final String FILTER_EXPRESSION_ATTRIBUTE = "expression";
    ~~~
2. 支持以";"分隔符的多包配置来获取多个包地址名称

    ~~~java
    String[] basePackages = StringUtils.tokenizeToStringArray(basePackage,ConfigurableApplicationContext.CONFIG_LOCATION_DELIMITERS);
    ~~~

3. 配置扫描器,想要完成对注入bean的扫描,需要一系列的准备工作,比如对各个属性节点的解析准备工作.

    ~~~java
    protected ClassPathBeanDefinitionScanner configureScanner(ParserContext parserContext, Element element) {
    		boolean useDefaultFilters = true;
    		//判断元素是否有use-default-filters属性
    		if (element.hasAttribute(USE_DEFAULT_FILTERS_ATTRIBUTE)) {
    			useDefaultFilters = Boolean.valueOf(element.getAttribute(USE_DEFAULT_FILTERS_ATTRIBUTE));
    		}

    		// Delegate bean definition registration to scanner class.
    		ClassPathBeanDefinitionScanner scanner = createScanner(parserContext.getReaderContext(), useDefaultFilters);
    		scanner.setBeanDefinitionDefaults(parserContext.getDelegate().getBeanDefinitionDefaults());
    		scanner.setAutowireCandidatePatterns(parserContext.getDelegate().getAutowireCandidatePatterns());
            //判断元素是否有resource-pattern属性
    		if (element.hasAttribute(RESOURCE_PATTERN_ATTRIBUTE)) {
    			scanner.setResourcePattern(element.getAttribute(RESOURCE_PATTERN_ATTRIBUTE));
    		}

    		try {
    		    //生成bean名称生成器,用于为注解类生成beanName,默认是类名首字母小写的驼峰式名称
    			parseBeanNameGenerator(element, scanner);
    		}
    		catch (Exception ex) {
    			parserContext.getReaderContext().error(ex.getMessage(), parserContext.extractSource(element), ex.getCause());
    		}

    		try {
    		    //解析该bean的作用范围,默认是单例
    			parseScope(element, scanner);
    		}
    		catch (Exception ex) {
    			parserContext.getReaderContext().error(ex.getMessage(), parserContext.extractSource(element), ex.getCause());
    		}

            //解析include-filter节点和exclude-filter节点
    		parseTypeFilters(element, scanner, parserContext);

    		return scanner;
    	}
    ~~~

4. 将配置的扫描的包中的所有需要注入到容器中的bean注册到CompositeComponentDefinition中去,用一个LinkList来保存
    ~~~java
    private final List<ComponentDefinition> nestedComponents = new LinkedList<ComponentDefinition>();
    ~~~
5. 真正开始扫描,就扫描到的类都注册成spring的bean,执行代码为Set<BeanDefinitionHolder> beanDefinitions = scanner.doScan(basePackages),方法源码如下:
    ~~~java
    protected Set<BeanDefinitionHolder> doScan(String... basePackages) {
    		Assert.notEmpty(basePackages, "At least one base package must be specified");
    		Set<BeanDefinitionHolder> beanDefinitions = new LinkedHashSet<BeanDefinitionHolder>();
    		for (String basePackage : basePackages) {
    			Set<BeanDefinition> candidates = findCandidateComponents(basePackage);
    			for (BeanDefinition candidate : candidates) {
    				ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(candidate);
    				candidate.setScope(scopeMetadata.getScopeName());
    				String beanName = this.beanNameGenerator.generateBeanName(candidate, this.registry);
    				if (candidate instanceof AbstractBeanDefinition) {
    					postProcessBeanDefinition((AbstractBeanDefinition) candidate, beanName);
    				}
    				if (candidate instanceof AnnotatedBeanDefinition) {
    					AnnotationConfigUtils.processCommonDefinitionAnnotations((AnnotatedBeanDefinition) candidate);
    				}
    				if (checkCandidate(beanName, candidate)) {
    					BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);
    					definitionHolder =
    							AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
    					beanDefinitions.add(definitionHolder);
    					registerBeanDefinition(definitionHolder, this.registry);
    				}
    			}
    		}
    		return beanDefinitions;
    	}
    ~~~
    > 我们看到核心代码为两个for循环,外层是定义的以";"分隔符的多包配置来获取多个包地址名称,内层循环是将完成package下的类注册工作,比如生成注解类的名称,设置注解类的scope范围,检查注解类是否已经注册,为注解类生成代理类,最后将该bean注册到spring容器中。

 6. 完成注册工作,整个过程是调用registerBeanDefinition(definitionHolder, this.registry)来完成,源码如下:
    ~~~java
    public static void registerBeanDefinition(
    			BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)
    			throws BeanDefinitionStoreException {

    		// Register bean definition under primary name.
    		String beanName = definitionHolder.getBeanName();
    		registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());

    		// Register aliases for bean name, if any.
    		String[] aliases = definitionHolder.getAliases();
    		if (aliases != null) {
    			for (String alias : aliases) {
    				registry.registerAlias(beanName, alias);
    			}
    		}
    	}
    ~~~
    - 将生成的注解类名称作为beanName将注解类注入到BeanFactory(这里是DefaultListableBeanFactory)中去
    - 为生成的注解类注册别名
    - 完成注册,实际上就是将注解类缓存到DefaultListableBeanFactory对应的实例map结构中去

##   context:component-scan对属性元素的处理

context:component-scan元素共有四个属性节点:
- base-package               <xsd:attribute name="base-package" type="xsd:string" use="required">
- use-default-filters       <xsd:attribute name="use-default-filters" type="xsd:boolean"default="true">
- resource-pattern         <xsd:attribute name="resource-pattern" type="xsd:string">
- annotation-config       <xsd:attribute name="annotation-config" type="xsd:boolean"default="true">
- name-generator          <xsd:attribute name="name-generator" type="xsd:string">

### base-package

base-package的作用:The comma/semicolon/space/tab/linefeed-separated list of packages to scan for annotated components.

> 配置需要扫描的包名,值为以逗号,分号,空格,制表符,换行符为分隔符

如以逗号为分隔符,base-package="com.vcg.community.dao;com.vcg.community.service;com.vcg.community.action"

### use-default-filters

use-default-filters的作用:Indicates whether automatic detection of classes annotated with @Component, @Repository, @Service,or @Controller should be enabled. Default is "true".

>默认为true表示过滤@Component、@ManagedBean、@Named注解的类,如果改为false默认将不过滤这些默认的注解来定义Bean,即这些注解类不能被过滤到,即不能通过这些注解进行Bean定义;

### resource-pattern

resource-pattern的作用:Controls the class files eligible for component detection. Defaults to "**/*.class", the recommended value.Consider use of the include-filter and exclude-filter elements for a more fine-grained approach.

>表示扫描注解类的后缀匹配模式,即"base-package+resource-pattern"将组成匹配模式用于匹配类路径中的组件,默认后缀为“**/*.class”,即指定包下的所有以.class结尾的类文件

NOTE:use-dafault-filters=”false”的情况下:<context:exclude-filter>指定的不扫描,<context:include-filter>指定的扫描

### annotation-config

annotation-config的作用:Indicates whether automatic detection of classes annotated with @Component, @Repository, @Service,or @Controller should be enabled. Default is "true".

> 表示是否自动支持注解实现Bean依赖注入,默认支持，如果设置为false,将关闭支持注解的依赖注入,需要通过<context:annotation-config/>开启

### name-generator

name-generator的作用:The fully-qualified class name of the BeanNameGenerator to be used for naming detected components.

> 默认情况下的Bean标识符生成策略,默认是 AnnotationBeanNameGenerator,其将生成以小写开头的类名(不包括包名);可以自定义自己的标识符生成策略


##   context:component-scan对字元素的处理

 context:component-scan共有两个子节点元素
 - include-filter               <xsd:element name="include-filter" type="filterType"minOccurs="0" maxOccurs="unbounded">
 - exclude-filter              <xsd:element name="exclude-filter" type="filterType"minOccurs="0" maxOccurs="unbounded">

### include-filter

include-filter的作用:表示过滤到的类将被注册为Spring管理Bean


### exclude-filter

exclude-filter的作用:表示过滤到的类将不被注册为Spring管理Bean，它比<context:include-filter>具有更高优先级

## 参考文章:

- spring 注解模式详解 http://www.tuicool.com/articles/Z7R7jy