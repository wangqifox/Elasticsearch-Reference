# 空白符分词器

空白符分词器在空白符处分割文本。

## 示例输出

```
curl -XPOST 'localhost:9200/_analyze?pretty' -d'
{
  "tokenizer": "whitespace",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog\u0027s bone."
}
'
```

产生以下输出：

```
[ The, 2, QUICK, Brown-Foxes, jumped, over, the, lazy, dog's, bone. ]
```

## 配置

空白符分词器不可配置
