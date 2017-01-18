## 经典分词器

经典分词器是一个基于语法的分词器，特别适用于英语文档。该分词器对首字母缩略词、公司名、email地址、网络主机名使用启发式的特殊对待。但是这些规则并非总是适用，该分词器对于大多数非英语的语言都不适用：

- 它在大多数标点符号上分割文本，删除标点符号。但是后面不跟着空格的点符号(.)被当成是分词的一部分。
- 它在连字号(-)上分割文本，除非分词中有数字，这种情况下整个分词解释为一个产品编号，不分割。
- 它将email地址和网络主机名识别为一个分词

## 示例输出

```
curl -XPOST 'localhost:9200/_analyze?pretty' -d'
{
  "tokenizer": "classic",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog\u0027s bone."
}
'
```

产生以下分词：

```
[ The, 2, QUICK, Brown, Foxes, jumped, over, the, lazy, dog's, bone ]
```

## 配置

经典分词器接受以下参数：

|参数|说明|
|---|----|
|max_token_length|最大的分词长度。如果分词长度超过了最大长度，则以最大长度为间隔分割分词。默认是255|

## 示例配置

该例子中，我们将经典分词器的`max_token_length`配置为5。

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
          "type": "classic",
          "max_token_length": 5
        }
      }
    }
  }
}
'
curl -XPOST 'localhost:9200/my_index/_analyze?pretty' -d'
{
  "analyzer": "my_analyzer",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}
'
```

产生以下分词：

```
[ The, 2, QUICK, Brown, Foxes, jumpe, d, over, the, lazy, dog's, bone ]
```
