# 排序

可以在特定的字段上进行一个或多个排序。排序可以反转。排序在每个字段级别上定义，还有一些特殊的字段：`_score`用来根据分值排序，`_doc`用来根据索引顺序排序。

假设下面是索引的映射：

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
    "mappings": {
        "my_type": {
            "properties": {
                "post_date": { "type": "date" },
                "user": {
                    "type": "keyword"
                },
                "name": {
                    "type": "keyword"
                },
                "age": { "type": "integer" }
            }
        }
    }
}'
```

```
curl -XGET 'localhost:9200/my_index/my_type/_search?pretty' -d'
{
    "sort" : [
        { "post_date" : {"order" : "asc"}},
        "user",
        { "name" : "desc" },
        { "age" : "desc" },
        "_score"
    ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}'
```

> `_doc`没有真正的用例，除了它是最有效的排序顺序。如果你不关心文档的返回顺序，可以使用`_doc`来排序。使用`scrolling`时这特别有用。

## 排序值

返回的每个文档的排序值也作为响应的一部分返回。

## 排序顺序

`order`可以有以下的值：

|值|说明|
|--|---|
|asc|升序排序|
|desc|降序排序|

对`_score`排序时默认用`desc`，对其他字段排序时默认使用`asc`

## 排序模式

Elasticsearch支持按数组或多值字段排序。`mode`选项控制选择用于对其所属文档进行排序的数组值。`mode`选项可以具有以下值：

|值|说明|
|--|---|
|min|选择最小值|
|max|选择最大值|
|sum|使用所有值的和作为排序值。仅适用于基于数字的数组字段|
|avg|使用所有值的平均作为排序值。仅适用于基于数字的数组字段|
|median|使用所有值的中值作为排序值。仅适用于基于数字的数组字段|

### 排序模式例子

下面的例子中price字段有多个值。匹配的结果会按照价格的平均值来升序排列。

```
curl -XPUT 'localhost:9200/my_index/my_type/1?refresh&pretty' -d'
{
   "product": "chocolate",
    "price": [20, 4]
}'
curl -XPOST 'localhost:9200/_search?pretty' -d'
{
   "query" : {
      "term" : { "product" : "chocolate" }
   },
   "sort" : [
      {"price" : {"order" : "asc", "mode" : "avg"}}
   ]
}'
```

## 对`nested`对象排序

Elasticsearch也支持对包含一个或多个嵌套对象的字段进行排序。嵌套字段排序在已有的排序选项之上还有以下参数：

### nested_path

定义要排序的嵌套对象。实际排序字段必须是这个嵌套对象的直接字段。对嵌套字段排序时，此字段是必须的。

### nested_filter

过滤器应该和嵌套路径中的内部对象相匹配，以便在排序时考虑这个字段的值。通常在嵌套过滤或查询中重复查询/过滤。默认没有`nested_filter`。

## 嵌套排序的例子

下面的例子中，`offer`是一个嵌套类型的字段。需要指定`nested_path`，否则elasticsearch不知道在哪一个嵌套级获取排序数据。

```
curl -XPOST 'localhost:9200/_search?pretty' -d'
{
   "query" : {
      "term" : { "product" : "chocolate" }
   },
   "sort" : [
       {
          "offer.price" : {
             "mode" :  "avg",
             "order" : "asc",
             "nested_path" : "offer",
             "nested_filter" : {
                "term" : { "offer.color" : "blue" }
             }
          }
       }
    ]
}'
```

根据脚本和地理距离排序也支持嵌套排序。

## 缺失值

`missing`参数指定了缺失字段的文档应该如何处理：`missing`值可以是`_last`，`_first`，或者一个特定的值（排序值缺失时使用）。

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "sort" : [
        { "price" : {"missing" : "_last"} }
    ],
    "query" : {
        "term" : { "product" : "chocolate" }
    }
}'
```

> 如果嵌套内部对象不匹配`nested_filter`，使用缺失的值。

## 忽略没有映射的字段

默认情况下，如果字段没有映射相关联，搜索请求会失败。`unmapped_type`选项允许忽略没有映射的且没有根据它们来排序的字段。这个参数的值用来确定哪些值可以排序。下面是如何使用这个参数的例子：

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "sort" : [
        { "price" : {"unmapped_type" : "long"} }
    ],
    "query" : {
        "term" : { "product" : "chocolate" }
    }
}'
```

如果用于查询的索引没有字段`price`的映射，Elasticsearch会像有一个`long`类型的映射来处理它，这个索引中所有的文档都没有这个字段的值。

## 地理距离排序

### Lat Lon 作为属性

### Lat Lon 作为字符串

### Geohash

### Lat Lon 作为数组

### 多个参照点

## 基于脚本的排序

允许基于特定的脚本来排序，如下所示：

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "query" : {
        "term" : { "user" : "kimchy" }
    },
    "sort" : {
        "_script" : {
            "type" : "number",
            "script" : {
                "lang": "painless",
                "inline": "doc[\u0027field_name\u0027].value * params.factor",
                "params" : {
                    "factor" : 1.1
                }
            },
            "order" : "asc"
        }
    }
}'
```

## 分值跟踪

当对一个字段排序时，不计算分值。通过设置`track_scores`为`true`，仍然可以计算跟踪分值。

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "track_scores": true,
    "sort" : [
        { "post_date" : {"order" : "desc"} },
        { "name" : "desc" },
        { "age" : "desc" }
    ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}'
```

## 考虑内存

排序时，相关排序字段的值加载到内存中。这意味着每个分片应该有内存来存储。对于基于字符串的类型，排序字段不应该被分词。对于数值类型，建议将类型设置地尽量窄（比如`short`,`intege`,`float`）。
