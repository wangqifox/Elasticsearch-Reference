# 嵌套数据类型

`nested`类型是一种对象类型的特殊版本，它允许索引对象数组，独立地索引每个对象。

## 如何使对象数组变扁平

内部类对象数组并不以你预料的方式工作。Lucene没有内部对象的概念，所以Elasticsearch将对象层次扁平化，转化成字段名字和值构成的简单列表。比如，以下的文档：

```
curl -XPUT 'localhost:9200/my_index/my_type/1?pretty' -d'
{
  "group" : "fans",
  "user" : [ 	// 1
    {
      "first" : "John",
      "last" :  "Smith"
    },
    {
      "first" : "Alice",
      "last" :  "White"
    }
  ]
}'
```

`user`字段作为对象动态添加

在内部被转化成如下格式的文档：

```
{
  "group" :        "fans",
  "user.first" : [ "alice", "john" ],
  "user.last" :  [ "smith", "white" ]
}
```

`user.first`和`user.last`扁平化为多值字段，`alice`和`white`的关联关系丢失了。导致这个文档错误地匹配对`alice`和`smith`的查询：

```
curl -XGET 'localhost:9200/my_index/_search?pretty' -d'
{
  "query": {
    "bool": {
      "must": [
        { "match": { "user.first": "Alice" }},
        { "match": { "user.last":  "Smith" }}
      ]
    }
  }
}'
```

## 使用`nested`字段对应`object`数组

如果你需要索引对象数组，并且保持数组中每个对象的独立性，你应该使用`nested`对象类型而不是`object`类型。`nested`对象将数组中每个对象作为独立隐藏文档来索引，这意味着每个嵌套对象都可以独立被搜索：

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "mappings": {
    "my_type": {
      "properties": {
        "user": {
          "type": "nested" 	// 1
        }
      }
    }
  }
}'
curl -XPUT 'localhost:9200/my_index/my_type/1?pretty' -d'
{
  "group" : "fans",
  "user" : [
    {
      "first" : "John",
      "last" :  "Smith"
    },
    {
      "first" : "Alice",
      "last" :  "White"
    }
  ]
}'
curl -XGET 'localhost:9200/my_index/_search?pretty' -d'
{
  "query": {
    "nested": {
      "path": "user",
      "query": {
        "bool": {
          "must": [
            { "match": { "user.first": "Alice" }},
            { "match": { "user.last":  "Smith" }} 	// 2
          ]
        }
      }
    }
  }
}'
curl -XGET 'localhost:9200/my_index/_search?pretty' -d'
{
  "query": {
    "nested": {
      "path": "user",
      "query": {
        "bool": {
          "must": [
            { "match": { "user.first": "Alice" }},
            { "match": { "user.last":  "White" }} 	// 3
          ]
        }
      },
      "inner_hits": { 	// 4
        "highlight": {
          "fields": {
            "user.first": {}
          }
        }
      }
    }
  }
}'
```

- 1 `user`字段映射为`nested`类型而不是`object`类型。
- 2 该查询没有匹配，因为`Alice`和`Smith`不在同一个嵌套类中
- 3 该查询有匹配，因为`Alice`和`White`在同一个嵌套类中
- 4 `inner_hits`可以高亮匹配的嵌套文档

嵌套文档可以：

- 使用`nested`查询来搜索
- 使用`nested`和`reverse_nested`聚合来分析
- 使用`nested`排序来排序
- 使用`nested inner hits`来检索和高亮

## `nested`字段参数

|参数|说明|
|---|---|
|dynamic||
|include_in_all||
|properties||

> 因为嵌套文档是作为单独的文档被索引的，所以嵌套文档只能被`nested`查询、`nested / reverse_nested`或者`nested inner hits`访问。
> 比如，一个`string`字段包含嵌套文档，嵌套文档中`index_options`设置为`offsets`以使用postings highlighter，这些偏移量在主要的高亮阶段是不可用的。必须通过`nested inner hits`来进行高亮操作。

## 限制`nested`字段的数量

索引一个包含100个`nested`字段的文档实际上就是索引101个文档，每个嵌套文档都作为一个独立文档来索引。为了防止过度定义嵌套字段的数量，每个索引可以定义的嵌套字段被限制在50个。
