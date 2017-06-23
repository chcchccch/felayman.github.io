---
layout: post
title:  "mysql中concat函数的使用"
date:   2017-05-17 11:43:01 +0800
categories: mysql
tag: mysql 原创
sid: 1495179680
---

##  mysql中concat函数的使用

最近在做一次多表联合查询后对两个表的多字段进行模糊搜索的小功能,就使用了mysql中的concat函数函数,其sql如下：
 ~~~
 select u.id,u.nick_name,t.update_time,t.type
   from tribe_user t, user u
   where t.tribe_id = '18fbd7f6bf694565b153b31c17e5e4c1'
         and  u.id=t.user_id
         and t.type in('COMMON')
         and concat(u.id,u.nick_name,user_name) like '%方向%' limit 0,10;
 ~~~

发现没有返回结果,然后把上面的sql替换成or的方式:
~~~
select u.id,u.nick_name,t.update_time,t.type
  from tribe_user t, user u
  where t.tribe_id = '18fbd7f6bf694565b153b31c17e5e4c1'
        and   u.id=t.user_id
        and t.type =('COMMON')
        and (u.id LIKE '%方向%' or u.nick_name LIKE '%方向%' or u.user_name LIKE '%方向%') limit 0,10;
~~~

发现有结果了,很神奇的样子！！！

后来详细查了concat函数的文档,发现如果concat连接的字段中如果某个字段是NULL,则整个concat()的返回结果都是NULL,然后就修改了下sql,因为本身确定u.id和u.nike_name肯定不能为null,sql修改如下:

~~~
select u.id,u.nick_name,t.update_time,t.type
  from tribe_user t, user u
  where t.tribe_id = '18fbd7f6bf694565b153b31c17e5e4c1'
        and  u.id=t.user_id
        and t.type in('COMMON')
        and concat(u.id,u.nick_name,ifnull(u.user_name,'')) like '%方向%' limit 0,10;
~~~

