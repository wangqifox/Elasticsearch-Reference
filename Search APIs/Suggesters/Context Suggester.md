# Context Suggester

完成建议器考虑索引中所有的文档，但是通常希望提供通过某些标准来过滤和/或提升建议。例如，通过某些艺术家过滤歌曲标题，或者根据其流派提升歌曲的标题。

为了做到过滤和/或提升建议，你可以在配置完成字段的时候增加上下文映射。你可以为完成字段定义多个上下文映射。每个上下文映射都有一个唯一的名字和一个类型。有两种类型：`category`和`geo`。上下文映射在字段映射的`contexts`参数下配置。

以下示例定义了类型，完成字段有两个上下文映射。

```
curl -XPUT 'localhost:9200/place?pretty' -H 'Content-Type: application/json' -d'
{
    "mappings": {
        "shops" : {
            "properties" : {
                "suggest" : {
                    "type" : "completion",
                    "contexts": [
                        {   // 1
                            "name": "place_type",
                            "type": "category",
                            "path": "cat"
                        },
                        {   // 2
                            "name": "location",
                            "type": "geo",
                            "precision": 4
                        }
                    ]
                }
            }
        }
    }
}
'
curl -XPUT 'localhost:9200/place_path_category?pretty' -H 'Content-Type: application/json' -d'
{
    "mappings": {
        "shops" : {
            "properties" : {
                "suggest" : {
                    "type" : "completion",
                    "contexts": [
                        {   // 3
                            "name": "place_type",
                            "type": "category",
                            "path": "cat"
                        },
                        {   // 4
                            "name": "location",
                            "type": "geo",
                            "precision": 4,
                            "path": "loc"
                        }
                    ]
                },
                "loc": {
                    "type": "geo_point"
                }
            }
        }
    }
}
'
```

- 1 定义了一个类型为`category`名为`place_type`的上下文，其中类别必须与建议一起发送。
- 2 定义了一个类型为`geo`名为`location`的上下文，其中类别必须与建议一起发送。
- 3 定义了一个类型为`category`名为`place_type`的上下文，其中类别从`cat`字段中读取。
- 4 定义了一个类型为`geo`名为`location`的上下文，其中类别从`cat`字段中读取。

> 添加上下文映射增加了完成字段的索引大小，完成索引完全驻留在堆中，你可以使用`Indices Stats`来监视完成字段的索引大小

## 类别上下文

`category`（类别）上下文允许你在索引阶段将一个或多个类别与建议相关联。在查询阶段，建议可以通过它们关联的类别来过滤和提升。

像上面的`place_type`字段一样设置映射。如果定义了`path`，从文档的这些路径中读取类别，否则必须在建议字段中传送类别：

```
curl -XPUT 'localhost:9200/place/shops/1?pretty' -H 'Content-Type: application/json' -d'
{
    "suggest": {
        "input": ["timmy's", "starbucks", "dunkin donuts"],
        "contexts": {
            "place_type": ["cafe", "food"]  // 1
        }
    }
}
'
```

- 1 建议与cafe和food类别关联

如果映射具有路径，则以下索引请求将足以添加类别：

```
curl -XPUT 'localhost:9200/place_path_category/shops/1?pretty' -H 'Content-Type: application/json' -d'
{
    "suggest": ["timmy\u0027s", "starbucks", "dunkin donuts"],
    "cat": ["cafe", "food"]     // 1
}
'
```

- 1 这些建议与cafe和food类别关联

> 如果上下文映射引用另一个字段，且类别被显式索引，则建议将使用这两个类别进行索引。

## 索引查询

建议可以通过一个或多个类别来过滤。以下示例通过多个类别来过滤建议：

```
curl -XPOST 'localhost:9200/place/_search?pretty&pretty' -H 'Content-Type: application/json' -d'
{
    "suggest": {
        "place_suggestion" : {
            "prefix" : "tim",
            "completion" : {
                "field" : "suggest",
                "size": 10,
                "contexts": {
                    "place_type": [ "cafe", "restaurants" ]
                }
            }
        }
    }
}
'
```

> 如果在查询阶段不提供类别，





















