# 存在查询

返回包含至少有一个非null值的原始字段的文档：

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query": {
        "exists" : { "field" : "user" }
    }
}
'
```

例如，以下的文档都会被上面的查询所匹配：

```
{ "user": "jane" }
{ "user": "" } 					// 1
{ "user": "-" } 				// 2
{ "user": ["jane"] }
{ "user": ["jane", null ] } 	// 3
```

- 1 空字符串是非null的值
- 2 即使标准分析器没有分词输出，原始字段依然是非null
- 3 只要有一个非null值，这个字段就是非null

以下文档不会匹配上面的查询:

```
{ "user": null }
{ "user": [] } 		// 1
{ "user": [null] } 	// 2
{ "foo":  "bar" } 	//3
```

- 1 该字段没有值
- 2 至少需要一个非null值
- 3 user字段完全丢失

## `null_value`映射

如果字段的映射包含了`null_value`设置，则显式的`null`值会被替换成指定的`null_value`。例如，如果`user`字段有如下的映射：

```
"user": {
    "type": "keyword",
    "null_value": "_null_"
  }
```

则显式的`null`值索引为字符串`_null_`，且下面的文档会匹配`exists`过滤：

```
{ "user": null }
{ "user": [null] }
```

但是，这些没有显式null值的文档在`user`字段仍然不会有值，因此不会匹配`exists`过滤：

```
{ "user": [] }
{ "foo": "bar" }
```

## `misssing`查询

`missing`查询已经被移除了，因为它可以在`must_not`子句中增加`exists`查询来替换：

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query": {
        "bool": {
            "must_not": {
                "exists": {
                    "field": "user"
                }
            }
        }
    }
}
'
```

该查询返回在user字段上没有值的文档
