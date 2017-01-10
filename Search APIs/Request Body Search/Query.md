# 查询

搜索请求体中的查询元素允许使用`Query DSL`来定义查询

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}'
```
