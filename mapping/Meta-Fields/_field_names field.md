# _field_names 字段

`_field_names`字段索引包含任何不为`null`的文档中每个字段的名字，该字段在`exists`查询中被用来寻找某个字段包含非null值的文档。

`_field_names`字段的值可以在查询中按如下形式来访问:

```
# Example documents
curl -XPUT 'localhost:9200/my_index/my_type/1?pretty' -H 'Content-Type: application/json' -d'
{
  "title": "This is a document"
}
'
curl -XPUT 'localhost:9200/my_index/my_type/2?refresh=true&pretty' -H 'Content-Type: application/json' -d'
{
  "title": "This is another document",
  "body": "This document has a body"
}
'
curl -XGET 'localhost:9200/my_index/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "terms": {
      "_field_names": [ "title" ] 		// 1
    }
  }
}
'
```

- 1 在`_field_names`字段上查询