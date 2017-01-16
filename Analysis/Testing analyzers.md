# 测试分析器

`analyze`API是一个非常重要的工具，用来查看分析器产生的分词。内建的分析器（或者和内建的分词器、分词过滤器、字符过滤器结合）可以在请求中内联指定。

```
curl -XPOST 'localhost:9200/_analyze?pretty' -d'
{
  "analyzer": "whitespace",
  "text":     "The quick brown fox."
}
'
curl -XPOST 'localhost:9200/_analyze?pretty' -d'
{
  "tokenizer": "standard",
  "filter":  [ "lowercase", "asciifolding" ],
  "text":      "Is this déja vu?"
}
'
```

> 位置和字符偏移
> 从`analyze`API的输出可以看出，分析器不仅仅将单词转化为分词，它们也记录每个分词的顺序或者相对位置（用来做短语查询或近似查询），以及每个分词的原始文本中的开始和结束字符偏移。

在某个索引中运行`analyze`API可以使用一个自定义的分析器。

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "settings": {
    "analysis": {
      "analyzer": {
        "std_folded": { 	// 1
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "asciifolding"
          ]
        }
      }
    }
  },
  "mappings": {
    "my_type": {
      "properties": {
        "my_text": {
          "type": "text",
          "analyzer": "std_folded" 	// 2
        }
      }
    }
  }
}
'
curl -XGET 'localhost:9200/my_index/_analyze ?pretty' -d'	// 3
{
  "analyzer": "std_folded", 	// 4
  "text":     "Is this déjà vu?"
}
'
curl -XGET 'localhost:9200/my_index/_analyze ?pretty' -d'	// 5
{
  "field": "my_text", 	// 6
  "text":  "Is this déjà vu?"
}
'
```

- 1 定义一个名为`std_folded`的分析器
- 2 字段`my_text`使用`std_folded`分析器
- 3 5 为了使用这个分析器，`analyze`API必须指定索引名。
- 4 通过名字使用分析器
- 6 通过字段`my_text`使用分析器
