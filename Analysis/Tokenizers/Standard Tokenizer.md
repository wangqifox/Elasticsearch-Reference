# 标准分词器

标准分词器提供基于语法的词汇切分（基于Unicode文本分割算法），适用于大部分语言。

## 示例输出

```
curl -XPOST 'localhost:9200/_analyze?pretty' -d'
{
  "tokenizer": "standard",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}
'
```

产生以下分词：

```
[ The, 2, QUICK, Brown, Foxes, jumped, over, the, lazy, dog's, bone ]
```

## 配置

标准分词器接受以下参数：

|参数|说明|
|---|----|
|max_token_length|最大的分词长度。如果分词长度超过了最大长度，则以最大长度为间隔分割分词。默认是255|

## 示例配置

该例子中，我们将标准分词器的`max_token_length`配置为5。

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
          "type": "standard",
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
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog\u0027s bone."
}
'
```

产生以下分词：

```
[ The, 2, QUICK, Brown, Foxes, jumpe, d, over, the, lazy, dog's, bone ]
```
