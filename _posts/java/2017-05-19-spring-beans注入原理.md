---
layout: post
title:  "spring-beans注入原理"
date:   2017-05-27 09:21:01 +0800
categories: java
tag: java 原创
sid: 1495848151
---



# spring-beans注入原理

今天在调用公司某一组件,看到代码中写有这样一段代码：
~~~java
public abstract class AbstractStrategy {


    private static Logger logger = Logger.getLogger(AbstractStrategy.class);

    @Autowired
    protected RedisService redisService;
    @Autowired
    protected MysqlService mysqlService;

    @Autowired
    protected RedisReadWrite m_RedisReadWrite;
    @Autowired
    protected ESearchReadWrite eSearchReadWrite;

    /**
     * 检查某两个id是否建立连接
     * @param fromId
     * @param toId
     * @param linkEnum
     * @return
     */
    public static boolean checkLinkState(String fromId, String toId, LinkEnum linkEnum ) {

        String columnOne  = ClassUtil.getKeyPrefix(linkEnum.fromType().objectClass().getName()).toLowerCase()+ StringUtil.getFirstUpperString(linkEnum.fromKey().dbKey())+"Id";
        String columnTwo = ClassUtil.getKeyPrefix(linkEnum.toType().objectClass().getName()).toLowerCase() + StringUtil.getFirstUpperString(linkEnum.toKey().dbKey())+"Id";

        MysqlService mysqlService = SpringUtils.getBean(MysqlService.class);
        Map<String , Object > map = new HashMap<String, Object>();
        map.put(columnOne , fromId);
        map.put(columnTwo , toId);
        List list = mysqlService.getByMapCondition(map , linkEnum.lClass());
        return list!=null && list.size()>0;
    }
}
~~~

有两个地方引起了我的注意：
1. 整个类中并没有引用到RedisService这个服务,但是却引入了,可见开发人员多么不细心
2. MysqlService服务已经通过Autowired注解引入,下方又通过MysqlService mysqlService = SpringUtils.getBean(MysqlService.class)来引入

问了相关同事为什么MysqlService服务要再次使用MysqlService mysqlService = SpringUtils.getBean(MysqlService.class)来获取,得到的回答是,在调试过程中
Autowired注解没有生效,在使用了MysqlService mysqlService = SpringUtils.getBean(MysqlService.class)就可以获取了,然后就这么用了。

我们看下SpringUtils中的getBean方法,该类是另一位同事写的一个工具类,源码如下：

~~~java
public final class SpringUtils implements BeanFactoryPostProcessor {

    private static ConfigurableListableBeanFactory beanFactory; // Spring应用上下文环境

    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        SpringUtils.beanFactory = beanFactory;
    }

    /**
     * 获取对象
     *
     * @param name
     * @return Object 一个以所给名字注册的bean的实例
     * @throws org.springframework.beans.BeansException
     *
     */
    @SuppressWarnings("unchecked")
    public static <T> T getBean(String name) throws BeansException {
        return (T) beanFactory.getBean(name);
    }

    /**
     * 获取类型为requiredType的对象
     *
     * @param clz
     * @return
     * @throws org.springframework.beans.BeansException
     *
     */
    public static <T> T getBean(Class<T> clz) throws BeansException {
        T result = (T) beanFactory.getBean(clz);
        return result;
    }

    /**
     * 根据类型查找bean
     *
     * @param serviceClassName
     * @return
     * @throws org.springframework.beans.BeansException
     *
     */
    public static Object getBeanByClassName(String serviceClassName) throws BeansException {
        try {
            Class c = Class.forName(serviceClassName);
            Object result =  beanFactory.getBean(c);
            return result;
        } catch (ClassNotFoundException e) {
            return null;
        }
    }


    /**
     * 如果BeanFactory包含一个与所给名称匹配的bean定义，则返回true
     *
     * @param name
     * @return boolean
     */
    public static boolean containsBean(String name) {
        return beanFactory.containsBean(name);
    }

    /**
     * 判断以给定名字注册的bean定义是一个singleton还是一个prototype。 如果与给定名字相应的bean定义没有被找到，将会抛出一个异常（NoSuchBeanDefinitionException）
     *
     * @param name
     * @return boolean
     * @throws org.springframework.beans.factory.NoSuchBeanDefinitionException
     *
     */
    public static boolean isSingleton(String name) throws NoSuchBeanDefinitionException {
        return beanFactory.isSingleton(name);
    }

    /**
     * @param name
     * @return Class 注册对象的类型
     * @throws org.springframework.beans.factory.NoSuchBeanDefinitionException
     *
     */
    public static Class<?> getType(String name) throws NoSuchBeanDefinitionException {
        return beanFactory.getType(name);
    }

    /**
     * 如果给定的bean名字在bean定义中有别名，则返回这些别名
     *
     * @param name
     * @return
     * @throws org.springframework.beans.factory.NoSuchBeanDefinitionException
     *
     */
    public static String[] getAliases(String name) throws NoSuchBeanDefinitionException {
        return beanFactory.getAliases(name);
    }


    public static <T> List<T> getBeans(Class<T> clz) throws BeansException {
        Map<String, T> map =  beanFactory.getBeansOfType(clz);
        List<T> list = new ArrayList<T>();
        if (map!=null){
            for (Map.Entry<String, T> entry :map.entrySet()){
                list.add(entry.getValue());
            }
        }
        return list;
    }

    public static <T> Map<String, T> getBeansOfType(Class<T> clz) throws BeansException {
        Map<String, T> map =  beanFactory.getBeansOfType(clz);
        return map;
    }
~~~

核心调用方法为:
~~~java
 public static <T> T getBean(Class<T> clz) throws BeansException {
        T result = (T) beanFactory.getBean(clz);
        return result;
    }
~~~

## SpringUtils类的使用

该工具类原理很简单,此类继承了BeanFactoryPostProcessor,使用了spring提供的扩展功能,将该工具注册到srping容器之后,能够自动获取到
容器本身,然后在容器中获取指定类型的bean(这里是MysqlService),实际上是BeanFactory的装饰类。

## @Autowired注解的使用

@Autowired注解是Spring 2.5 引入的,它可以对类成员变量、方法及构造函数进行标注，完成自动装配的工作。 通过 @Autowired的使用来消除 set ,get方法.

## 使用哪一种方式获取MysqlService呢?
采用SpringUtils.getBean()与@Autowired本质上是一样的,一个是将依赖的bean自动注入到某个bean中去,一个是在使用的时候直接去BeanFactory中获取,
那么应该使用哪种方式呢?这个需要看业务本身,MysqlService是通过注解注入到spring容器中去,那么上面的两种方式,第一种就是不管使用使用,在spring启动
的过程中就完成依赖注入，第二种则是开始不使用,在使用的时候再去容器中获取,类似于延迟加载一样。


## 参考
- BeanFactoryPostProcessor相关知识 http://www.infoq.com/cn/articles/spring-2.5-part-1
- JSR 330标准注解 http://ifeve.com/spring-beans-standard-annotations/#more-32817