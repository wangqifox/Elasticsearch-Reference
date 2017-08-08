# Parent Id 查询

> 5.0.0中增加

`parent_id`查询用于寻找属于某个父文档的子文档。

如下的映射定义：

```
curl -XPUT 'localhost:9200/my_index?pretty' -H 'Content-Type: application/json' -d'
{
    "mappings": {
        "blog_post": {
            "properties": {
                "name": {
                    "type": "keyword"
                }
            }
        },
        "blog_tag": {
            "_parent": {
                "type": "blog_post"
            },
            "_routing": {
                "required": true
            }
        }
    }
}
'
```

```
curl -XGET 'localhost:9200/my_index/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query": {
        "parent_id" : {
            "type" : "blog_tag",
            "id" : "1"
        }
    }
}
'
```

上面的查询在功能上等于下面的`has_parent`查询，但是性能好一些，因为它不需要做连接：

```
curl -XGET 'localhost:9200/my_index/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "has_parent": {
      "parent_type": "blog_post",
        "query": {
          "term": {
            "_id": "1"
        }
      }
    }
  }
}
'
```

## 参数

该查询有两个需要的参数：

|参数|说明|
|---|----|
|`type`|子类型。该类型必须有`_parent`字段|
|`id`|文档必须引用的父文档id|
|`ignore_unmapped`|当设置为`true`时，该查询忽略未映射的`type`，不会匹配任何文档。当查询多个索引时是有用的，因为这些索引可能有不同的映射。当将其设置为`false`(默认值)时，如果类型没有映射的话查询会抛出异常。|
