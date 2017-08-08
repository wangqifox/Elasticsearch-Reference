# _routing 字段

文档使用下面的公式路由到一个特定的分片中：

```
shard_num = hash(_routing) % num_primary_shards
```

`routing`的默认值是文档的id，如果有`_parent`ID则使用`_parent`ID。

可以为每个文档指定自定义的`routing`值来实现自定义路由。例如：

```
curl -XPUT 'localhost:9200/my_index/my_type/1?routing=user1&refresh=true&pretty' -H 'Content-Type: application/json' -d'		// 1
{
  "title": "This is a document"
}
'
curl -XGET 'localhost:9200/my_index/my_type/1?routing=user1&pretty'		// 2
```

- 1 文档使用`user1`作为路由值，而非ID
- 2 在对文档执行`getting`, `deleting`, `updating`操作的时候，需要提供相同的`routing`值

`_routing`字段的值可以再查询中访问：

```
curl -XGET 'localhost:9200/my_index/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "terms": {
      "_routing": [ "user1" ] 			// 1
    }
  }
}
'
```

在`_routing`字段查询(也见`ids`查询)

## 使用自定义的路由来搜索

自定义路由可以减少搜索的影响。避免将搜索请求发送到索引的每个分片上，请求可以带上指定的路由值以便将其发送到特定的分片：

```
curl -XGET 'localhost:9200/my_index/_search?routing=user1,user2&pretty' -H 'Content-Type: application/json' -d'				// 1
{
  "query": {
    "match": {
      "title": "document"
    }
  }
}
'
```

- 1 这个搜索请求只会在`user1`和`user2`路由值关联的分片中执行

## 使路由值必须

当使用自定义的路由值时，无论是`indexing`, `getting`, `deleting`, `updating`操作，提供路由值都是非常重要的。

忘记路由值的话会导致文档在一个或多个分片中索引。为了安全起见，可以配置`_routing`字段使得所有的CRUD操作都必须提供`routing`值：

```
curl -XPUT 'localhost:9200/my_index2?pretty' -H 'Content-Type: application/json' -d'
{
  "mappings": {
    "my_type": {
      "_routing": {
        "required": true 		// 1
      }
    }
  }
}
'
curl -XPUT 'localhost:9200/my_index2/my_type/1?pretty' -H 'Content-Type: application/json' -d'		// 2
{
  "text": "No routing value provided"
}
'
```

- 1 `my_type`文档需要路由值
- 2 索引会抛出一个`routing_missing_exception`

## 带自定义路由的唯一IDs

当索引文档时指定了自定义的`_routing`，一个索引不同分片中的`_id`不能保证唯一性。事实上，如果使用不同的`_routing`值来索引，`_id`相同的文档可能在不同的分片中。

由用户来决定索引中的IDs是否是唯一的。

## 路由到一个索引分区

可以配置索引，使自定义的路由值到分片的一个子集而不是一个单独的分片。这样可以减少不平衡集群的风险，也减少了搜索的影响。

通过在索引创建的时候提供索引级别的设置`index.routing_partition_size`来达到这个目的。随着分区大小的增加，数据分布的越均匀，搜索的时候就需要查询更多的分片。

当存在此设置时，分片计算的公式变成：

```
shard_num = (hash(_routing) + hash(_id) % routing_partition_size) % num_primary_shards
```

`_routing`字段用来计算分片的集合，`_id`用来在集合中取出一个分片

为了开启这个功能，`index.routing_partition_size`应该大于1且小于`index.number_of_shards`。

一旦开启了该功能，分区索引就有以下限制：

- 父子关系的映射不能创建在其上
- 这个索引中的映射必须有标记为必须的`_routing`字段
