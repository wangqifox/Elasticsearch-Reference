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


