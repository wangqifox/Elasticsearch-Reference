# 停词分析器

停词分析器和简单分析器一样，但是它支持删除停词。默认使用`_english_`停词。

## 定义

由以下组件组成：

- 分词器
	- Lower Case Tokenizer
- 分词过滤器
	- Stop Token Filter

## 示例输出

```
curl -XPOST 'localhost:9200/_analyze?pretty' -d'
{
  "analyzer": "stop",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog\u0027s bone."
}
'
```

产生以下分词：

```
[ quick, brown, foxes, jumped, over, lazy, dog, s, bone ]
```

## 配置

停词分析器接受以下参数：

|参数|说明|
|---|----|
|stopwords|预定义的停词比如`_english_`，或者包含一系列停词的数组。默认是`_english_`|
|stopwords_path|停词文件的路径|

查看`Stop Token Filter`获取更多停词配置的信息

## 示例配置

在该例子中，我们使用一系列单词作为停词列表来配置停词分析器：

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_stop_analyzer": {
          "type": "stop",
          "stopwords": ["the", "over"]
        }
      }
    }
  }
}
'
curl -XPOST 'localhost:9200/my_index/_analyze?pretty' -d'
{
  "analyzer": "my_stop_analyzer",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog\u0027s bone."
}
'
```

产生以下分词：

```
[ quick, brown, foxes, jumped, lazy, dog, s, bone ]
```
