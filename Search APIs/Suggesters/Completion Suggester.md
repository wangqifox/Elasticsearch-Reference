# Completion Suggester

`completion`建议器提供了自动完成/search-as-you-type功能。这是一项导航性的功能，将用户输入的结果引导到相关的结果，提升搜索精度。它不用于拼写校正或did-you-mean功能，像`term`或`phrase`建议器。

理想情况下，自动完成功能应该和用户的输入一样快，对用户的输入立即提供反馈。因此，`completion`建议器优化了速度。建议器使用允许快速查询的数据结构，但是构建成本高，且存储在内存中。

## 映射

为了使用该功能，需要为字段指定一个特殊的映射，将字段的值以快速完成的形式索引。

```
curl -XPUT 'localhost:9200/music?pretty' -H 'Content-Type: application/json' -d'
{
    "mappings": {
        "song" : {
            "properties" : {
                "suggest" : {
                    "type" : "completion"
                },
                "title" : {
                    "type": "keyword"
                }
            }
        }
    }
}
'
```

映射支持以下参数：

- `analyzer`：索引分析器，默认是`simple`。你可能会疑惑我们为什么不选择`standard`分析器。这里我们尝试简单地理解行为，如果我们索引字段内容`At the Drive-in`，对于`a`和`d`无法获得任何建议。
- `search_analyzer`：搜索分析器，默认是`analyzer`的值。
- `preserve_separators`：维持分割器，默认是`true`。如果禁用这个值，查询`foog`建议的时候，会得到以`Foo Fighters`开头的字段。
- `preserve_position_increments`：启用位置增量，默认是`true`。如果禁用该功能，使用停用词分析器，查询`b`建议值的时候会得到以`The Beatles`开头的字段。注意：你可以索引两个输入`Beatles`和`The Beatles`来做到这一点，如果你可以丰富数据，则不需要改变简单分析器。
- `max_input_length`：限制单个输入的长度，默认为50个UTF-16代码点。此限制只在索引阶段使用，用来减少每个输入字符串的字符数量以免大量的输入使底层数据结构膨胀。大多数用例不会受默认值的影响，因为前缀长度不会很长。

## 索引

像其他字段一样来索引建议。建议由`input`和可选的`weight`属性构成。`input`是建议查询预期匹配的文本，`weight`决定建议如何评分。索引建议如下所示：

```
curl -XPUT 'localhost:9200/music/song/1?refresh&pretty' -H 'Content-Type: application/json' -d'
{
    "suggest" : {
        "input": [ "Nevermind", "Nirvana" ],
        "weight" : 34
    }
}
'
```

支持以下参数：

- `input`：输入。可以是字符串数组或者仅仅一个字符串。这个字段是必须的。
- `weight`：一个正整数或者包含正整数的字符串，定义权重且对建议排序。该字段是可选的。

如下为一个文档索引多个建议：

```
curl -XPUT 'localhost:9200/music/song/1?refresh&pretty' -H 'Content-Type: application/json' -d'
{
    "suggest" : [
        {
            "input": "Nevermind",
            "weight" : 10
        },
        {
            "input": "Nirvana",
            "weight" : 3
        }
    ]
}
'
```

你可以使用如下的简写形式。可以不为建议指定权重。

```
curl -XPUT 'localhost:9200/music/song/1?refresh&pretty' -H 'Content-Type: application/json' -d'
{
  "suggest" : [ "Nevermind", "Nirvana" ]
}
'
```

## 查询

建议像往常一样工作，除了你必须将建议类型指定为`completion`。建议时接近实时的，这意味着新的建议使用`refresh`变得可见，文档一旦删除就不可见。

请求：

```
curl -XPOST 'localhost:9200/music/_search?pretty&pretty' -H 'Content-Type: application/json' -d'
{
    "suggest": {
        "song-suggest" : {
            "prefix" : "nir",
            "completion" : {
                "field" : "suggest"
            }
        }
    }
}
'
```

返回以下响应：

```
{
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  },
  "hits": ...
  "took": 2,
  "timed_out": false,
  "suggest": {
    "song-suggest" : [ {
      "text" : "nir",
      "offset" : 0,
      "length" : 3,
      "options" : [ {
        "text" : "Nirvana",
        "_index": "music",
        "_type": "song",
        "_id": "1",
        "_score": 1.0,
        "_source": {
          "suggest": ["Nevermind", "Nirvana"]
        }
      } ]
    } ]
  }
}
```

> 元字段`_source`必须开启，默认在建议中同步返回`_source`。

