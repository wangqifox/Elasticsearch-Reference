# Suggesters（建议器）

建议功能基于提供的文本来建议相似的分词。部分建议功能仍然在开发中。

建议请求部分在`_search`请求中跟随查询一起定义。

> `_suggest`已被弃用，改成通过`_search`来使用建议功能。5.0中，`_search`已经过优化，仅用于建议搜索请求。

```
curl -XPOST 'localhost:9200/twitter/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query" : {
    "match": {
      "message": "tring out Elasticsearch"
    }
  },
  "suggest" : {
    "my-suggestion" : {
      "text" : "trying out Elasticsearch",
      "term" : {
        "field" : "message"
      }
    }
  }
}
'
```

一次请求中可以指定多个建议。每个建议可以通过随意的名字来标识。下面的例子中请求了两个建议。`my-suggest-1`和`my-suggest-2`建议都使用了`term`建议器，但是文本不一样。

```
curl -XPOST 'localhost:9200/_suggest?pretty' -H 'Content-Type: application/json' -d'
{
  "my-suggest-1" : {
    "text" : "tring out Elasticsearch",
    "term" : {
      "field" : "message"
    }
  },
  "my-suggest-2" : {
    "text" : "kmichy",
    "term" : {
      "field" : "user"
    }
  }
}
'
```

下面是`my-suggest-1`和`my-suggest-2`的响应。每个建议响应包含多个条目。每个条目实际上是来自建议文本的分词，包含建议条目文本、初始的起始偏移量、建议文本的长度，随意数量的选项。

```
{
  "_shards": ...
  "my-suggest-1": [ {
    "text": "tring",
    "offset": 0,
    "length": 5,
    "options": [ {"text": "trying", "score": 0.8, "freq": 1 } ]
  }, {
    "text": "out",
    "offset": 6,
    "length": 3,
    "options": []
  }, {
    "text": "elasticsearch",
    "offset": 10,
    "length": 13,
    "options": []
  } ],
  "my-suggest-2": ...
}
```

每个选项数组包含一个选项对象，选项对象包括建议文本、文档频率、与建议输入文本相比较的分数。该分数的意义依赖于使用的建议器。`term`建议器的分数基于编辑距离。

## 全局建议文本

为了避免重复建议文本，可以定义一个全局文本。下面的例子中，建议文本在全局中定义，同时应用在`my-suggest-1`和`my-suggest-2`。

```
curl -XPOST 'localhost:9200/_suggest?pretty' -H 'Content-Type: application/json' -d'
{
  "text" : "tring out Elasticsearch",
  "my-suggest-1" : {
    "term" : {
      "field" : "message"
    }
  },
  "my-suggest-2" : {
    "term" : {
      "field" : "user"
    }
  }
}
'
```

上面例子中的建议文本可以作为建议特定选项来指定。建议级别指定的建议文本会覆盖全局级别的建议文本。
