# _id 字段

每个文档在索引时都和`_type`和`_id`相关联。`_id`字段不会被索引，因为它的值可以从`_uid`字段导出。

`_id`字段的值可以在某些查询中访问，比如term, terms, match, query_string, simple_query_string，但是聚合、脚本、排序查询中不能访问，这些查询中应该使用`_uid`来代替。

```
# Example documents
curl -XPUT 'localhost:9200/my_index/my_type/1?pretty' -d'
{
  "text": "Document with ID 1"
}'
curl -XPUT 'localhost:9200/my_index/my_type/2&refresh=true?pretty' -d'
{
  "text": "Document with ID 2"
}'
curl -XGET 'localhost:9200/my_index/_search?pretty' -d'
{
  "query": {
    "terms": {
      "_id": [ "1", "2" ] 	// 1
    }
  }
}'
```

- 1 查询`_id`字段（另见`ids`查询）
