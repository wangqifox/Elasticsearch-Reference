# 类型查询

过滤匹配文档/映射类型的文档

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query": {
        "type" : {
            "value" : "my_type"
        }
    }
}
'
```
