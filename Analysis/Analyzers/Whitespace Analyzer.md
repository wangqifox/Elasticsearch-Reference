# 空白符分析器

空白符分析器在空白符处将文本拆分为分词。

## 定义

它由以下组件组成：

- 分词器
	- Whitespace Tokenizer

## 示例输出

```
curl -XPOST 'localhost:9200/_analyze?pretty' -d'
{
  "analyzer": "whitespace",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog\u0027s bone."
}
'
```

产生以下分词：

```
[ The, 2, QUICK, Brown-Foxes, jumped, over, the, lazy, dog's, bone. ]
```

## 配置

空白符分析器不可配置
