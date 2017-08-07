# _uid 字段

每个经过索引的文档都和`_type`和`_id`关联。这些值结合成`{type}#{id}`的形式，并且作为`_uid`字段来索引

`_uid`字段的值可以再查询、聚合、脚本、排序中访问：

```
# Example documents
curl -XPUT 'localhost:9200/my_index/my_type/1?pretty' -H 'Content-Type: application/json' -d'
{
  "text": "Document with ID 1"
}
'
curl -XPUT 'localhost:9200/my_index/my_type/2?refresh=true&pretty' -H 'Content-Type: application/json' -d'
{
  "text": "Document with ID 2"
}
'
curl -XGET 'localhost:9200/my_index/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "terms": {
      "_uid": [ "my_type#1", "my_type#2" ] 	// 1
    }
  },
  "aggs": {
    "UIDs": {
      "terms": {
        "field": "_uid", 					// 2
        "size": 10
      }
    }
  },
  "sort": [
    {
      "_uid": { 							// 3
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
}
'
```

- 1 在`_uid`字段上查询
- 2 在`_uid`字段上聚合
- 3 在`_uid`字段上排序
- 4 在`_uid`字段上执行脚本
