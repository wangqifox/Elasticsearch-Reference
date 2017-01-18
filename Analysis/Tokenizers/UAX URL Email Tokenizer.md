# UAX URL Email分词器

`uax_url_email`分词器和标准分词器一样，不过它可以识别URLs和email地址，将它们作为一个单独的分词。

## 示例输出

```
curl -XPOST 'localhost:9200/_analyze?pretty' -d'
{
  "tokenizer": "uax_url_email",
  "text": "Email me at john.smith@global-international.com"
}
'
```

产生以下分词：

```
[ Email, me, at, john.smith@global-international.com ]
```

如果使用标准分词器会产生以下分词：

```
[ Email, me, at, john.smith, global, international.com ]
```

## 配置

`uax_url_email`分词器接受以下参数：

|参数|说明|
|---|---|
|max_token_length|最大的分词长度。如果分词长度超过了最大长度，则以最大长度为间隔分割分词。默认是255|

## 示例配置

该示例中，我们配置`uax_url_email`分词器的`max_token_length`参数为5。

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
          "type": "uax_url_email",
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
  "text": "john.smith@global-international.com"
}
'
```

产生以下分词：

```
[ john, smith, globa, l, inter, natio, nal.c, om ]
```
