# Match Phrase Prefix Query

`match_phrase_prefix`和`match_phrase`一样，除了文本中最后一个分词允许前缀匹配。

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "query": {
        "match_phrase_prefix" : {
            "message" : "quick brown f"
        }
    }
}'
```

它接受和短语类型一样的参数。另外，它也接受`max_expansions`参数（默认是50），控制最后的分词可以扩展到多少个前缀。强烈建议设置一个可接受的值来控制查询的执行时间。

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "query": {
        "match_phrase_prefix" : {
            "message" : {
                "query" : "quick brown f",
                "max_expansions" : 10
            }
        }
    }
}'
```

> `match_phrase_prefix`是一个简易的自动完成。它非常容易使用，可以让你快速开始，结果通常是不错的，不多有时会让人迷惑。
> 比如查询字符串为`quick brown f`。该查询使用分词`quick`和`brown`来创建短语查询（`quick`必须存在且后面必须跟着`brown`）。然后查看排序的分词字典来找到以`f`开头的前50个分词，并将这些分词添加到短语查询。
> 问题时前50个分词可能不包括`fox`，所以短语`quick brown fox`可能找不到。通常这不是个问题，用户会接着输入更多的字符，直到他们要找的单词出现
> 如果想要更好的解决方案，查看`completion suggester`和`Index-Time Search-as-You-Type`

