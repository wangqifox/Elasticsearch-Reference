# `_all`字段

`_all`是一个特殊的全量字段，它把其他所有字段的值连接起来组成一个大字符串，使用空格作为分隔符，然后索引该字符串但是不保存原始值。这意味着该字段只能用于搜索而不能用于检索。

`_all`字段允许你在不知道哪个字段包含搜索值的情况下搜索文档。一开始处理新的数据集时这个字段会很有用。

```
curl -XPUT 'localhost:9200/my_index/user/1 ?pretty' -d'	// 1
{
  "first_name":    "John",
  "last_name":     "Smith",
  "date_of_birth": "1970-10-24"
}'
curl -XGET 'localhost:9200/my_index/_search?pretty' -d'
{
  "query": {
    "match": {
      "_all": "john smith 1970"
    }
  }
}'
```

- 1 `_all`字段包含分词：["john", "smith", "1970", "10", "24"]

> 所有的值都被当做字符串
> 上面的例子中`date_of_birth`字段识别为日期字段，以`1970-10-24 00:00:00 UTC`的形式索引。但是`_all`字段中把所有的值都当做字符串，所以日期以三个分词："1970","24","10"来索引。
> 需要注意的是，`_all`字段将其他字段的原始值整合为一个字符串，而不是整合每个字段的分词。

`_all`字段是一个`text`字段，接受和其他字符串字段一样的参数，包括`analyzer`, `term_vectors`, `index_options`, `store`。

`_all`字段在使用简单的过滤器来搜索新数据时尤其有用。但是把字段值都整合成一个大的字符串丢失了短字段（相关性更高）和长字段（相关性更低）的差别。如果相关性的重要性很高，最好使用单个字段的查询。

使用`_all`字段并不是没有代价的：它需要消耗额外的CPU和磁盘空间。如果不需要这个功能，可以将它完全关闭，或者指定某些字段不包含在`_all`中。

## 查询中使用`_all`字段

`query_string`和`simple_query_string`默认查询`_all`字段，除非指定了另外的字段。

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
  "query": {
    "query_string": {
      "query": "john smith 1970"
    }
  }
}'
```

该查询和`URI search requests`使用`?q`参数一样（内部被重写为`query_string`查询）：

```
GET _search?q=john+smith+1970
```

其他查询，比如`match`和`term`查询需要显式指定`_all`字段。

## 关闭`_all`字段

每个类型都可以通过设置`enabled`为`false`来关闭`_all`字段：

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "mappings": {
    "type_1": { 	// 1
      "properties": {...}
    },
    "type_2": { 	// 2
      "_all": {
        "enabled": false
      },
      "properties": {...}
    }
  }
}'
```

- 1 `type_1`中`_all`字段开启
- 2 `type_2`中`_all`字段关闭

如果`_all`字段是关闭的，`URI`、`query_string`、`simple_query_string`查询就不能使用`_all`字段来查询了。你可以设置`index.query.default_field`来使用不用的字段。

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "mappings": {
    "my_type": {
      "_all": {
        "enabled": false 	// 1
      },
      "properties": {
        "content": {
          "type": "text"
        }
      }
    }
  },
  "settings": {
    "index.query.default_field": "content" 	// 2
  }
}'
```

- 1 `my_type`类型中`_all`字段被关闭
- 2 `query_string`会查询`content`字段

## 从`_all`中排除字段

使用`include_in_all`可以设置单个字段在`_all`字段中包含或者从`_all`字段中排除。

## 索引boost和`_all`字段

可以使用`boost`参数在索引阶段boost单个字段。`_all`字段会考虑这些boost：

```
curl -XPUT 'localhost:9200/myindex?pretty' -d'
{
  "mappings": {
    "mytype": {
      "properties": {
        "title": { 	// 1
          "type": "text",
          "boost": 2
        },
        "content": { 	// 2
          "type": "text"
        }
      }
    }
  }
}'
```

- 1 2 当查询`_all`字段时，来源于`title`字段的单词相关性是来源于`content`字段的单词的两倍。

> `_all`字段上使用索引阶段的boost会对查询性能造成显著影响。通常更好地解决办法是在查询时使用boost

## 自定义`_all`字段

每个索引中只有一个单独的`_all`字段，使用`copy_to`参数可以创建多个自定义的`_all`字段。比如，`first_name`和`last_name`可以组合成`full_name`字段。

```
curl -XPUT 'localhost:9200/myindex?pretty' -d'
{
  "mappings": {
    "mytype": {
      "properties": {
        "first_name": {
          "type":    "text",
          "copy_to": "full_name" 	// 1
        },
        "last_name": {
          "type":    "text",
          "copy_to": "full_name" 	// 2
        },
        "full_name": {
          "type":    "text"
        }
      }
    }
  }
}'
curl -XPUT 'localhost:9200/myindex/mytype/1?pretty' -d'
{
  "first_name": "John",
  "last_name": "Smith"
}'
curl -XGET 'localhost:9200/myindex/_search?pretty' -d'
{
  "query": {
    "match": {
      "full_name": "John Smith"
    }
  }
}'
```

- 1 2 `first_name`和`last_name`的值被复制到`full_name`字段

## 高亮和`_all`字段

如果原始字符串值可用，则字段只能用于高亮显示，无论是从`_source`字段还是作为存储字段。

`_all`字段不存在与`_source`字段中，默认情况下不存储，因此不能高亮显示。有两个解决办法，存储`_all`字段或者高亮显示原始字段。

### 存储`_all`字段

如果`store`被设置为`true`，原始字段值就可以被检索以及高亮显示。

```
curl -XPUT 'localhost:9200/myindex?pretty' -d'
{
  "mappings": {
    "mytype": {
      "_all": {
        "store": true
      }
    }
  }
}'
curl -XPUT 'localhost:9200/myindex/mytype/1?pretty' -d'
{
  "first_name": "John",
  "last_name": "Smith"
}'
curl -XGET 'localhost:9200/_search?pretty' -d'
{
  "query": {
    "match": {
      "_all": "John Smith"
    }
  },
  "highlight": {
    "fields": {
      "_all": {}
    }
  }
}'
```

当然，存储`_all`字段会造成显著的磁盘空间占用，并且由于`_all`字段是其他字段的组合，可能会导致奇怪的高亮结果。

`_all`字段也接受`term_vector`和`index_options`参数，可以使用`fast vector highlighter`和`postings highlighter`。

### 高亮原始字段

你可以查询`_all`字段，但是使用原始字段来做高亮，如下所示：

```
curl -XPUT 'localhost:9200/myindex?pretty' -d'
{
  "mappings": {
    "mytype": {
      "_all": {}
    }
  }
}'
curl -XPUT 'localhost:9200/myindex/mytype/1?pretty' -d'
{
  "first_name": "John",
  "last_name": "Smith"
}'
curl -XGET 'localhost:9200/_search?pretty' -d'
{
  "query": {
    "match": {
      "_all": "John Smith" 	// 1
    }
  },
  "highlight": {
    "fields": {
      "*_name": { 	// 2
        "require_field_match": false  	// 3
      }
    }
  }
}'
```

- 1 搜索`_all`字段来匹配字段
- 2 在两个名字字段上进行高亮操作，数据可以从`_source`中获得。
- 3 未对名字字段运行查询，所以将`require_field_match`设置为`false`
