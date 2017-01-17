# 指纹分析器

指纹分析器实现了`fingerprinting algorithm`，该算法在OpenRefine项目中使用，以辅助聚类。

输入本文转化为小写，规范化以移除扩展字符，排序，删除重复数据，连接成单个分词。如果配置了停词列表，停词也会被删除。

## 定义

它由以下组件构成：

- 分词器
	+ Standard Tokenizer
- 分词过滤器
	+ Lower Case Token Filter
	+ ASCII Folding Token Filter
	+ Stop Token Filter(默认关闭)
	+ Fingerprint Token Filter

## 示例输出

```
curl -XPOST 'localhost:9200/_analyze?pretty' -d'
{
  "analyzer": "fingerprint",
  "text": "Yes yes, Gödel said this sentence is consistent and."
}
'
```

产生以下单个分词：

```
[ and consistent godel is said sentence this yes ]
```

## 配置

指纹分析器接受以下参数：

|separator|用来接连分词的字符。默认是空格|
|max_output_size|分词最大的长度。默认是255。长度大于该值的分词会被丢弃。|
|stopwords||
|||

