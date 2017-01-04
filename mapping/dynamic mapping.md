# 动态映射

Elasticsearch一个最重要的特征是它可以让你尽快地开始探索数据而不用过多地关注其他。你不用为了索引文档而在一开始创建索引、定义映射类型、定义字段，你只需要建立一个文档索引，索引、类型、字段会自动生成。

```
curl -XPUT 'localhost:9200/data/counters/1 ?pretty' -d' // 1
{ "count": 5 }'
```

- 1 创建一个名为`data`的索引，名为`counters`的映射类型，名为`count`的字段，字段的数据类型为`long`

自定检测添加新类型和字段的功能叫动态映射。动态映射的规则可以根据需求来自定义：

- `_default_`映射
  - 配置新类型使用的基本映射
- 动态字段映射
  - 控制动态字段检测的规则
- 动态模板
  - 自定义规则来配置动态增加字段的映射

**`index templates`允许你设置新索引默认的映射、配置、别名，不论新索引是自动创建还是显式创建的**

## 关闭自动创建类型

设置`index.mapper.dynamic`为`false`可以关闭某个索引上自动类型创建。

```
curl -XPUT 'localhost:9200/data/_settings?pretty' -d'
{
  "index.mapper.dynamic":false // 1
}'
```

- 1 关闭索引`data`上自动类型创建

设置索引模板可以关闭所有索引上自动类型创建。

```
curl -XPUT 'localhost:9200/_template/template_all?pretty' -d'
{
  "template": "*",
  "order":0,
  "settings": {
    "index.mapper.dynamic": false   // 1
  }
}'
```
- 1 关闭所有索引上自动类型创建

不管这些设置的值如何，类型仍然可以在创建索引或使用`PUT mapping`接口时显式添加
