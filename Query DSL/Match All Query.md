# 匹配所有的查询

最简单的查询就是匹配所有的文档，所有结果的分值都是1.0

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "query": {
        "match_all": {}
    }
}'
```

`_score`可以通过`boost`参数来改变

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "query": {
        "match_all": { "boost" : 1.2 }
    }
}'
```

## 不匹配查询

这是`match_all`查询的反面，不匹配任何文档：

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "query": {
        "match_none": {}
    }
}'
```
