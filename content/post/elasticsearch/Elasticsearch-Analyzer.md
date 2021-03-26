---
title: "Elasticsearch Analyzer"
date: 2020-12-11T10:48:05+08:00
lastmod: 2020-12-11T10:48:05+08:00
draft: false
keywords: ["Elasticsearch", "Analyzer"]
description: "Elasticsearch Analyzer"
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

只有 `text` 字段支持 `analyzer` 参数.

analyzer 由三部分组成L `character filters`, `tokenizers`, `token filters`.

### Character filters

一个分析器可以有零个或多个 character filters, 按顺序执行。

预处理字符流，添加、删除或更改字符，然后在将其传递给令牌生成器。

内置的 character filters

- HTML Strip Character Filter

   `html_strip`  去除 HTML 元素

  ```
  PUT my-index-000001
  {
    "settings": {
      "analysis": {
        "analyzer": {
          "my_analyzer": {
            "tokenizer": "keyword",
            "char_filter": [
              "my_custom_html_strip_char_filter"
            ]
          }
        },
        "char_filter": {
          "my_custom_html_strip_char_filter": {
            "type": "html_strip",
            "escaped_tags": [
              "b"
            ]
          }
        }
      }
    }
  }
  ```

  

- Mapping Character Filter 

  替换出现的指定字符串

  ```
  PUT /my-index-000001
  {
    "settings": {
      "analysis": {
        "analyzer": {
          "my_analyzer": {
            "tokenizer": "standard",
            "char_filter": [
              "my_mappings_char_filter"
            ]
          }
        },
        "char_filter": {
          "my_mappings_char_filter": {
            "type": "mapping",
            "mappings": [
              ":) => _happy_",
              ":( => _sad_"
            ]
          }
        }
      }
    }
  }
  ```

  

- Pattern Replace Character Filter

  根据正则表达式进行替换

  ```
  PUT my-index-00001
  {
    "settings": {
      "analysis": {
        "analyzer": {
          "my_analyzer": {
            "tokenizer": "standard",
            "char_filter": [
              "my_char_filter"
            ]
          }
        },
        "char_filter": {
          "my_char_filter": {
            "type": "pattern_replace",
            "pattern": "(\\d+)-(?=\\d)",
            "replacement": "$1_"
          }
        }
      }
    }
  }
  ```

  

### Tokenizer

分词器还负责记录每个术语的顺序或位置，以及开始和结束字符偏移量。

分析器只能有一个 tokenizer。

### Token filters

过滤器不允许更改每个令牌的位置或字符偏移量。

分析器可能具有零个或多个 token filter，按顺序执行。

- `lowercase` 将所有token转换为小写字母。

- `stop` 从 所有token中删除常用词。

- `synonym` 引入同义词。

#### index and search analysis

test analysis 在两种情况会发生。

- 数据写入时，`text` 字段会被分析
- 数据查询时，默认和数据写入时使用相同的分析器，除非指定。

#### Stemming

词干，将单词还原成词根形式的过程，可以确保在搜索过程中单词匹配。

例如: walking 和 walked 还原成词根 walk。词干依赖于语言，通常涉及从单词中删除前缀和后缀。

##### Stemmer token filter

- algorithmic stemmers 基于一组规则来生成词干

  优点：

  - 需要很少的设置，通常开箱即用
  - 使用很少的内存
  - 比 dictionary 分析器快

- dictionary stemmers 在字典中查找

  非常适合

  - 阻止不规则单词
  - 辨别拼写相似但概念上不相关的单词，例如： 'organ'和'organization', 'broker'和 'broken'

  缺点：

  - 词典质量, 需要定期更新
  - 大小和性能，必须将词典中的所有单词，前缀和后缀加载到内存中，会占用大量RAM。
  - 低质量词典在删除前缀和后缀时可能效率较低，会大大减慢词干的处理速度。

  algorithmic优于 dictionary stemmers

- Control stemming

  有时拼写相近但没有关系的词可以产生相同的词干，例如： skies 和 skiing 产生相同词干 ski。

  为了防止这种情况并更好控制，可以使用 以下token filter:

  - stemmer_override 阻止特定令牌的规则
  - keyword_marker 将制定的标记标记为关键字，关键字令牌不会被后续的词干令牌过滤器阻止。
  - conditional 类似于 keyword_marker

  对于内置的 language analyzers, 可以使用 stem_exclusion 参数定义不会被词干的列表。

#### Token graphs

empty

### Test an analyzer

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
             "my_custom_analyzer": {
              "type": "custom", 
              "tokenizer": "standard",
              "char_filter": [
                "html_strip"
              ],
              "filter": [
                "lowercase",
                "asciifolding"
              ]
            },
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
            },
            "semicolon_analyzer": {
              "type": "custom",
              "tokenizer": "semicolon_tokenizer"
            }
         },
         "tokenizer": {
          "semicolon_tokenizer": 
            {
              "type": "char_group",
              "tokenize_on_chars": [
                ";"
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

GET /_analyze
{
  "tokenizer": "keyword",
  "char_filter": [
    "html_strip"
  ],
  "text": "<p>I&apos;m so <b>happy</b>!</p>"
}

POST my-index-000001/_analyze
{
  "analyzer": "my_custom_analyzer",
  "text": "Is this <b>déjà vu</b>?"
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



### Normalizers

规范化器, normalizer 是keyword的一个属性，对keyword生成的单一`term`在做进一步的处理，例如大小写转换，字符转换.

Elasticsearch 内置 `lowercase` 规范器，其他的需要自定义使用，`arabic_normalization`, `asciifolding`, `bengali_normalization`, `cjk_width`, `decimal_digit`, `elision`, `german_normalization`, `hindi_normalization`, `indic_normalization`, `lowercase`, `persian_normalization`, `scandinavian_folding`, `serbian_normalization`, `sorani_normalization`, `uppercase`。 

```
PUT my-index-000001
{
  "settings": {
    "analysis": {
      "char_filter": {
        "quote": {
          "type": "mapping",
          "mappings": [
            "« => \"",
            "» => \""
          ]
        }
      },
      "normalizer": {
        "my_normalizer": {
          "type": "custom",
          "char_filter": ["quote"],
          "filter": ["lowercase", "asciifolding"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "foo": {
        "type": "keyword",
        "normalizer": "my_normalizer"
      }
    }
  }
}

// keyword 忽略大小写
PUT kyle-keyword
{
  "settings": {
    "analysis": {
      "analyzer": {
        "keyword_analyzer": {
          "type": "custom",
          "tokenizer": "keyword",
          "filter": [
            "lowercase"
          ]
        }
      },
      "normalizer": {
        "my_normalizer": {
          "type": "custom",
          "filter": [
            "lowercase"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "groups": {
        "type": "text",
        "analyzer": "keyword_analyzer"
      },
      "email": {
        "type": "keyword",
        "normalizer": "my_normalizer"
      }
    }
  }
}


POST my-index-000001/_doc
{
  "foo": "«Abc»"
}

// 可以查询出来结果
GET my-index-000001/_search
{
  "query": {
    "query_string": {
      "query": "«abc»"
    }
  }
}
// 可以查询出来结果
GET my-index-000001/_search
{
  "query": {
    "term": {
      "foo": {
        "value": "\"abc»"
      }
    }
  }
}
// 可以查询出来结果
GET my-index-000001/_search
{
  "query": {
    "term": {
      "foo": {
        "value": "\"abc»"
      }
    }
  }
}

// 可以查询出来结果
GET my-index-000001/_search
{
  "query": {
    "term": {
      "foo": {
        "value": "\"Abc\""
      }
    }
  }
}




```

