# Highlighting(高亮)

允许在一个或多个字段中高亮显示搜索结果。在实践中使用lucene`plain`高亮器、快速向量高亮器(fvh)或`postings`高亮器。下面是搜索请求体的示例：

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query" : {
        "match": { "content": "kimchy" }
    },
    "highlight" : {
        "fields" : {
            "content" : {}
        }
    }
}
'
```

在上面的示例中，`content`字段中每个搜索命中的词都会高亮显示(每个命中的搜索都会增加一个称为`highlight`的的元素，包含高亮显示的字段和高亮的片段)

> 为了执行高亮，字段真实的内容是必须的。如果查询的字段存储了（映射中`store`设置为`true`）则会使用这个字段，否则会加载真实的`_source`，从中提取出相关的字段。
> `_all`字段无法从`_source`中提取，所以只有当它在映射中设置为`true`才可以用于映射。

字段名称支持通配符标记。比如，使用`comment_*`可以使匹配该表达式的`text`和`keyword`字段（5.0之前的`string`字段）高亮。注意，所有的其他字段都不会被高亮。如果你是使用自定义映射，且想要高亮一个字段，必须显式提供字段名称。

## Plain highlighter

高亮器默认选择的类型是`plain`，使用Lucene高亮器。在短语查询中，理解单词重要性和任何单词定位标准方面，都很难反应查询匹配逻辑。

> 如果你想在大量文档中使用复杂查询来高亮大量的字段，这个高亮器不会很快。该高亮器致力于准确反映查询逻辑，它创建一个微小的内存索引，并通过Lucene的查询执行计划重新执行原始查询条件，以获取当前文档的低级匹配信息。对于需要高亮显示的每个字段和每个文档都会重复此操作。如果在你的系统中出现了性能问题，请考虑使用其他高亮器。

## Postings highlighter

如果映射中`index_options`设置为`offsets`，则会使用postings高亮器而不是plain高亮器。postings高亮器：

- 更快，因为它不需要重新分析用于高亮的文本：文档体积越大，获得的性能越好
- 相比于fast vector highlighter需要的term_vectors，所需的磁盘空间更少
- 将文本切分成句子再高亮显示。对于自然语言表现良好，但是对于包含html的字段则有问题。
- 把文档当成一个整体的文集，使用BM25算法对单独的句子打分就像它们是这个文集中的文档。

下面的示例中，在索引映射中设置`content`字段，允许使用postings高亮器来高亮显示:

```
{
    "type_name" : {
        "content" : {"index_options" : "offsets"}
    }
}
```

> 注意，postings高亮器意味着执行简单查询分词高亮，不管它们的位置。这意味着当用于与短语查询结合使用时，它将高亮显示查询所构成的所有分词，而不管它们实际上是查询匹配的一部分，有效地忽略了它们的位置。

> postings高亮器不支持高亮显示一些复杂的查询，例如类型设置为`match_phrase_prefix`的匹配查询。在这种情况下，不会返回高亮显示的片段。

## Fast vector highlighter

如果在映射中将`term_vector`设置为`with_positions_offsets`，则会使用fast vector highlighter而不是plain highlighter。fast vector highlighter：

- 更快，尤其是对于大字段（>1MB）
- 可以通过`boundary_chars`,`boundary_max_scan`,`fragment_offset`来自定义
- 必须将`term_vector`设置为`with_position_offsets`，这也会增加索引的大小
- 可以将多个字段中匹配的内容整合成单个结果，查看`matched_fields`
- 可以对不同位置的匹配词赋予不同的权重，当高亮显示boosting查询时将短语匹配的权重提升至高于分词匹配，允许短语匹配在排序时排在分词匹配的前面。

下面的示例中设置`content`字段，以使用fast vector highlighter来高亮显示（索引将变得更大）：

```
{
    "type_name" : {
        "content" : {"term_vector" : "with_positions_offsets"}
    }
}
```

## Unified Highlighter

> 该功能是实验性的，可能在将来的版本中完全更改或删除。Elastic将采取最大的努力来解决任何问题，但实验功能不受SLA官方功能的支持。

unified highlighter可以从postings, term vectors, 或者通过重新分析的文本中提取偏移量。它使用Lucene的UnifiedHighlighter，根据字段和查询选择策略来高亮显示。该策略的独特之处在于，使用BM25算法将文本拆分成句子，就像是文集中的文档一样对单个句子评分。它支持准确的短语和多分词（模糊、前缀、正则表达式）的高亮显示，可以使用以下选项：

- force_source
- encoder
- highlight_query
- pre_tags 和 post_tags
- require_field_match

## 强制高亮类型

`type`字段允许强制定义一个特定的类型。当实例在一个开启了`term_vectors`的字段上需要使用plain highlighter时，`type`字段就有用了。允许的值是：`plain`,`postings`,`fvh`。以下示例强制使用plain highlighter：

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query" : {
        "match": { "user": "kimchy" }
    },
    "highlight" : {
        "fields" : {
            "content" : {"type" : "plain"}
        }
    }
}
'
```

## 在source中强制开启高亮显示

