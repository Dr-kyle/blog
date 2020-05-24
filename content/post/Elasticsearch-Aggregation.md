---
title: "Elasticsearch 聚合查询学习"
date: 2020-05-24T17:52:14+08:00
lastmod: 2020-05-24T17:52:14+08:00
draft: false
keywords: ["Elasticsearch", "aggretation", "es 聚合"]
description: "Elasticsearch aggretation 学习"
tags: ["Elasticsearch"]
categories: ["Elasticsearch"]
author: "Kyle Zhao"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: false
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."hugo 
contentCopyright: false
reward: true
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""


---

Elasticsearch 聚合查询学习

<!--more-->

# 聚合查询

## Bucketing

每个桶和一个文档的key相关联，符合条件的数据落入一个桶内

### Adjacency Matrix Aggregation

邻接矩阵聚合,也就是求交集，假设es中的数据，  name: kyle 有2条， name: lola 有4条， name: aiden 有1条

```json
// 如果两个过滤条件没有交集，则不会出现，例如 a 和 c 没有交集，返回的聚合结果中就没有 a&c
query:
{
  "size": 0,
  "aggs": {
    "interactions": {
      "adjacency_matrix": {
        "filters": {
          "a": {
            "terms": {
              "name.keyword": ["kyle", "lola"]
            }
          },
          "b": {
            "terms": {
              "name.keyword": ["lola", "aiden"]
            }
          },
          "c": {
            "terms": {
              "name.keyword": ["aiden"]
            }
          }
        }
      }
    }
  }
}
response:
"aggregations" : {
    "interactions" : {
      "buckets" : [
        {
          // kyle 和 lola的总共有 6条
          "key" : "a",
          "doc_count" : 6
        },
        {
          // a 和 b 相交的结果，也就是 kyle, lola 和 lola, aiden相交的结果是 lola， 有4条
          "key" : "a&b",
          "doc_count" : 4
        },
        {
          // lola 和 aiden 总共有5条
          "key" : "b",
          "doc_count" : 5
        },
        {
          // lola aiden和aiden的相交结果是 aiden， 结果有1条
          "key" : "b&c",
          "doc_count" : 1
        },
        {
          // aiden 有1条
          "key" : "c",
          "doc_count" : 1
        }
      ]
    }
  }

```



|      | a    | b    | c    |
| ---- | ---- | ---- | ---- |
| a    | a    | a&b  | a&c  |
| b    |      | b    | b&c  |
| c    |      |      | c    |

### Auto-interval Date Histogram Aggregation

```json
POST /sales/_search?size=0
{
    "aggs" : {
        "sales_over_time" : {
            "auto_date_histogram" : {
                "field" : "date",
                // default
                "buckets" : 10,
                // default is first date format
                "format" : "yyyy-MM-dd",
                // 指定时区，ES默认使用UTC
                "time_zone": "-01:00",
                // "time_zone": "America/Los_Angeles"
                // 指定最小舍入间隔
                "minimum_interval": "minute",
                // 对于缺少值得文档的对待方式，默认丢弃，也可以通过以下添加该值
                "missing": "2000-01-01"
            }
        }
    }
}
Reponse:
{
	"aggregations": {
        "sales_over_time": {
            "buckets": [
                {
                    "key_as_string": "2015-01-01",
                    "key": 1420070400000,
                    "doc_count": 3
                },
                {
                    "key_as_string": "2015-02-01",
                    "key": 1422748800000,
                    "doc_count": 2
                },
                {
                    "key_as_string": "2015-03-01",
                    "key": 1425168000000,
                    "doc_count": 2
                }
            ],
            "interval": "1M"
        }
    }
}
```

### Composite aggregation

组合查询

```js
{
    "keyword": ["foo", "bar"],
    "number": [23, 65, 76]
}
```

```js
{ "keyword": "foo", "number": 23 }
{ "keyword": "foo", "number": 65 }
{ "keyword": "foo", "number": 76 }
{ "keyword": "bar", "number": 23 }
{ "keyword": "bar", "number": 65 }
{ "keyword": "bar", "number": 76 }
"aggs": {
    "k": {
      "composite": {
        "sources": [
          {
            "k1": {
              "terms": {
                "field": "keyword.keyword"
              }
            }
          },{
            "k2": {
              "terms": {
                "field": "number"
                
              }
            }
          }
        ]
      }
    }
  }
```

