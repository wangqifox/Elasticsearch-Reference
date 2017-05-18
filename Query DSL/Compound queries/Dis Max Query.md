# Dis Max Query

一个查询由它的子查询来产生文档的并集，每个子查询产生的最高分数最为该文档的分数，并对任何其他匹配子查询的增加分数增量。

当以不同的boost因素在多个字段上搜索一个单词时，该查询时有用的（多个字段不能等价地结合成单个的字段）。我们想要的是将主要的分值和最高boost关联，而不是字段分数的总和（如Boolean查询）。如果查询是"albino elephant"，"albino"匹配一个字段，"elephant"匹配另一个字段，它得到的分数比"albino"要高。为了得到这个结果，同时使用`Boolean`查询和`DisjunctionMax`查询：`DisjunctionMax`查询搜索每个字段，这些`DisjunctionMax`查询集合组合成一个`Boolean`查询。

`tie breaker`能力使得多个字段中包含相同分词的结果能够比多个字段中的最佳字段包含该分词的结果更好地判断，而不会与多个字段中两个不同分词这种更好的情况相混淆。默认的`tie_breaker`是`0.0`。

该查询对应于Lucene的`DisjunctionMaxQuery`。

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "query": {
        "dis_max" : {
            "tie_breaker" : 0.7,
            "boost" : 1.2,
            "queries" : [
                {
                    "term" : { "age" : 34 }
                },
                {
                    "term" : { "age" : 35 }
                }
            ]
        }
    }
}
'
```