基于source强制高亮显示字段，即使字段是单独存储的。默认为`false`。

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query" : {
        "match": { "user": "kimchy" }
    },
    "highlight" : {
        "fields" : {
            "content" : {"force_source" : true}
        }
    }
}
'
```

## 高亮显示标签

默认情况下，使用`<em>`和`</em>`来包裹高亮的文本。这个可以通过设置`pre_tags`和`post_tags`来控制，例如：

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query" : {
        "match": { "user": "kimchy" }
    },
    "highlight" : {
        "pre_tags" : ["<tag1>"],
        "post_tags" : ["</tag1>"],
        "fields" : {
            "_all" : {}
        }
    }
}
'
```

使用fast vector highlighter可以有更多的标签，且重要性被排序。

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query" : {
        "match": { "user": "kimchy" }
    },
    "highlight" : {
        "pre_tags" : ["<tag1>", "<tag2>"],
        "post_tags" : ["</tag1>", "</tag2>"],
        "fields" : {
            "_all" : {}
        }
    }
}
'
```

还有内置的“标签”模式，目前有一个单一的模式，称为`styled`，具有以下`pre_tags`：

```
<em class="hlt1">, <em class="hlt2">, <em class="hlt3">,
<em class="hlt4">, <em class="hlt5">, <em class="hlt6">,
<em class="hlt7">, <em class="hlt8">, <em class="hlt9">,
<em class="hlt10">
```

`post_tags`为`</em>`。如果你认为有更好的内置标签模式，只需发送电子邮件到邮件列表或打开一个issue。以下是切换标记模式的示例：

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query" : {
        "match": { "user": "kimchy" }
    },
    "highlight" : {
        "tags_schema" : "styled",
        "fields" : {
            "content" : {}
        }
    }
}
'
```

## Encoder

`encoder`参数可以用来定义高亮的文本如何编码。可以是`default`（不编码）或者`html`（编码html，如果你使用html高亮编码）

## 高亮片段

每个字段高亮可以控制高亮片段的字符数（默认为5）。

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query" : {
        "match": { "user": "kimchy" }
    },
    "highlight" : {
        "fields" : {
            "content" : {"fragment_size" : 150, "number_of_fragments" : 3}
        }
    }
}
'
```

当使用postings highlighter时`fragment_size`会被忽略，因为输出的句子会忽略长度。

除此之外，可以指定高亮片段按分数排序：

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query" : {
        "match": { "user": "kimchy" }
    },
    "highlight" : {
        "order" : "score",
        "fields" : {
            "content" : {"fragment_size" : 150, "number_of_fragments" : 3}
        }
    }
}
'
```

如果`number_of_fragments`值设置为0，则不会产生任何碎片，而是返回该字段的整个内容，当然也会高亮显示。如果需要高亮显示短文本（如文档标题或地址），但不需要分片，这可能非常方便。注意：这种情况下，fragment_size将被忽略。

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query" : {
        "match": { "user": "kimchy" }
    },
    "highlight" : {
        "fields" : {
            "_all" : {},
            "bio.title" : {"number_of_fragments" : 0}
        }
    }
}
'
```

使用`fvh`时，可以使用`fragment_offset`参数来控制开始高亮显示的便宜量。

在没有匹配片段高亮显示的情况下，默认不返回任何东西。相反，我们可以通过将`no_match_size`（默认为0）设置为要返回的文本的长度，从字段的开始返回一段文本。实际长度可能会比指定的长度短，因为它试图在单词边界上断开。当使用postings highlighter时，无法控制片段的实际大小，因此只要`no_match_size`大于0，第一个句子就会返回。

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query" : {
        "match": { "user": "kimchy" }
    },
    "highlight" : {
        "fields" : {
            "content" : {
                "fragment_size" : 150,
                "number_of_fragments" : 3,
                "no_match_size": 150
            }
        }
    }
}
'
```

## 高亮查询

还可以通过设置`highlight_query`来高亮显示搜索查询以外的查询。如果你使用rescore查询，这是非常有用的，因为默认情况下高亮显示不会考虑这些。Elasticsearch不会以任何方式验证`highlight_query`包含的搜索查询，所以可以定义它，这样合法的查询结果就不会高亮显示。一般来说，最好是在highlight_query中加入搜索查询。以下是在highlight_query中包含搜索查询和rescore查询的示例。

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "stored_fields": [ "_id" ],
    "query" : {
        "match": {
            "content": {
                "query": "foo bar"
            }
        }
    },
    "rescore": {
        "window_size": 50,
        "query": {
            "rescore_query" : {
                "match_phrase": {
                    "content": {
                        "query": "foo bar",
                        "slop": 1
                    }
                }
            },
            "rescore_query_weight" : 10
        }
    },
    "highlight" : {
        "order" : "score",
        "fields" : {
            "content" : {
                "fragment_size" : 150,
                "number_of_fragments" : 3,
                "highlight_query": {
                    "bool": {
                        "must": {
                            "match": {
                                "content": {
                                    "query": "foo bar"
                                }
                            }
                        },
                        "should": {
                            "match_phrase": {
                                "content": {
                                    "query": "foo bar",
                                    "slop": 1,
                                    "boost": 10.0
                                }
                            }
                        },
                        "minimum_should_match": 0
                    }
                }
            }
        }
    }
}
'
```

注意这种情况下，文本片段的分值通过Lucene高亮框架来计算。你可以查看`ScoreOrderFragmentsBuilder.java`类来获取执行的细节。另一方面，当使用postings highlighter时，如上所述使用BM25算法对片段进行评分。

## 全局设置

高亮设置可以在全局级别来设置，在字段级别可以覆盖设置

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query" : {
        "match": { "user": "kimchy" }
    },
    "highlight" : {
        "number_of_fragments" : 3,
        "fragment_size" : 150,
        "fields" : {
            "_all" : { "pre_tags" : ["<em>"], "post_tags" : ["</em>"] },
            "bio.title" : { "number_of_fragments" : 0 },
            "bio.author" : { "number_of_fragments" : 0 },
            "bio.content" : { "number_of_fragments" : 5, "order" : "score" }
        }
    }
}
'
```

