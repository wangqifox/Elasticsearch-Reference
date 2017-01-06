# fielddata

默认情况下，大部分字段都会被索引，这使得它们可以搜索。但是，对脚本中的字段值进行排序、聚合、访问需要与搜索不同的访问模式。

搜索需要回答的问题是“哪些文档包含此分词”，但是排序和聚合需要回答不同的问题“文档中此字段的值是什么”。

大多数字段都可以使用索引阶段在磁盘中生成的`doc_values`以适应这样的数据访问模式，但是`text`字段不支持`doc_values`。

`text`字段使用查询阶段在内存中生成的数据结构称为`fielddata`。这个数据结构在第一次用于聚合、排序或脚本是的时候按需构建的。通过从磁盘中读取整个段的反向索引，反转分词与文档的关系，将结果存储在内存(JVM heap)中。

## fielddata默认在`text`字段是关闭的

`fielddata`会消耗大量的堆空间，特别是加载高基数`text`字段。一旦fielddata被加载进堆内存，它在整个段的生命期都会存在。加载fielddata也是一个复杂的过程，会导致命中延迟。这也是为什么fielddata默认情况是关闭的。

当你尝试使用脚本对一个`text`字段进行排序、聚合、访问，你将会看到此异常：

> fielddata默认在`text`字段上是关闭的。在字段上设置`fielddata=true`会将fielddata加载进内存。注意着将消耗可观的内存。

## 开启fielddata之前

开启fielddata之前，考虑一下你为什么要在排序、聚合、脚本中使用`text`字段。通常这样做是没有意义的。

`text`字段在索引前会进行分析，因此在搜索`new`或`york`时可以找到`New York`。当你想要一个称为`New York`的桶时，在这个字段上的聚合会返回一个`new`桶和一个`york`桶。

你需要一个用作全文搜索的`text`字段，以及一个未做分析的`keyword`字段，开启`doc_values`用作聚合。

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "mappings": {
    "my_type": {
      "properties": {
        "my_field": { 	// 1
          "type": "text",
          "fields": {
            "keyword": { 	// 2
              "type": "keyword"
            }
          }
        }
      }
    }
  }
}'
```

- 1 `my_field`字段用作搜索
- 2 `my_field.keyword`字段用作聚合、排序、脚本中访问。

## `text`字段开启fielddata

你可以使用`PUT mapping API`在已存在的字段上打开fielddata。

```
curl -XPUT 'localhost:9200/my_index/_mapping/my_type?pretty' -d'
{
  "properties": {
    "my_field": { 	// 1
      "type":     "text",
      "fielddata": true
    }
  }
}'
```

- 1 `my_field`指定的映射应该由该字段上已经存在的映射组成，再加上`fielddata`参数

> 同一个索引中相同名字的字段必须有相同`fielddata.*`参数设置。`fielddata.*`参数值可以使用`PUT mapping API`来更新。

## 全部序数

全部序数是在`fielddata`和`doc values`之上的数据结构，它以字典顺序维护每个唯一term的增量编号。每个`term`具有一个唯一的编号，term A的编号低于term B的编号。只有`text`和`keyword`字段支持全部序数。

`fielddata`和`doc values`也有序数，它是特定分段和字段中所有term的唯一编号。全部序数通过提供段序数和全局序数之间的映射建立在`fielddata`和`doc values`之上，全局序数在整个分片中是唯一的。

全局序数在使用段序数的功能中使用，比如排序和term聚合，以改善执行时间。term聚合单纯依赖于全局序数在分片级别进行聚合，然后将全局序数转化为真实的term用作最后的reduce阶段，reduce阶段将不同的分片合并成最后的结果。

指定字段的全局序数绑定到一个分片的所有段，`fielddata`和`doc values`序数绑定到一个单独的段，其与绑定到单个段的特定字段的字段数据不同。因为这个原因，一旦新的段变得可见，整个全局序数需要重建。

全局序数的加载时间取决于字段中term的数量，不过一般来说是很低的，因为`source`字段的数据已经加载了。全局序数的内存开销很小，因为它被高效地压缩了。全局序数的立即加载将加载时间从第一次搜索请求移动到刷新本身。

## fielddata_frequency_filter

fielddata过滤可以减少加载进内存的term的数量，减少了内存的消耗。term可以通过频率来过滤：

频率过滤器允许你只加载文档频率在`min`和`max`之间的的term，频率值可以是一个绝对值（大于1.0的值）或者是一个百分数（0.01是1%，1.0是100%）。每个段都会计算频率。百分比基于具有字段值的文档数，而不是段中所有的文档。

使用`min_segment_size`可以指定段中最少的文档数，这样就可以排除一些比较小的段：

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "mappings": {
    "my_type": {
      "properties": {
        "tag": {
          "type": "text",
          "fielddata": true,
          "fielddata_frequency_filter": {
            "min": 0.001,
            "max": 0.1,
            "min_segment_size": 500
          }
        }
      }
    }
  }
}'
```

