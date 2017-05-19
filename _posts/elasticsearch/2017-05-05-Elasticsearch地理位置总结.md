---
layout: post
title:  "Elasticsearch地理位置总结"
date:   2017-05-05 23:43:01 +0800
categories: elasticsearch
tag: elasticsearch
sid: 1495122680
---

翻译版本：https://es.xiaoleilu.com/310_Geopoints/00_Intro.html
官方原文：https://www.elastic.co/guide/en/elasticsearch/guide/current/geoloc.html

**本文只是针对这些内容通过具体的例子用java来实现其具体细节，如果只想看java实现部分，请直接往下面代码实现部分看**
    
地理坐标点（geo-point） 是指地球表面可以用经纬度描述的一个点。地理坐标点可以用来计算两个坐标位置间的距离，或者判断一个点是否在一个区域中。地理坐标点不能被动态映射（dynamic mapping）自动检测，而是需要显式声明对应字段类型为 geo_point 。
```
PUT /attractions
{
  "mappings": {
    "restaurant": {
      "properties": {
        "name": {
          "type": "string"
        },
        "location": {
          "type": "geo_point"
        }
      }
    }
  }
}
```
   如上例，location 被声明为 geo_point 后，我们就可以索引包含了经纬度信息的文档了。经纬度信息的形式可以是字符串，数组或者对象。
```
PUT /attractions/restaurant/1
{
  "name":     "Chipotle Mexican Grill",
  "location": "40.715, -74.011" <1>
}

PUT /attractions/restaurant/2
{
  "name":     "Pala Pizza",
  "location": { <2>
    "lat":     40.722,
    "lon":    -73.989
  }
}

PUT /attractions/restaurant/3
{
  "name":     "Mini Munchies Pizza",
  "location": [ -73.983, 40.719 ] <3>
}
```

- <1> 以半角逗号分割的字符串形式 "lat,lon"；
- <2> 明确以 lat 和 lon 作为属性的对象；
- <3> 数组形式表示 [lon,lat]。

> 注意 :
> 可能所有人都至少踩过一次这个坑：地理坐标点用字符串形式表示时是纬度在前，经度在后（"latitude,longitude"），而数组形式表示时刚好相反，是经度在前，纬度在后（[longitude,latitude]）。其实，在 Elasticesearch 内部，不管字符串形式还是数组形式，都是纬度在前，经度在后。不过早期为了适配 GeoJSON 的格式规范，调整了数组形式的表示方式。因此，在使用地理位置（geolocation）的路上就出现了这么一个“捕熊器”，专坑那些不了解这个陷阱的使用者。
    
##**通过地理坐标点过滤**##
有四种地理坐标点相关的过滤方式可以用来选中或者排除文档：

 - geo_bounding_box::
找出落在指定矩形框中的坐标点
 - geo_distance::
找出与指定位置在给定距离内的点
 - geo_distance_range::
找出与指定点距离在给定最小距离和最大距离之间的点
 - geo_polygon::
找出落在多边形中的点。这个过滤器使用代价很大。当你觉得自己需要使用它，最好先看看 geo-shapes

所有这些过滤器的工作方式都相似： 把 索引中所有文档（而不仅仅是查询中匹配到的部分文档，见 fielddata-intro）的经纬度信息都载入内存，然后每个过滤器执行一个轻量级的计算去判断当前点是否落在指定区域。
提示

地理坐标过滤器使用代价昂贵 —— 所以最好在文档集合尽可能少的场景使用。 你可以先使用那些简单快捷的过滤器，比如 term 或者 range，来过滤掉尽可能多的文档，最后才交给地理坐标过滤器处理。

布尔型过滤器（bool filter）会自动帮你做这件事。 它会优先让那些基于“bitset”的简单过滤器(见 filter-caching)来过滤掉尽可能多的文档，然后依次才是地理坐标过滤器或者脚本类的过滤器。

