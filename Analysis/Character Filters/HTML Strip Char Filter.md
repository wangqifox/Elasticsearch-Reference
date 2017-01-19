# HTML Strip Char字符过滤器

`html_strip`字符过滤器去除文本中的HTML元素，将HTML实体替换为解码后的值（比如替换`&amp;`为`&`）。

## 示例输出

```
curl -XPOST 'localhost:9200/_analyze?pretty' -d'
{
  "tokenizer":      "keyword", 	// 1
  "char_filter":  [ "html_strip" ],
  "text": "<p>I&apos;m so <b>happy</b>!</p>"
}
'
```

- 1 `keyword`分词器返回单个分词

返回以下分词：

```
[ \nI'm so happy!\n ]
```

如果使用标准分词器，返回以下分词：

```
[ I'm, so, happy ]
```

## 配置

`html_strip`字符过滤器接受以下参数：

|参数|说明|
|---|----|
|escaped_tags|HTML标签列表，这些标签不应从原始文档中去除|

## 示例配置

该例子中，我们配置`html_strip`字符过滤器来保留`<b>`标签。

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "keyword",
          "char_filter": ["my_char_filter"]
        }
      },
      "char_filter": {
        "my_char_filter": {
          "type": "html_strip",
          "escaped_tags": ["b"]
        }
      }
    }
  }
}
'
curl -XPOST 'localhost:9200/my_index/_analyze?pretty' -d'
{
  "analyzer": "my_analyzer",
  "text": "<p>I&apos;m so <b>happy</b>!</p>"
}
'
```

产生以下分词：

```
[ \nI'm so <b>happy</b>!\n ]
```