## Require Field Match

`require_field_match`设置为`false`会使任意字段高亮显示，不管查询是否匹配这些字段。默认行为是`true`，意味着只有匹配查询的字段会高亮显示。

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query" : {
        "match": { "user": "kimchy" }
    },
    "highlight" : {
        "require_field_match": false,
        "fields": {
                "_all" : { "pre_tags" : ["<em>"], "post_tags" : ["</em>"] }
        }
    }
}
'
```

## 边界符号

当使用fast vector highlighter来高亮显示字段时，可以配置`boundary_chars`来定义高亮显示由什么边界构成。它是一个单个字符串，其中定义了每个边界字符。默认为`.,!? \t\n`

`boundary_max_scan`控制查找边界符号的距离有多远，默认为20

## 匹配字段

fast vector highlighter可以使用`matched_fields`将高亮显示的多个字段整合成一个单独的字段。这对于使用不同方式分析相同字符串的多个字段来说是很直观的。所有的`matched_fields`都必须将`term_vector`设置为`with_position_offsets`，但是只有匹配整合到的字段才会加载，所以只有那个字段才会从`store`设置为`yes`中受益。

下面的例子中，`content`使用`english`分析器来分析，`content.plain`使用`standard`分析器来分析。

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query": {
        "query_string": {
            "query": "content.plain:running scissors",
            "fields": ["content"]
        }
    },
    "highlight": {
        "order": "score",
        "fields": {
            "content": {
                "matched_fields": ["content", "content.plain"],
                "type" : "fvh"
            }
        }
    }
}
'
```

上面的例子匹配"run with scissors"和"running with scissors"，会高亮显示"running"和"scissors"，但是不会高亮显示"run"。如果两个短语都出现在一个大文档中，片段列表中"running with scissors"会排在"run with scissors"前面，因为它匹配了更多的片段。

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query": {
        "query_string": {
            "query": "running scissors",
            "fields": ["content", "content.plain^10"]
        }
    },
    "highlight": {
        "order": "score",
        "fields": {
            "content": {
                "matched_fields": ["content", "content.plain"],
                "type" : "fvh"
            }
        }
    }
}
'
```

上面的例子高亮显示"run","running","scissors"，但是"running with scissors"仍然排在"run with scissors"之前，因为plain匹配（"running"）被提升了。

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query": {
        "query_string": {
            "query": "running scissors",
            "fields": ["content", "content.plain^10"]
        }
    },
    "highlight": {
        "order": "score",
        "fields": {
            "content": {
                "matched_fields": ["content.plain"],
                "type" : "fvh"
            }
        }
    }
}
'
```

上面的查询不会高亮"run"或"scissor"，但是表明在匹配字段中不列出匹配组合（内容）的字段是好的。

> 技术上来讲，将字段添加到`matched_fields`中，在技术上，将匹配的字段添加到match_fields中，并且不会与匹配组合的字段共享相同的底层字符串。 结果可能没有什么意义，如果其中一个匹配结束了文本，则整个查询将失败。

> 将matched_fields设置为非空数组时所需的开销很少，因此总是倾向将
>
	"highlight": {
        "fields": {
            "content": {}
        }
    }

> 设置为
>
	"highlight": {
        "fields": {
            "content": {
                "matched_fields": ["content"],
                "type" : "fvh"
            }
        }
    }

## 短语限制

fast vector highlighter有一个`phrase_limit`参数，防止分析过多的短语而导致消耗过多的内存。默认为256，因此只考虑文档中最开始的256个匹配短语。你可以增加`phrase_limit`参数的值，但是记住对越多的短语评分消耗越多的时间和内存。

如果使用`matched_fields`，记住每个匹配字段的`phrase_limit`短语都被考虑。

## 字段高亮顺序

Elasticsearch以字段被传送的顺序来高亮显示字段。每个json spec对象是无序的，但是如果你需要明确指出字段被高亮显示的顺序，那么你可以使用这样的字段的数组：

    "highlight": {
        "fields": [
            {"title":{ /*params*/ }},
            {"text":{ /*params*/ }}
        ]
    }

Elasticsearch内置的高亮器都不关心字段高亮的顺序，但是有插件可以实现
