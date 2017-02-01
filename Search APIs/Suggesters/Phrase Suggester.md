# Phrase Suggester（短语建议器）

`term`建议器提供了一个非常方便的API，在某个字符串距离内在每个分词的基础上访问可供替换的单词。API允许在流中单独访问每个分词，建议选择则留给API使用者。然而，建议通常需要预先选择以呈现给最终用户。`phrase`建议器在`term`建议器之上添加了附加逻辑，以选择整个经过校正的短语，而不是基于`ngram`语言模型加权的单个分词。在实践中，该建议器可以基于共现和频率来做出关于选择哪些分词的更好的选择。

## API示例

通常情况下，`phrase`建议器需要之前有特殊的映射。此页面上的`phrase`建议器示例需要以下的映射。`reverse`分析器仅仅使用在最后的例子中。

```
curl -XPUT 'localhost:9200/test?pretty' -H 'Content-Type: application/json' -d'
{
  "settings": {
    "index": {
      "number_of_shards": 1,
      "analysis": {
        "analyzer": {
          "trigram": {
            "type": "custom",
            "tokenizer": "standard",
            "filter": ["standard", "shingle"]
          },
          "reverse": {
            "type": "custom",
            "tokenizer": "standard",
            "filter": ["standard", "reverse"]
          }
        },
        "filter": {
          "shingle": {
            "type": "shingle",
            "min_shingle_size": 2,
            "max_shingle_size": 3
          }
        }
      }
    }
  },
  "mappings": {
    "test": {
      "properties": {
        "title": {
          "type": "text",
          "fields": {
            "trigram": {
              "type": "text",
              "analyzer": "trigram"
            },
            "reverse": {
              "type": "text",
              "analyzer": "reverse"
            }
          }
        }
      }
    }
  }
}
'
curl -XPOST 'localhost:9200/test/test?refresh=true&pretty' -H 'Content-Type: application/json' -d'
{"title": "noble warriors"}
'
curl -XPOST 'localhost:9200/test/test?refresh=true&pretty' -H 'Content-Type: application/json' -d'
{"title": "nobel prize"}
'
```

一旦设置了分析器和映射，你可以在`term`建议器使用的任何地方使用`phrase`建议器：

```
curl -XPOST 'localhost:9200/_suggest?pretty' -H 'Content-Type: application/json' -d'
{
  "text": "noble prize",
  "simple_phrase": {
    "phrase": {
      "field": "title.trigram",
      "size": 1,
      "gram_size": 3,
      "direct_generator": [ {
        "field": "title.trigram",
        "suggest_mode": "always"
      } ],
      "highlight": {
        "pre_tag": "<em>",
        "post_tag": "</em>"
      }
    }
  }
}
'
```

响应包含由最可能的拼写纠正评分的建议。这个例子中，我们收到了符合预期的经过校正的"nobel prize"。

```
{
  "_shards": ...
  "simple_phrase" : [
    {
      "text" : "noble prize",
      "offset" : 0,
      "length" : 11,
      "options" : [ {
        "text" : "nobel prize",
        "highlighted": "<em>nobel</em> prize",
        "score" : 0.5962314
      }]
    }
  ]
}
```

## 基本的短语建议API参数

