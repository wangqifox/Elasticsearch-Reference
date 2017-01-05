# properties

类型映射、`object`字段、`nested`字段包含称为`properties`的子字段。这些属性可以是任何数据类型，包括`object`、`nested`。以下情况可以增加属性：

- 创建索引时定义显式定义属性
- 使用`PUT mapping`接口增加或者更新映射类型时显式定义属性
- 索引包含新字段的文档时动态地增加属性

下面的例子演示了向映射类型、`object`字段、`nested`字段中添加属性：

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "mappings": {
    "my_type": { 	// 1
      "properties": {
        "manager": { 	// 2
          "properties": {
            "age":  { "type": "integer" },
            "name": { "type": "text"  }
          }
        },
        "employees": { 	// 3
          "type": "nested",
          "properties": {
            "age":  { "type": "integer" },
            "name": { "type": "text"  }
          }
        }
      }
    }
  }
}'
curl -XPUT 'localhost:9200/my_index/my_type/1 ?pretty' -d'	// 4
{
  "region": "US",
  "manager": {
    "name": "Alice White",
    "age": 30
  },
  "employees": [
    {
      "name": "John Smith",
      "age": 34
    },
    {
      "name": "Peter Brown",
      "age": 26
    }
  ]
}'
```

- 1 `my_type`映射类型下的属性
- 2 `manager`对象字段下的属性
- 3 `employees`嵌套字段下的属性
- 4 对应以上映射的文档

> 同一个索引中相同名字的字段可以有不同的`properties`。可以通过`PUT mapping API`向已存在的字段增加`properties`值。

## 点符号

使用点符号可以在查询、聚集等中引用内部字段：

```
curl -XGET 'localhost:9200/my_index/_search?pretty' -d'
{
  "query": {
    "match": {
      "manager.name": "Alice White" 	// 1
    }
  },
  "aggs": {
    "Employees": {
      "nested": {
        "path": "employees"
      },
      "aggs": {
        "Employee Ages": {
          "histogram": {
            "field": "employees.age", 	// 2
            "interval": 5
          }
        }
      }
    }
  }
}'
```

> 必须指定内部字段的完整路径
