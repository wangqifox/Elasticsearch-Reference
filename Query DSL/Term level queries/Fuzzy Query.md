# 模糊查询

模糊查询使用基于Levenshtein编辑距离的相似性

## 字符串字段

`fuzzy`查询生成所有在最大编辑距离内可能匹配的分词，最大编辑距离在`fuzziness`中指定，然后检查分词字典，找出这些生成的分词中哪些在索引中存在。

这儿有个简单的例子：

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query": {
       "fuzzy" : { "user" : "ki" }
    }
}
'
```

或者是更多的高级设置：

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query": {
        "fuzzy" : {
            "user" : {
                "value" :         "ki",
                "boost" :         1.0,
                "fuzziness" :     2,
                "prefix_length" : 0,
                "max_expansions": 100
            }
        }
    }
}
'
```

参数：

- `fuzziness`: 最大的编辑距离。默认是`AUTO`。
- `prefix_length`: 不进行模糊匹配的初始字符的数量。用来减少必须检查的分词数量。默认是`0`
- `max_expansions`: `fuzzy`查询扩展的分词最大数量。默认是`50`

> 该查询会非常重，如果`prefix_length`设置为`0`，`max_expansions`设置为一个很大的数字。这会导致索引中每个分词都会被检查