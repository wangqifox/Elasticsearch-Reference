# 日期类型

JSON没有日期类型，所以Elasticsearch中的日期有一下几种形式：

- 包含格式化日期的字符串，比如:"2015-01-01"或者"2015/01/01 12:10:30"
- 长整型，表示距离epoch(1970 年 1 月 1 日子时格林威治标准时间)的毫秒数
- 整型，表示距离epoch的秒数

实际上，日期被转换成UTC（如果指定了时区），以长整型的形式保存，表示距离epoch的毫秒数

日期格式可以指定，如果没有指定`format`会使用默认的格式：

```
"strict_date_optional_time||epoch_millis"
```

这意味着可以接受可选的时间戳，遵照`strict_date_optional_time`支持的格式或者距离epoch的毫秒数

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "mappings": {
    "my_type": {
      "properties": {
        "date": {
          "type": "date"  // 1
        }
      }
    }
  }
}'
curl -XPUT 'localhost:9200/my_index/my_type/1?pretty' -d'
{ "date": "2015-01-01" } '  // 2
curl -XPUT 'localhost:9200/my_index/my_type/2?pretty' -d'
{ "date": "2015-01-01T12:10:30Z" } '  // 3
curl -XPUT 'localhost:9200/my_index/my_type/3?pretty' -d'
{ "date": 1420070400001 } ' // 4
curl -XGET 'localhost:9200/my_index/_search?pretty' -d'
{
  "sort": { "date": "asc"}  // 5
}'
```

- 1 `date`字段使用默认的日期格式
- 2 文档使用普通的日期
- 3 文档包括时间
- 4 文档使用距离epoch的毫秒数
- 5 排序结果都以距离epoch的毫秒数为格式返回

## 多个日期格式

可以指定多个日期格式，以`||`分隔。索引时，依次尝试每个格式直到找到匹配的格式。查询数据时，使用第一个格式将距离epoch的毫秒数转换为字符串。

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "mappings": {
    "my_type": {
      "properties": {
        "date": {
          "type":   "date",
          "format": "yyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
        }
      }
    }
  }
}'
```

`date`字段用到的参数

|参数|说明|
|---|---|
|boost|映射字段级别的查询时boost。接受一个浮点数，默认为1.0|
|doc_values|该字段是否以跨列的方式保存，以便排序、聚集、执行脚本。接受`true`（默认）或者`false`|
|format|日期格式。默认是`strict_date_optional_time || epoch_millis`|
|locale|用来解析日期的`local`，因为在所有语言中月份名称和缩写都是不同的。默认为`ROOT`|
|ignore_malformed|如果设置为`true`，不正确的数字会被忽略。如果设置为`false`（默认），不正确的数字会抛出异常然后拒绝整个文档|
|include_in_all|字段是否应该被包含在`_all`字段中。如果`index`被设置为`false`或者父`object`字段的`include_in_all`被设置为`false`，则该字段的`include_in_all`也被设置为`false`。除此以外默认都是`true`|
|index|该字段是否可以被搜索。接收`true`（默认）和`false`|
|null_value|接受一个符合`format`格式的日期值，整个值可以替换任何显式的`null`值。默认为`null`，这意味着该字段被当成缺失的。|
|store|该字段的值是否可以独立获取，而不仅仅包含在`_source`字段中。接受`true`和`false`（默认）|
