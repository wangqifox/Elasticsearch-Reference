# 路径层级分词

`path_hierarchy`分词器将层级值当做像文件系统路径一样来处理，在路径分隔符处分割文本，并为树中的每个组件生成一个分词。

## 示例输出

```
curl -XPOST 'localhost:9200/_analyze?pretty' -d'
{
  "tokenizer": "path_hierarchy",
  "text": "/one/two/three"
}
'
```

产生以下分词：

```
[ /one, /one/two, /one/two/three ]
```

## 配置

`path_hierarchy`分词器接受以下参数：

|参数|说明|
|---|---|
|delimiter|作为路径分隔符的字符。默认是`/`|
|replacement|用作分隔符的可选替换字符。默认为delimiter|
|buffer_size|单次传递中，读入分词缓存的字符数量。默认是1024。分词缓存会按该大小增长直到所有的文本都被消耗。建议不用改变这个设置。|
|reverse|如果设置为`true`，逆转分词的顺序。默认是`false`|
|skip|跳过初始分词的数量。默认是0|

## 示例配置

该例子中，我们配置`path_hierarchy`分词器，在`-`上分割文本，将`-`替换为`/`。跳过起始的两个分词：

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "my_tokenizer"
        }
      },
      "tokenizer": {
        "my_tokenizer": {
          "type": "path_hierarchy",
          "delimiter": "-",
          "replacement": "/",
          "skip": 2
        }
      }
    }
  }
}
'
curl -XPOST 'localhost:9200/my_index/_analyze?pretty' -d'
{
  "analyzer": "my_analyzer",
  "text": "one-two-three-four-five"
}
'
```

产生以下的分词：

```
[ /three, /three/four, /three/four/five ]
```

如果将`reverse`设置为`true`，或产生以下的分词：

```
[ one/two/three/, two/three/, three/ ]
```