建议的权重作为`_score`返回。`text`字段使用经过索引的建议的`input`。建议默认返回完整的文档`_source`。`_source`的大小会影响性能，由于磁盘读取和网络传输的开销。为了节约网络开销，可以使用`source filtering`过滤`_source`中不需要的字段来减少`_source`的大小。注意`_suggest`请求不支持source过滤但是在`_search`请求中使用建议支持source过滤：

```
curl -XPOST 'localhost:9200/music/_search?size=0&pretty' -H 'Content-Type: application/json' -d'
{
    "_source": "suggest",
    "suggest": {
        "song-suggest" : {
            "prefix" : "nir",
            "completion" : {
                "field" : "suggest"
            }
        }
    }
}
'
```

响应如下：

```
{
    "took": 6,
    "timed_out": false,
    "_shards" : {
        "total" : 5,
        "successful" : 5,
        "failed" : 0
    },
    "hits": {
        "total" : 0,
        "max_score" : 0.0,
        "hits" : []
    },
    "suggest": {
        "song-suggest" : [ {
            "text" : "nir",
            "offset" : 0,
            "length" : 3,
            "options" : [ {
                "text" : "Nirvana",
                "_index": "music",
                "_type": "song",
                "_id": "1",
                "_score": 1.0,
                "_source": {
                    "suggest": ["Nevermind", "Nirvana"]
                }
            } ]
        } ]
    }
}
```

基本的`completion`建议器查询支持以下参数：

- `field`：在其中运行查询的字段的名称（必须）
- `size`：返回的建议数量（默认为5）

> `completion`建议器考虑索引中所有的文档。有关如何查询文档子集的解释，请参阅`Context Suggester`

> 在完成查询跨越多个分片的情况下，建议在两个阶段中执行，最后一个阶段从分片中获取相关的文档，这意味着当建议跨越多个分片时，对单个碎片执行完成请求由于文档提取开销而更加高效。为了获取最佳的完成性能，建议将完成索引到单个分片索引中。在由于分片大小而导致的堆使用率过高的情况下，仍然建议将索引拆分为多个分片，而不是优化完成性能。

## 模糊查询

完成建议器也支持模糊查询——这意味着即使搜索中有拼写错误也可以获得正确的结果

```
curl -XPOST 'localhost:9200/music/_search?pretty&pretty' -H 'Content-Type: application/json' -d'
{
    "suggest": {
        "song-suggest" : {
            "prefix" : "nor",
            "completion" : {
                "field" : "suggest",
                "fuzzy" : {
                    "fuzziness" : 2
                }
            }
        }
    }
}
'
```

和查询`prefix`共享最长前缀的建议具有更高的分值。

模糊查询可以采用特定的模糊参数。支持以下参数：

|参数|说明|
|---|----|
|`fuzziness`|模糊因子，默认是`AUTO`。|
|`transpositions`|如果设置为`true`，换位被记为一次更改而不是两次，默认为`true`|
|`min_length`|模糊建议器返回前输入的最小长度，默认为3|
|`prefix_length`|输入的最小长度（未针对模糊替代项进行检查）默认为1|
|`unicode_aware`|如果设置为`true`，则所有的度量都以unicode代码点而不是字节为单位。这比原始字节稍慢，因此默认情况下设置为`false`。|

> 如果你想坚持使用默认值，但仍然使用模糊查询，你可以使用`fuzzy:{}`或`fuzzy:true`

## 正则查询

`completion`建议器也支持正则查询，这意味着你可以将前缀表达为一个正则表达式。

```
curl -XPOST 'localhost:9200/music/_search?pretty&pretty' -H 'Content-Type: application/json' -d'
{
    "suggest": {
        "song-suggest" : {
            "regex" : "n[ever|i]r",
            "completion" : {
                "field" : "suggest"
            }
        }
    }
}
'
```

正则查询可以采用特定的正则参数。支持以下参数：

|参数|说明|
|---|----|
|`flags`|可能的标志是`ALL`（默认）,`ANYSTRING`, `COMPLEMENT`, `EMPTY`, `INTERSECTION`, `INTERVAL`, `NONE`。具体的意义查看`regexp-syntax`|
|`max_determinized_states`|正则表达式是危险的，很容易在无意中创建了无恶意的表达式，需要指数数量的内部确定的自动机状态（以及相应的RAM和CPU）来执行Lucene。Lucene使用`max_determinized_states`设置（默认为10000）阻止这些操作。你可以提高此限制以允许执行更复杂的正则表达式。|

