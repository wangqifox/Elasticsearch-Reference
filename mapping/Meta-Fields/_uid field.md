# `_uid`字段

每个文档在索引时都和`_type`和`_id`相关联。这些值结合成`{type}#{id}`的格式，索引到`_uid`字段。

`_uid`字段的值在查询、聚合、脚本、排序中可以访问。

```
# Example documents
curl -XPUT 'localhost:9200/my_index/my_type/1?pretty' -d'
{
  "text": "Document with ID 1"
}'
curl -XPUT 'localhost:9200/my_index/my_type/2?refresh=true&pretty' -d'
{
  "text": "Document with ID 2"
}'
curl -XGET 'localhost:9200/my_index/_search?pretty' -d'
{
  "query": {
    "terms": {
      "_uid": [ "my_type#1", "my_type#2" ] 	// 1
    }
  },
  "aggs": {
    "UIDs": {
      "terms": {
        "field": "_uid", 	// 2
        "size": 10
      }
    }
  },
  "sort": [
    {
      "_uid": { 	// 3
        "order": "desc"
      }
    }
  ],
  "script_fields": {
    "UID": {
      "script": {
         "lang": "painless",
         "inline": "doc[\u0027_uid\u0027]" 	// 4
      }
    }
  }
}'
```

- 1 查询`_uid`字段
- 2 对`_uid`字段进行聚合
- 3 对`_uid`字段进行排序
- 4 脚本中访问`_uid`字段
