---
layout: post
title:  "Java8—StringJoiner对字符串操作"
date:   2017-06-23 13:41:01 +0800
categories: java
tag: java 原创
sid: 1498200579
---

> 虽然项目暂时还是使用了JDK1.7,但是在日常开发中还是体会到java8的一些升级给开发带来极大的便利

## 前言

经常在业务中需要将字符串和列表进行互相转换,下面为我封装的工具类:
**列表转换成字符**
~~~java
public static String list2Str(List <String>list,String patten){
		assert list != null;
		StringBuilder stringBuilder = new StringBuilder();
		for (int i=0;i<list.size();i++){
				stringBuilder.append(list.get(i)+patten);
			}else {
				stringBuilder.append(list.get(i));
			}
		}
		return stringBuilder.toString();
	}
~~~
**成字符转换成列表**
~~~java
public static List<String> str2List(String params,String patten,boolean isDeDuplication){
		assert params != null;
		String [] ids = params.split(patten);
		List<String> idList = new ArrayList<>();
		for (String id : ids){
		if (isDeDuplication){
		if (!idList.contains(id)){
                idList.add(id);
            }
		}else{
		    idList.add(id);
		}
    }
		return idList;
	}
~~~

而在Java8中,我们很轻松实现上面的转换:
~~~java
  List<String> list = Arrays.asList("1","2","4","5","6");
String string = String.join(";",list);
~~~
这样就完成了将一个列表转换成以指定分隔符分割的字符串了,但是该方法无法完成去重。

值得注意的是String.join()方法的源码中并没有使用StringBuilder或StringBuffer来实现,而是使用了一个Java8新类StringJoiner来完成.

而StringJoiner的内部也是通过StringBuilder来完成字符串的拼接,只是StringJoiner中少了对分隔符的拼接工作,本质上并没有什么区别

StringJoiner的add操作:
~~~java
 public StringJoiner add(CharSequence newElement) {
        prepareBuilder().append(newElement);
        return this;
    }
~~~

prepareBuilder()方法:
~~~java
 private StringBuilder prepareBuilder() {
        if (value != null) {
            value.append(delimiter);
        } else {
            value = new StringBuilder().append(prefix);
        }
        return value;
    }
~~~

StringJoiner内部也是依靠StringBuilder来操作字符串的,换汤不换药,而String中在JDK8新增的大量join方法中,都是借助于StringJoiner来完成。
