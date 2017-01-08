# 数组类型

Elasticsearch中没有专用的`array`类型。任何类型默认都可以包含0个或多个值，但是数组中的值必须是同一类型的。比如：

- 字符串数组：["one", "two"]
- 整形数组：[1, 2]
- 包含数组的数组：[1, [2, 3]]，等价于[1, 2, 3]
- 对象的数组：[{"name":"Mary", "age":12}, {"name":"John", "age":10}]

> 对象数组的工作方式和你预期的不一样：不能单独查询数组中的每个对象。如果你需要这样查询必须使用`nested`类型而不是`object`类型。更详细的解释参照`Nested datatype`

动态增加字段时，数组中第一个值决定了字段的类型。数组中其他的值必须和这个类型相同或者至少可以强制转化成一致的类型。

不支持包含混合数据类型的数组：[10, "some string"]

包含`null`值的数组，`null`值可以被`null_value`替换或者整个数组跳过。空数组[]作为缺失的字段——没有值的字段。

支持数组不需要预先设置，它们是开箱即用的：

```
curl -XPUT 'localhost:9200/my_index/my_type/1?pretty' -d'
{
  "message": "some arrays in this document...",
  "tags":  [ "elasticsearch", "wow" ], 	// 1
  "lists": [ 	// 2
    {
      "name": "prog_list",
      "description": "programming list"
    },
    {
      "name": "cool_list",
      "description": "cool stuff list"
    }
  ]
}'
curl -XPUT 'localhost:9200/my_index/my_type/2 ?pretty' -d'	// 3
{
  "message": "no arrays in this document...",
  "tags":  "elasticsearch",
  "lists": {
    "name": "prog_list",
    "description": "programming list"
  }
}'
curl -XGET 'localhost:9200/my_index/_search?pretty' -d'
{
  "query": {
    "match": {
      "tags": "elasticsearch" 	// 4
    }
  }
}'
```

- 1 `tags`字段作为`string`动态添加
- 2 `lists`字段作为`object`动态添加
- 3 第二个文档不包含数组，但是可以索引到相同的字段
- 4 在`tag`字段中搜索`elasticsearch`，匹配两个文档。

## 多值字段和倒排索引

所有的字段类型都支持多值字段，这是Lucene起源的结果。Lucene作为全文搜索引擎设计。为了搜索大文本中单独的词，Lucene将文本拆成单独的分词，将每个分词分别添加到倒排索引。

这意味着即使是简单的文本字段也必须默认支持多个值。当添加其他的数据类型时，比如数值、日期，使用和文本相同的数据结构，这样可以支持多值。
