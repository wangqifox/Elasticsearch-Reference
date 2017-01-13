# Constant Score Query(常数分值查询)

常数分值查询包裹另一个查询，返回分值等于过滤器中查询`boost`的所有文档。相对于Lucene的`ConstantScoreQuery`。

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "query": {
        "constant_score" : {
            "filter" : {
                "term" : { "user" : "kimchy"}
            },
            "boost" : 1.2
        }
    }
}'
```

过滤器子句在过滤器上下文中执行，这意味着忽略评分，并且考虑用于缓存的子句。

