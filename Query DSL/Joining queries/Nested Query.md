# 嵌套查询（Nested Query）

嵌套查询允许查询嵌套对象/文档。该查询针对嵌套对象/文档执行，就像嵌套对象/文档在内部以单独的文档形式索引，并生成根父文档（或父嵌套映射）。以下是我们使用的示例映射：

```
curl -XPUT 'localhost:9200/my_index?pretty' -H 'Content-Type: application/json' -d'
{
    "mappings": {
        "type1" : {
            "properties" : {
                "obj1" : {
                    "type" : "nested"
                }
            }
        }
    }
}
'
```

以下是一个嵌套查询的示例：

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query": {
        "nested" : {
            "path" : "obj1",
            "score_mode" : "avg",
            "query" : {
                "bool" : {
                    "must" : [
                    { "match" : {"obj1.name" : "blue"} },
                    { "range" : {"obj1.count" : {"gt" : 5}} }
                    ]
                }
            }
        }
    }
}
'
```

查询`path`指向嵌套对象的路径，`query`包含运行在匹配直接路径的嵌套文档上，结合根父文档。注意，查询中引用的任何字段必须使用完整路径（完全限定）。

`score_mode`允许设置内部子查询匹配如何影响父查询的评分。默认为avg，但可以是sum,min,max,none

还有一个ignore_unmapped选项，当设置为true时，将忽略未映射的路径，并且将不匹配此查询的任何文档。当查询可能具有不同映射的多个索引时，这可能很有用。当设置为false(默认)时，如果路径未映射，则查询将抛出异常。

自动支持和检测多级嵌套，导致内部嵌套查询自动匹配相关的嵌套级别（而不是root），如果它存在于另一个嵌套查询中。
