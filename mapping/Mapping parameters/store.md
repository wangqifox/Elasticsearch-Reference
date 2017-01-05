# store

默认情况下，字段值被编入索引以便使其可搜索，但是不会保存它们。这意味着字段可以查询，但是无法检索原始的字段值。

通常这没有问题。默认情况下，字段的值已经是`_source`字段的一部分了。如果你只是想检索单个的字段或者几个字段，只需要通过`source filtering`来查询，而不需要查询整个`_source`字段。

在某些情况下保存一个字段是有意义的。比如，文档有一个`title`字段、一个`date`字段和一个非常巨大的`content`字段，你可能只想检索`title`和`date`而是不从一个巨大的`_source`字段中抽取这两个字段。

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "mappings": {
    "my_type": {
      "properties": {
        "title": {
          "type": "text",
          "store": true     // 1
        },
        "date": {
          "type": "date",
          "store": true     // 2
        },
        "content": {
          "type": "text"
        }
      }
    }
  }
}'
curl -XPUT 'localhost:9200/my_index/my_type/1?pretty' -d'
{
  "title":   "Some short title",
  "date":    "2015-01-01",
  "content": "A very long content field..."
}'
curl -XGET 'localhost:9200/my_index/_search?pretty' -d'
{
  "stored_fields": [ "title", "date" ]  // 3
}'
```

- 1 2 `title`和`date`字段设置为存储原始值
- 3 这个请求会检索`title`和`date`字段的值

> 为了保持一致性，存储原始值的字段总是以一个数组的形式返回，因为没有办法知道原始值是单个值、多个值还是一个空数组。
> 如果你需要原始的值，你应该从`_source`字段来检索

存储字段值另一个有意义的情况是，有些字段不会出现在`_source`中（比如`copy_to`字段）。
