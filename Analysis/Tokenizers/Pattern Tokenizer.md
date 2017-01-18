# 模式分词器

模式分词器使用正则表达式将文本分割为分词，当遇见匹配的单词分隔符，或者捕获匹配的文本作为分词。

默认的模式是`\W+`，在非单词字符处分割文本。

> 防止病态的正则表达式。模式分析器使用`Java Regular Expressions`。糟糕的正则表达式书写会造成运行缓慢甚至栈溢出错误，导致正在运行的节点突然退出。

## 示例输出

```
curl -XPOST 'localhost:9200/_analyze?pretty' -d'
{
  "tokenizer": "pattern",
  "text": "The foo_bar_size's default is 5."
}
'
```

产生以下分词：

```
[ The, foo_bar_size, s, default, is, 5 ]
```

## 配置

模式分词器接受以下参数：

|参数|说明|
|---|----|
|pattern|Java正则表达式，默认是`\W+`|
|flags|Java正则表达式标识。以管道符(|)分隔，比如"CASE_INSENSITIVE|COMMENTS"|
|group|哪个捕获的组提取为分词。默认是-1|

## 示例配置

该示例中，我们将模式分词器配置为在逗号(,)处分割文本。

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
          "type": "pattern",
          "pattern": ","
        }
      }
    }
  }
}
'
curl -XPOST 'localhost:9200/my_index/_analyze?pretty' -d'
{
  "analyzer": "my_analyzer",
  "text": "comma,separated,values"
}
'
```

产生以下分词：

```
[ comma, separated, values ]
```

接下去的例子中，我们将模式分词器配置为捕获双引号内的值（忽略嵌入的转义引号）。正则表达式为：

```
"((?:\\"|[^"]|\\")*)"
```

读取方式如下：

- 一个双引号`"`
- 开始捕获
	+ 一个`\"`或者除了`"`的任意字符
	+ 重复直到没有字符匹配
- 另一个闭合的双引号`"`

由于正则表达式是在JSON中指定，`"`和`\`需要被转义，所以最后的表达式如下所示：

```
\"((?:\\\\\"|[^\"]|\\\\\")+)\"
```

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
          "type": "pattern",
          "pattern": "\"((?:\\\\\"|[^\"]|\\\\\")+)\"",
          "group": 1
        }
      }
    }
  }
}
'
curl -XPOST 'localhost:9200/my_index/_analyze?pretty' -d'
{
  "analyzer": "my_analyzer",
  "text": "\"value\", \"value with embedded \\\" quote\""
}
'
```

产生以下的分词：

```
[ value, value with embedded \" quote ]
```
