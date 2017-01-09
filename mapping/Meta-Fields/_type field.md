# `_type`字段

每个文档在索引时都和`_type`和`_id`相关联。`_type`字段用来根据类型名快速搜索。

`_type`字段的值可以在查询、聚合、脚本、排序时访问。

```
# Example documents
curl -XPUT 'localhost:9200/my_index/type_1/1?pretty' -d'
{
  "text": "Document with type 1"
}'
curl -XPUT 'localhost:9200/my_index/type_2/2?refresh=true&pretty' -d'
{
  "text": "Document with type 2"
}'
curl -XGET 'localhost:9200/my_index/type_*/_search?pretty' -d'
{
  "query": {
    "terms": {
      "_type": [ "type_1", "type_2" ] 	// 1
    }
  },
  "aggs": {
    "types": {
      "terms": {
        "field": "_type", 	// 2
        "size": 10
      }
    }
  },
  "sort": [
    {
      "_type": { 	// 3
        "order": "desc"
      }
    }
  ],
  "script_fields": {
    "type": {
      "script": {
        "lang": "painless",
        "inline": "doc[\u0027_type\u0027]" 	// 4
      }
    }
  }
}'
```

- 1 查询`_type`字段
- 2 聚合`_type`字段
- 3 排序`_type`字段
- 4 脚本中访问`_type`字段。
