---
layout: post
title:  "Elasticsearch5.0新的API规范"
date:   2017-05-12 23:51:01 +0800
categories: elasticsearch
tag: elasticsearch 翻译
sid: 1495172681
---

Elasticsearch的REST API 是基于JSON格式通过HTTP协议进行暴露的。
下面是一些在使用REST API中的概念。

 - [多重索引](https://www.elastic.co/guide/en/elasticsearch/reference/current/multi-index.html)
 - [索引名称支持日期公式](https://www.elastic.co/guide/en/elasticsearch/reference/current/date-math-index-names.html)
 - [常规选项](https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html)
 - [基于URL的访问控制](https://www.elastic.co/guide/en/elasticsearch/reference/current/url-access-control.html)

##多重索引##

Elasticsearch5的大部分api在涉及到索引参数的时候，都支持多重索引，比如简单的像test1,test2,test3这样简单的罗列（或者使用_all来替代）。同时也支持通配符的方式，比如test* or *test or te*t or *test*等，甚至支持“+”(新增)，“-”（移除），比如 +test*,-test3。

所有涉及到多重索引的的API都支持如下的字符串查询参数：

####**ignore_unavailable**####

控制是否忽略那些不存在或者已经关闭的索引，true 或者 false.

####**allow_no_indices**####

是否允许没有索引。控制当涉及到一个含有通配符的表达式没有执行结果的时候是否进行失败处理。比如当我们使用foo*来指定索引名称的时候，如果发现没有任何结果的时候，这个请求会失败。这个参数同样在_all,* 或者在没有索引的时候被指定，也能被用于别名中，但是被关闭的索引除外。

####**expand_wildcards**####

指定通配符作用于哪种具体的索引上，当该参数被指定为open，则作用于所有已经打开的索引中,如果被指定为closed，则被作用于所有被关闭的索引中。

##日期计算##

让日期支持数学计算，这个方案能够让你在一系列时间序列的索引中进行范围查找而不是全部查找，可以过滤部分结果，同时也能维护别名。

减少搜索数量能够在搜索的时候减少集群负载，并且能提高执行效率。比如你正在你每天的日志中去搜寻错误信息，你可以使用日期计算来限制你的搜索范围在附近两天的日志中进行搜索。

**所有的APIS都拥有一个索引参数，用来支持日期计算，格式如下：**

```
<static_name{date_math_expr{date_format|time_zone}}>
```

 - static_name
     固定的文本参数名，作为参数的一部分
 - date_math_expr
    一个动态的日期数学表达式
 - date_format
	日期格式，比如 YYYY-MM-dd，默认为YYYY.MM.dd.
 - time_zone
  时区选项，默认为utc

**使用方式：**

你必须把一个含有日期表达式放入到一个大括号中，所有的指定的字符集必须被URI编码。比如：

```
# GET /<logstash-{now/d}>/_search
GET /%3Clogstash-%7Bnow%2Fd%7D%3E/_search
{
  "query" : {
    "match": {
      "test": "data"
    }
  }
}
```

下面是一些特殊字符的转义说明：

```
<

%3C

>

%3E

/

%2F

{

%7B

}

%7D

|

%7C

+

%2B

:

%3A

,

%2C
```

下面的例子展示了不同形式的索引名称，假如今天是2017-2-22

| 表达式 | 含义 |
|------------- |:-------------:| -----:|
|< logstash-{now/M} >  | logstash-2017.02.22 |
|< logstash-{now/M}> | logstash-2017.02.01 |
|< logstash-{now/M{YYYY.MM}}>  | logstash-2017.02|
|< logstash-{now/M-1M{YYYY.MM}}>  | logstash-2017.01 |
|< logstash-{now/d{YYYY.MM.dd l+12:00}} >| logstash-2017.02.23 |


参考地址：[https://www.elastic.co/guide/en/elasticsearch/reference/current/date-math-index-names.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/date-math-index-names.html)


----------

##常规选项##

**美化结果**

我们可以通过在url后追加?pretty=true，让我们的返回结果会以格式化好的JSON格式返回给我们。同样我们也可以追加?format=yaml，来让我们的返回结果以yaml的格式返回给我们。

**让人可读性更强的输出**

返回结果更加详细，可读性更强。我们可以通过在url后追加?human=false参数来控制是否让返回结果更加具体。

**日期表达式**

 - +1h
添加一个小时
 - -1d
减少一天
 - /d
四舍五入到最近的一天

**简写说明**
| 缩写  | 含义 |
| ------------- | -----:|
| y  | years |
| M  | 	months |
| w | weeks |
| d | days |
| h | hours |
| H |hours |
| m | minutes |
| s | seconds |

一些具体的例子

| 表达式  | 含义 |
| ------------- | -----:|
| now+1h | 当前时间加一小时 |
| now+1h+1m | 当前时间加一小时再加一分钟 |
| now+1h/d | 当前时间加一小时后四舍五入到最近的一天(这里没有优先级) |
| 2015-01-01 ll +1M/d| 2015-01-01加上一天后四舍五入到最进一天

**响应过滤**

所有的REST API都支持在url后面添加一个过滤参数：filter_path，通常用来减少返回结果的信息，就是过滤一些自己想要的结果信息。如：

```
GET /_search?q=elasticsearch&filter_path=took,hits.hits._id,hits.hits._score

```

返回结果如下：

```
{
  "took" : 3,
  "hits" : {
    "hits" : [
      {
        "_id" : "0",
        "_score" : 1.6375021
      }
    ]
  }
}
```
该参数中可以使用通配符来作为返回结果字段的一部分，如：

```
GET /_cluster/state?filter_path=metadata.indices.*.stat*
```

返回结果为：

```
{
  "metadata" : {
    "indices" : {
      "twitter": {"state": "open"}
    }
  }
}
```
如果在一个索引中存在级联关系，而且我们不知道某个字段在哪个层级上，我们可以使用 “**”来作为过滤路径的一部分：

```
GET /_cluster/state?filter_path=routing_table.indices.**.state
```

返回结果：

```
{
  "routing_table": {
    "indices": {
      "twitter": {
        "shards": {
          "0": [{"state": "STARTED"}, {"state": "UNASSIGNED"}],
          "1": [{"state": "STARTED"}, {"state": "UNASSIGNED"}],
          "2": [{"state": "STARTED"}, {"state": "UNASSIGNED"}],
          "3": [{"state": "STARTED"}, {"state": "UNASSIGNED"}],
          "4": [{"state": "STARTED"}, {"state": "UNASSIGNED"}]
        }
      }
    }
  }
}
```

同样支持使用前缀"_"，作为排除一个或多个字段的标志。如：

```
GET /_count?filter_path=-_shards
```
返回结果为：

```
{
  "count" : 5
}
```

下面是一个比较复杂的组合过滤查询,如：

```
GET /_cluster/state?filter_path=metadata.indices.*.state,-metadata.indices.logstash-*

```
返回结果为：

```
{
  "metadata" : {
    "indices" : {
      "index-1" : {"state" : "open"},
      "index-2" : {"state" : "open"},
      "index-3" : {"state" : "open"}
    }
  }
}
```

过滤_source中的字段查询，如：

```
POST /library/book?refresh
{"title": "Book #1", "rating": 200.1}
POST /library/book?refresh
{"title": "Book #2", "rating": 1.7}
POST /library/book?refresh
{"title": "Book #3", "rating": 0.1}
GET /_search?filter_path=hits.hits._source&_source=title&sort=rating:desc

```
返回结果为：

```
{
  "hits" : {
    "hits" : [ {
      "_source":{"title":"Book #1"}
    }, {
      "_source":{"title":"Book #2"}
    }, {
      "_source":{"title":"Book #3"}
    } ]
  }
}
```

**扁平化的设置**

通过flat_settings参数，我们可以获取某个索引中的一些常规信息，这些信息被扁平化为一个层级，比如：

```
curl --user elastic:changeme -XGET 'localhost:9200/merchant/_settings?flat_settings=true'
```

返回结果为：

```
{"merchant":{"settings":{"index.creation_date":"1487663655158","index.number_of_replicas":"1","index.number_of_shards":"5","index.provided_name":"merchant","index.uuid":"ET0mOIQwQZ-40vau3OspvQ","index.version.created":"5020199"}}}
```

如果我们使用：

```
curl --user elastic:changeme -XGET 'localhost:9200/merchant/_settings?flat_settings=false'
```
返回结果则如下：

```
{"merchant":{"settings":{"index":{"creation_date":"1487663655158","number_of_shards":"5","number_of_replicas":"1","uuid":"ET0mOIQwQZ-40vau3OspvQ","version":{"created":"5020199"},"provided_name":"merchant"}}}}
```
**开启错误堆栈**

默认情况下，Elasticsearch在返回结果中不会包含详细的错误堆栈信息。现在我们可以通过一个参数error_trace来强制开启，当设置error_trace=true的时候，返回结果中会包含错误信息的具体堆栈，比如：

```
POST /twitter/_search?size=surprise_me&error_trace=true
```

返回结果为：

```
{
  "error": {
    "root_cause": [
      {
        "type": "illegal_argument_exception",
        "reason": "Failed to parse int parameter [size] with value [surprise_me]",
        "stack_trace": "Failed to parse int parameter [size] with value [surprise_me]]; nested: IllegalArgumentException..."
      }
    ],
    "type": "illegal_argument_exception",
    "reason": "Failed to parse int parameter [size] with value [surprise_me]",
    "stack_trace": "java.lang.IllegalArgumentException: Failed to parse int parameter [size] with value [surprise_me]\n    at org.elasticsearch.rest.RestRequest.paramAsInt(RestRequest.java:175)...",
    "caused_by": {
      "type": "number_format_exception",
      "reason": "For input string: \"surprise_me\"",
      "stack_trace": "java.lang.NumberFormatException: For input string: \"surprise_me\"\n    at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)..."
    }
  },
  "status": 400
}
```