### Date histogram aggregation

```console
POST /sales/_search?size=0
{
    "aggs" : {
        "sales_over_time" : {
            "date_histogram" : {
                "field" : "date",
                // minute(1m), hour(1h),day(1d),week(1w),month(1M),quarter(1q),year(1y)
                "calendar_interval" : "month"
                // milliseconds(ms), seconds(s), minutes(m), hours(h), days(d)
                // "fixed_interval" : "30d"
            }
        }
    }
}
```

### Date Range Aggregation

range aggregation this is dedicated for date values. date 字段专用 

```console
以下创建两个bucket, 第一个 10个月以前的，第二个10个月以后的
POST /sales/_search?size=0
{
    "aggs": {
        "range": {
            "date_range": {
                "field": "date",
                // "keyed": true
                "format": "MM-yyyy",
                // how documents that are missing a value should be treated. 默认忽略
                "missing": "11-1976",
                "ranges": [
                    // < now minus 10 months, rounded down to the start of the month.
                    { "to": "now-10M/M" }, 
                    // >= now minus 10 months, rounded down to the start of the month.
                    { "from": "now-10M/M" }
                    // now-10M <= range < now-5M
                    { 
                    	// 指定该bucket返回的key， 默认 和format相关， 
                    	"key":"from-to",
                    	"to": "now-5M/M",
                    	"from": "now-10M/M" }
                ]
            }
        }
    }
}
Response:
"aggregations" : {
    "range" : {
      "buckets" : [
        {
          "key" : "*-04-2020",
          "to" : 1.5856992E12,
          "to_as_string" : "04-2020",
          "doc_count" : 3
        },
        {
          "key" : "04-2020-*",
          "from" : 1.5856992E12,
          "from_as_string" : "04-2020",
          "to" : 1.5882912E12,
          "to_as_string" : "05-2020",
          "doc_count" : 3
        },
        {
          "key" : "from-to",
          "from" : 1.5882912E12,
          "from_as_string" : "05-2020",
          "doc_count" : 3
        }
      ]
    }
  }
  
  // 如果加入"keyed": true 返回的 buckets的格式则为如下
  "buckets" : {
  	"*-04-2020": 
  			{
          "to" : 1.5856992E12,
          "to_as_string" : "04-2020",
          "doc_count" : 3
        },
       "04-2020-*":
        {
          "from" : 1.5856992E12,
          "from_as_string" : "04-2020",
          "to" : 1.5882912E12,
          "to_as_string" : "05-2020",
          "doc_count" : 3
        },
        "from-to":
        {
          "from" : 1.5882912E12,
          "from_as_string" : "05-2020",
          "doc_count" : 3
        }
  }

```

### Filter Aggregation

通常用于将聚合应用于一组特定的文档。

```console
// 计算 t-shirt 的平均价格
POST /sales/_search?size=0
{
    "aggs" : {
        "t_shirts" : {
            "filter" : { "term": { "type": "t-shirt" } },
            "aggs" : {
                "avg_price" : { "avg" : { "field" : "price" } }
            }
        }
    }
}
```

### Filters Aggregation

