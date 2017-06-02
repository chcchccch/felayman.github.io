---
layout: post
title:  "java8函数式接口FunctionalInterface"
date:   2017-06-02 13:41:01 +0800
categories: java
tag: java 原创
sid: 1496382152
---

>

## 函数式接口

我们先来看看java8之前我们如何创建一个Runnable实例

~~~java
Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println("hello,world");
            }
        };
~~~

我们再来看看java8之后我们如何创建一个Runnable实例
~~~java
 Runnable runnable = () -> System.out.println("hello,world");
~~~

我们发现会变得非常简洁,我们查看Runnable的源码,发现该接口被标注了一个FunctionalInterface注解:

~~~java
@FunctionalInterface
public interface Runnable {
    /**
     * When an object implementing interface <code>Runnable</code> is used
     * to create a thread, starting the thread causes the object's
     * <code>run</code> method to be called in that separately executing
     * thread.
     * <p>
     * The general contract of the method <code>run</code> is that it may
     * take any action whatsoever.
     *
     * @see     java.lang.Thread#run()
     */
    public abstract void run();
}
~~~

@FunctionalInterface注解式java8为函数式接口引入了一个新注解,主要用于编译级错误检查，加上该注解，当你写的接口不符合函数式接口定义的时候，编译器会报错。
被@FunctionalInterface标注的接口只能有一个方法
比如:
~~~java
@FunctionalInterface
public interface DataProceeService {

    void method1();

    void method2();

}

~~~
上述的接口在编译期间就会报错。被@FunctionalInterface标注的接口一般推荐使用Lambda表达式,虽然你也可以使用传统的方法来实现该接口

在之前很多开源项目中,如果使用了Runnable实例来完成一些多线程任务,无非有两种途径:

- 命名多个Runnable接口的实现
- 创建多个Runnable接口的匿名内部类

这样做的缺点很明显:

- 代码冗余
- 逻辑结构不清晰

我在之前有过类似的痛苦经历:

JSONExtender类,用于扩展和修改原有的JSONArray的内容
~~~java
public interface JSONExtender {
    JSONArray extend(JSONArray json);
}
~~~

WebResponse类,用于返回给前端,返回请求状态、返回信息和请求结果
~~~java
public final class WebResponse {

	private String  message;

	private String status;

	private String error;

	private JSON data;

	public String getMessage() {
		return message;
	}

	public void setMessage(String message) {
		this.message = message;
	}

	public String getStatus() {
		return status;
	}

	public void setStatus(String status) {
		this.status = status;
	}

	public String getError() {
		return error;
	}

	public void setError(String error) {
		this.error = error;
	}

	public JSON getData() {
		return data;
	}

	public void setData(JSON data) {
		this.data = data;
	}

	public WebResponse(){}

	public WebResponse(String message, String status) {
		this.message = message;
		this.status = status;
	}

	public WebResponse(String message, String status, JSONObject data) {
		this.message = message;
		this.status = status;
		this.data = data;
	}

	public static String success(){
		WebResponse response = new WebResponse();
		response.message = "success";
		response.status = "200";
		return JSONObject.toJSONString(response);
	}

	public static String success(JSON data){
		WebResponse response = new WebResponse();
		response.message = "success";
		response.status = "200";
		if (data == null){
			response.data = new JSONArray();
		}else {
			response.data = data;
		}
		return JSONObject.toJSONString(response);
	}

	public static String success(String key,Object value){
		WebResponse response = new WebResponse();
		response.message = "success";
		response.status = "200";
		JSONObject json = new JSONObject();
		json.put(key,value);
		response.data = json;
		return JSONObject.toJSONString(response);
	}

	public static String success(JSONExtender jsonExtender, JSONArray json){
		WebResponse response = new WebResponse();
		response.message = "success";
		response.status = "200";
		response.data = jsonExtender.extend(json);
		return JSONObject.toJSONString(response);
	}

	public static String fail(){
		WebResponse response = new WebResponse();
		response.message = "fail";
		response.status = "500";
		return JSONObject.toJSONString(response);
	}

	public static String fail(String error){
		WebResponse response = new WebResponse();
		response.message = "fail";
		response.status = "500";
		response.error = error;
		return JSONObject.toJSONString(response);
	}

