## Has Parent查询

`has_parent`查询接收一个查询以及父类型。查询在父文档空间中执行，通过父类型来指定。该查询返回匹配的关联父文档。`has_parent`查询其他的选项以及工作方式和`has_child`查询一致。

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query": {
        "has_parent" : {
            "parent_type" : "blog",
            "query" : {
                "term" : {
                    "tag" : "something"
                }
            }
        }
    }
}
'
```

## 评分能力

`has_parent`也支持评分。默认是`false`，忽略来自父文档的评分。在这种情况下，评分等于`has_parent`查询的`boost`(默认为1)。如果评分设置为`true`，匹配的父文档的评分被聚合到子文档中。评分模式可以在`has_parent`查询中的`score`字段指定。

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query": {
        "has_parent" : {
            "parent_type" : "blog",
            "score" : true,
            "query" : {
                "term" : {
                    "tag" : "something"
                }
            }
        }
    }
}
'
```

## 忽略未映射的文档

当把`ignore_unmapped`选项设置为`true`时，则忽略未映射的类型，该查询不会匹配任何文档。当查询多个索引时是有用的，因为这些索引可能有不同的映射。当将其设置为`false`(默认值)时，如果类型没有映射的话查询会抛出异常。

## 排序

通过平常的排序选项，子文档无法根据匹配的父文档的字段来排序。如果你需要根据父文档的字段来排序子文档，你必须使用`function_score`查询，且只能根据`_score`来排序。

根据父文档的`view_count`字段来排序：

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query": {
        "has_parent" : {
            "parent_type" : "blog",
            "score" : true,
            "query" : {
                "function_score" : {
                    "script_score": {
                        "script": "_score * doc['view_count'].value"
                    }
                }
            }
        }
    }
}
'
```
