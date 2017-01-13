# Common Terms Query

`common`分词查询用来代替停词，它提高了搜索结果的精确度和召回（考虑到停用词），且不牺牲性能。

## 问题

查询中的每个分词都有成本。搜索"The brown fox"需要三个`term`查询，分别是对"the", "brown", "fox"的查询。所有这些查询针对索引中的所有文档执行。对"the"的查询几乎会匹配索引中所有的文档，因此它比另外两个分词在相关性上的影响小得多。

以前，解决这个问题的方法是忽略高频的分词。将"the"当成是停词，减小索引的大小且减少了需要执行的term查询数量。

这个方法的问题是，尽管停词对相关性的影响很小，但是它们仍然是很重要的。如果我们删除停词，我们会丢失精确性（比如无法分辨"happy"和"not happy"）且丢失召回（比如"The The"或"To be or not to be"在索引中就不存在了）。

## 解决方法

`common terms`查询将查询的分词划分为两组：更重要的（低频分词）和不太重要的（高频分词，比如之前的停词）。

首先，搜索匹配更重要分词的文档。这些分词出现的文档较少，且对相关性有更大的影响。

然后，再执行对不太重要分词的查询，这些分词出现的频率高，对相关性的影响较小。不同于对所有匹配文档计算相关分值，它只计算那些在之前查询中已经匹配的文档的分值。使用这种方法高频分词可以改善相关性计算的性能。

如果查询仅由高频分词构成，则单个查询被作为`AND`（连接）查询来执行，换句话说，所有的分词都是需要的。尽管单个的分词会匹配很多文档，多个分词的组合会将结果缩小到最相关的结果集。使用特定的`minimun_should_match`，单个分词被当做`OR`来执行，这种情况下应该使用足够高的值。

分词基于`cutoff_frequency`来分配给高频组和低频组，`cutoff_frequency`是一个特定的绝对频率(>=1)或一个相对频率(0.0 ... 1.0)。（记住，文档频率在分片级别上计算，就像`Relevance is broken`中介绍的）

也许这个查询最有趣的性质是，它能自动适应特定域上的停词。比如，在视频网站上，诸如"clip"和"video"之类的常用词会自动当成停词，而无需手动维护列表。

## 例子：

在这个例子中，文档频率大于0.1%（比如"this"和"is"）的单词被当成常见词。

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "query": {
        "common": {
            "body": {
                "query": "this is bonsai cool",
                    "cutoff_frequency": 0.001
            }
        }
    }
}'
```

匹配的分词数量可以通过以下的参数来控制：`minimum_should_match`(`high_freq`, `low_freq`), `low_freq_operator`(默认是`or`), `high_freq_operator`(默认是`or`)。

对于低频分词，将`low_freq_operator`设置为"and"来查询所有的分词：

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "query": {
        "common": {
            "body": {
                "query": "nelly the elephant as a cartoon",
                    "cutoff_frequency": 0.001,
                    "low_freq_operator": "and"
            }
        }
    }
}'
```

这个查询大致相当于：

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "query": {
        "bool": {
            "must": [
            { "term": { "body": "nelly"}},
            { "term": { "body": "elephant"}},
            { "term": { "body": "cartoon"}}
            ],
            "should": [
            { "term": { "body": "the"}},
            { "term": { "body": "as"}},
            { "term": { "body": "a"}}
            ]
        }
    }
}'
```

也可以使用`minimum_should_match`来指定低频分词必须出现的最小数量或百分比，比如：

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "query": {
        "common": {
            "body": {
                "query": "nelly the elephant as a cartoon",
                "cutoff_frequency": 0.001,
                "minimum_should_match": 2
            }
        }
    }
}'
```

大致上和以下的查询相等：

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "query": {
        "bool": {
            "must": {
                "bool": {
                    "should": [
                    { "term": { "body": "nelly"}},
                    { "term": { "body": "elephant"}},
                    { "term": { "body": "cartoon"}}
                    ],
                    "minimum_should_match": 2
                }
            },
            "should": [
                { "term": { "body": "the"}},
                { "term": { "body": "as"}},
                { "term": { "body": "a"}}
                ]
        }
    }
}'
```

### minimum_should_match

使用`low_freq`和`high_freq`参数可以在低频和高频分词上使用不同的`minimum_should_match`。以下是提供额外参数的例子：

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "query": {
        "common": {
            "body": {
                "query": "nelly the elephant not as a cartoon",
                    "cutoff_frequency": 0.001,
                    "minimum_should_match": {
                        "low_freq" : 2,
                        "high_freq" : 3
                    }
            }
        }
    }
}'
```

大致上和以下的查询相等：

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "query": {
        "bool": {
            "must": {
                "bool": {
                    "should": [
                    { "term": { "body": "nelly"}},
                    { "term": { "body": "elephant"}},
                    { "term": { "body": "cartoon"}}
                    ],
                    "minimum_should_match": 2
                }
            },
            "should": {
                "bool": {
                    "should": [
                    { "term": { "body": "the"}},
                    { "term": { "body": "not"}},
                    { "term": { "body": "as"}},
                    { "term": { "body": "a"}}
                    ],
                    "minimum_should_match": 3
                }
            }
        }
    }
}'
```

在这种情况下，意味着高频分词只有当匹配到三个分词时才会对相关性产生影响。`minimum_should_match`对高频分词最有趣的使用是当只有高频分词时：

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "query": {
        "common": {
            "body": {
                "query": "how not to be",
                    "cutoff_frequency": 0.001,
                    "minimum_should_match": {
                        "low_freq" : 2,
                        "high_freq" : 3
                    }
            }
        }
    }
}'
```

大致上和以下的查询相等：

```curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "query": {
        "bool": {
            "should": [
            { "term": { "body": "how"}},
            { "term": { "body": "not"}},
            { "term": { "body": "to"}},
            { "term": { "body": "be"}}
            ],
            "minimum_should_match": "3<50%"
        }
    }
}'
```

高频率生成的查询比`AND`查询的限制性稍小。

`common terms`查询也支持`boost`, `analyzer`, `disable_coord`参数。
