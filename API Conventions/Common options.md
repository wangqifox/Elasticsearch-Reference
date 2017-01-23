# Common options

以下的选项所有的REST APIs都适用。

## Pretty Results

在任意请求中附加`?pretty=true`，返回的JSON会被格式化（只用于debug）。另一个选项是设置`?format=yaml`，结果会以更具可读性的yaml格式返回。

## Human readable output

统计信息会以更适合人类（比如"exists_time":"1h"或"size":"1kb"）和更适合计算机（比如"exists_time_in_millis":3600000或"size_in_bytes":1024）的格式返回。可以在查询字符串中增加`?human=false`来关闭人类可读值。当统计结果传给监控工具使用而不是给用户使用时是有意义的。`human`标签的默认值是`false`。

## Date Math

大多数接受格式化日期值的参数都理解`date math`。比如范围查询中的`gt`和`lt`，或者日期范围聚合中的`from`和`to`。

表达式以一个锚日期起始，它可以是一个`now`或者以`||`结尾的日期字符串。锚日期可以跟一个或更多的数学表达式：

- `+1h`增加一个小时
- `-1d`减去一天
- `/d`向下舍入到最近的一天

支持的时间单位和持续时间的时间单位不同：

|单位|说明|
|---|---|
|y|年|
|M|月|
|w|周|
|d|天|
|h|小时|
|H|小时|
|m|分钟|
|s|秒|

示例：

|表达式|解释|
|-----|---|
|`now+1h`|当前时间加一小时，精确到毫秒|
|`now+1h+1m`|当前时间加一小时一分钟，精确到毫秒|
|`now+1h/d`|当前时间加一小时，向下舍入到最近的一天|
|`2015-01-01||+1M/d`|`2015-01-01`加一个月，向下舍入到最近的一天|

## Response Filtering（返回结果过滤）

所有的REST API接受`filter_path`参数，用来减少elasticsearch返回的值。`filter_path`参数的值以`.`来表示，用`,`分隔：

```
curl -XGET 'localhost:9200/_search?q=elasticsearch&filter_path=took,hits.hits._id,hits.hits._score&pretty'
```

返回值：

```
{
  "took" : 3,
  "hits" : {
    "hits" : [
      {
        "_id" : "0",
        "_score" : 1.6375021
      }
    ]
  }
}
```

也支持`*`通配符来匹配任意字段或者字段名字的一部分：

```
curl -XGET 'localhost:9200/_cluster/state?filter_path=metadata.indices.*.stat*&pretty'
```

返回值：

```
{
  "metadata" : {
    "indices" : {
      "twitter": {"state": "open"}
    }
  }
}
```

`**`通配符用来包含不知道确切路径的字段。比如，该请求返回每个段的Lucene版本：

```
curl -XGET 'localhost:9200/_cluster/state?filter_path=routing_table.indices.**.state&pretty'
```

返回值：

```
{
  "routing_table": {
    "indices": {
      "twitter": {
        "shards": {
          "0": [{"state": "STARTED"}, {"state": "UNASSIGNED"}],
          "1": [{"state": "STARTED"}, {"state": "UNASSIGNED"}],
          "2": [{"state": "STARTED"}, {"state": "UNASSIGNED"}],
          "3": [{"state": "STARTED"}, {"state": "UNASSIGNED"}],
          "4": [{"state": "STARTED"}, {"state": "UNASSIGNED"}]
        }
      }
    }
  }
}
```

也可以在开头加上`-`来排除一个或多个字段：

```
curl -XGET 'localhost:9200/_count?filter_path=-_shards&pretty'
```

返回值：

```
{
  "count" : 5
}
```

包含和排除过滤器可以组合在同一个表达式中。首先执行排除过滤器，排除后的结果再使用包含过滤器过滤：

```
curl -XGET 'localhost:9200/_cluster/state?filter_path=metadata.indices.*.state,-metadata.indices.logstash-*&pretty'
```

