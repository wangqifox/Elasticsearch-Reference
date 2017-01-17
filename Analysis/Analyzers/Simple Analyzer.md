# 简单分析器

简单分析器在非字母字符处将文本拆分为分词。所有分词转化为小写字母。

## 定义

由以下组件构成：

- 分词器
	- Lower Case Tokenizer

## 示例输出

```
curl -XPOST 'localhost:9200/_analyze?pretty' -d'
{
  "analyzer": "simple",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog\u0027s bone."
}
'
```

产生以下分词：

```
[ the, quick, brown, foxes, jumped, over, the, lazy, dog, s, bone ]
```

## 配置

简单分析器不可配置