|参数|说明|
|---|---|
|`field`|用于对语言模型进行n-gram查找的字段名称。建议器使用此字段获取统计信息来进行评分更正。此字段是必填字段。|
|`gram_size`|设置字段中n-grams(shingles)的最大值。如果字段不包含n-grams(shingles)则应省略或者设为1。注意，Elasticsearch尝试根据指定的字段来检测gram的大小。如果字段使用`shingle`过滤器，若未明确设置，`gram_size`设置为`max_shingle_size`。|
|`real_word_error_likelihood`|即使分词存在于字典中，但仍然是拼写错误的可能性。默认是0.95，对应5%的真实单词拼写错误。|
|`confidence`|置信水平定义了应用于输入短语分数的因子，它被用于其他建议候选的阈值。只有得分高于阈值的候选建议才会包括在结果中。比如置信水平为1.0时，返回分数大于输入短语的建议。如果设置为0.0，返回前N个候选值。默认是1.0|
|`max_errors`|为了生成错误校正，最多被认为是拼写错误的分词的最大百分比。如果是范围`[0..1)`的浮点值则作为实际查询项的分数，如果是`>=0`的值则作为查询项的绝对数。默认值为1.0，对应于返回最多1个拼写错误项的更正。注意，将其设置得过高会对性能产生负面影响。建议使用1或者2这样的值，否则建议调用的时间花费可能超过查询执行的时间花费。|
|`separator`|在bigram字段中用来分隔分词。如果没有设置该值，使用空格作为分隔符|
|`size`|每个查询分词产生的候选词的数量，通常3或5可以产生比较好的结果。提高这个值会产生编辑距离过大的分词。默认是5|
|`analyzer`|设置分析建议文本的分析器。默认是通过`field`传入建议字段的搜索分析器。|
|`shard_size`|设置从每个单独分片获得的建议分词的最大数量。在reduce阶段，基于`size`选项返回前N个建议。默认是5|
|`text`|设置提供建议的文本或者查询|
|`highlight`|设置建议高亮。如果没有设置则不返回`highlighted`字段。如果设置了该值，则必须包含精确的`pre_tag`和`post_tag`，包裹改变的分词。如果一行中有多个改变的分词，则包裹分词的整个短语而不是每个分词。|
|`collate`|针对指定的查询检查每个建议，以修剪那些索引中没有文档匹配的建议。对于建议的校对查询只运行在本地分片中，建议从本地分片中生成。必须指定查询，且作为`template`查询运行。当前的建议自动作为`{{suggestion}}`变量，在查询中使用。你仍然可以指定自己的模板`params`——`suggestion`值将添加到你指定的变量。此外，你可以指定`prune`来控制是否返回所有短语建议，当设置为`true`时，建议会有一个附加选项`collate_match`，如果找到匹配短语的文档则为`true`，否则为`false`。`prune`的默认值为`false`。|

```
curl -XPOST 'localhost:9200/_suggest?pretty' -H 'Content-Type: application/json' -d'
{
  "text" : "noble prize",
  "simple_phrase" : {
    "phrase" : {
      "field" :  "title.trigram",
      "size" :   1,
      "direct_generator" : [ {
        "field" :            "title.trigram",
        "suggest_mode" :     "always",
        "min_word_length" :  1
      } ],
      "collate": {
        "query": {  // 1
          "inline" : {
            "match": {
              "{{field_name}}" : "{{suggestion}}"   // 2
            }
          }
        },
        "params": {"field_name" : "title"},     // 3
        "prune": true   // 4 
      }
    }
  }
}
'
```

- 1 该查询对于每个建议只运行一次
- 2 `{{suggestion}}`变量会被每个建议文本替换。
- 3 额外的`field_name`变量在`params`中指定，在`match`查询中使用。
- 4 所有的建议都会返回一个额外的`collate_match`选项，表示生成的短语是否匹配任何文档。

## 平滑模型

`phrase`建议器支持多种平滑模型在不频繁grams（未出现在索引中的grams）和频繁grams（在索引中出现至少一次的gram）之间平衡权重。

|模型|说明|
|---|----|
|`stupid_backoff`|简单的回退模型，如果高阶计数为0并且通过常数因子折扣低阶n-gram模型，则回退到低阶n-gram模型。默认折扣为0.4。`stupid_backoff`是默认模型。|
|`laplace`|使用添加平滑的平滑模型，其中将常数（通常为1.0或更小）添加到所有计数以平衡权重。默认`alpha`为0.5|
|`linear_interpolation`|平滑模型，其基于用户提供的权重(lambdas)获取`unigrams`,`bigrams`,`trigrams`的加权平均值。线性插值没有任何默认值。必须提供所有参数(`trigram_lambda`,`bigram_lambda`,`unigram_lambda`)。|

## 候选值生成器