```
PUT /kyle_test_filters/_bulk?refresh
{ "index" : { "_id" : 1 } }
{ "body" : "warning: page could not be rendered" }
{ "index" : { "_id" : 2 } }
{ "body" : "authentication error" }
{ "index" : { "_id" : 3 } }
{ "body" : "warning: connection timed out" }
{ "index" : { "_id" : 4 } }
{ "body" : "connection timed out" }

GET kyle_test_filters/_search
{
  "size": 0,
  "aggs" : {
    "messages" : {
      "filters" : {
        // 其他的值
        "other_bucket_key": "other_messages",
        "filters" : {
          "errors" :   { "match" : { "body" : "error"   }},
          "warnings" : { "match" : { "body" : "warning" }}
        }
      }
    }
  }
}
response:
"aggregations" : {
    "messages" : {
      "buckets" : {
        "errors" : {
          "doc_count" : 1
        },
        "warnings" : {
          "doc_count" : 2
        },
        "other_messages" : {
          "doc_count" : 1
        }
      }
    }
  }
GET kyle_test_filters/_search
{
  "size": 0,
  "aggs" : {
    "messages" : {
      "filters" : {
        "filters" : [
          { "match" : { "body" : "error"   }},
          { "match" : { "body" : "warning" }},
          { "match" : { "body" : "info" }}
        ]
      }
    }
  }
}
response:
"aggregations" : {
    "messages" : {
      "buckets" : [
        {
          "doc_count" : 1
        },
        {
          "doc_count" : 2
        },
        {
          "doc_count" : 0
        }
      ]
    }
  }
```

### Global Aggreation

Global aggregators can only be placed as top level aggregators

```console
// 计算 t_shirts 价格的平均值和全部产品的平均值
POST /sales/_search?size=0
{
    "query" : {
        "match" : { "type" : "t-shirt" }
    },
    "aggs" : {
        "all_products" : {
            "global" : {}, 
            "aggs" : { 
                "avg_price" : { "avg" : { "field" : "price" } }
            }
        },
        "t_shirts": { "avg" : { "field" : "price" } }
    }
}
response: 
"aggregations" : {
        "all_products" : {
            "doc_count" : 7, 
            "avg_price" : {
                "value" : 140.71428571428572 
            }
        },
        "t_shirts": {
            "value" : 128.33333333333334 
        }
    }
```

### Histogram Aggregation

```console
POST /sales/_search?size=0
{
    "aggs" : {
        "prices" : {
            "histogram" : {
                "field" : "price",
                "interval" : 50
                // "min_doc_count" : 1
           
            }
        }
    }
}

Response:
POST /sales/_search?size=0
{
    ...
    "aggregations": {
        "prices" : {
            "buckets": [
                {
                    "key": 0.0,
                    "doc_count": 1
                },
                {
                    "key": 50.0,
                    "doc_count": 1
                },
                // 如果设置了 "min_doc_count" : 1，则该条不会返回
                {
                    "key": 100.0,
                    "doc_count": 0
                },
                {
                    "key": 150.0,
                    "doc_count": 2
                },
                {
                    "key": 200.0,
                    "doc_count": 3
                }
            ]
        }
    }
}
```

### Missing Aggregation

```
该聚合器通常将与其他字段数据存储桶聚合器（例如范围）结合使用,以返回由于缺少字段数据值而无法放置在任何其他存储桶中的所有文档的信息。
POST /sales/_search?size=0
{
    "aggs" : {
        "products_without_a_price" : {
            "missing" : { "field" : "price" }
        }
    }
}

{
    ...
    "aggregations" : {
        "products_without_a_price" : {
            "doc_count" : 00
        }
    }
}
```

### Nested Aggregation

用于nested字段类型的聚合

```
PUT /products/_doc/0
{
  "name": "LED TV", 
  "resellers": [
    {
      "reseller": "companyA",
      "price": 350
    },
    {
      "reseller": "companyB",
      "price": 500
    }
  ]
}
GET /products/_search
{
    "query" : {
        "match" : { "name" : "led tv" }
    },
    "aggs" : {
        "resellers" : {
            "nested" : {
                "path" : "resellers"
            },
            "aggs" : {
                "min_price" : { "min" : { "field" : "resellers.price" } }
            }
        }
    }
}
```

### Reverse nested Aggregation

待详细研究

### Range Aggregation

参考 Date Range Aggregation，类似

### Rare Terms

就像是一个terms按_count 升序排序的聚合，实际上通过计数升序对术语agg进行排序具有无限误差

```console
GET /_search
{
    "aggs" : {
        "genres" : {
            "rare_terms" : {
                "field" : "genre",
                // 最大100， 默认1
                "max_doc_count": 2,
                "include" : ["swing", "rock"],
                "exclude" : "electro*"
            }
        }
    }
}
```

### Significant_terms Aggregation



