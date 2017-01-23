# Common options

以下的选项所有的REST APIs都适用。

## Pretty Results

## Human readable output

## Date Math

大多数接受格式化日期值的参数都理解`date math`。比如范围查询中的`gt`和`lt`，或者日期范围聚合中的`from`和`to`。

表达式以一个锚日期起始，它可以是一个`now`或者以`||`结尾的日期字符串。锚日期可以跟一个或更多的数学表达式：

- `+1h`增加一个小时
- `-1d`减去一天
- `/d`向下舍入到最近的天

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
