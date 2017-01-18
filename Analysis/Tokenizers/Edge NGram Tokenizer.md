# Edge NGram 分词器

`edge_ngram`分词器遇到指定字符列表中的字符时将文本分割成单词，然后针对每个单词产生指定长度的`N-gram`，`N-gram`的起始锚定在单词的开始。

Edge N-Gram对于`按输入搜索`很有用。

> 当你需要的`按输入搜索`文本按一个为人所知的顺序时（比如电影或者歌曲标题），`completion suggester`是比`edge N-gram`更高效的选择。当需要自动完成以任意顺序出现的单词时，`edge N-gram`会更有优势。

## 示例输出

默认设置下，`edge_ngram`将原始文本当做单个分词，使用最小长度1和最大长度2来生成N-gram。

```
curl -XPOST 'localhost:9200/_analyze?pretty' -d'
{
  "tokenizer": "edge_ngram",
  "text": "Quick Fox"
}
'
```

产生以下分词：

```
[ Q, Qu ]
```

> 默认的gram长度几乎完全无用。使用`edge_ngram`前必须先配置。

## 配置

`edge_ngram`分词器接受以下参数：

|参数|说明|
|---|----|
|min_gram|字符最小长度。默认是1|
|max_gram|字符最大长度。默认是2|
|token_chars|包含在分词中的字符类。Elasticsearch会在不属于该分类的字符处分割文本。默认是`[]`（保留所有字符）|

字符类可以是以下的任意一种

- `letter`—— `a`, `b`, `ï`, `京`
- `digit`—— `3`, `7`
- `whitespace`——`" "`, `"\n"`
- `punctuation`——`!`, `"`
- `symbol`——`$`, `√`

## 示例配置

在该例子中，我们配置`ngram`分词器，将字母和数字当成是分词，产生最小长度为2最大长度为10的gram。

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "my_tokenizer"
        }
      },
      "tokenizer": {
        "my_tokenizer": {
          "type": "edge_ngram",
          "min_gram": 2,
          "max_gram": 10,
          "token_chars": [
            "letter",
            "digit"
          ]
        }
      }
    }
  }
}
'
curl -XPOST 'localhost:9200/my_index/_analyze?pretty' -d'
{
  "analyzer": "my_analyzer",
  "text": "2 Quick Foxes."
}
'
```

产生以下分词：

```
[ Qu, Qui, Quic, Quick, Fo, Fox, Foxe, Foxes ]
```

通常我们推荐在索引阶段和搜索阶段使用相同的分析器。但是在使用`edge_ngram`分词器时，该建议有些不同。只有在索引阶段使用`edge_ngram`分词器才有意义，以确保索引中部分单词可用于匹配。在搜索阶段，直接使用用户输入的分词，比如：`Quick Fo`。

下面是一个例子，展示了如何建立一个`按输入搜索`的字段：

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "settings": {
    "analysis": {
      "analyzer": {
        "autocomplete": {
          "tokenizer": "autocomplete",
          "filter": [
            "lowercase"
          ]
        },
        "autocomplete_search": {
          "tokenizer": "lowercase"
        }
      },
      "tokenizer": {
        "autocomplete": {
          "type": "edge_ngram",
          "min_gram": 2,
          "max_gram": 10,
          "token_chars": [
            "letter"
          ]
        }
      }
    }
  },
  "mappings": {
    "doc": {
      "properties": {
        "title": {
          "type": "text",
          "analyzer": "autocomplete",
          "search_analyzer": "autocomplete_search"
        }
      }
    }
  }
}
'
curl -XPUT 'localhost:9200/my_index/doc/1?pretty' -d'
{
  "title": "Quick Foxes" 	// 1
}
'
curl -XPOST 'localhost:9200/my_index/_refresh?pretty'
curl -XGET 'localhost:9200/my_index/_search?pretty' -d'
{
  "query": {
    "match": {
      "title": {
        "query": "Quick Fo", 	// 2
        "operator": "and"
      }
    }
  }
}
'
```

- 1 `autocomplete`分析器索引以下的分词：`[qu, qui, quic, quick, fo, fox, foxe, foxes]`
- 2 `autocomplete_search`分析器搜索以下分词：`[quick, fo]`，两个都出现在索引中。
