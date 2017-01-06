# fields

为了不同的目的通常会将相同的字段用不同的方式来索引。这就是多字段(multi-fields)的目的。比如，`string`字段可以映射为`text`字段来做全文搜索，也可以映射为`keyword`来做排序或者聚集。

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "mappings": {
    "my_type": {
      "properties": {
        "city": {
          "type": "text",
          "fields": {
            "raw": { 	// 1
              "type":  "keyword"
            }
          }
        }
      }
    }
  }
}'
curl -XPUT 'localhost:9200/my_index/my_type/1?pretty' -d'
{
  "city": "New York"
}'
curl -XPUT 'localhost:9200/my_index/my_type/2?pretty' -d'
{
  "city": "York"
}'
curl -XGET 'localhost:9200/my_index/_search?pretty' -d'
{
  "query": {
    "match": {
      "city": "york" 	// 2
    }
  },
  "sort": {
    "city.raw": "asc" 	// 3
  },
  "aggs": {
    "Cities": {
      "terms": {
        "field": "city.raw" 	// 4
      }
    }
  }
}'
```

- 1 `city.raw`字段是`city`字段的`keyword`版本
- 2 `city`字段可以用于全文搜索
- 3 4 `city.raw`字段可以用于排序和聚集

> 多字段不会改变原始的`_source`字段

> 同一个索引中相同名字的字段可以有不同的`fields`。可以通过`PUT mapping API`向已存在的字段增加多字段(multi-fields)。

## 多字段使用多个分析器

多字段的另一个用处是用不同的分析器来拆分字段以取得更好地相关性。比如，我们可以用标准分析器来索引字段，将文本拆分成单词，然后再次使用英语分析器将单词转换成根形式。

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
curl -XPUT 'localhost:9200/my_index/my_type/1?pretty' -d'
{ "text": "quick brown fox" } '	// 3
curl -XPUT 'localhost:9200/my_index/my_type/2?pretty' -d'
{ "text": "quick brown foxes" } '	// 4
curl -XGET 'localhost:9200/my_index/_search?pretty' -d'
{
  "query": {
    "multi_match": {
      "query": "quick brown foxes",
      "fields": [ 	// 5
        "text",
        "text.english"
      ],
      "type": "most_fields" 	// 6
    }
  }
}'
```

- 1 `text`字段使用标准分析器
- 2 `text.english`字段使用英语分析器
- 3 4 索引两个文档，一个带`fox`另一个带`foxes`
- 5 6 查询`text`和`text.english`字段，结合两者的分值

第一个文档的`text`字段包含了`fox`，第二个文档包含了`foxes`。两个文档的`text.english`字段都包含了`fox`，因为`foxes`被转换成了`fox`。

查询`text`字段时查询字段由标准分析器来分析，查询`text.english`字段时由英语分析器来分析。经过英语分析器处理的字段允许`foxes`的查询匹配到只包含`fox`的文档。这样可以让我们匹配到尽量多的文档。通过查询未处理的`text`字段，改善了只匹配`foxes`文档的相关性分数。
