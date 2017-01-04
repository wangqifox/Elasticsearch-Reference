# doc_values

大多数字段都是以默认方式索引，这些字段都是可以被搜索的。倒排索引允许查询在唯一排序的搜索项列表中查找某一项，通过这个列表可以直接找到包含搜索项的文档列表。

排序、聚集、通过脚本访问字段值需要不同的数据访问方式。和查询搜索项来找到文档的方式不同，我们需要能够查询文档来找到搜索项。

doc值(doc values)是保存在磁盘中的数据结构，它在文档索引的时候建立，使得上面的数据访问方式成为可能。doc值保存和`_source`中相同的值，但是以一种面向列的方式来存储，这样可以更高效地来排序和聚集。doc值支持几乎所有的字段类型，除了`analyzed`字段。

支持doc值的所有字段默认都是打开doc值的。如果你确定不需要对某个字段进行排序、聚集、通过脚本访问字段值，可以关闭doc值以节约磁盘空间。

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "mappings": {
    "my_type": {
      "properties": {
        "status_code": { 	// 1
          "type":       "keyword"
        },
        "session_id": { 	// 2
          "type":       "keyword",
          "doc_values": false
        }
      }
    }
  }
}'
```

- 1 `status_code`字段中`doc_values`默认是打开的
- 2 `session_id`中关闭了`doc_values`，但是仍然可以被查询

> 同一个索引中相同名字的字段可以有不同的`doc_values`。可以通过`PUT mapping API`关闭以存在字段的`doc_values`值。
