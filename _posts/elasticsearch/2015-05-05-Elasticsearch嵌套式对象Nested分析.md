---
layout: post
title:  "Elasticsearch嵌套式对象Nested分析"
date:   2017-05-05 23:44:01 +0800
categories: elasticsearch
tag: elasticsearch
---


nested结构是Elasticsearch提供关系存储的一种特殊的结构,是NOSQL的一种高级特性,在传统的关系型sql中,很难做到一行记录中存储某个实体以及附属的内容，比如某个用户下评论数据，或某个订单下的所有商品等这种关系比较强的内容。当然传统sql也能做到,比如我们当想存储一个订单和该订单下的商品内容。我们可以定义一个text类型的字段,以json的方式存储不同的商品信息,但是这样有一个致命的缺点,就是性能非常差，其次无法高效的支持搜索。
我们现在模拟这样一个场景。

我们设计了一个博客系统.需要支持搜索该文章的评论信息。

我们当然可以利用传统的sql来做存储,我们可以这样设计文章与评论的表结构。

post表结构：
```
id,title,body,tags,comment_id
```
comments表结构：

```
id,name,comment,age,starts,date
```

然后利用post表中的comment_id与comments表中的id做外键关联，这样就可以存储文章与评论直接的关系,大部分设计也是如此。

然而问题来了,当我需要对评论内容进行模糊查询的时候,我们可能需要对两个表进行关联查询,
比如我想这样查询：查询title中包含"elasticsearch教程"的文章的评论中包含"视频教程地址"
这个时候我们要面临两个困难,当表稍微大一些的时候,需要对2个表的数据进行like查询,还需要对两个表进行联合查询,这个性能是无法忍受的。

