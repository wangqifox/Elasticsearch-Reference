# null_value

`null`值不能被索引和查询。当一个字段被设置为`null`（一个空数组或者一个都是`null`的数组），该字段被视作没有值。

`null_value`参数允许你将`null`值替换成一个指定的值，这样就可以被索引和查询了。

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "mappings": {
    "my_type": {
      "properties": {
        "status_code": {
          "type":       "keyword",
          "null_value": "NULL"  // 1
        }
      }
    }
  }
}'
curl -XPUT 'localhost:9200/my_index/my_type/1?pretty' -d'
{
  "status_code": null
}'
curl -XPUT 'localhost:9200/my_index/my_type/2?pretty' -d'
{
  "status_code": []     // 2
}'
curl -XGET 'localhost:9200/my_index/_search?pretty' -d'
{
  "query": {
    "term": {
      "status_code": "NULL"     // 3
    }
  }
}'
```

- 1 将`null`值替换成字符串"NULL"
- 2 空数组不包含显式的`null`，所以不会被替换成`null_value`值
- 3 查询`NULL`会返回文档1，而不是文档2

> `null_value`必须和字段的数据类型一样，比如字段是`long`类型就不能有一个`string`类型的`null_value`。
