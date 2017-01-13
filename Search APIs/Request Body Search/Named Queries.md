## Named Queries

每个过滤和查询在顶层定义中接受一个`_name`参数。

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "query": {
        "bool" : {
            "should" : [
                {"match" : { "name.first" : {"query" : "shay", "_name" : "first"} }},
                {"match" : { "name.last" : {"query" : "banon", "_name" : "last"} }}
            ],
            "filter" : {
                "terms" : {
                    "name.last" : ["banon", "kimchy"],
                    "_name" : "test"
                }
            }
        }
    }
}'
```

查询的响应在它匹配的文档中会包含一个`matched_queries`。查询和过滤器的标记只对bool查询有意义。
