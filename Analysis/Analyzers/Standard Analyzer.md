# Standard Analyzer（标准分析器）

如果不指定分析器，则默认使用标准分析器。它提供基于语法的分词器（基于Unicode文本分隔算法）且对于大部分语言都适用。

## 定义

它由以下部分组成：

- Tokenizer（分词器）
	- Standard Tokenizer（标准分词器）
- Token Filters（分词过滤器）
	- Standard Token Filter（标准分词过滤器）
	- Lower Case Token Filter（小写分词过滤器）
	- Stop Token Filter（停词分词过滤器，默认禁用）

## 输出的例子

```
curl -XPOST 'localhost:9200/_analyze?pretty' -d'
{
  "analyzer": "standard",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog\u0027s bone."
}
'
```

上面的句子会产生以下分词：

```
[ the, 2, quick, brown, foxes, jumped, over, the, lazy, dog's, bone ]
```

## 配置

标准分析器接受以下参数：

|参数|说明|
|---|----|
|max_token_length|最大分词长度，如果分词长度超过了这个长度，它会以`max_token_length`的间隔分割。|
|stopwords|预定义的停词，比如`_english_`或者是包含一系列停词的数组。默认是`_none_`|
|stopwords_path|包含停词的文件的路径|

## 配置示例

在这个例子中，我们配置标准分析器，`max_token_length`为5，使用预定义的英文停词。

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_english_analyzer": {
          "type": "standard",
          "max_token_length": 5,
          "stopwords": "_english_"
        }
      }
    }
  }
}
'
curl -XPOST 'localhost:9200/my_index/_analyze?pretty' -d'
{
  "analyzer": "my_english_analyzer",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog\u0027s bone."
}
'
```

上面的例子产生以下的分词：

```
[ 2, quick, brown, foxes, jumpe, d, over, lazy, dog's, bone ]
```
