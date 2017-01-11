# Match查询

`match`查询接受文本、数值、日期，经过分析，构建查询。

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "query": {
        "match" : {
            "message" : "this is a test"
        }
    }
}'
```

注意`message`是字段名，你可以使用任意字段的名称（包括`_all`）来代替。

## match

`match`查询时一个布尔查询。这意味着文本经过分析，使用提供的文本构建一个布尔查询。`operator`标志可以设置为`or`或`and`（默认是`or`）来控制布尔子句。可选的`should`子句匹配的最小数量可以使用`minimum_should_match`参数来设置。

可以设置分析器来控制文本的分析过程。默认是映射中定义的分析器，或者默认的搜索分析器。

`lenient`参数可以设置为`true`（默认为`false`）来忽略数据类型失配引起的异常，比如使用文本查询字符串来查询数值字段。

## Fuzziness

`fuzziness`允许基于查询字段类型的模糊匹配。查看`Fuzziness`章节。

设置`prefix_length`和`max_expansions`来控制模糊查询过程。如果设置了模糊选项，查询会使用`top_terms_blended_freqs_${max_expansions}`作为重写方法，`fuzzy_rewrite`参数控制查询如何重写。

模糊变换(ab->ba)默认是允许的，不过可以设置`fuzzy_transposition`为`false`来禁用。

以下是提供了额外参数的查询：

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "query": {
        "match" : {
            "message" : {
                "query" : "this is a test",
                "operator" : "and"
            }
        }
    }
}'
```

## Zero terms query

如果查询中分析器（比如`stop`过滤器）删除了所有的分词，默认匹配不到任何文档。可以使用`zero_terms_query`选项来改变这种行为，接受`none`（默认）和`all`（相当于`match_all`查询）。

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "query": {
        "match" : {
            "message" : {
                "query" : "to be or not to be",
                "operator" : "and",
                "zero_terms_query": "all"
            }
        }
    }
}'
```

## 截止频率

`match`查询支持`cutoff_frequency`，允许指定一个绝对或相对的文档频率，高频分词被移动到一个可选的子查询中，如果`or`操作中一个低频分词或者`and`操作中所有的低频分词匹配，对高频分词打分。

此查询允许在运行时动态处理`stopwords`，独立于域且不需要停用词文件。它防止评分/迭代高频分词，如果一个更有意义/低频的分词匹配文档，高频分词仅仅纳入考虑。如果所有的查询分词都高于`cutoff_frequency`，查询自动地转变成联合（and）查询以便快速执行。

如果值在[0..1)的范围内`cutoff_frequency`是相对于文档的总数；如果大于等于1.0，`cutoff_frequency`是绝对值。

以下是一个由停词组成的查询的例子：

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "query": {
        "match" : {
            "message" : {
                "query" : "to be or not to be",
                "cutoff_frequency" : 0.001
            }
        }
    }
}'
```

> `cutoff_frequency`选项在分片级别上操作。这意味着当在文档数量很少的测试索引上使用，你应该遵循`Relevance is broken`中的建议。

## Comparison to query_string / field

`match`查询系列不经过查询解析的过程。也不支持前缀、通配符、或其他高级特征的字段名。因此，查询失败的机会是很少的（几乎没有），当它作为查询行为（通常是一个文本搜索框）只用来分析和执行时提供了一个很好的行为。此外，`phrase_prefix`类型可以提供一个`as you type`行为来自动加载查询结果。
