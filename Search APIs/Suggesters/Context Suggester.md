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

> 如果在查询阶段不提供类别，将考虑所有的索引文档。应该避免在启用类别的完成字段上执行不带类别的查询，这会降低搜索性能。

对某些类别的建议可以提升其相对于其他建议的分值。以下示例通过类别来过滤建议，提升某些类别关联的建议。

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
                    "place_type": [     // 1
                        { "context" : "cafe" },
                        { "context" : "restaurants", "boost": 2 }
                     ]
                }
            }
        }
    }
}
'
```

- 1 上下文查询过滤关联类别`cafe`和`restaurants`的建议，将关联`restaurants`的建议提升因子2。

除了接受类别值之外，上下文查询可以由多个类别上下文子句组成。类别上下文子句支持以下参数：

|参数|说明|
|---|----|
|`context`|类别值，用于过滤或提升。该参数是强制的|
|`boost`|建议的分值应该提升的因子，通过将建议的权重乘以`boost`值来计算分值。默认是1|
|`prefix`|类别值是否应该被看做前缀。比如，如果将其设置为`true`，通过指定类别前缀`type`就可以过滤`type1`,`type2`等等。默认为`false`|

## 地理位置上下文

`geo`上下文允许你在索引阶段将一个或多个地理点或`geohash`关联到建议。在查询阶段，如果建议在指定地理位置的某个距离内，可以过滤或提升建议。

在内部，地理点被编码为具有指定精度的geohash。

### 地理映射

除了`path`设置，`geo`上下文映射接受以下设置。

|参数|说明|
|---|---|
|`precision`|geohash索引精度的定义。可以指定为一个距离值（5m, 10km等）或者一个原始的geohash精度（1..12）。默认是一个原始geohash精度，值为6。|

> 索引阶段的`precision`设置了查询阶段使用的最大geohash精度。

### 索引地理上下文

`geo`上下文可以随建议一起显式设置，或者通过`path`参数从文档的地理点字段索引，类似于`category`上下文。将多个地理位置上下文与建议关联，会对每个地理位置的建议建立索引。以下示例对具有两个地理位置上下文的建议进行索引。

```
curl -XPUT 'localhost:9200/place/shops/1?pretty' -H 'Content-Type: application/json' -d'
{
    "suggest": {
        "input": "timmy\u0027s",
        "contexts": {
            "location": [
                {
                    "lat": 43.6624803,
                    "lon": -79.3863353
                },
                {
                    "lat": 43.6624718,
                    "lon": -79.3873227
                }
            ]
        }
    }
}
'
```

## 地理位置查询

建议可以根据它们与一个或多个地理点的接近程度而被过滤和提升。以下示例过滤落在区域的建议，区域由geohash编码的地理点表示。

```
curl -XPOST 'localhost:9200/place/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "suggest": {
        "place_suggestion" : {
            "prefix" : "tim",
            "completion" : {
                "field" : "suggest",
                "size": 10,
                "contexts": {
                    "location": {
                        "lat": 43.662,
                        "lon": -79.380
                    }
                }
            }
        }
    }
}
'
```

> 当查询阶段指定了较低精度的位置时，将考虑落入该区域被的所有建议。

落入区域中的建议相较于其他建议在分值上有所提升，如下所示：

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
                    "location": [   // 1
                        {
                            "lat": 43.6624803,
                            "lon": -79.3863353,
                            "precision": 2
                        },
                        {
                            "context": {
                                "lat": 43.6624803,
                                "lon": -79.3863353
                            },
                            "boost": 2
                        }
                     ]
                }
            }
        }
    }
}
'
```

- 1 上下文查询过滤落在地理位置上的建议，地理位置(43.662, -79.380)以geohash的格式，精度为2。落在精度为6的地理位置(43.6624803, -79.3863353)上的建议通过因子2提升。

除了接受上下文值，上下文查询可以由多个上下文子句组成。`category`上下文子句支持以下参数：

|参数|说明|
|---|----|
|`context`|地理点对象或者geohash字符串，用于过滤或提升建议。该参数为必须|
|`boost`|用于建议分值提升的因子，通过将建议的权重乘以该因子来计算分值，默认为1|
|`precision`|geohash编码为查询地理点的精度。可以指定为距离值(5m, 10km等)，或者原始geohash精度(1..12)。默认是索引阶段的精度级别。|
|`neighbours`|接受精度值数组，其中应该考虑相邻的geohash。精度值可以为距离值(5m, 10km等)，或者原始geohash精度(1..12)。默认生成索引阶段精度级别的邻居。|

