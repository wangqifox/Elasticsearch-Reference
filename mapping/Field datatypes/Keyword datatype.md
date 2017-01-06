# Keyword数据类型

用于索引结构化的数据的字段，比如email地址、主机名、状态码、邮政编码或者标签，通常用于过滤（查找所有状态是published的博客文章）、排序、聚合。`keyword`字段只能通过精确值来搜索。

如果需要索引全文内容，比如email体或者产品描述，那么你可能需要使用`text`字段。

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "mappings": {
    "my_type": {
      "properties": {
        "tags": {
          "type":  "keyword"
        }
      }
    }
  }
}'
```

## `keyword`字段接收的参数

|参数|说明|
|---|---|
|boost||
|doc_values||
|eager_global_ordinals||
|fields||
|ignore_above||
|include_in_all||
|index||
|index_options||
|norms||
|null_value||
|store||
|search_analyzer||
|similarity||

> 2.x版本导入的索引不支持`keyword`。它们会尝试将`keyword`降级到`string`。这允许你将新映射和旧映射合并。长期索引在升级到6.x之前必须重新创建，但是映射降级允许你根据自己的安排来进行重新创建。