### Sampler Aggregation



### Diversified Sampler Aggregation

先看significant_terms aggregation

## Metrics

计算一组文档的指标

### Cardinality Aggregation

计算不同值得近似计数

```console
POST /sales/_search?size=0
{
    "aggs" : {
        "type_count" : {
            "cardinality" : {
                "field" : "type"
            }
        }
    }
}
```

### Stats Aggregation

The stats that are returned consist of: `min`, `max`, `sum`, `count` and `avg`

```console
POST /exams/_search?size=0
{
    "aggs" : {
        "grades_stats" : { "stats" : { "field" : "grade", "missing": 0 } }
    }
}
POST /exams/_search?size=0
{
    "aggs" : {
        "grades_stats" : {
             "stats" : {
                 "script" : {
                     "lang": "painless",
                     "source": "doc['grade'].value"
                 }
             }
         }
    }
}
POST /exams/_search?size=0
{
    "aggs" : {
        "grades_stats" : {
            "stats" : {
                "script" : {
                    "id": "my_script",
                    "params" : {
                        "field" : "grade"
                    }
                }
            }
        }
    }
}
POST /exams/_search?size=0
{
    "aggs" : {
        "grades_stats" : {
            "stats" : {
                "field" : "grade",
                "script" : {
                    "lang": "painless",
                    "source": "_value * params.correction",
                    "params" : {
                        "correction" : 1.2
                    }
                }
            }
        }
    }
}
response:
{
    ...

    "aggregations": {
        "grades_stats": {
            "count": 2,
            "min": 50.0,
            "max": 100.0,
            "avg": 75.0,
            "sum": 150.0
        }
    }
}
```

### Extended Stats Aggregation

对stats的扩展

```console
GET /exams/_search
{
    "size": 0,
    "aggs" : {
        "grades_stats" : { "extended_stats" : { "field" : "grade" } }
    }
}
{
    ...

    "aggregations": {
        "grades_stats": {
           "count": 2,
           "min": 50.0,
           "max": 100.0,
           "avg": 75.0,
           "sum": 150.0,
           "sum_of_squares": 12500.0,
           "variance": 625.0,
           "std_deviation": 25.0,
           "std_deviation_bounds": {
            "upper": 125.0,
            "lower": 25.0
           }
        }
    }
}
```

### Median Absolute Deviation Aggregation

中位数绝对偏差

```console
GET reviews/_search
{
  "size": 0,
  "aggs": {
    "review_average": {
      "avg": {
        "field": "rating"
      }
    },
    "review_variability": {
      "median_absolute_deviation": {
        "field": "rating" 
      }
    }
  }
}
```

### Scripted Metric Aggregation

未明白含义

### String Stats Aggregation



```console
POST /twitter/_search?size=0
{
    "aggs" : {
        "message_stats" : { "string_stats" : { "field" : "message.keyword" } }
    }
}
Response:
{
    ...

    "aggregations": {
        "message_stats" : {
            // count - The number of non-empty fields counted.
            "count" : 5,
            // min_length - The length of the shortest term.
            "min_length" : 24,
            // max_length - The length of the longest term.
            "max_length" : 30,
            // avg_length - The average length computed over all terms.
            "avg_length" : 28.8,
            // 没懂什么含义。。
            "entropy" : 3.94617750050791
        }
    }
}
```

### Top Hits Aggregation

```console
POST /sales/_search?size=0
{
    "aggs": {
        "top_tags": {
            "terms": {
                "field": "type",
                "size": 3
            },
            "aggs": {
                "top_sales_hits": {
                    "top_hits": {
                        "sort": [
                            {
                                "date": {
                                    "order": "desc"
                                }
                            }
                        ],
                        "_source": {
                            "includes": [ "date", "price" ]
                        },
                        // "from": 0
                        "size" : 1
                    }
                }
            }
        }
    }
}
```

### Top Hits Aggregation

值得研究，暂未看懂

## Matrix

不支持脚本，可以在多个字段上操作，并根据请求的文档字段中提取的值生成矩阵结果。

## Pipeline

聚合其他聚合的输出结果及其相关指标