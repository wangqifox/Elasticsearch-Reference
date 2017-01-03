# 字段数据类型

Elasticsearch支持很多不同的数据类型

## 核心数据类型

- 字符串
  - `text`和`keyword`
- 数值类型
  - `long`, `integer`, `short`, `byte`, `double`, `float`
- 日期类型
  - `date`
- 布尔类型
  - `boolean`
- 二进制类型
  - `binary`

## 复杂数据类型

- 数组类型
  - 数据不需要专门类型来支持
- 对象类型
  - `object`，用于单个JSON对象
- 嵌套类型
  - `nested`，用于JSON对象数组

## 地理数据类型

- 地理坐标类型
  - `geo_point` 经度(longitude)/纬度(latitude)点
- 地理形状类型
  - `geo_shape` 多边形一样的复杂形状

## 专门的数据类型

- IP类型
  - `ip` IPv4和IPv6地址
- 完成类型
  - `completion` 提供自动补全提示
- token计数类型
  - `token_count` 计算字符串中token的个数
- `mapper-murmur3`
  - `murmur3` 计算索引时和保存时值的hash值
- 附件类型
  - 查看`mapper-attachments`插件，这个插件支持附件的索引，附件类型包括`Microsoft Office`, `Open Documents format`, `ePub`, `HTML`等等。
- 过滤器类型
  - 接受来自query-dsl的查询

## 多字段

针对不同的目的，将相同的字段以不同的方式索引往往是很有用的。比如，`string`字段可以当成`text`来索引用作全文索引，或者当成`keyword`来索引用作排序或聚集。或者你还可以用`standard analyzer`, `english analyzer`, `french analyzer`来索引`string`字段。

这就是多字段(multi-fields)的目的。通过`fields`参数，大多数数据类型支持多字段。
