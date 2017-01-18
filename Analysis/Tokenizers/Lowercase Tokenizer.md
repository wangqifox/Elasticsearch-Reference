# 小写分词器

小写分词器和字母分词器一样，在非字母处分割文本，但是小写分词器将所有的分词转化为小写。功能上等价于字母分词器和小写分词过滤器的组合，但是效率更高因为它在一个步骤中执行这两步。

## 示例输出

```
curl -XPOST 'localhost:9200/_analyze?pretty' -d'
{
  "tokenizer": "lowercase",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog\u0027s bone."
}
'
```

产生以下分词：

```
[ the, quick, brown, foxes, jumped, over, the, lazy, dog, s, bone ]
```

## 配置

小写分词器不可配置
