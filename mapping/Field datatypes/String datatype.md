## String数据类型

5.x中创建的索引不支持`string`字段，而是支持`text`和`keyword`字段。如果尝试在5.0中创建一个包含`string`字段的索引，Elasticsearch会尝试将`string`升级成恰当的`text`或者`keyword`字段。返回一个HTTP警告头告诉你`string`已经被废弃了。这样的升级过程并不总是完美的，因为一些`string`支持的功能组合并不支持`text`或者`keyword`。因此使用`text`或者`keyword`会更好。

从2.x导入的索引只支持`string`而不支持`text`和`keyword`。为了简化从2.0的升级，Elasticsearch会把应用于2.x索引的映射从`text`和`keyword`降级到`string`。虽然长期索引最终需要针对5.x重新创建，然后升级到6.x，但是这样的降级可以使版本的升级变得平滑。