短语建议器使用候选值生成器来产生给定文本中每个分词的可能分词列表。单个候选生成器类似于对文本中每个单独分词调用的分词建议。随后，将生成器的输出与来自其他项的候选分词一起打分用于候选值的建议。

目前只支持一种类型的候选生成器，`direct_generator`。Phrase建议API接受在关键`direct_generator`下的生成器列表，列表中每个生成器在原始文本中被称为每个term。

## 直接生成器

直接生成器支持以下参数：

|参数|说明|
|---|----|
|`field`|从中获取候选建议的字段。这是必须的选项，需要设置全局或每个建议。|
|`size`|每个建议文本分词返回的最大更正值。|
|`suggest_mode`|建议模式，控制每个分片上生成的建议中包含哪些建议。除了`always`之外的所有值可以认为是优化手段，产生更少的建议在每个分片中测试，整合每个分片中产生的建议时不用重新检查。因此，对于不包含它们的分片，即使其他分片包含它们，`missing`模式也会生成对分片的建议。应该使用`confidence`过滤掉它们。可以指定三个可能的值：- `missing`：只对不在分片中的分词产生建议。默认。 - `popular`：只对分片中比原始分词出现在更多文档中的分词提供建议。 - `always`：基于在建议文本中的分词提供任何匹配的建议。|
|`max_edits`|候选建议被认为是建议可以具有的最大编辑距离。只能是介于1和2。任何其他值都会导致请求抛出错误。默认是2|
|`prefix_length`|候选建议必须匹配的最少前缀字符数量。默认是1。增加这个值可以提高拼写检查的性能。通常分词开始的时候不会发生拼写错误。（旧名称`prefix_len`已废弃）|
|`min_word_length`|建议文本分词必须具有的最少长度。默认是4。（旧名称`min_word_len`已废弃）|
|`max_inspections`|与`shards_size`相乘，用来在共享级别检查更多候选的拼写校正。以性能为代价提高精度。默认为5。|
|`min_doc_freq`|建议出现的文档的最小数量阈值。可以以绝对数量的形式指定或者以文档数量的相对百分比指定。只对高频分词提供建议可以提供质量。默认是`0f`，禁用。如果指定了一个大于1的值，则该值不能为小数。分片级别的文档频率被用于此选项。|
|`max_term_freq`|建议文本分词存在的文档的最大数量阈值。可以是一个相对百分比（比如0.4）或者一个绝对值代表文档频率。如果指定了一个大于1的值，则不可以是小数。默认是0.01f。该参数可以用来将高频分词排除于拼写检查外。高频分词通常拼写是正确的，这也提高了拼写检查的性能。分片级别的文档频率被用于此选项。|
|`pre_filter`|应用于传递到该候选生成器的每个分词的过滤器（分析器）。在生成候选值之前，此过滤器应用于原始分词。|
|`post_filter`|在它们被传递给实际短语计分器之前应用于所生成的每个分词的过滤器（分析器）。|

下面的示例展示了`phrase`建议器和两个生成器一起调用，第一个使用包含普通索引分词的字段；第二个使用字段，该字段使用经过`reverse`过滤器索引的分词（分词以逆序来索引）。这是为了克服直接生成器的限制——需要恒定的前缀来提供高性能建议。`pre_filter`和`post_filter`选项接受普通的分析器名字。

```
curl -XPOST 'localhost:9200/_suggest?pretty' -H 'Content-Type: application/json' -d'
{
  "text" : "obel prize",
  "simple_phrase" : {
    "phrase" : {
      "field" : "title.trigram",
      "size" : 1,
      "direct_generator" : [ {
        "field" : "title.trigram",
        "suggest_mode" : "always"
      }, {
        "field" : "title.reverse",
        "suggest_mode" : "always",
        "pre_filter" : "reverse",
        "post_filter" : "reverse"
      } ]
    }
  }
}
'
```

`pre_filter`和`post_filter`可以用来在候选词生成之后注入同义词。比如查询`captain usq`，针对分词`usq`产生候选词`usa`，`usa`的同义词是`america`，如果短语的分值够高可以将`captain america`呈现给用户。

