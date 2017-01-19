# Pattern Replace 字符过滤器

`pattern_relace`字符过滤器使用正则表达式来匹配字符串，将匹配到的字符串替换为指定的字符串。替换字符串可以指定为正则表达式中的捕获组。

> 防止病态的正则表达式。模式分析器使用`Java Regular Expressions`。糟糕的正则表达式书写会造成运行缓慢甚至栈溢出错误，导致正在运行的节点突然退出。

## 配置

`pattern_replace`字符过滤器接受以下参数：

|参数|说明|
|---|----|
|pattern|Java正则表达式（必须）|
|replacement|替换字符串。可以引用捕获组，以`$1...$9`这样的语法。|
|flags|Java正则表达式标识。以管道符(|)分隔，比如"CASE_INSENSITIVE|COMMENTS"|

## 示例配置

该例子中，我们配置`pattern_replace`字符过滤器来将数字中嵌入的划线(dash)替换成下划线，比如`123-456-789`->`123_456_789`。

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "standard",
          "char_filter": [
            "my_char_filter"
          ]
        }
      },
      "char_filter": {
        "my_char_filter": {
          "type": "pattern_replace",
          "pattern": "(\\d+)-(?=\\d)",
          "replacement": "$1_"
        }
      }
    }
  }
}
'
curl -XPOST 'localhost:9200/my_index/_analyze?pretty' -d'
{
  "analyzer": "my_analyzer",
  "text": "My credit card is 123-456-789"
}
'
```

产生以下分词：

```
[ My, credit, card, is 123_456_789 ]
```

> 替换字符串改变了原始文本的长度，对于搜索来说没问题，但是对高亮来说是错误的。下面的例子可以看到。

以下的例子在小写字母后跟大写字母的位置插入空格（比如`fooBarBaz`->`foo Bar Baz`），使驼峰形式的单词可以单独查询。

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "standard",
          "char_filter": [
            "my_char_filter"
          ],
          "filter": [
            "lowercase"
          ]
        }
      },
      "char_filter": {
        "my_char_filter": {
          "type": "pattern_replace",
          "pattern": "(?<=\\p{Lower})(?=\\p{Upper})",
          "replacement": " "
        }
      }
    }
  },
  "mappings": {
    "my_type": {
      "properties": {
        "text": {
          "type": "text",
          "analyzer": "my_analyzer"
        }
      }
    }
  }
}
'
curl -XPOST 'localhost:9200/my_index/_analyze?pretty' -d'
{
  "analyzer": "my_analyzer",
  "text": "The fooBarBaz method"
}
'
```

产生以下分词：

```
[ the, foo, bar, baz, method ]
```

查询`bar`可以正确找到文档，但是高亮查询结果会有错误，因为字符过滤器改变了原始文本的长度。

```
curl -XPUT 'localhost:9200/my_index/my_doc/1?refresh&pretty' -d'
{
  "text": "The fooBarBaz method"
}
'
curl -XGET 'localhost:9200/my_index/_search?pretty' -d'
{
  "query": {
    "match": {
      "text": "bar"
    }
  },
  "highlight": {
    "fields": {
      "text": {}
    }
  }
}
'
```

查询结果如下所示：

```
{
  "timed_out": false,
  "took": $body.took,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.2824934,
    "hits": [
      {
        "_index": "my_index",
        "_type": "my_doc",
        "_id": "1",
        "_score": 0.2824934,
        "_source": {
          "text": "The fooBarBaz method"
        },
        "highlight": {
          "text": [
            "The foo<em>Ba</em>rBaz method" 	// 1
          ]
        }
      }
    ]
  }
}
```

- 1 注意高亮的位置有误
