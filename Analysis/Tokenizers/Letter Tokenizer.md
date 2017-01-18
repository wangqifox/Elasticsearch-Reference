# 字母分词器

字母分词器在非字母处分割文本。该分词器对于大多数的欧洲语言是合理的，但是对于不使用空格来分隔的亚洲语言来说是很糟糕的。

## 示例输出

```
curl -XPOST 'localhost:9200/_analyze?pretty' -d'
{
  "tokenizer": "letter",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog\u0027s bone."
}
'
```

产生以下分词：

```
[ The, QUICK, Brown, Foxes, jumped, over, the, lazy, dog, s, bone ]
```

## 配置

字母分词器不可配置
