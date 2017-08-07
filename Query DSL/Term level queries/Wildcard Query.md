# 通配符查询

查询字段匹配通配符表达式(不经过分析)的文档。支持的通配符有`*`(匹配任意的字符序列)，`?`(匹配单个字符)。注意该查询可能比较慢，因为它需要遍历很多分词。为了放置非常慢的通配符查询，通配符尽量不要以`*`或`?`开头。通配符查询映射到Lucene的`WildcardQuery`。

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query": {
        "wildcard" : { "user" : "ki*y" }
    }
}
'
```

该查询也可以关联boost：

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query": {
        "wildcard" : { "user" : { "value" : "ki*y", "boost" : 2.0 } }
    }
}
'
```

或者

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query": {
        "wildcard" : { "user" : { "wildcard" : "ki*y", "boost" : 2.0 } }
    }
}
'
```

多个分词查询可以使用`rewrite`参数来控制如何获取重写