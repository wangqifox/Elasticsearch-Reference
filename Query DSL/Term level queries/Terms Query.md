# Terms查询

过滤指定字段匹配任意一个给定分词（不经过分析）的文档。

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "query": {
        "constant_score" : {
            "filter" : {
                "terms" : { "user" : ["kimchy", "elasticsearch"]}
            }
        }
    }
}'
```

## 分词查找机制

当需要指定一个包含大量分词的分词过滤器时，从一个索引的文档中获取这些分词的值是很有利的。一个具体的例子是过滤你关注者推送的tweet。`term`过滤器中指定的用户ID数量可能会很多。这种情况下，使用term过滤器的term查找机制就很有意义。

term查找机制支持以下选项：

|选项|说明|
|---|---|
|index|从中获取term值的索引。默认是当前索引|
|type|从中获取term值的类型|
|id|从中获取term值的文档id|
|path|term过滤器获取真实值的路径|
|routing|检索外部分词文档时使用的自定义路由值|

`term`过滤器的值从特定类型和索引中特定id的文档中的字段获取。在内部，执行get请求从特定的路径来获取值。目前，这个功能工作需要保存`_source`。

如果引用的term数据不大，考虑使用单个分片的索引，并在所有节点中完全复制。如果可能的话，`term`过滤器偏好在本地节点中执行get请求，减少网络的需求。

## term查找的例子

首先，我们将用户的信息添加到id为2的索引中，其关注者索引来自id为1的用户的推文。最后我们搜索匹配用户2关注者的推文。

```
curl -XPUT 'localhost:9200/users/user/2?pretty' -d'
{
    "followers" : ["1", "3"]
}'
curl -XPUT 'localhost:9200/tweets/tweet/1?pretty' -d'
{
    "user" : "1"
}'
curl -XGET 'localhost:9200/tweets/_search?pretty' -d'
{
    "query" : {
        "terms" : {
            "user" : {
                "index" : "users",
                "type" : "user",
                "id" : "2",
                "path" : "followers"
            }
        }
    }
}'
```

外部分词文档的结构也可以包含内部对象的数组：

```
curl -XPUT localhost:9200/users/user/2 -d '{
 "followers" : [
   {
     "id" : "1"
   },
   {
     "id" : "2"
   }
 ]
}'
```

在这种情况下，查找路径将是followers.id