返回值：

```
{
  "metadata" : {
    "indices" : {
      "index-1" : {"state" : "open"},
      "index-2" : {"state" : "open"},
      "index-3" : {"state" : "open"}
    }
  }
}
```

注意，elasticsearch有时会直接返回字段的原始值，比如`_source`字段。如果想要过滤`_source`字段，应该将`_source`参数和`filter_path`参数结合：

```
curl -XPOST 'localhost:9200/library/book?refresh&pretty' -d'
{"title": "Book #1", "rating": 200.1}
'
curl -XPOST 'localhost:9200/library/book?refresh&pretty' -d'
{"title": "Book #2", "rating": 1.7}
'
curl -XPOST 'localhost:9200/library/book?refresh&pretty' -d'
{"title": "Book #3", "rating": 0.1}
'
curl -XGET 'localhost:9200/_search?filter_path=hits.hits._source&_source=title&sort=rating:desc&pretty'
```

返回值：

```
{
  "hits" : {
    "hits" : [ {
      "_source":{"title":"Book #1"}
    }, {
      "_source":{"title":"Book #2"}
    }, {
      "_source":{"title":"Book #3"}
    } ]
  }
}
```

## Flat Settings（扁平化设置）

`flat_settings`标识影响设置列表的呈现。当`flat_settings`标识为`true`时，设置以扁平的格式返回：

```
curl -XGET 'localhost:9200/twitter/_settings?flat_settings=true&pretty'
```

返回：

```
{
  "twitter" : {
    "settings": {
      "index.number_of_replicas": "1",
      "index.number_of_shards": "1",
      "index.creation_date": "1474389951325",
      "index.uuid": "n6gzFZTgS664GUfx0Xrpjw",
      "index.version.created": ...,
      "index.provided_name" : "twitter"
    }
  }
}
```

当`flat_settings`标识设置为`false`时，设置以人类更可读的结构化格式返回：

```
curl -XGET 'localhost:9200/twitter/_settings?flat_settings=false&pretty'
```

返回值：

```
{
  "twitter" : {
    "settings" : {
      "index" : {
        "number_of_replicas": "1",
        "number_of_shards": "1",
        "creation_date": "1474389951325",
        "uuid": "n6gzFZTgS664GUfx0Xrpjw",
        "version": {
          "created": ...
        },
        "provided_name" : "twitter"
      }
    }
  }
}
```

`flat_settings`默认为`false`

## Parameters（参数）

Rest参数（当使用HTTP，映射到HTTP URL参数）遵循使用下划线的惯例。

## Boolean Values（布尔值）

所有的REST API参数（请求参数和JSON体）中布尔值"false"可以为以下值：`false`,`0`,`no`,`off`。所有其他的值都被认为是`true`。注意，这和文档中当做布尔字段来索引的字段无关。

## Number Values（数值）

所有的REST API支持以字符串的形式来提供数值参数，以支持本地JSON数值类型。

## Time units（时间单位）

当需要指定持续时间时，比如`timeout`参数，持续时间必须带单位，像`2d`表示两天。支持的单位：

|单位|说明|
|---|---|
|d|天|
|h|小时|
|m|分钟|
|s|秒|
|ms|毫秒|
|micros|微秒|
|nanos|纳秒|

## Byte size units（字节大小单位）

当需要指定数据的字节大小时，比如设置缓存大小参数，必须指定单位，比如`10kb`代表10千字节。支持的单位有：

|单位|说明|
|---|----|
|b|字节|
|kb|千字节|
|mb|兆字节|
|gb|吉字节|
|tb|太字节|
|pb|拍字节|

## Unit-less quantities（没有单位的数量）

没有单位的数量意味着它们没有`bytes`、`Hertz`、`meter`、`long time`这样的单位。

如果数量非常巨大，我们可以将`10000000`表示为`10m`或者将`7000`表示为`7k`。仍然使用`87`来表示`87`。以下是支持的单位：

