# boost

使用`boost`参数，单个字段可以在查询时被自动boost，计算相关性得分时更多的考虑该字段：

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "mappings": {
    "my_type": {
      "properties": {
        "title": {
          "type": "text",
          "boost": 2  // 1
        },
        "content": {
          "type": "text"
        }
      }
    }
  }
}'

```
- 1 `title`字段上的匹配度会比`content`字段上的匹配度高出两倍，默认的`boost`值为1.0

> `boost`仅适用于`term queries`（prefix, range, fuzzy查询不适用）

查询时直接使用boost参数也可以达到相同的效果。

```
curl -XPOST 'localhost:9200/_search?pretty' -d'
{
    "query": {
        "match" : {
            "title": {
                "query": "quick brown fox"
            }
        }
    }
}'

curl -XPOST 'localhost:9200/_search?pretty' -d'
{
    "query": {
        "match" : {
            "title": {
                "query": "quick brown fox",
                "boost": 2
            }
        }
    }
}'

当字段带有boost时，两种查询的效果是相同的
```

boost在处理`_all`字段时也是适用的。这意味着，当查询`_all`字段时，来源于`title`字段的词比来源于`content`字段的词分值更大。这个功能也是有代价的，使用了boost之后查询`_all`字段会变慢。

> 索引时boost在5.0.0中已经废弃了，被查询时boost所代替。不过5.0.0之前创建的索引boost仍然可用。

> 索引时boost不是个好主意，原因如下：
- 除非将所有的文档重新索引，否则无法改变索引时boost的值。
- 使用查询时boost可以达到相同的效果，而且不用重新索引就可以改变boost的值
- 索引时boost作为`norm`的一部分来保存，而`norm`只有1字节。这降低了字段长度归一化因子的分辨率，这可能导致相关性计算的质量变低。
