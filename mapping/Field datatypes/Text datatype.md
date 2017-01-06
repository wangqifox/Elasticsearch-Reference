# Text数据类型

索引全文值的字段，比如email的正文或者产品的描述。这些字段会被分析，也就是说在被索引之前它们被传递给分析器，分析器将字符串拆开成一系列的独立分词。经过这个分析过程，Elasticsearch才可以搜索全文中单独的单词。`text`字段不能用来排序，也很少用来聚合（尽管`significant terms aggregation`是一个例外）。

如果你需要索引结构化的数据，比如email地址、主机名、状态码、邮政编码或者标签，最好使用`keyword`字段。

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "mappings": {
    "my_type": {
      "properties": {
        "full_name": {
          "type":  "text"
        }
      }
    }
  }
}'
```

有时同一个字段需要同时拥有全文(text)和关键字(keyword)版本：一个用作全文搜索，另一个用作聚合和排序。这可以通过`multi-fields`做到。

## `text`字段参数

|参数|说明|
|---|---|
|analyzer||
|boost||
|eager_global_ordinals||
|fielddata||
|fielddata_frequency_filter||
|fields||
|include_in_all||
|index||
|index_options||
|norms||
|position_increment_gap||
|store||
|search_analyzer||
|search_quote_analyzer||
|similarity||
|term_vector||

> 从2.x导入的索引不支持`text`。会尝试将`text`降级到`string`。这样就可以合并新映射和旧映射。长期索引在升级到6.x之前必须重新创建，映射降级使得你可以根据自己的安排来进行重新创建。
