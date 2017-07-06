---
layout: post
title:  "2017-07-04-Elasticsearch常用统计命令"
date:   2017-07-04 23:44:01 +0800
categories: elasticsearch
tag: elasticsearch
sid: 1499181834
---

> 列举Elasticsearch5.4.3中常用到的一些统计命令

- GET _all  获取所有索引的相关信息
- GET _alias 获取所有索引的别名信息
- GET _aliases 获取所有索引的别名信息
- GET _analyze {"text": "中华人民共和国"} 获取每个字符串的分词结果
- GET _cluster/health 获取集群的健康状态
-