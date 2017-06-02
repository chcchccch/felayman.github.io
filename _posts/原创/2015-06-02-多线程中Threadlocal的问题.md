---
layout: post
title:  "多线程中Threadlocal的问题"
date:   2017-06-02 11:30:01 +0800
categories: 原创
tag: spring 原创
sid: 1496374259
---

>  最近在总结自己两年项目开发过程中的一些没有总结过的问题,今天看到了之前自己的一次对于多线程环境中使用Threadlocal的问题

### 问题始末

1. 最开始的时候,我们对封装了一个RPC服务,作为请求转发的一个路由处理器,部分代码如下:

    ~~~java
     //每日记录相关日志
        if(inputObject.getType()!=-101){
            DateFormat dateFormat = new SimpleDateFormat("YYYY-MM-dd");
            String date = dateFormat.format(new Date());
            LocalLogUtil.appendLog("/data/logs/dataprocess/dataprocess.log."+date , inputObject.toMQMessage());
        }
    ~~~

上述代码的作用就是拦截指定类型的错误请求调用,然后通过日志工具路由到MQ组件,然后异步到日志文件中去。

该服务每天的调用次数大概几十万次,我们看到,每次调用我们都需要执行DateFormat dateFormat = new SimpleDateFormat("YYYY-MM-dd");
因为每次调用该服务的时候都需要创建DateFormat实例,这样不存在线程安全问题,但是每次开销很大,后来codeReview的时候觉得不妥,于是将
上述代码修改成全局变量

2. 对上述代码进行升级,将DateFormat变成一个全局静态实例

    ~~~java
    private static DateFormat dateFormat = new SimpleDateFormat("YYYY-MM-dd");
    ~~~


3. 后来查看日志的时候,发现日志中的时间出现异常,混乱.再回头来看代码,发现上面的静态实例有线程安全问题,关于SimpleDateFormat的线程安全问题,参考:http://blog.csdn.net/zxh87/article/details/19414885
简单的查阅了一些资料和文章,再次对上述内容进行改写:

    ~~~java
    private static  ThreadLocal<SimpleDateFormat> dateFormat = new ThreadLocal<SimpleDateFormat>(){
            protected SimpleDateFormat initialValue() {
                return new SimpleDateFormat("YYYY-MM-dd");
            }
        };
    ~~~

这样将共享变量变为独享，线程独享肯定能比方法独享在并发环境中能减少不少创建对象的开销。如果对性能要求比较高的情况下,一般推荐使用这种方法。

### 问题深追

上面的问题