##**地理坐标盒模型过滤器**##
这是目前为止最有效的 地理坐标过滤器了，因为它计算起来非常简单。 你指定一个矩形的 顶部（top）, 底部（bottom）, 左边界（left）, 和 右边界（right）， 然后它只需判断坐标的经度是否在左右边界之间，纬度是否在上下边界之间。（译注：原文似乎写反了）
```
GET /attractions/restaurant/_search
{
  "query": {
    "filtered": {
      "filter": {
        "geo_bounding_box": {
          "location": { <1>
            "top_left": {
              "lat":  40.8,
              "lon": -74.0
            },
            "bottom_right": {
              "lat":  40.7,
              "lon": -73.0
            }
          }
        }
      }
    }
  }
}
```

- <1> 盒模型信息也可以用 bottom_left（左下方点）和 top_right（右上方点） 来表示。
##**优化盒模型**##
地理坐标盒模型过滤器不需要把所有坐标点都加载到内存里。 因为它要做的只是简单判断 纬度 和 经度 坐标数值是否在给定的范围内，所以它可以用倒排索引来做一个范围（range）过滤。
要使用这种优化方式，需要把 geo_point 字段用 纬度（lat）和经度（lon）方式表示并分别索引。
```

PUT /attractions
{
  "mappings": {
    "restaurant": {
      "properties": {
        "name": {
          "type": "string"
        },
        "location": {
          "type":    "geo_point",
          "lat_lon": true <1>
        }
      }
    }
  }
}
```

- <1> location.lat 和 location.lon 字段将被分别索引。它们可以被用于检索，但是不会在检索结果中返回。

然后，查询时你需要告诉 Elasticesearch 使用已索引的 lat和lon。

```
GET /attractions/restaurant/_search
{
  "query": {
    "filtered": {
      "filter": {
        "geo_bounding_box": {
          "type":    "indexed", <1>
          "location": {
            "top_left": {
              "lat":  40.8,
              "lon": -74.0
            },
            "bottom_right": {
              "lat":  40.7,
              "lon":  -73.0
            }
          }
        }
      }
    }
  }
}
```

- <1> 设置 type 参数为 indexed (默认为 memory) 来明确告诉 Elasticsearch 对这个过滤器使用倒排索引。
> 注意：

geo_point 类型可以包含多个地理坐标点，但是针对经度纬度分别索引的这种优化方式（lat_lon）只对单个坐标点的方式有效。	

##**地理距离过滤器**##

地理距离过滤器（geo_distance）以给定位置为圆心画一个圆，来找出那些位置落在其中的文档：

```
GET /attractions/restaurant/_search
{
  "query": {
    "filtered": {
      "filter": {
        "geo_distance": {
          "distance": "1km", <1>
          "location": { <2>
            "lat":  40.715,
            "lon": -73.988
          }
        }
      }
    }
  }
}
```

- <1> 找出所有与指定点距离在1公里（1km）内的 location 字段。访问 Distance Units 查看所支持的距离表示单位