|单位|说明|
|---|----|
|``|单独|
|`k`|千|
|`m`|兆|
|`g`|千兆|
|`t`|兆兆|
|`p`|千兆兆|

## Distance Units（距离单位）

当需要指定距离时，比如`Geo Distance`查询中的`distance`参数，如果不指定的话，默认的单位是米。距离可以使用其他距离来指定，比如`1km`或`2mi`（2英里）。

距离单位列表：

1英里 = 5,280英尺 = 63,360英寸 ≈ 1609.344米 ≈ 1.609344公里

|说明|单位|
|---|----|
|`mile`英里|`mi`或`miles`|
|`yard`码（3英尺）|`yd`或`yards`|
|`feet`英尺|`ft`或`feet`|
|`inch`英寸|`in`或`inch`|
|`kilometer`千米|`km`或`kilometers`|
|`meter`米|`m`或`meters`|
|`centimeter`厘米|`cm`或`centimeters`|
|`millimeter`毫米|`mm`或`millimeters`|
|`nautical mile`纳米|`NM`,`nmi`或`nauticalmiles`|

## Fuzziness

一些查询或者API支持参数以允许使用模糊性参数进行不精确的模糊匹配。

当查询`text`或者`keyword`字段时，`fuzziness`解释为`levenshtein`编辑距离——将一个字符串转变为另一个字符串需要改变的字符数量。

`fuzziness`参数可以指定为以下值：

- 0, 1, 2
    + 最大允许的编辑距离
- AUTO
    + 根据分词的长度生成编辑距离。
        * 0..2 : 必须精确比配
        * 3..5 : 编辑距离为1
        * >5 : 编辑距离为2
    + AUTO是`fuzziness`的首选值

## Enabling stack traces（开启栈追踪）

默认情况下，如果请求返回了错误，Elasticsearch不会包含错误的栈追踪。可以将`error_trace`参数设置为`true`来开启错误栈追踪。比如，在`_search`API中传入无效的`size`参数：

```
curl -XPOST 'localhost:9200/twitter/_search?size=surprise_me&pretty'
```

返回值如下所示：

```
{
  "error" : {
    "root_cause" : [
      {
        "type" : "illegal_argument_exception",
        "reason" : "Failed to parse int parameter [size] with value [surprise_me]"
      }
    ],
    "type" : "illegal_argument_exception",
    "reason" : "Failed to parse int parameter [size] with value [surprise_me]",
    "caused_by" : {
      "type" : "number_format_exception",
      "reason" : "For input string: \"surprise_me\""
    }
  },
  "status" : 400
}
```

如果设置了`error_trace=true`：

```
curl -XPOST 'localhost:9200/twitter/_search?size=surprise_me&error_trace=true&pretty'
```

返回值如下所示：

```
{
  "error": {
    "root_cause": [
      {
        "type": "illegal_argument_exception",
        "reason": "Failed to parse int parameter [size] with value [surprise_me]",
        "stack_trace": "Failed to parse int parameter [size] with value [surprise_me]]; nested: IllegalArgumentException..."
      }
    ],
    "type": "illegal_argument_exception",
    "reason": "Failed to parse int parameter [size] with value [surprise_me]",
    "stack_trace": "java.lang.IllegalArgumentException: Failed to parse int parameter [size] with value [surprise_me]\n    at org.elasticsearch.rest.RestRequest.paramAsInt(RestRequest.java:175)...",
    "caused_by": {
      "type": "number_format_exception",
      "reason": "For input string: \"surprise_me\"",
      "stack_trace": "java.lang.NumberFormatException: For input string: \"surprise_me\"\n    at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)..."
    }
  },
  "status": 400
}
```

## Request body in query string（查询字符串中的请求体）

对于非POST请求不接受请求体的库来说，你可以将请求正文作为源查询字符串参数传递。
