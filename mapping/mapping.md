# 映射(mapping)

映射用来定义文档和文档中的字段如何存储、索引。例如，使用映射定义：

- 哪个string字段应该被当做全文字段
- 哪个字段包含数字、日期、地理位置
- 文档中所有的字段值是否应该被`_all`字段索引
- 日期的格式
- 控制动态增加字段的自定义规则

## 映射类型

每个索引都有一个或多个映射类型，用来将索引中的文档划分成逻辑组。User文档可能存储在`user`类型，blog posts存储在`blogpost`类型。

每个映射类型拥有：

- 元字段(Meta-fields)
  - 元字段用来规定文档关联的元数据如何处理。元字段包括：`_index`, `_type`, `_id`, `_source`
- 字段或属性(Fields or properties)
  - 每个映射类型包括一系列相关的字段或者属性。`user`类型可能包含`title`, `name`, `age`字段，`blogpost`类型可能包含`title`, `body`, `user_id`, `created`字段。一个索引不同的映射类型中，相同名字的字段必须有相同的映射。

## 字段数据类型

每个字段都有数据类型：

- 简单的类型，比如：`text`, `keyword`, `date`, `long`, `double`, `boolean`, `ip`
- 支持JSON分层特性的类型，比如`object`, `nested`
- 专门的类型，比如`geo_point`, `geo_shape`, `completion`

针对不同的目的，将相同的字段以不同的方式索引往往是很有用的。比如，`string`字段可以当成`text`来索引用作全文索引，或者当成`keyword`来索引用作排序或聚集。或者你还可以用`standard analyzer`, `english analyzer`, `french analyzer`来索引`string`字段。

这就是多字段(multi-fields)的目的。通过`fields`参数，大多数数据类型支持多字段。

### 防止映射爆炸(mappings explosion)

下面的设置允许你限制手动或者自动可以创建的字段映射的数量，以防止坏的文档引起映射爆炸。

- index.mapping.total_fields.limit
  - 一个索引最大的字段数量。默认值是1000
- index.mapping.depth.limit
  - 字段最大的深度，深度指的是内部对象的数量。比如，如果所有的字段都定义在根对象层级上，那么深度是1；如果有一个对象映射，那么深度是2。默认值是20
- index.mapping.nested_fields.limit
  - 索引中`nested`字段最大的数量。将1个文档用100个嵌套字段来索引，实际上索引了101个文档，因为每个嵌套的文档都被当做一个单独的隐藏文档来索引。


## 动态映射(dynamic mapping)

字段和映射类型不需要在使用前被定义。因为动态映射，新的映射类型和新的字段名称在索引的时候会被自动添加。新的字段会被添加到最高层的映射类型、内部对象、嵌套字段。

可以配置动态映射规则来自定义用于新类型和新字段的映射。

## 显示映射(explicit mapping)

你对数据的了解肯定比Elasticsearch所猜测的要多，所以动态映射在开始的时候很有用，但是在某一时刻，你会想指定显示的映射。

你可以在创建索引的时候创建映射类型和字段映射，你也可以通过`PUT mapping API`给已存在的映射添加映射类型和映射字段。

## 更新已存在的映射

除了记录在哪里，**已存在的类型映射和字段映射** 不能被更新。更改映射意味着所有已经被索引的文档都会失效。所以，你必须创建一个新的索引然后将数据重新建立索引。

## 字段通过映射类型共享

映射类型用来给字段分组，但是每个映射类型中字段不是互相独立的。

字段拥有：
- 相同的名字
- 在同一个索引
- 在不同的映射类型
- 实际映射到相同的字段

必须拥有相同的映射。

如果`title`字段同时存在于`user`和`blogpost`映射类型，`title`字段在每个类型中必须有相同的映射。这个规则唯一的例外是`copy_to`, `dynamic`, `enabled`, `ignore_above`, `include_in_all`, `properties`，这些参数在每个字段中可以有不同的设置。

通常相同名字的字段也包含相同的数据类型，因此拥有相同的映射也不会有问题。如果发生了冲突，可以将这些字段替换成根据描述性的名字，比如`user_title`, `blog_title`。

## 例子

如下所示，创建索引时可以指定如上所述的映射。

```
curl -XPUT 'localhost:9200/my_index ?pretty' -d'  // 1
{
  "mappings": {
    "user": { // 2
      "_all":       { "enabled": false  },  // 3
      "properties": { // 4
        "title":    { "type": "text"  },  // 5
        "name":     { "type": "text"  },  // 6
        "age":      { "type": "integer" }   // 7
      }
    },
    "blogpost": { // 8
      "_all":       { "enabled": false  },  // 9
      "properties": { // 10
        "title":    { "type": "text"  },  // 11
        "body":     { "type": "text"  },  // 12
        "user_id":  {
          "type":   "keyword" // 13
        },
        "created":  {
          "type":   "date", // 14
          "format": "strict_date_optional_time||epoch_millis"
        }
      }
    }
  }
}'
```


- 1 创建一个名叫`my_index`
- 2 8 添加名叫`user`和`blogpost`的映射类型
- 3 9 `user`映射类型中将`_all`元字段设为禁用
- 4 10 为每个映射类型指定字段或者属性
- 5 6 7 11 12 13 14 指定数据类型和每个字段的映射
