# NGram分词器

`ngram`分词器遇到指定字符列表中的字符时将文本分割成单词，然后针对每个单词产生指定长度的`N-gram`。

N-gram就像是一个在单词上移动的滑动窗口——指定长度的连续字符序列。它们对于查询不使用空格的语言或德语这样有长复合词的语言很有用。

## 示例输出

默认设置下，`ngram`分词器将原始文本视为单个分词，使用最小长度1和最大长度2来生成N-gram。

```
curl -XPOST 'localhost:9200/_analyze?pretty' -d'
{
  "tokenizer": "ngram",
  "text": "Quick Fox"
}
'
```

产生以下分词：

```
[ Q, Qu, u, ui, i, ic, c, ck, k, "k ", " ", " F", F, Fo, o, ox, x ]
```

## 配置

`ngram`分词器接受以下参数：

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

> 将`min_gram`和`max_gram`设置为同一个值通常是有意义的。长度越小，匹配的文档越多但是匹配的质量越差；长度越长，匹配得越详细。通常`tri-gram`（长度为3）是一个比较好的值。

## 示例配置

在该例子中，我们配置`ngram`分词器，将字母和数字当成是分词，产生`tri-gram`（长度为3）：

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
          "type": "ngram",
          "min_gram": 3,
          "max_gram": 3,
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
[ Qui, uic, ick, Fox, oxe, xes ]
```