利用Elasticsearch的Nested结构可以很好的解决这个问题，下面是Nested的一些常识，可以参考：
[nested结构介绍](https://es.xiaoleilu.com/402_Nested/30_Nested_objects.html)
```
PUT /my_index/blogpost/1
{
  "title": "Nest eggs",
  "body":  "Making your money work...",
  "tags":  [ "cash", "shares" ],
  "comments": [ <1>
    {
      "name":    "John Smith",
      "comment": "Great article",
      "age":     28,
      "stars":   4,
      "date":    "2014-09-01"
    },
    {
      "name":    "Alice White",
      "comment": "More like this please",
      "age":     31,
      "stars":   5,
      "date":    "2014-10-22"
    }
  ]
}
```
我们如果直接在marvel插件中这样利用动态映射的话,comments栏位会被自动建立为一个object栏位，因为所有内容都在同一个文档中，使搜寻时并不需要连接(join)blog文章与回应，因此搜寻表现更加优异。
问题在於以上的文档可能会如下所示的匹配一个搜寻：

```
GET /_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "name": "Alice" }},
        { "match": { "age":  28      }} <1>
      ]
    }
  }
}
```
**Alice是31岁，而不是28岁！**

造成跨对象配对的原因如同我们在对象阵列中所讨论到，在于我们优美结构的JSON文档在索引中被扁平化为下方的 键-值 形式：

```
{
  "title":            [ eggs, nest ],
  "body":             [ making, money, work, your ],
  "tags":             [ cash, shares ],
  "comments.name":    [ alice, john, smith, white ],
  "comments.comment": [ article, great, like, more, please, this ],
  "comments.age":     [ 28, 31 ],
  "comments.stars":   [ 4, 5 ],
  "comments.date":    [ 2014-09-01, 2014-10-22 ]
}
```
Alice与31 以及 John与2014-09-01 之间的关联已经无法挽回的消失了。 当object类型的栏位用于储存单一对象是非常有用的。 从搜寻的角度来看，对於排序一个对象阵列来说关联是不需要的东西。
这是嵌套对象被设计来解决的问题。 藉由映射commments栏位为nested类型而不是object类型， 每个嵌套对象会被索引为一个隐藏分割文档，例如：

```
{ <1>
  "comments.name":    [ john, smith ],
  "comments.comment": [ article, great ],
  "comments.age":     [ 28 ],
  "comments.stars":   [ 4 ],
  "comments.date":    [ 2014-09-01 ]
}
{ <2>
  "comments.name":    [ alice, white ],
  "comments.comment": [ like, more, please, this ],
  "comments.age":     [ 31 ],
  "comments.stars":   [ 5 ],
  "comments.date":    [ 2014-10-22 ]
}
{ <3>
  "title":            [ eggs, nest ],
  "body":             [ making, money, work, your ],
  "tags":             [ cash, shares ]
}
```
<1> 第一个嵌套对象
<2> 第二个嵌套对象
<3> 根或是父文档
藉由分别索引每个嵌套对象，对象的栏位中保持了其关联。 我们的查询可以只在同一个嵌套对象都匹配时才回应。
不仅如此，因嵌套对象都被索引了，连接嵌套对象至根文档的查询速度非常快--几乎与查询单一文档一样快。
这些额外的嵌套对象被隐藏起来，我们无法直接访问他们。 为了要新增丶修改或移除一个嵌套对象，我们必须重新索引整个文档。 要牢记搜寻要求的结果并不是只有嵌套对象，而是整个文档。

## **嵌套对象映射** ##
设定一个nested栏位很简单--在你会设定为object类型的地方，改为nested类型：

```
PUT /my_index
{
  "mappings": {
    "blogpost": {
      "properties": {
        "comments": {
          "type": "nested", <1>
          "properties": {
            "name":    { "type": "string"  },
            "comment": { "type": "string"  },
            "age":     { "type": "short"   },
            "stars":   { "type": "short"   },
            "date":    { "type": "date"    }
          }
        }
      }
    }
  }
}
```
<1> 一个nested栏位接受与object类型相同的参数。
所需仅此而已。 任何comments对象会被索引为分离嵌套对象。 参考更多 [nested type reference docs](http://bit.ly/1KNQEP9)

## **查询嵌套对象** ##

因嵌套对象(nested objects)会被索引为分离的隐藏文档，我们不能直接查询它们。而是使用 nested查询或 nested 过滤器来存取它们：

```
GET /my_index/blogpost/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "title": "eggs" }}, <1>
        {
          "nested": {
            "path": "comments", <2>
            "query": {
              "bool": {
                "must": [ <3>
                  { "match": { "comments.name": "john" }},
                  { "match": { "comments.age":  28     }}
                ]
        }}}}
      ]
}}}
```
<1> title条件运作在根文档上
<2> nested条件深入嵌套的comments栏位。它不会在存取根文档的栏位，或是其他嵌套文档的栏位。
<3> comments.name以及comments.age运作在相同的嵌套文档。

> 一个nested栏位可以包含其他nested栏位。 相同的，一个nested查询可以包含其他nested查询。 嵌套阶层会如同你预期的运作。

当然，一个nested查询可以匹配多个嵌套文档。 每个文档的匹配会有各自的关联分数，但多个分数必须减少至单一分数才能应用至根文档。

在预设中，它会平均所有嵌套文档匹配的分数。这可以藉由设定score_mode参数为avg, max, sum或甚至none(为了防止根文档永远获得1.0的匹配分数时)来控制。

```
GET /my_index/blogpost/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "title": "eggs" }},
        {
          "nested": {
            "path":       "comments",
            "score_mode": "max", <1>
            "query": {
              "bool": {
                "must": [
                  { "match": { "comments.name": "john" }},
                  { "match": { "comments.age":  28     }}
                ]
        }}}}
      ]
}}}
```

<1> 从最匹配的嵌套文档中给予根文档的_score值。

> nested过滤器类似於nested查询，除了无法使用score_mode参数。 只能使用在filter context—例如在filtered查询中--其作用类似其他的过滤器： 包含或不包含，但不评分。
nested过滤器的结果本身不会缓存，通常缓存规则会被应用於nested过滤器之中的过滤器。

## **嵌套排序** ##

我们可以依照嵌套栏位中的值来排序，甚至藉由分离嵌套文档中的值。为了使其结果更加有趣，我们加入另一个记录：

```
PUT /my_index/blogpost/2
{
  "title": "Investment secrets",
  "body":  "What they don't tell you ...",
  "tags":  [ "shares", "equities" ],
  "comments": [
    {
      "name":    "Mary Brown",
      "comment": "Lies, lies, lies",
      "age":     42,
      "stars":   1,
      "date":    "2014-10-18"
    },
    {
      "name":    "John Smith",
      "comment": "You're making it up!",
      "age":     28,
      "stars":   2,
      "date":    "2014-10-16"
    }
  ]
}
```
想像我们要取回在十月中有收到回应的blog文章，并依照所取回的各个blog文章中最少stars数量的顺序作排序。 这个搜寻请求如下：

```
GET /_search
{
  "query": {
    "nested": { <1>
      "path": "comments",
      "filter": {
        "range": {
          "comments.date": {
            "gte": "2014-10-01",
            "lt":  "2014-11-01"
          }
        }
      }
    }
  },
  "sort": {
    "comments.stars": { <2>
      "order": "asc",   <2>
      "mode":  "min",   <2>
      "nested_filter": { <3>
        "range": {
          "comments.date": {
            "gte": "2014-10-01",
            "lt":  "2014-11-01"
          }
        }
      }
    }
  }
}
```
<1> nested查询限制了结果为十月份收到回应的blog文章。
<2> 结果在所有匹配的回应中依照comment.stars栏位的最小值(min)作递增(asc)的排序。
<3> 排序条件中的nested_filter与主查询query条件中的nested查询相同。 於下一个下方解释。
为什么我们要在nested_filter重复写上查询条件？ 原因是排序在於执行查询后才发生。 此查询匹配了在十月中有收到回应的blog文章，回传blog文章文档作为结果。 如果我们不加上nested_filter条件，我们最後会依照任何blog文章曾经收到过的回应作排序，而不是在十月份收到的。

## **嵌套-聚合** ##

如同我们在查询时需要使用nested查询来存取嵌套对象，专门的nested集合使我们可以取得嵌套对象中栏位的集合：

```
GET /my_index/blogpost/_search?search_type=count
{
  "aggs": {
    "comments": { <1>
      "nested": {
        "path": "comments"
      },
      "aggs": {
        "by_month": {
          "date_histogram": { <2>
            "field":    "comments.date",
            "interval": "month",
            "format":   "yyyy-MM"
          },
          "aggs": {
            "avg_stars": {
              "avg": { <3>
                "field": "comments.stars"
              }
            }
          }
        }
      }
    }
  }
}
```
<1> nested集合深入嵌套对象的comments栏位
<2> 评论基於comments.date栏位被分至各个月份分段
<3> 每个月份分段单独计算星号的平均数
结果显示集合发生於嵌套文档层级：

```
...
"aggregations": {
  "comments": {
     "doc_count": 4, <1>
     "by_month": {
        "buckets": [
           {
              "key_as_string": "2014-09",
              "key": 1409529600000,
              "doc_count": 1, <1>
              "avg_stars": {
                 "value": 4
              }
           },
           {
              "key_as_string": "2014-10",
              "key": 1412121600000,
              "doc_count": 3, <1>
              "avg_stars": {
                 "value": 2.6666666666666665
              }
           }
        ]
     }
  }
}
...
```
<1> 此处总共有四个comments: 一个在九月以及三个在十月

## **什麽时候要使用嵌套对象** ##

嵌套对象对於当有一个主要实体(如blogpost)，加上有限数量的紧密相关实体(如comments)是非常有用的。 有办法能以评论内容找到blog文章很有用，且nested查询及过滤器提供短查询时间连接(fast query-time joins)。
嵌套模型的缺点如下：
如欲新增丶修改或删除一个嵌套文档，则必须重新索引整个文档。因此越多嵌套文档造成越多的成本。
搜寻请求回传整个文档，而非只有匹配的嵌套文档。 虽然有个进行中的计画要支持只回传根文档及最匹配的嵌套文档，但目前并未支持。
有时你需要完整分离主要文档及其关连实体。 父-子关系提供这一个功能。

参考地址：[nested](https://es.xiaoleilu.com/402_Nested/30_Nested_objects.html)


----------

基于上面的介绍,我们下面来介绍nested结构实际的应用(图片搜索)。

我们拿一家图片搜索网站https://www.500px.me来进行举例说明.

图片是一个包含很多附加属性的类型。为此我们为图片创建一个映射(比较简单,实际很复杂)

```
PUT /photo/
{
  "settings": {
    "number_of_shards": 4,
    "number_of_replicas": 0
  },
  "mappings": {
    "photo": {
      "properties": {
        "id": {
          "type": "String"
        },
        "title": {
          "type": "String"
        },
        "category":{
          "type": "String"
        },
        "uploader_name": {
          "type": "String"
        },
        "uploader_id": {
          "type": "String"
        },
        "keyword": {
          "properties":{
            "categoryId":{
              "type": "String"
            },
            "content":{
               "type": "String"
            }
          }
        }
      }
    }
  }
}
```
其中我们需要关注的是keyword这个字段,我们开始的时候为这个字段设置普通的类型或者依靠动态映射创建默认类型。为什么会有这种数据格式呢？因为图片本身会有一个分类，而关键词一般也会有一个分类(当然也会有有其他额外信息)，

然后我们插入几条数据。

```
POST /photo/photo/1
{
  "id":"1",
  "title":"北京天坛风景",
  "uploader_name":"felayman",
  "uploader_id":"1",
  "keyword":[
    {
      "categoryId":"1",
      "content":"北京"
    },
    {
      "categoryId":"2",
      "content":"天坛"
    },
    {
      "categoryId":"3",
      "content":"秋天"
    },
    {
      "categoryId":"4",
      "content":"旅游"
    }
    ]
}
POST /photo/photo/2
{
  "id":"2",
  "title":"美国波士顿",
  "uploader_name":"felayman",
  "uploader_id":"1",
  "keyword":[
    {
      "categoryId":"1",
      "content":"波士顿"
    },
    {
      "categoryId":"2",
      "content":"夏天"
    },
    {
      "categoryId":"3",
      "content":"旅游"
    }
    ]
}
POST /photo/photo/3
{
  "id":"3",
  "title":"北京西单商业街",
  "uploader_name":"felayman",
  "uploader_id":"1",
  "keyword":[
    {
      "categoryId":"1",
      "content":"北京"
    },
    {
      "categoryId":"2",
      "content":"西单"
    },
    {
      "categoryId":"3",
      "content":"夏天"
    },
    {
      "categoryId":"4",
      "content":"旅游"
    },
    {
       "categoryId":"5",
      "content":"中国"
    },
    {
       "categoryId":"6",
      "content":"龙"
    }
    ]
}
```
我们尝试进行如下查询：

```
GET /photo/photo/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "content": "夏天"
          }
        },
        {
          "match": {
            "categoryId": "3"
          }
        }
      ]
    }
  }
}
```
查询结果为:

```
{
   "took": 4,
   "timed_out": false,
   "_shards": {
      "total": 4,
      "successful": 4,
      "failed": 0
   },
   "hits": {
      "total": 2,
      "max_score": 0.19178301,
      "hits": [
         {
            "_index": "photo",
            "_type": "photo",
            "_id": "2",
            "_score": 0.19178301,
            "_source": {
               "id": "2",
               "title": "美国波士顿",
               "uploader_name": "felayman",
               "uploader_id": "1",
               "keyword": [
                  {
                     "categoryId": "1",
                     "content": "波士顿"
                  },
                  {
                     "categoryId": "2",
                     "content": "夏天"
                  },
                  {
                     "categoryId": "3",
                     "content": "旅游"
                  }
               ]
            }
         },
         {
            "_index": "photo",
            "_type": "photo",
            "_id": "3",
            "_score": 0.17260471,
            "_source": {
               "id": "3",
               "title": "北京西单商业街",
               "uploader_name": "felayman",
               "uploader_id": "1",
               "keyword": [
                  {
                     "categoryId": "1",
                     "content": "北京"
                  },
                  {
                     "categoryId": "2",
                     "content": "西单"
                  },
                  {
                     "categoryId": "3",
                     "content": "夏天"
                  },
                  {
                     "categoryId": "4",
                     "content": "旅游"
                  },
                  {
                     "categoryId": "5",
                     "content": "中国"
                  },
                  {
                     "categoryId": "6",
                     "content": "龙"
                  }
               ]
            }
         }
      ]
   }
}
```

其实我们的目的是查询关键词中含有分类为3，内容为"夏天"的图片,搜索结果出现了2条结果，很显然，第一条并不满足我们的需求。因为其分类为2.

造成这个原因的本质就是在创建映射的时候 photo作为主体，与keyword的关联性不强。

这个时候我们把keyword修改成nested结构，我们再试下结果：

```
PUT /photo/
{
  "settings": {
    "number_of_shards": 4,
    "number_of_replicas": 0
  },
  "mappings": {
    "photo": {
      "properties": {
        "id": {
          "type": "String"
        },
        "title": {
          "type": "String"
        },
        "category":{
          "type": "String"
        },
        "uploader_name": {
          "type": "String"
        },
        "uploader_id": {
          "type": "String"
        },
        "keyword": {
          "type":"nested",
          "properties":{
            "categoryId":{
              "type": "String"
            },
            "content":{
               "type": "String"
            }
          }
        }
      }
    }
  }
}
```
同样插入几条数据：

```
POST /photo/photo/1
{
  "id":"1",
  "title":"北京天坛风景",
  "uploader_name":"felayman",
  "uploader_id":"1",
  "keyword":[
    {
      "categoryId":"1",
      "content":"北京"
    },
    {
      "categoryId":"2",
      "content":"天坛"
    },
    {
      "categoryId":"3",
      "content":"秋天"
    },
    {
      "categoryId":"4",
      "content":"旅游"
    }
    ]
}
POST /photo/photo/2
{
  "id":"2",
  "title":"美国波士顿",
  "uploader_name":"felayman",
  "uploader_id":"1",
  "keyword":[
    {
      "categoryId":"1",
      "content":"波士顿"
    },
    {
      "categoryId":"2",
      "content":"夏天"
    },
    {
      "categoryId":"3",
      "content":"旅游"
    }
    ]
}
POST /photo/photo/3
{
  "id":"3",
  "title":"北京西单商业街",
  "uploader_name":"felayman",
  "uploader_id":"1",
  "keyword":[
    {
      "categoryId":"1",
      "content":"北京"
    },
    {
      "categoryId":"2",
      "content":"西单"
    },
    {
      "categoryId":"3",
      "content":"夏天"
    },
    {
      "categoryId":"4",
      "content":"旅游"
    },
    {
       "categoryId":"5",
      "content":"中国"
    },
    {
       "categoryId":"6",
      "content":"龙"
    }
    ]
}
```

执行同样的查询：

```
GET /photo/photo/_search
{
  GET /photo/photo/_search
{
  "query": {
    "nested": {
      "path": "keyword",
      "query": {
        "bool": {
          "must": [
            {
              "match": {
                "content": "夏天"
              }
            },
            {
              "match": {
                "categoryId": "3"
              }
            }
          ]
        }
      }
    }
  }
}
```
需要注意的是,在进行nested语法查询的时候,我们需要用一个nested把进行的查询包起来，并加上查询的路径来识别该查询是 一个nested结构。

这个时候,查询结果为:

```
{
   "took": 5,
   "timed_out": false,
   "_shards": {
      "total": 4,
      "successful": 4,
      "failed": 0
   },
   "hits": {
      "total": 1,
      "max_score": 2.8159537,
      "hits": [
         {
            "_index": "photo",
            "_type": "photo",
            "_id": "3",
            "_score": 2.8159537,
            "_source": {
               "id": "3",
               "title": "北京西单商业街",
               "uploader_name": "felayman",
               "uploader_id": "1",
               "keyword": [
                  {
                     "categoryId": "1",
                     "content": "北京"
                  },
                  {
                     "categoryId": "2",
                     "content": "西单"
                  },
                  {
                     "categoryId": "3",
                     "content": "夏天"
                  },
                  {
                     "categoryId": "4",
                     "content": "旅游"
                  },
                  {
                     "categoryId": "5",
                     "content": "中国"
                  },
                  {
                     "categoryId": "6",
                     "content": "龙"
                  }
               ]
            }
         }
      ]
   }
}
```

我们可以看到,查询的结果正是我们想要的结果。

上面的例子可能不能完全说明nested的重要性，下面我们更深入一点，我们会拿出一个更实用的场景,一般情况下(在https://500px.me),图片网站进行搜索的时候会更新关键词的权重,来调节用户搜索效果，比如我们搜索含有 "城市" 的图片，搜索结果可能会随用户的点击对相对搜索的关键词进行调整,比如某张图片含有"城市"的权重为0.9，我们可能因为用户真实点击来将其修改成"0.91"来增加该关键词在该张图片的搜索权重。

我们重新设计我们的图片的映射:

```
{
   "index_v2": {
      "mappings": {
         "resource": {
            "properties": {
               "categoryId": {
                  "type": "string"
               },
               "commentCount": {
                  "type": "long"
               },
               "contestTags": {
                  "type": "string"
               },
               "createdDate": {
                  "type": "long"
               },
               "createdTime": {
                  "type": "long"
               },
               "description": {
                  "type": "string"
               },
               "doc": {
                  "properties": {
                     "keywords": {
                        "properties": {
                           "keyword": {
                              "type": "string"
                           },
                           "score": {
                              "type": "double"
                           }
                        }
                     }
                  }
               },
               "exifInfo": {
                  "properties": {
                     "aperture": {
                        "type": "string"
                     },
                     "dateTimeDigitized": {
                        "type": "long"
                     },
                     "dateTimeOriginal": {
                        "type": "long"
                     },
                     "exposureTime": {
                        "type": "string"
                     },
                     "exposureTimeVcg": {
                        "type": "string"
                     },
                     "focalLength": {
                        "type": "string"
                     },
                     "gpsAltitudeVcg": {
                        "type": "string"
                     },
                     "gpsLatitude": {
                        "type": "string"
                     },
                     "gpsLocationVcg": {
                        "type": "string"
                     },
                     "gpsLongitude": {
                        "type": "string"
                     },
                     "iso": {
                        "type": "string"
                     },
                     "lens": {
                        "type": "string"
                     },
                     "make": {
                        "type": "string"
                     },
                     "model": {
                        "type": "string"
                     },
                     "modelVcg": {
                        "type": "string"
                     },
                     "resourceId": {
                        "type": "string"
                     },
                     "uploadTime": {
                        "type": "long"
                     }
                  }
               },
               "geoCoordinates": {
                  "type": "geo_point"
               },
               "hasCover": {
                  "type": "integer"
               },
               "height": {
                  "type": "integer"
               },
               "hotUpDate": {
                  "type": "long"
               },
               "id": {
                  "type": "string"
               },
               "keywords": {
                  "type": "nested",
                  "properties": {
                     "keyword": {
                        "type": "string"
                     },
                     "score": {
                        "type": "double"
                     }
                  }
               },
               "latitude": {
                  "type": "long"
               },
               "licenceId": {
                  "type": "string"
               },
               "likeCount": {
                  "type": "long"
               },
               "location": {
                  "type": "string"
               },
               "longitude": {
                  "type": "long"
               },
               "openState": {
                  "type": "string"
               },
               "origin": {
                  "type": "string"
               },
               "originId": {
                  "type": "integer"
               },
               "photoCount": {
                  "type": "long"
               },
               "photos": {
                  "type": "nested",
                  "properties": {
                     "createdTime": {
                        "type": "long"
                     },
                     "description": {
                        "type": "string"
                     },
                     "exifInfo": {
                        "type": "string"
                     },
                     "groupId": {
                        "type": "string"
                     },
                     "height": {
                        "type": "long"
                     },
                     "id": {
                        "type": "string"
                     },
                     "state": {
                        "type": "long"
                     },
                     "updateTime": {
                        "type": "long"
                     },
                     "url": {
                        "type": "string"
                     },
                     "userId": {
                        "type": "string"
                     },
                     "width": {
                        "type": "long"
                     }
                  }
               },
               "privacy": {
                  "type": "integer"
               },
               "profileSortTime": {
                  "type": "long"
               },
               "rating": {
                  "type": "double"
               },
               "rating2": {
                  "type": "long"
               },
               "ratingMax": {
                  "type": "double"
               },
               "ratingMax2": {
                  "type": "long"
               },
               "ratingMaxDate": {
                  "type": "long"
               },
               "recommendTime": {
                  "type": "long"
               },
               "refer": {
                  "type": "string"
               },
               "resourceId": {
                  "type": "string"
               },
               "resourceType": {
                  "type": "integer"
               },
               "riseUpDate": {
                  "type": "long"
               },
               "state": {
                  "type": "integer"
               },
               "tag": {
                  "type": "string"
               },
               "title": {
                  "type": "string"
               },
               "tribeTags": {
                  "type": "string"
               },
               "uploadedDate": {
                  "type": "long"
               },
               "uploaderId": {
                  "type": "string"
               },
               "uploaderName": {
                  "type": "string"
               },
               "url": {
                  "type": "string"
               },
               "weather": {
                  "type": "string"
               },
               "width": {
                  "type": "integer"
               }
            }
         }
      }
   }
}
```
上面的映射中，我们需要注意的是：

```
"keywords": {
                  "type": "nested",
                  "properties": {
                     "keyword": {
                        "type": "string"
                     },
                     "score": {
                        "type": "double"
                     }
                  }
               }
```

每张图片都可能有若干个关键词,每个关键词有不同的score，为什么会这样设计呢？因为任何一张图片在随着用户上传的时候都可能任意设置,我们如何去除哪些与图内容关联度不高的关键词呢？那就需要来调节每一张图片中所有关键词的权重来识别。

比如一张图片中同时包含"草坪","露天","露营","吃饭","家庭","夏天","午后","公园",其实这张图片的主题就是“一家人在某个公园的草坪上露营吃饭”,整张照片可能突出"露营"这个关键词,而与其他关键词关联度并不是很高,这个时候我们就需要来调节这些关键词的score值,当用户在数亿张图片库中搜索关键词中含有"露营"的时候,我们会针对所有含有"露营"这个关键词的图片进行查询,然后对这个关键词自带的score进行排序,那么搜索的结果就更符合用户的搜索(这里不使用elasticsearch自带的评分机制)。

如果我们不将keywords设置成nested结构的时候,我们很遗憾就无法去利用score来对相同的关键词进行排序，因为会出现最上述的情况,每个photo文档都被分割成互五关联的键-值 形式。