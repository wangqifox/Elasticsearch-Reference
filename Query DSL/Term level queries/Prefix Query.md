# 前缀查询

查询字段以特定前缀(不经过分析)开头的文档。前缀查询映射到Lucene的`PrefixQuery`。下面的查询匹配包含以`ki`开头的user字段的文档：

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{ "query": {
    "prefix" : { "user" : "ki" }
  }
}
'
```

查询可以关联boost：

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{ "query": {
    "prefix" : { "user" :  { "value" : "ki", "boost" : 2.0 } }
  }
}
'
```

或者关联`prefix`语法(在5.0.0版本中已废弃)：

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{ "query": {
    "prefix" : { "user" :  { "prefix" : "ki", "boost" : 2.0 } }
  }
}
'
```

多个分词查询可以使用`rewrite`参数来控制如何获取重写