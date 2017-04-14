# copy_to

`copy_to`参数允许你创建自定义的`_all`字段。换句话说，多个字段的值可以组合成一组字段，作为一个单独的字段用作查询。比如，`first_name`和`last_name`字段可以组合成`full_name`字段。

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "mappings": {
    "my_type": {
      "properties": {
        "first_name": {
          "type": "text",
          "copy_to": "full_name"    // 1
        },
        "last_name": {
          "type": "text",
          "copy_to": "full_name"    // 2
        },
        "full_name": {
          "type": "text"
        }
      }
    }
  }
}'
curl -XPUT 'localhost:9200/my_index/my_type/1?pretty' -d'
{
  "first_name": "John",
  "last_name": "Smith"
}'
curl -XGET 'localhost:9200/my_index/_search?pretty' -d'
{
  "query": {
    "match": {
      "full_name": {    // 3
        "query": "John Smith",
        "operator": "and"
      }
    }
  }
}'
```

- 1 2 `first_name`和`last_name`字段的值组合成`full_name`字段。
- 3 `first_name`和`last_name`字段仍然可以分别查询，但是`full_name`字段可以以姓名组合的方式来查询。

几个重要的点：

- 复制的是字段值，而不是terms（由分析过程产生的分词）
- 原始的`_source`字段不会为了显示复制的值而修改
- 相同的值可以复制到多个字段中，如`"copy_to":["field_1","field_2"]`
