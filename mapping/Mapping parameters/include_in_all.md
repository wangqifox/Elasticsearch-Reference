# include_in_all

`include_in_all`参数控制每个字段是否包括在`_all`字段，默认是`true`。如果`index`设置为`false`，`include_in_all`则为`false`。

下面的例子演示了如何在`_all`字段中将`date`字段排除在外：

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "mappings": {
    "my_type": {
      "properties": {
        "title": { 	// 1
          "type": "text"
        },
        "content": { 	// 2
          "type": "text"
        },
        "date": { 	// 3
          "type": "date",
          "include_in_all": false
        }
      }
    }
  }
}'
```

- 1 2 `title`和`content`字段包含在`_all`字段中
- 3 `date`字段不包含在`_all`字段中

> 同一个索引中相同名字的字段可以有不同的`include_in_all`。可以通过`PUT mapping API`更新已存在字段的`include_in_all`值。

`include_in_all`参数可以在类型级别和`object`或者`nested`字段上配置，子字段可以继承这样的设置。

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "mappings": {
    "my_type": {
      "include_in_all": false, 	// 1
      "properties": {
        "title":          { "type": "text" },
        "author": {
          "include_in_all": true, 	// 2
          "properties": {
            "first_name": { "type": "text" },
            "last_name":  { "type": "text" }
          }
        },
        "editor": {
          "properties": {
            "first_name": { "type": "text" }, 	// 3
            "last_name":  { "type": "text", "include_in_all": true } 	// 4
          }
        }
      }
    }
  }
}'
```

- 1 `my_type`中的所有字段都从`_all`中排除
- 2 `author.first_name`和`author.last_name`字段包含在`_all`
- 3 4 只有`editor.last_name`包含在`_all`中，`editor_first_name`继承了类型级别的设置，排除在`_all`之外。

> 原始字段值将添加到`_all`字段，字段的分析器生成的内容没有在`_all`中。 因此，在多字段上将`include_in_all`设置为`true`没有任何意义，因为每个多字段与其父字段具有完全相同的值。
