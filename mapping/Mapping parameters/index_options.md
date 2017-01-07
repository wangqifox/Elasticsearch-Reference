# index_options

`index_options`参数控制什么信息可以被加入到倒排索引中，用来搜索和高亮。它接受以下设置：

|设置|说明|
|---|----|
|docs|仅仅索引文档编号。可以知道字段中是否存在某个分词|
|freqs|索引文档编号和分词频率。出现频率高的分词分值高于频率低的分词|
|positions|索引文档编号、分词频率、分词位置（或者顺序）。位置信息在`proximity or phrase queries`时使用|
|offsets|索引文档编号、分词频率、分词位置、起止字符的偏移量（将分词反向映射到原始的字符串）。偏移量在`postings highlighter`中使用。|

经过分析的字符串字段默认使用`positions`，其他所有的字段都默认使用`docs`。

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "mappings": {
    "my_type": {
      "properties": {
        "text": {
          "type": "text",
          "index_options": "offsets"
        }
      }
    }
  }
}'
curl -XPUT 'localhost:9200/my_index/my_type/1?pretty' -d'
{
  "text": "Quick brown fox"
}'
curl -XGET 'localhost:9200/my_index/_search?pretty' -d'
{
  "query": {
    "match": {
      "text": "brown fox"
    }
  },
  "highlight": {
    "fields": {
      "text": {} 	// 1
    }
  }
}'
```

- 1 `text`字段默认使用`postings highlighter`，因为索引中包含`offsets`信息。