	public static String fail(String error, String status){
		WebResponse response = new WebResponse();
		response.message = "fail";
		response.status = status;
		response.error = error;
		return JSONObject.toJSONString(response);
	}

	public static String addField(WebResponse response,String key,Object value){
		JSONObject jsonObject = JSON.parseObject(JSONObject.toJSONString(response));
		jsonObject.put(key,value);
		return jsonObject.toJSONString();
	}
}
~~~

该类中有一个方法:
~~~java
public static String success(JSONExtender jsonExtender, JSONArray json){
		WebResponse response = new WebResponse();
		response.message = "success";
		response.status = "200";
		response.data = jsonExtender.extend(json);
		return JSONObject.toJSONString(response);
	}
~~~
接受两个参数,第一个参数为JSONExtender实例,用于扩展第二个参数,项目中有一个搜索接口,需要对返回的数据做修改和填充其他数据:
~~~java
private String  fillingExtraInfo(final JSONArray jsonArray, final String userId, final String imgSize, final String avatarSize, SearchType searchType, final boolean isFilterContest){
        JSONArray copyJsonArr = new JSONArray();
        assert searchType != null;
        switch (searchType) {
            case group:
            case graphic:
            case set:
                return WebResponse.success(new JSONExtender() {
                    @Override
                    public JSONArray extend(JSONArray json) {
                        json = fillingExtraSetInfo(json,imgSize,avatarSize);
                        return json;
                    }
                },jsonArray);
            case photoAndGroup:
            case photo:
                return WebResponse.success(new JSONExtender() {
                    public JSONArray extend(JSONArray json) {
                        json = fillingExtraPhotoInfo(json,userId,imgSize,avatarSize);
                        return json;
                    }
                }, jsonArray);
            case tribe:
                return WebResponse.success(new JSONExtender() {
                    JSONArray jsonArray = new JSONArray();
                    public JSONArray extend(JSONArray json) {
                        json = finllingExtraTribeInfo(json,userId,imgSize,avatarSize);
                        return json;
                    }
                }, jsonArray);
        return null;
    }
~~~
当然这也的写法,并不是很合理,可以为每个case,创建一个独立的JSONExtender实现,这样就能把修改和扩展的业务代码包装在指定的实现类中,但是也有比较特殊的时候,
比如,需要在每个case中调用上下文的方法,当需要在匿名内部类中调用上下文的方法的时候,这个时候就只能使用匿名内部类的方式。

当然 我们可以使用java8提供的Lambda表达式来改写上述冗余的代码:
~~~java
private String  fillingExtraInfo2(final JSONArray jsonArray, final String userId, final String imgSize, final String avatarSize, SearchType searchType, final boolean isFilterContest){
        JSONArray copyJsonArr = new JSONArray();
        assert searchType != null;
        switch (searchType) {
            case group:
            case graphic:
                return WebResponse.success();
            case set:
                return WebResponse.success(json -> fillingExtraSetInfo(jsonArray,imgSize,avatarSize),jsonArray);
            case photoAndGroup:
            case photo:
                return WebResponse.success(json -> fillingExtraPhotoInfo(jsonArray,userId,imgSize,avatarSize),jsonArray);
            case tribe:
                return WebResponse.success(json -> finllingExtraTribeInfo(json,userId,imgSize,avatarSize), jsonArray);
        }
        return null;
    }
~~~

我们可以看到,使用Lambda表达式来修改后的代码,格外清爽和简洁。


## Lambda表达式

> Lambda 表达式”(lambda expression)是一个匿名函数，Lambda表达式基于数学中的λ演算得名，直接对应于其中的lambda抽象(lambda abstraction)，是一个匿名函数，即没有函数名的函数。Lambda表达式可以表示闭包（注意和数学传统意义上的不同）

我们继续拿上面的JSONExtender接口来说,我们创建JSONExtender实例的方法有下面几种

1. 指明返回类
~~~java
JSONExtender jsonExtender  = (JSONArray json) -> new JSONArray();
~~~
2. 不指明返回类型
~~~java
JSONExtender jsonExtender = json -> new JSONArray();
~~~
3. 使用语句块
~~~java
JSONExtender jsonExtender = json -> {return new JSONArray();};
~~~

可见lambda表达式有三部分组成：参数列表,箭头(->),以及一个表达式或语句块
