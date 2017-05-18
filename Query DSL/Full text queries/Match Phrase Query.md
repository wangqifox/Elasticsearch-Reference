# 短语匹配查询

`match_phrase`查询分析文本，然后从经过分析的文本中创建`phrase`查询。

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "query": {
        "match_phrase" : {
            "message" : "this is a test"
        }
    }
}'
```

短语查询根据可配置的`slop`按照任意顺序匹配分词。转置分词的`slop`为2。

可以设置`analyzer`来控制使用哪个分析器在文本上执行分析过程。可以在映射中定义映射，或者使用默认的搜索分析器。

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "query": {
        "match_phrase" : {
            "message" : {
                "query" : "this is a test",
                "analyzer" : "my_analyzer"
            }
        }
    }
}'
```
