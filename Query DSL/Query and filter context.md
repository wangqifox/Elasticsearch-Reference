# 查询过滤上下文

查询子句的行为取决于在`query context`还是`filter context`中使用。

## 上下文查询

上下文查询中使用的查询子句回答了：文档匹配该查询子句的程度。除了判断文档是否匹配，查询子句还计算了`_score`来表示某个文档相对于其他文档的匹配程度。

无论何时查询子句传递到`query`参数（比如`search`API中的`query`参数）中，上下文查询都是有效的。

## 上下文过滤

在上下文过滤中，查询子句回答了：文档是否匹配查询子句。回答是简单的`Yes`或者`No`——不计算分数。上下文过滤通常用在过滤结构化数据上，比如：

- `timestamp`是否落在2015到2016的区间上
- `status`字段是否被设置为"published"

频繁使用的过滤会被自动缓存，来提升性能

无论何时查询子句传递到`filter`参数（比如`bool`查询中的`filter`和`must_not`参数，`constant_score`查询或者`filter`聚合中的`filter`参数，）中，上下文过滤都是有效的。

下面的例子展示了`search`API中上下文查询、上下文过滤使用的查询子句。该查询匹配满足如下条件的文档：

- 包含单词`search`的`title`字段
- 包含单词`elasticsearch`的`content`字段
- `status`字段为`published`
- `publish_date`字段包含2015-01-01之后的日期

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
  "query": { 	// 1
    "bool": { 	// 2
      "must": [
        { "match": { "title":   "Search"        }}, 	// 3
        { "match": { "content": "Elasticsearch" }}  	// 4
      ],
      "filter": [ 	// 5
        { "term":  { "status": "published" }}, 	// 6
        { "range": { "publish_date": { "gte": "2015-01-01" }}} 	// 7
      ]
    }
  }
}'
```

- 1 `query`参数指定了上下文查询
- 2 3 4 上下文查询中使用了`bool`和两个`match`子句。意味着使用它们来计算文档匹配的程度
- 5 `filter`参数指定了上下文过滤
- 6 7 上下文过滤中使用了`term`和`range`子句。它们将不匹配的文档过滤掉，但是不会影响匹配文档的分值。

> 影响文档匹配分支的条件在上下文查询的查询子句中使用，其他查询子句在上下文过滤中使用。
