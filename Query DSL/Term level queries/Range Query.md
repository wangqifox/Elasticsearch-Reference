# 范围查询

匹配字段中分词在某个范围内的文档。Lucene查询的类型依赖于字段的类型，对于`string`类型使用`TermRangeQuery`，对于数字/日期类型使用`NumericRangeQuery`。下面的例子返回`age`在10和20之间的所有文档。

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "query": {
        "range" : {
            "age" : {
                "gte" : 10,
                "lte" : 20,
                "boost" : 2.0
            }
        }
    }
}'
```

`range`查询接受以下参数：

|参数|说明|
|---|----|
|gte|大于等于|
|gt|大于|
|lte|小于等于|
|lt|小于|
|boost|设置查询的boost值，默认是1.0|

## 日期字段的范围

当在日期类型的字段上运行`range`查询，范围可以使用`Date Math`来指定。

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "query": {
        "range" : {
            "date" : {
                "gte" : "now-1d/d",
                "lt" :  "now/d"
            }
        }
    }
}'
```

## Date math 和四舍五入

当使用`date math`来将日期四舍五入到最近的天、月、小时等，经过四舍五入的日期取决于范围的结束是包含还是排除。

向上舍入移动到舍入范围的最后一个毫秒，并向下舍入到舍入范围的第一个毫秒。例如：

|参数|说明|
|---|----|
|gt|大于日期向上舍入：`2014-11-18||/M`变成`2014-11-30T23:59:59.999`，即不包括整个月|
|gte|大于等于日期向下舍入：`2014-11-18||/M`变成`2014-11-01`，包括整个月|
|lt|小于日期向下舍入：`2014-11-18||/M`变成`2014-11-01`，不包括整个月|
|lte|小于等于日期向上舍入：`2014-11-18|//M`变成`2014-11-30T23:59:59.999`，包括整个月|

## 范围查询中的日期格式

格式化日期默认使用日期字段上指定的`format`来解析，但是它可以通过`format`参数来覆盖：

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "query": {
        "range" : {
            "born" : {
                "gte": "01/01/2012",
                "lte": "2013",
                "format": "dd/MM/yyyy||yyyy"
            }
        }
    }
}'
```

## 范围查询中的时区

可以通过在日期值本身指定时区来将另一个时区转换为UTC，或者可以指定`time_zone`参数：

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "query": {
        "range" : {
            "timestamp" : {
                "gte": "2015-01-01 00:00:00", 	// 1
                "lte": "now", 	// 2
                "time_zone": "+01:00"
            }
        }
    }
}'
```

- 1 日期转换为`2014-12-31T23:00:00 UTC`
- 2 `now`不受`time_zone`参数的影响（日期必须作为UTC来存储）


