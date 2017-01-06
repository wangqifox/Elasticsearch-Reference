# search_analyzer

通常，索引阶段和查询阶段应该使用相同的分析器，以保证查询时的分词和倒排索引的分词格式上一致。

尽管如此，有时候在查询时使用不同的分析器是有意义的。比如在使用`edge_ngram`分词器来做自动完成功能时。

默认情况下，使用字段映射时定义的分析器来做查询，不过也可以使用`search_analyzer`来自定义查询时的分析器。

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "settings": {
    "analysis": {
      "filter": {
        "autocomplete_filter": {
          "type": "edge_ngram",
          "min_gram": 1,
          "max_gram": 20
        }
      },
      "analyzer": {
        "autocomplete": { 	// 1
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "autocomplete_filter"
          ]
        }
      }
    }
  },
  "mappings": {
    "my_type": {
      "properties": {
        "text": {
          "type": "text",
          "analyzer": "autocomplete", 	// 2
          "search_analyzer": "standard" 	// 3
        }
      }
    }
  }
}'
curl -XPUT 'localhost:9200/my_index/my_type/1?pretty' -d'
{
  "text": "Quick Brown Fox" 	// 4
}'
curl -XGET 'localhost:9200/my_index/_search?pretty' -d'
{
  "query": {
    "match": {
      "text": {
        "query": "Quick Br", 	// 5
        "operator": "and"
      }
    }
  }
}'
```

- 1 `analysis`设置定义了自定义的`autocomplete`分析器。
- 2 3 `text`字段在索引阶段使用`autocomplete`分析器，但是在索引阶段使用`standard`分析器
- 4 该字段被索引成以下的分词：[q, qu, qui, quic, quick, b, br, bro, brow, brown, f, fo, fox]
- 5 该查询搜索以下的分词：[quick, br]


