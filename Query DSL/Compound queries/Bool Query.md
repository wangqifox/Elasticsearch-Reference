# bool查询

这个查询匹配的文档是其他查询的布尔组合。`bool`查询对应于Lucene的`BooleanQuery`。它使用一个或多个布尔子句，每个子句都有一个发生类型。发生类型有这些：

|发生|说明|
|---|----|
|must|查询子句必须出现在匹配的文档中，对分值有影响|
|filter|查询子句必须出现在匹配的文档中。但是不像`must`，查询的分值会被忽略。过滤子句在过滤器上下文中执行，意味着分值被忽略，且子句考虑用于缓存。|
|should|查询子句应该出现在匹配的文档中。在没有`must`或`filter`子句的布尔查询中，一个或多个`should`子句必须匹配文档。`should`子句匹配的最小值可以使用`minimum_should_match`来设置。|
|must_not|查询子句必须不出现在匹配文档中。子句在过滤器上下文中执行，意味着分值会被忽略且子句考虑用于缓存。因为分值被忽略，返回分值为0的所有文档。|

> 如果查询在过滤器上下文中使用，且包含`should`子句，则需要匹配至少一个`should`子句

bool查询也支持`disable_coord`参数（默认是`false`）。坐标相似性基于文档包含的所有分词的分数来计算得分因子。有关更多详细信息，请参阅Lucene的BooleanQuery。

bool查询采用“匹配越多越好”的方法，所以，每个匹配的`must`或`should`子句的总分作为每个文档最终的得分。

```
curl -XPOST 'localhost:9200/_search?pretty' -d'
{
  "query": {
    "bool" : {
      "must" : {
        "term" : { "user" : "kimchy" }
      },
      "filter": {
        "term" : { "tag" : "tech" }
      },
      "must_not" : {
        "range" : {
          "age" : { "from" : 10, "to" : 20 }
        }
      },
      "should" : [
        { "term" : { "tag" : "wow" } },
        { "term" : { "tag" : "elasticsearch" } }
      ],
      "minimum_should_match" : 1,
      "boost" : 1.0
    }
  }
}'
```

## 使用`bool.filter`滚动

`filter`元素下的查询对得分没有影响——返回0作为得分。分数只受被指定查询的影响。比如，下面的三个查询返回`status`字段包含`active`分词的所有文档。

第一个查询将所有的文档的分数都赋值为0，因为没有指定计算分数的查询：

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
  "query": {
    "bool": {
      "filter": {
        "term": {
          "status": "active"
        }
      }
    }
  }
}'
```

下面这个`bool`查询有一个`match_all`查询，将所有的文档都分值都设为1.0：

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "term": {
          "status": "active"
        }
      }
    }
  }
}'
```

`constant_score`查询和上面第二个例子的行为是一样的。`constant_score`查询对所有匹配的文档都分配分值为1.0.

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "status": "active"
        }
      }
    }
  }
}'
```

## 使用命名查询来查看匹配哪个子句

如果你需要知道bool查询匹配哪些文档，你可以使用`named queries`来给每个子句分配一个名字。
