---
title: "Analyzer"
date: 2020-12-11T10:48:05+08:00
lastmod: 2020-12-11T10:48:05+08:00
draft: true
keywords: []
description: "Elasticsearch Analyzer"
tags: []
categories: []
author: ""

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: false
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
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

Elasticsearch Analyzer

<!--more-->

# [Analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analyzer.html)

只有 `text` 字段支持 `analyzer` 参数

建议在生产环境使用前，进行分词测试，See [Test an analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/test-analyzer.html)

1. An `analyzer` setting for indexing all terms including stop words
2. A `search_analyzer` setting for non-phrase queries that will remove stop words
3. A `search_quote_analyzer` setting for phrase queries that will not remove stop words

```
PUT my-index-000001
{
   "settings":{
      "analysis":{
         "analyzer":{
            "my_analyzer":{ 
               "type":"custom",
               "tokenizer":"standard",
               "filter":[
                  "lowercase"
               ]
            },
            "my_stop_analyzer":{ 
               "type":"custom",
               "tokenizer":"standard",
               "filter":[
                  "lowercase",
                  "english_stop"
               ]
            }
         },
         "filter":{
            "english_stop":{
               "type":"stop",
               "stopwords":"_english_"
            }
         }
      }
   },
   "mappings":{
       "properties":{
          "title": {
             "type":"text",
             // setting that points to the my_analyzer analyzer which will be used at index time
             "analyzer":"my_analyzer", 
             // setting that points to the my_stop_analyzer and removes stop words for non-phrase queries
             "search_analyzer":"my_stop_analyzer", 
             // setting that points to the my_analyzer analyzer and ensures that stop words are not removed from phrase queries
             "search_quote_analyzer":"my_analyzer" 
         }
      }
   }
}

PUT my-index-000001/_doc/1
{
   "title":"The Quick Brown Fox"
}

PUT my-index-000001/_doc/2
{
   "title":"A Quick Brown Fox"
}

GET my-index-000001/_search
{
   "query":{
      "query_string":{
         "query":"\"the quick brown fox\"" 
      }
   }
}

因为 "query":"\"the quick brown fox\"" 使用引号括起来的，所以是短语查询，所以 search_quote_analyzer 启用确保不会从query移除停止词。
my_analyzer analyzer 会返回词组 [the, quick, brown, fox] ，将匹配其中的一个文档。
同时，term queries 将会使用 my_stop_analyzer 分析器进行分析，分析器会过滤掉停止词，因此一个搜索例如：
The quick brown fox 或 A quick brown fox 将返回两个文档， 因为这两个文档包含 tokens [quick, brown, fox]。 
如果没有search_quote_analyzer，则无法对短语查询进行精确匹配，因为短语查询中的停止词将被删除，从而导致两个文档都匹配。
```



