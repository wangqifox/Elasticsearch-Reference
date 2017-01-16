# 配置内建的分析器

内建的分析器可以不经配置而直接使用。不过一些分析器支持配置选项来改变行为。例如，标准分析器可以通过配置来支持一系列停词：

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "settings": {
    "analysis": {
      "analyzer": {
        "std_english": { 	// 1
          "type":      "standard",
          "stopwords": "_english_"
        }
      }
    }
  },
  "mappings": {
    "my_type": {
      "properties": {
        "my_text": {
          "type":     "text",
          "analyzer": "standard", 	// 2
          "fields": {
            "english": {
              "type":     "text",
              "analyzer": "std_english" 	// 3
            }
          }
        }
      }
    }
  }
}
'
curl -XPOST 'localhost:9200/my_index/_analyze?pretty' -d'
{
  "field": "my_text", 	// 4
  "text": "The old brown cow"
}
'
curl -XPOST 'localhost:9200/my_index/_analyze?pretty' -d'
{
  "field": "my_text.english", 	// 5
  "text": "The old brown cow"
}
'
```

- 1 我们根据标准分析器来定义`std_english`，通过配置删除预定义的一系列英文停词
- 2 4 `my_text`字段直接使用标准分析器，没有任何配置。该字段不会删除停词。分词结果是：`[ the, old, brown, cow ]`
- 3 5 `my_text.english`字段使用`std_english`分析器，所以英文停词会被删除。分词结果是：`[ the, old, brown, cow ]`
