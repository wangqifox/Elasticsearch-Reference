# Index API

索引API在特定索引中添加或更新类型化的JSON文档，使其可搜索。一下示例将JSON文档插入到"twitter"索引中，类型名为"tweet"，ID为1：

```
curl -XPUT 'localhost:9200/twitter/tweet/1?pretty' -H 'Content-Type: application/json' -d'
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
'
```

上面操作的结果是：

```
{
    "_shards" : {
        "total" : 2,
        "failed" : 0,
        "successful" : 2
    },
    "_index" : "twitter",
    "_type" : "tweet",
    "_id" : "1",
    "_version" : 1,
    "created" : true,
    "result" : created
}
```

`_shards`头提供了关于索引操作复制过程的信息。

- `total`：表明索引操作在多少个分片副本中执行（主分片和副本分片）
- `successful`：表明索引操作中有多少个分片副本执行成功
- `failed`：索引操作在副本分片上失败的情况下，包含复制相关错误的数组

在`successful`至少为1的情况下索引操作成功。

> 当索引操作成功返回时，可能不会全部启动副本分片（默认情况下，只需要主分片，但是可以更改此行为）。这这种情况下，`total`将等于基于`number_of_replicas`设置的总分片，并且成功将等于已启动的分片数（主分片和副本）。如果没有失败，失败将是0。



























