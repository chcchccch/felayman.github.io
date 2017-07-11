---
layout: post
title:  "Elasticsearch创建索引的一些常用设置"
date:   2017-05-12 23:51:01 +0800
categories: elasticsearch
tag: elasticsearch 翻译
sid: 1495172680
---

常见的设置如下：
```
"settings": {
    "analysis": {},
    "cache.filter.expire": "2h",
    "cache.filter.max_size": "2gb",
    "gateway.snapshot_interval": "10s",
    "blocks.metadata": true,
    "blocks.read": true,
    "blocks.read_only": false,
    "index.cache.query.enable": true,
    "translog.disable_flush": true,
    "translog.flush_threshold_ops": 5000,
    "translog.flush_threshold_period": "30m",
    "translog.flush_threshold_size": "200mb",
    "number_of_shards": 5,
    "number_of_replicas": 1,
    "recovery.initial_shards": "quorum",
    "term_index_interval": 32,
    "refresh_interval": "1s",
    "ttl.disable_purge": true,
```

 - analysis
 - cache.filter.expire   过滤器的过期时间
 - cache.filter.max_size  过滤器的最大数量
 - gateway.snapshot_interval
 - blocks.metadata
 - blocks.read
 - blocks.read_only
 - index.cache.query.enable
 - translog.disable_flush
 - translog.flush_threshold_ops
 - translog.flush_threshold_period
 - translog.flush_threshold_size
 - number_of_shards  主分片数量
 - number_of_replicas 复制分片数量
 - recovery.initial_shards
 - term_index_interval
 - refresh_interval
 - ttl.disable_purge



其中最需要注意的的是analysis的设置,设置类似如下：

```
"char_filter": {
            "amp_and": {
              "type": "mapping",
              "mappings": [
                "&=> and "
              ]
            },
            "punctuation": {
              "type": "mapping",
              "mappings": [
                ".=> "
              ]
            }
          },
          "filter": {
            "preserved_asciifolding": {
              "type": "asciifolding",
              "preserve_original": "true"
            },
            "large_prefixer": {
              "max_gram": "100",
              "min_gram": "1",
              "type": "edgeNGram",
              "side": "front"
            },
            "prefixer": {
              "max_gram": "8",
              "type": "edgeNGram",
              "min_gram": "2",
              "side": "front"
            },
            "german_stemmer": {
              "type": "stemmer",
              "language": "light_german"
            },
            "german_stop": {
              "type": "stop",
              "stopwords": "_german_"
            },
            "fivegrammer": {
              "min_gram": "5",
              "type": "nGram",
              "max_gram": "5"
            },
            "synonyms": {
              "type": "synonym",
              "synonyms_path": "analysis/wn_s.pl",
              "format": "wordnet"
            },
            "trigrammer": {
              "type": "nGram",
              "min_gram": "3",
              "max_gram": "3"
            },
            "custom_stems": {
              "type": "stemmer_override",
              "rules_path": "analysis/custom_stems.txt"
            }
          },
          "analyzer": {
            "exact_stemmed_synonyms": {
              "type": "custom",
              "char_filter": [
                "amp_and"
              ],
              "filter": [
                "asciifolding",
                "lowercase",
                "trim",
                "custom_stems",
                "kstem",
                "synonyms",
                "custom_stems",
                "stop"
              ],
              "tokenizer": "keyword"
            },
            "stemmed": {
              "filter": [
                "standard",
                "lowercase",
                "custom_stems",
                "stop",
                "kstem"
              ],
              "tokenizer": "standard"
            },
            "exact_stemmed_synonyms_search": {
              "type": "custom",
              "char_filter": [
                "amp_and"
              ],
              "filter": [
                "standard",
                "asciifolding",
                "lowercase",
                "trim",
                "custom_stems",
                "stop",
                "kstem"
              ],
              "tokenizer": "standard"
            },
            "synonyms": {
              "type": "custom",
              "char_filter": [
                "amp_and"
              ],
              "filter": [
                "standard",
                "lowercase",
                "synonyms"
              ],
              "tokenizer": "standard"
            },
            "partial": {
              "filter": [
                "preserved_asciifolding",
                "large_prefixer"
              ],
              "tokenizer": "lowercase"
            },
            "prefix_search": {
              "tokenizer": "lowercase"
            },
            "stemmed_synonyms": {
              "type": "custom",
              "char_filter": [
                "amp_and"
              ],
              "filter": [
                "standard",
                "asciifolding",
                "lowercase",
                "trim",
                "custom_stems",
                "kstem",
                "synonyms",
                "custom_stems",
                "stop"
              ],
              "tokenizer": "standard"
            },
            "fivegram_ascii": {
              "filter": [
                "standard",
                "asciifolding",
                "lowercase",
                "trim",
                "fivegrammer"
              ],
              "tokenizer": "standard"
            },
            "prefix": {
              "filter": [
                "preserved_asciifolding",
                "prefixer"
              ],
              "tokenizer": "lowercase"
            },
            "exact": {
              "type": "custom",
              "char_filter": [
                "amp_and"
              ],
              "filter": [
                "asciifolding",
                "lowercase",
                "trim"
              ],
              "tokenizer": "keyword"
            },
            "stemmed_synonyms_search": {
              "type": "custom",
              "char_filter": [
                "amp_and"
              ],
              "filter": [
                "standard",
                "asciifolding",
                "lowercase",
                "trim",
                "custom_stems",
                "stop",
                "kstem"
              ],
              "tokenizer": "standard"
            },
            "trigram": {
              "filter": [
                "lowercase",
                "trim",
                "trigrammer"
              ],
              "tokenizer": "keyword"
            },
            "stemmed_de": {
              "filter": [
                "standard",
                "asciifolding",
                "lowercase",
                "german_stop",
                "german_normalization",
                "german_stemmer"
              ],
              "tokenizer": "standard"
            },
            "partial_search": {
              "tokenizer": "lowercase"
            }
          }
```

可以设置字符过滤器(char_filter),过滤器(filter),分析器(analyzer)
