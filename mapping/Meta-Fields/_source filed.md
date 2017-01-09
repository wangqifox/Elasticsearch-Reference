# `_source`字段

`_source`字段包含索引阶段传入的原始JSON文档正文。`_source`字段本身不参与索引（因此它无法被搜索），但是它被保存了，所以可以用于获取请求，比如`get`或者`search`。

## 关闭`_source`字段

尽管非常便利，`source`字段确实会在索引中增加磁盘开销。因此可以按以下方式禁用：

```
curl -XPUT 'localhost:9200/tweets?pretty' -d'
{
  "mappings": {
    "tweet": {
      "_source": {
        "enabled": false
      }
    }
  }
}'
```

> 用于经常不考考后果就禁用`_source`字段，结果后悔不已。如果`_source`字段不可用，以下的特性就不支持：
> - `update`, `update_by_query`, `reindex`接口
> - `highlighting`
> - 从一个Elasticsearch索引重新索引到另一个的能力，改变映射或者分析，更新索引到新版本
> - 通过查看索引时的原始文档来调试查询或者聚合的能力
> - 未来的潜能，自动修复索引损坏的能力

> 如果磁盘空间紧张，最好考虑增加压缩级别而不是禁用`_source`字段

## 度量用例

度量用例和其他基于时间或日志的用例不同，有许多只包含数字、日期、关键字的小文档。这些用例没有更新，没有高亮请求，数据快速老化，所以不需要重新索引。搜索请求通常使用简单的查询按日期或标签来过滤数据，结果作为聚合返回。

这种情况下，禁用`_source`字段可以节省空间且减少I/O，同时也建议禁用`_all`字段。

## `_source`字段中包含或者排除字段

文档索引之后`_source`字段存储之前，修改`_source`字段内容的能力是一项专家级的功能。

> 从`_source`中删除字段与禁用`_source`具有类型的缺点，特别是你无法将文档从一个Elasticsearch索引重新索引到另一个索引。应该考虑使用`source filtering`替代。

`includes/excludes`参数（接受通配符）可以像下面这样使用：

```
curl -XPUT 'localhost:9200/logs?pretty' -d'
{
  "mappings": {
    "event": {
      "_source": {
        "includes": [
          "*.count",
          "meta.*"
        ],
        "excludes": [
          "meta.description",
          "meta.other.*"
        ]
      }
    }
  }
}'
curl -XPUT 'localhost:9200/logs/event/1?pretty' -d'
{
  "requests": {
    "count": 10,
    "foo": "bar" 	// 1
  },
  "meta": {
    "name": "Some metric",
    "description": "Some metric description", 	// 2
    "other": {
      "foo": "one", 	// 3
      "baz": "two" 		// 4
    }
  }
}'
curl -XGET 'localhost:9200/logs/event/_search?pretty' -d'
{
  "query": {
    "match": {
      "meta.other.foo": "one" 	// 5
    }
  }
}'
```

- 1 2 3 4 这些字段从`_source`中删除
- 5 尽管没有保存在`_source`中，但是我们仍然可以搜索这些字段
