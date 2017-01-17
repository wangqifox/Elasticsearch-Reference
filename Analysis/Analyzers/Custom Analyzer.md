# 自定义分析器

当内建的分析器无法满足需要时，可以创建一个自定义的分析器，包含以下内容：

- 0个或多个字符过滤器
- 1个分词器
- 0个或多个分词过滤器

## 配置

自定义的分析器接受以下参数：

|参数|说明|
|---|----|
|tokenizer|内建或自定义的分词器（必须）|
|char_filter|内建或自定义的字符过滤器数组（可选）|
|filter|内建或自定义的分词过滤器数组（可选）|
|position_increment_gap|当索引一个文本数组时，Elasticsearch在一个数组的最后一个分词和下一个数组的第一个分词之间插入一个伪间隔，以保证短语查询不会匹配到不同数组元素中的两个分词。默认是`100`。|

## 配置示例

该示例组合以下的组件：

- 字符过滤器
	- HTML Strip Character Filter
- 分词器
	- Standard Tokenizer
- 分词过滤器
	- Lowercase Token Filter
	- ASCII-Folding Token Filter

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "type":      "custom",
          "tokenizer": "standard",
          "char_filter": [
            "html_strip"
          ],
          "filter": [
            "lowercase",
            "asciifolding"
          ]
        }
      }
    }
  }
}
'
curl -XPOST 'localhost:9200/my_index/_analyze?pretty' -d'
{
  "analyzer": "my_custom_analyzer",
  "text": "Is this <b>déjà vu</b>?"
}
'
```

例子产生以下的分词：

```
[ is, this, deja, vu ]
```

上面的例子使用默认配置的分词器、分词过滤器、字符过滤器，可以创建每个配置的版本，在自定义分析器中使用。

以下是一个更复杂的例子，组合以下的组件：

- 字符过滤器
	- Mapping Character Filter，使用`_happy_`代替`:)`、`_sad_`代替`:(`
- 分词器
	- Pattern Tokenizer，在标点符号处分割
- 分词过滤器
	- Lowercase Token Filter
	- Stop Token Filter，使用预定义的英文停词

示例：

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "char_filter": [
            "emoticons" 	// 1
          ],
          "tokenizer": "punctuation", 	// 2
          "filter": [
            "lowercase",
            "english_stop" 	// 3
          ]
        }
      },
      "tokenizer": {
        "punctuation": { 	// 4
          "type": "pattern",
          "pattern": "[ .,!?]"
        }
      },
      "char_filter": {
        "emoticons": { 	// 5
          "type": "mapping",
          "mappings": [
            ":) => _happy_",
            ":( => _sad_"
          ]
        }
      },
      "filter": {
        "english_stop": { 	// 6
          "type": "stop",
          "stopwords": "_english_"
        }
      }
    }
  }
}
'
curl -XPOST 'localhost:9200/my_index/_analyze?pretty' -d'
{
  "analyzer": "my_custom_analyzer",
  "text":     "I'm a :) person, and you?"
}
'
```

- 1 2 3 4 5 6 `emotion`字符过滤器、`punctuation`分词器、`english_stop`分词过滤器在同一个索引设置中定义。

以上例子产生以下的分词:

```
[ i'm, _happy_, person, you ]
```

