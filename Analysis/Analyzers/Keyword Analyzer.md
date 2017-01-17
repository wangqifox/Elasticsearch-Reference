# 关键词分析器

关键词分析器是一个“空”分析器，将整个字符串作为单个分词。

## 定义

它由以下组件构成：

- 分词器
	+ Keyword Tokenizer

## 示例输出

```
curl -XPOST 'localhost:9200/_analyze?pretty' -d'
{
  "analyzer": "keyword",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog\u0027s bone."
}
'
```

产生以下分词：

```
[ The 2 QUICK Brown-Foxes jumped over the lazy dog's bone. ]
```

## 配置

关键词分析器不可配置
