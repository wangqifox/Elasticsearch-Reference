# Term 查询

`term`查询在倒排索引中寻找包含指定精确分词的文档。比如：

```
curl -XPOST 'localhost:9200/_search?pretty' -d'
{
  "query": {
    "term" : { "user" : "Kimchy" } 	// 1
  }
}'
```

- 1 查询`user`字段的倒排索引中包含精确分词`Kimchy`的文档。

指定`boost`参数可以给这个`term`查询一个更好的相关性分值，比如：

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "status": {
              "value": "urgent",
              "boost": 2.0 	// 1
            }
          }
        },
        {
          "term": {
            "status": "normal" 	// 2
          }
        }
      ]
    }
  }
}'
```

- 1 `urgent`查询子句有一个2.0的`boost`，这意味着它比`normal`的查询子句重要两倍。
- 2 `normal`子句的`boost`是默认的1.0

## 为什么`term`查询不匹配文档

字符串字段可以是`text`类型（当成全文，就像email的正文）或者`keyword`类型（当成精确的值，就像email地址或者邮政编码）。精确值（比如数字、日期、关键字）将字段中的确切值添加到反向索引中，以便使其可搜索。

但是，文本字段是经过分析的。这意味着他们的值要经过一个分析器，产生一系列的分词，然后添加到倒排索引中。

有很多方式来分析文本：默认是标准分析器，它删除标点符号，将文本拆分成独立的单词，再将单词转变成小写。比如，标准分析器将字符串"Quick Brown Fox!"转变成[`quick`, `brown`, `fox`]。

`term`查询在字段的倒排索引中查找精确的分词，它不关心字段的分析器。它可以用来在关键字字段、数值字段、日期字段中查找值。如果需要查询全文字段，使用`match`查询，它理解字段是如何被分析的。

如下所示，首先创建索引，指定字段映射，索引文档：

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "mappings": {
    "my_type": {
      "properties": {
        "full_text": {
          "type":  "text" 	// 1
        },
        "exact_value": {
          "type":  "keyword" 	// 2
        }
      }
    }
  }
}'
curl -XPUT 'localhost:9200/my_index/my_type/1?pretty' -d'
{
  "full_text":   "Quick Foxes!", 	// 3
  "exact_value": "Quick Foxes!"  	// 4
}'
```

- 1 `full_text`字段是`text`类型，会经过分析器分析。
- 2 `exact_value`字段是`keyword`类型，不会经过分析器分析。
- 3 `full_text`倒排索引包含以下分词:[quick, foxes]
- 4 `exact_value`倒排索引包含精确的分词:[Quick Foxes!]

比较`term`查询和`match`查询的结果：

```
curl -XGET 'localhost:9200/my_index/my_type/_search?pretty' -d'
{
  "query": {
    "term": {
      "exact_value": "Quick Foxes!" 	// 1
    }
  }
}'
curl -XGET 'localhost:9200/my_index/my_type/_search?pretty' -d'
{
  "query": {
    "term": {
      "full_text": "Quick Foxes!" 	// 2
    }
  }
}'
curl -XGET 'localhost:9200/my_index/my_type/_search?pretty' -d'
{
  "query": {
    "term": {
      "full_text": "foxes" 	// 3
    }
  }
}'
curl -XGET 'localhost:9200/my_index/my_type/_search?pretty' -d'
{
  "query": {
    "match": {
      "full_text": "Quick Foxes!" 	// 4
    }
  }
}'
```

- 1 这个查询匹配，因为`exact_value`字段包含精确的分词`Quick Foxes!`
- 2 这个查询不匹配，因为`full_text`字段只包含分词`quick`和`foxes`。不包含分词`Quick Foxes!`
- 3 对于分词`foxes`的`term`查询匹配`full_text`字段
- 4 `full_text`字段的`match`查询首先分析查询字符串，然后查找包含`quick`或`foxes`或两者的文档
