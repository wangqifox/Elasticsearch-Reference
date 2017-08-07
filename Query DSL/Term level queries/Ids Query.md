# Ids查询

过滤符合所提供的ids的文档。注意，该查询使用`_uid`字段

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query": {
        "ids" : {
            "type" : "my_type",
            "values" : ["1", "4", "100"]
        }
    }
}
'
```

`type`是可选的，可以被忽略，也可以接受一个数组。如果不指定type，则尝试索引映射中定义的所有type。