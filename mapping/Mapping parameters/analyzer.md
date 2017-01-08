# 分析器

字符串字段的值通过分析器将字符串转换为分词流。比如，字符串"The quick Brown Foxes"根据分析器被拆分为分词：`quick`,`brown`,`fox`。这些是字段索引真正的数据项，这样就可以高效地查询大文本中单独的单词。

分析过程不仅仅需要在索引阶段发生，也需要发生在查询阶段：查询字段需要经过相同（或相似）的分析器处理，这样尝试搜索的分词格式就和索引中已经存在的分词格式相同。

Elasticsearch附带了许多预制的分析器，可以在不进一步配置的情况下使用。它也附带了许多字符过滤器、分词器、分词过滤器，结合起来可以给每个索引配置自定义的分析器。

每个查询、每个字段、每个索引都可以指定分析器。在索引阶段，Elasticsearch会按照下面的顺序来寻找分析器：

- 字段映射中定义的分析器
- 索引设置中名为`default`的分析器
- 标准分析器

查询阶段有更多层：

- 全文查询中定义的`analyzer`
- 字段映射中定义的`search_analyzer`
- 字段映射中定义的`analyzer`
- 索引设置中定义的`default_search`
- 索引设置中定义的`default`
- 标准分析器

某个字段中指定分析器最简单的方式是在字段映射中定义。

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "mappings": {
    "my_type": {
      "properties": {
        "text": { 	// 1
          "type": "text",
          "fields": {
            "english": { 	// 2
              "type":     "text",
              "analyzer": "english"
            }
          }
        }
      }
    }
  }
}'
curl -XGET 'localhost:9200/my_index/_analyze ?pretty' -d'	// 3
{
  "field": "text",
  "text": "The quick Brown Foxes."
}'
curl -XGET 'localhost:9200/my_index/_analyze ?pretty' -d'	// 4
{
  "field": "text.english",
  "text": "The quick Brown Foxes."
}'
```

- 1 `text`字段使用默认的标准分析器
- 2 `text.english`多字段使用`english`分析器，它去除停词并且生成词干
- 3 返回分词：[the, quick, brown, foxes]
- 4 返回分词：[quick, brown, fox]

## `search_quote_analyzer`

`search_quote_analyzer`允许你为短语指定一个分析器，当处理禁用短语查询的停用词时，这特别有用。

要禁用短语的停用词，字段使用三种分析器设置：

1. `analyzer`设置用于索引包含停词的所有分词
2. `search_analyzer`设置用于删除停词的非短语查询
3. `search_quote_analyzer`设置用于不删除停词的短语查询

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
   "settings":{
      "analysis":{
         "analyzer":{
            "my_analyzer":{ 	// 1
               "type":"custom",
               "tokenizer":"standard",
               "filter":[
                  "lowercase"
               ]
            },
            "my_stop_analyzer":{ 	// 2
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
      "my_type":{
         "properties":{
            "title": {
               "type":"text",
               "analyzer":"my_analyzer", 	// 3
               "search_analyzer":"my_stop_analyzer", 	// 4
               "search_quote_analyzer":"my_analyzer" 	// 5
            }
         }
      }
   }
}'

PUT my_index/my_type/1
{
   "title":"The Quick Brown Fox"
}

PUT my_index/my_type/2
{
   "title":"A Quick Brown Fox"
}

GET my_index/my_type/_search
{
   "query":{
      "query_string":{
         "query":"\"the quick brown fox\"" 	// 1
      }
   }
}
```

- 1 `my_analyzer`分析器使用包含停词的所有分词
- 2 `my_stop_analyzer`分析器删除停词
- 3 `analyzer`设置指向`my_analyzer`分析器，在索引时使用
- 4 `search_analyzer`设置指向`my_stop_analyzer`，在非短语查询时删除停词
- 5 `search_quote_analyzer`设置指向`my_analyzer`分析器，确保短语查询时不删除停词
- 1 因为查询字符串包着引号，经检测作为短语查询。因此`search_quote_analyzer`生效，确保停词不从查询中删除。`my_analyzer`分析器返回以下的分词[`the`, `quick`, `brown`, `fox`]，这匹配其中一个文档。同时`term query`使用`my_stop_analyzer`分析器，分析器会过滤掉停词。因此不管是`The quick brown fox`还是`A quick brown fox`都会返回两个文档，因为两个文档都包含以下分词[`quick`, `brown`, `fox`]。没有`search_quote_analyzer`，短语查询是不可能做到精确匹配的，因为短语查询的停词会被删除，导致匹配到两个文档。

