---
layout: post
title:  "spring-alias别名使用及源码解析"
date:   2017-05-23 23:03:01 +0800
categories: spring
tag: spring java 原创
sid: 1495553913
---

##  spring-alias别名使用

创建一个普通的bean,如下：
~~~java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="commonBean" class="com.vcg.community.commonbean.CommonBean"/>
    <alias name="commonBean" alias="commonBean_1"/>
    <alias name="commonBean" alias="commonBean_2"/>
    <alias name="commonBean" alias="commonBean_3"/>
</beans>
~~~

##  spring-alias源码分析

在spring的AbstractBeanFactory类中的有如下代码：
~~~java
final String beanName = transformedBeanName(name);
~~~
其中我们凡是调用BeanFactory中的getBean(String name)该方法,不管我们传入的name是别名(alias),还是name,还是id,调用如上代码之后,都会返回一个bean的name,
我们注意到返回值被final修饰后,防止在后面被修改,以此来保证该值的唯一性。

transformedBeanName(name)方法的核心调用了canonicalName(String name)方法,该方法的源码如下：
~~~java
public String canonicalName(String name) {
		String canonicalName = name;
		// Handle aliasing...
		String resolvedName;
		do {
			resolvedName = this.aliasMap.get(canonicalName);
			if (resolvedName != null) {
				canonicalName = resolvedName;
			}
		}
		while (resolvedName != null);
		return canonicalName;
	}
~~~
其中注释说的很清楚,Handle aliasing。其中存储别名只是用了一个ConcurrentHashMap,key是别名,value是bean的真正name(见下文别名的注册)。
~~~java
private final Map<String, String> aliasMap = new ConcurrentHashMap<String, String>(16);
~~~

##  spring-alias别名注册源码

在spring的DefaultBeanDefinitionDocumentReader类中的parseDefaultElement方法中,在解析xml文件的时候会解析并注册别名到指定的bean上去,源码如下:
~~~java
private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
		if (delegate.nodeNameEquals(ele, IMPORT_ELEMENT)) {
			importBeanDefinitionResource(ele);
		}
		else if (delegate.nodeNameEquals(ele, ALIAS_ELEMENT)) {
			processAliasRegistration(ele);
		}
		else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) {
			processBeanDefinition(ele, delegate);
		}
		else if (delegate.nodeNameEquals(ele, NESTED_BEANS_ELEMENT)) {
			// recurse
			doRegisterBeanDefinitions(ele);
		}
	}
~~~
其中判断了节点类型,如果是ALIAS_ELEMENT(public static final String ALIAS_ELEMENT = "alias"),则对该节点进行解析调用processAliasRegistration(ele);。
processAliasRegistration方法的源码如下:
~~~java
protected void processAliasRegistration(Element ele) {
		String name = ele.getAttribute(NAME_ATTRIBUTE);
		String alias = ele.getAttribute(ALIAS_ATTRIBUTE);
		boolean valid = true;
		if (!StringUtils.hasText(name)) {
			getReaderContext().error("Name must not be empty", ele);
			valid = false;
		}
		if (!StringUtils.hasText(alias)) {
			getReaderContext().error("Alias must not be empty", ele);
			valid = false;
		}
		if (valid) {
			try {
				getReaderContext().getRegistry().registerAlias(name, alias);
			}
			catch (Exception ex) {
				getReaderContext().error("Failed to register alias '" + alias +
						"' for bean with name '" + name + "'", ele, ex);
			}
			getReaderContext().fireAliasRegistered(name, alias, extractSource(ele));
		}
	}
~~~
核心代码为getReaderContext().getRegistry().registerAlias(name, alias);最终调用代码为SimpleAliasRegistry中的registerAlias(String name, String alias)方法,源码如下：
~~~java
public void registerAlias(String name, String alias) {
		Assert.hasText(name, "'name' must not be empty");
		Assert.hasText(alias, "'alias' must not be empty");
		if (alias.equals(name)) {
			this.aliasMap.remove(alias);
		}
		else {
			String registeredName = this.aliasMap.get(alias);
			if (registeredName != null) {
				if (registeredName.equals(name)) {
					// An existing alias - no need to re-register
					return;
				}
				if (!allowAliasOverriding()) {
					throw new IllegalStateException("Cannot register alias '" + alias + "' for name '" +
							name + "': It is already registered for name '" + registeredName + "'.");
				}
			}
			checkForAliasCircle(name, alias);
			this.aliasMap.put(alias, name);
		}
	}
~~~
核心代码为
~~~java
checkForAliasCircle(name, alias);
this.aliasMap.put(alias, name);
~~~
最终会将别名存储在aliasMap中,其中存储别名只是用了一个ConcurrentHashMap,key是别名,value是bean的真正name(见上文别名的解析)。

其中,spring在其核心容器BeanFactory(其子类),大量的使用了ConcurrentHashMap来存储或缓存一些bean。

spring对别名的处理为AliasRegistry及其子类,其中spring默认使用SampleAliasRegistry来完成对别名的操作,SampleAliasRegistry内部使用map来存储别名
