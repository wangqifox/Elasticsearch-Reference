# 强制类型转换

数据并不总是干净的。根据其生成方式，数字可能在JSON正文中以真正的JSON数字呈现（比如5），也可以作为字符串呈现（比如"5"）。需要整型数字的地方可能会以浮点数呈现（比如5.0，甚至"5.0）。

强制类型转换尝试清理脏数据来适应字段的数据类型。例如：

- 字符串强制转型成数字
- 浮点数截断成整型值

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "mappings": {
    "my_type": {
      "properties": {
        "number_one": {
          "type": "integer"
        },
        "number_two": {
          "type": "integer",
          "coerce": false
        }
      }
    }
  }
}'
curl -XPUT 'localhost:9200/my_index/my_type/1?pretty' -d'
{
  "number_one": "10" 	// 1
}'
curl -XPUT 'localhost:9200/my_index/my_type/2?pretty' -d'
{
  "number_two": "10" 	// 2
}'
```

- 1 `number_one`字段赋值为`10`
- 2 该文档会被拒绝，因为关闭了强制类型转换

> 同一个索引中相同名字的字段可以有不同的`coerce`设置。可以通过`PUT mapping API`更新已存在字段的`coerce`值。

## 索引级别的默认设置

`index.mapping.coerce`可以在索引级别设置来关闭全局的强制类型转换：

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "settings": {
    "index.mapping.coerce": false
  },
  "mappings": {
    "my_type": {
      "properties": {
        "number_one": {
          "type": "integer",
          "coerce": true
        },
        "number_two": {
          "type": "integer"
        }
      }
    }
  }
}'
curl -XPUT 'localhost:9200/my_index/my_type/1?pretty' -d'
{ "number_one": "10" } '	// 1
curl -XPUT 'localhost:9200/my_index/my_type/2?pretty' -d'
{ "number_two": "10" } '	// 2
```

- 1 `number_one`字段覆盖了索引级别的设置，开启了强制类型转换
- 2 文档会被拒绝，因为`number_two`字段继承了索引级别的强制类型设置。
