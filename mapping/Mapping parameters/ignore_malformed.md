# ignore_malformed

有时候没办法控制接收到的数据。一个用户可能会向`login`字段发送一个日期数据，另一个用户可能会向`login`字段发送一个email地址。

索引错误数据到字段默认会引发异常，并拒绝整个文档。如果将`ignore_malformed`参数设置为`true`，可以忽略整个异常。错误的字段不会被索引，但是文档中其他的字段正常处理。

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "mappings": {
    "my_type": {
      "properties": {
        "number_one": {
          "type": "integer",
          "ignore_malformed": true
        },
        "number_two": {
          "type": "integer"
        }
      }
    }
  }
}'
curl -XPUT 'localhost:9200/my_index/my_type/1?pretty' -d'
{
  "text":       "Some text value",
  "number_one": "foo" 	// 1
}'
curl -XPUT 'localhost:9200/my_index/my_type/2?pretty' -d'
{
  "text":       "Some text value",
  "number_two": "foo" 	// 2
}'
```

- 1 `text`字段正常被索引，`number_one`字段忽略
- 2 整个文档都会被拒绝，因为`number_two`不允许错误的值。

> 同一个索引中相同名字的字段可以有不同的`ignore_malformed`。可以通过`PUT mapping API`关闭已存在字段的`ignore_malformed`值。

## 索引级别的默认值

`index.mapping.ignore_malformed`可以在索引级别设置，以允许在所有映射类型中全局忽略错误内容。

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "settings": {
    "index.mapping.ignore_malformed": true 	// 1
  },
  "mappings": {
    "my_type": {
      "properties": {
        "number_one": { 	// 2
          "type": "byte"
        },
        "number_two": {
          "type": "integer",
          "ignore_malformed": false 	// 3
        }
      }
    }
  }
}'
```

- 1 2 `number_one`字段继承了索引级别的设置
- 3 `number_two`字段覆盖了`ignore_malformed`的索引级别设置。
