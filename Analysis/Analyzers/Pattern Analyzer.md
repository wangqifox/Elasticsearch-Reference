# 模式分析器

模式分析器使用正则表达式对文本进行分词。注意正则表达式匹配的是分隔符而不是分词本身。正则表达式默认是`\W+`（所有非单词字符）。

> 防止病态的正则表达式。模式分析器使用`Java Regular Expressions`。糟糕的正则表达式书写会造成运行缓慢甚至栈溢出错误，导致正在运行的节点突然退出。

## 定义

它由以下组件构成：

- 分词器
	+ Pattern Tokenizer
- 分词过滤器
	+ Lower Case Token Filter
	+ Stop Token Filter（默认禁用）

## 示例输出

```
curl -XPOST 'localhost:9200/_analyze?pretty' -d'
{
  "analyzer": "pattern",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog\u0027s bone."
}
'
```

产生以下分词：

```
[ the, 2, quick, brown, foxes, jumped, over, the, lazy, dog, s, bone ]
```

## 配置

模式分析器接受以下参数：

|参数|说明|
|---|---|
|pattern|Java正则表达式，默认是`\W+`|
|flags|Java正则表达式标志。标志通过通道符号分隔，如"CASE_INSENSITIVE|COMMENTS"|
|lowercase|分词是否转化为小写。默认为`true`|
|stopwords|预定义的停词列表，比如`_english_`或者是包含一系列停词的数组。默认是`_none_`|
|stopwords_path|停词文件的路径|

## 配置示例

在该示例中，我们配置模式分析器在非单词字符或者下划线中切分文本，再将结果转变为小写。

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_email_analyzer": {
          "type":      "pattern",
          "pattern":   "\\W|_", 	// 1
          "lowercase": true
        }
      }
    }
  }
}
'
curl -XPOST 'localhost:9200/my_index/_analyze?pretty' -d'
{
  "analyzer": "my_email_analyzer",
  "text": "John_Smith@foo-bar.com"
}
'
```

- 1 模式中的反斜杠需要被转义

产生以下的分词

```
[ john, smith, foo, bar, com ]
```

## 驼峰式分词器

以下是一个更复杂的示例对驼峰文本分词：

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "settings": {
    "analysis": {
      "analyzer": {
        "camel": {
          "type": "pattern",
          "pattern": "([^\\p{L}\\d]+)|(?<=\\D)(?=\\d)|(?<=\\d)(?=\\D)|(?<=[\\p{L}&&[^\\p{Lu}]])(?=\\p{Lu})|(?<=\\p{Lu})(?=\\p{Lu}[\\p{L}&&[^\\p{Lu}]])"
        }
      }
    }
  }
}
'
curl -XGET 'localhost:9200/my_index/_analyze?pretty' -d'
{
  "analyzer": "camel",
  "text": "MooseX::FTPClass2_beta"
}
'
```

产生以下分词：

```
[ moose, x, ftp, class, 2, beta ]
```

上面正则表达式的说明:

```
  ([^\p{L}\d]+)                 # swallow non letters and numbers,
| (?<=\D)(?=\d)                 # or non-number followed by number,
| (?<=\d)(?=\D)                 # or number followed by non-number,
| (?<=[ \p{L} && [^\p{Lu}]])    # or lower case
  (?=\p{Lu})                    #   followed by upper case,
| (?<=\p{Lu})                   # or upper case
  (?=\p{Lu}                     #   followed by upper case
    [\p{L}&&[^\p{Lu}]]          #   then lower case
  )
 ```
