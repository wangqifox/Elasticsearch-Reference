# 搜索

搜索API用来执行搜索查询，返回匹配的搜索结果。可以使用`query string as a parameter`或者`request body`来查询。

## 多索引，所类型

所有搜索API都可以应用于索引内的多个类型，跨越支持多索引语法的多个索引。例如，我们可以搜索`twitter`索引中所有的类型：

```
curl -XGET 'localhost:9200/twitter/_search?q=user:kimchy&pretty'
```

也可以搜索特定的类型：

```
curl -XGET 'localhost:9200/twitter/tweet,user/_search?q=user:kimchy&pretty'
```

我们也可以搜索多个索引中的`tweets`类型（比如每个用户都有自己的索引）：

```
curl -XGET 'localhost:9200/kimchy,elasticsearch/tweet/_search?q=tag:wow&pretty'
```

或者使用`_all`来搜索所有索引中的`tweets`类型

```
curl -XGET 'localhost:9200/_all/tweet/_search?q=tag:wow&pretty'
```

甚至搜索所有索引中的所有类型：

```
curl -XGET 'localhost:9200/_search?q=tag:wow&pretty'
```

elasticsearch默认会拒绝查询大于1000个分片的搜索请求。原因是这么大数量的分片使得协调节点耗费大量的CPU和内存。组织数据使得大分片更少通常是个好主意。如果你想绕过此限制（不鼓励这么做），可以更新`action.search.shard_count.limit`为一个更大的值。
