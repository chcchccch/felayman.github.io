---
layout: post
title:  "spring-alias别名使用及源码解析"
date:   2017-05-23 23:03:01 +0800
categories: spring java
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
其中注释说的很清楚,Handle aliasing。其中存储别名只是用了一个ConcurrentHashMap,key是别名,value是bean的真正name。
~~~java
private final Map<String, String> aliasMap = new ConcurrentHashMap<String, String>(16);
~~~