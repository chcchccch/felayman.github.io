---
layout: post
title:  "Spring通过AOP拦截@RequestParam参数值"
date:   2017-06-13 23:03:01 +0800
categories: spring
tag: spring 原创
sid: 1497363742
---

## Spring通过AOP拦截@RequestParam参数值

我们都知道AOP可以针对某些Controller做一些切面操作,比如权限,日志处理,现在我们来看看如何处理在Controller中的@RequestParam参数进行统一的拦截处理

### 需求背景

> 在做部落产品的时候,有两种权限1.普通成员 2.管理员/首领,在管理员权限下,可以做一些对部落的一些管理操作,比如删除成员,修改部落信息,删除作品等普通成员无法操作的权限.

### 需求来源

> 在TribeAdminAction中许多方法都存在如下的权限判断代码,非常冗余.

~~~java
Tribe tribe = tribeService.getTribe(tribeId);
//校验部落是否否在
        if (tribe == null) {
            request.setAttribute(result, WebResponse.fail(TRIBE_NOT_EXISTS + tribeId));
            return json;
        }
//当前用户是否是部落管理员
 boolean isLeader= tribeService.isLeader(tribeId, userId);
        if (!isLeader) {
            request.setAttribute(result, WebResponse.fail("非部落首领不允许操作"));
            return json;
}

~~~

而针对此类代码的方法,都有两个参数tribeId以及能在Session中获取到的userId

在对代码进行重构的时候,把此类反复使用的代码进行AOP化,通过Spring的拦截器进行统一处理,处理代码如下：



~~~java
@Around("@annotation(com.vcg.community.validate.PageValidator)")
    public Object pageValidateAround(ProceedingJoinPoint joinPoint) throws Throwable{
        String methodName = joinPoint.getSignature().getName();
        for (Method method : joinPoint.getTarget().getClass().getMethods()) {
            if(method.getName().equalsIgnoreCase(methodName)){
                Class<?>[] parameterTypes = method.getParameterTypes();
                for (Class<?> parameterType : parameterTypes) {
                    RequestParam annotation = parameterType.getAnnotation(RequestParam.class);
                    String value = annotation.value();
                    if ("tribeId".equals(value)){
                        //TODO 对tribeId进行判断操作
                    }
                }
            }
        }
~~~

@TribeAdminValidator如下:
~~~java

/**
 * 拦截部落管理员的校验器,判断部落是否存在,当前用户是否是部落首领
 * @author lei.fang@shijue.me
 * @since 2017-05-19 上午2:42
 */
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface TribeAdminValidator {
}
~~~