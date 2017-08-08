# Has Child Query

`has_child`过滤器接受一个请求以及一个子类型，结果是包含匹配子文档的父文档。比如：

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query": {
        "has_child" : {
            "type" : "blog_tag",
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

`has_child`也支持评分。支持的评分模式有`min`,`max`,`sum`,`avg`,`none`。默认是`none`，行为与之前的版本相同。如果评分模式被设置成非`none`的值，所有匹配的子文档的分数都被聚合到相关联的父文档。评分类型可以通过`has_child`查询中的`score_mode`字段来指定。

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query": {
        "has_child" : {
            "type" : "blog_tag",
            "score_mode" : "min",
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

## Min/Max Children

`has_child`查询允许你指定父文档中匹配的最少(和/或)最多的子文档的数量:

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query": {
        "has_child" : {
            "type" : "blog_tag",
            "score_mode" : "min",
            "min_children": 2, 				// 1
            "max_children": 10, 			// 2
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

- 1 2 `min_children`和`max_children`都是可选的

`min_children`和`max_children`参数可以和`score_mode`参数相结合。

## 忽略未映射的文档

当把`ignore_unmapped`选项设置为`true`时，则忽略未映射的类型，该查询不会匹配任何文档。当查询多个索引时是有用的，因为这些索引可能有不同的映射。当将其设置为`false`(默认值)时，如果类型没有映射的话查询会抛出异常。

## 排序

通过平常的排序选线，父文档无法根据匹配的子文档的字段来排序。如果你需要根据子文档的字段来排序父文档，你必须使用`function_score`查询，只能根据`_score`来排序。

根据子文档的`click_count`字段来排序：

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query": {
        "has_child" : {
            "type" : "blog_tag",
            "score_mode" : "max",
            "query" : {
                "function_score" : {
                    "script_score": {
                        "script": "_score * doc['click_count'].value"
                    }
                }
            }
        }
    }
}
'
```