- <2> 中心点可以表示为字符串，数组或者（如示例中的）对象。详见 [lat-lon-formats](https://es.xiaoleilu.com/310_Geopoints/lat-lon-formats)。

地理距离过滤器计算代价昂贵。 为了优化性能，Elasticsearch 先画一个矩形框（边长为2倍距离）来围住整个圆形， 这样就可以用消耗较少的盒模型计算方式来排除掉那些不在盒子内（自然也不在圆形内）的文档， 然后只对落在盒模型内的这部分点用地理坐标计算方式处理。

> 提示
> 你需要判断你的使用场景，是否需要如此精确的使用圆模型来做距离过滤？ 通常使用矩形模型是更高效的方式，并且往往也能满足应用需求。
##**更快的地理距离计算**##
两点间的距离计算，有多种性能换精度的算法：

 - arc::
最慢但是最精确是弧形（arc）计算方式，这种方式把世界当作是球体来处理。 不过这种方式精度还是有限，因为这个世界并不是完全的球体。
 - plane::
平面（plane）计算方式，((("plane distance calculation")))把地球当成是平坦的。 这种方式快一些但是精度略逊；在赤道附近位置精度最好，而靠近两极则变差。
 - sloppy_arc::
如此命名，是因为它使用了 Lucene 的 SloppyMath 类。 这是一种用精度换取速度的计算方式，它使用 Haversine formula 来计算距离； 它比弧形（arc）计算方式快4~5倍, 并且距离精度达99.9%。这也是默认的计算方式。

你可以参考下例来指定不同的计算方式：

```
GET /attractions/restaurant/_search
{
  "query": {
    "filtered": {
      "filter": {
        "geo_distance": {
          "distance":      "1km",
          "distance_type": "plane", <1>
          "location": {
            "lat":  40.715,
            "lon": -73.988
          }
        }
      }
    }
  }
}
```

- <1> 使用更快但精度稍差的平面（plane）计算方式。

>提示： 你的用户真的会在意一个宾馆落在指定圆形区域数米之外了吗？ 一些地理位置相关的应用会有较高的精度要求；但大部分实际应用场景中，使用精度较低但响应更快的计算方式可能就挺好。

##**地理距离区间过滤器**##

地理距离过滤器（geo_distance）和地理距离区间过滤器（geo_distance_range）的唯一差别在于后者是一个环状的，它会排除掉落在内圈中的那部分文档。

指定到中心点的距离也可以换一种表示方式： 指定一个最小距离（使用 gt或者gte）和最大距离（使用lt或者lte），就像使用区间（range）过滤器一样。

```

GET /attractions/restaurant/_search
{
  "query": {
    "filtered": {
      "filter": {
        "geo_distance_range": {
          "gte":    "1km", <1>
          "lt":     "2km", <1>
          "location": {
            "lat":  40.715,
            "lon": -73.988
          }
        }
      }
    }
  }
}
```
- 匹配那些距离中心点超过1公里而小于2公里的位置。
###**缓存地理位置过滤器**### 
因为如下两个原因，地理位置过滤器默认是不被缓存的：

- 地理位置过滤器通常是用于查找用户当前位置附近的东西。但是用户是在移动的，并且没有两个用户的位置完全相同，因此缓存的过滤器基本不会被重复使用到。
- 过滤器是被缓存为比特位集合来表示段（segment）内的文档。假如我们的查询排除了几乎所有文档，只剩一个保存在这个特别的段内。一个未缓存的地理位置过滤器只需要检查这一个文档就行了，但是一个缓存的地理位置过滤器则需要检查所有在段内的文档。


缓存对于地理位置过滤器也可以很有效。 假设你的索引里包含了所有美国的宾馆。一个在纽约的用户是不会对旧金山的宾馆感兴趣的。 所以我们可以认为纽约是一个热点（hot spot），然后画一个边框把它和附近的区域围起来。

如果这个地理盒模型过滤器（geo_bounding_box）被缓存起来，那么当有位于纽约市的用户访问时它就可以被重复使用了。 它可以直接排除国内其它区域的宾馆。然后我们使用未缓存的，更加明确的地理盒模型过滤器（geo_bounding_box）或者地理距离过滤器（geo_distance）来在剩下的结果集中把范围进一步缩小到用户附近：

```
GET /attractions/restaurant/_search
{
  "query": {
    "filtered": {
      "filter": {
        "bool": {
          "must": [
            {
              "geo_bounding_box": {
                "type": "indexed",
                "_cache": true, <1>
                "location": {
                  "top_left": {
                    "lat":  40,8,
                    "lon": -74.1
                  },
                  "bottom_right": {
                    "lat":  40.4,
                    "lon": -73.7
                  }
                }
              }
            },
            {
              "geo_distance": { <2>
                "distance": "1km",
                "location": {
                  "lat":  40.715,
                  "lon": -73.988
                }
              }
            }
          ]
        }
      }
    }
  }
}
```
- <1> 缓存的地理盒模型过滤器把结果集缩小到了纽约市。
-  <2> 代价更高的地理距离过滤器（geo_distance）让结果集缩小到1km内的用户。

###**减少内存占用**###

每一个 经纬度（lat/lon）组合需要占用16个字节的内存。要知道内存可是供不应求的。 使用这种占用16字节内存的方式可以得到非常精确的结果。不过就像之前提到的一样，实际应用中几乎都不需要这么精确。
你可以通过这种方式来减少内存使用量： 设置一个压缩的（compressed）数据字段格式并明确指定你的地理坐标点所需的精度。 即使只是将精度降低到1毫米（1mm）级别，也可以减少1/3的内存使用。 更实际的，将精度设置到3米（3m）内存占用可以减少62%，而设置到1公里（1km）则节省75%之多。

这个设置项可以通过 update-mapping API 来对实时索引进行调整：

```
POST /attractions/_mapping/restaurant
{
  "location": {
    "type": "geo_point",
    "fielddata": {
      "format":    "compressed",
      "precision": "1km" <1>
    }
  }
}
```
- <1> 每一个经纬度（lat/lon）组合现在只需要4个字节，而不是16个。

另外，你还可以这样做来避免把所有地理坐标点全部同时加载到内存中： 使用在优化盒模型（optimize-bounding-box）中提到的技术， 或者把地理坐标点当作文档值（doc values）来存储。

```

PUT /attractions
{
  "mappings": {
    "restaurant": {
      "properties": {
        "name": {
          "type": "string"
        },
        "location": {
          "type":       "geo_point",
          "doc_values": true <1>
        }
      }
    }
  }
}
```
- <1> 地理坐标点现在不会被加载到内存，而是保存在磁盘中。

将地理坐标点映射为文档值的方式只能是在这个字段第一次被创建时。 相比使用字段值，使用文档值会有一些小的性能代价，不过考虑到它对内存的节省，这种方式通常是还值得的。

###**按距离排序**###
检索结果可以按跟指定点的距离排序：
> 提示 当你可以（can）按距离排序时，按距离打分（scoring-by-distance）通常是一个更好的解决方案。

```
GET /attractions/restaurant/_search
{
  "query": {
    "filtered": {
      "filter": {
        "geo_bounding_box": {
          "type":       "indexed",
          "location": {
            "top_left": {
              "lat":  40,8,
              "lon": -74.0
            },
            "bottom_right": {
              "lat":  40.4,
              "lon": -73.0
            }
          }
        }
      }
    }
  },
  "sort": [
    {
      "_geo_distance": {
        "location": { <1>
          "lat":  40.715,
          "lon": -73.998
        },
        "order":         "asc",
        "unit":          "km", <2>
        "distance_type": "plane" <3>
      }
    }
  ]
}
```
- <1> 计算每个文档中 location 字段与指定的 lat/lon 点间的距离。
- <2> 以 公里（km）为单位，将距离设置到每个返回结果的 sort 键中。
- <3> 使用快速但精度略差的平面（plane）计算方式。

你可能想问：为什么要制定距离的单位（unit）呢？ 用于排序的话，我们并不关心比较距离的尺度是英里，公里还是光年。 原因是，这个用于排序的值会设置在每个返回结果的 sort 元素中。

```
...
  "hits": [
     {
        "_index": "attractions",
        "_type": "restaurant",
        "_id": "2",
        "_score": null,
        "_source": {
           "name": "New Malaysia",
           "location": {
              "lat": 40.715,
              "lon": -73.997
           }
        },
        "sort": [
           0.08425653647614346 <1>
        ]
     },
...
```
- <1> 宾馆距离我们的指定位置距离是 0.084km。
- 你可以通过设置单位（unit）来让返回值的形式跟你应用中想要的匹配。

> 提示
> 地理距离排序可以对多个坐标点来使用，不管（这些坐标点）是在文档中还是排序参数中。 使用 sort_mode 来指定是否需要使用位置集合的 最小（min），最大（max）或者平均（avg）距离。 这样就可以返回离我的工作地和家最近的朋友这样的结果了。

###**按距离打分**###

有可能距离只是决定返回结果排序的唯一重要因素，不过更常见的情况是距离会和其它因素， 比如全文检索匹配度，流行程度或者价格一起决定排序结果。

遇到这种场景你需要在查询分值计算（function_score query）中指定方式让我们把这些因子处理得到一个综合分。 decay-functions中有个一个例子就是地理距离影响排序得分的。

另外按距离排序还有个缺点就是性能：需要对每一个匹配到的文档都进行距离计算。 而 function_score请求，在 rescore phase阶段有可能只需要对前 n 个结果进行计算处理。









