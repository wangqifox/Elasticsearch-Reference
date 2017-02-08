# 聚合执行(Executing Aggregations)

聚合提供了从数据中分组和提取统计信息的功能。最简单的考虑聚合的方法是大致等价于SQL GROUP BY和SQL聚合函数。在Elasticsearch中，你可以在搜索执行返回匹配的同时返回一个针对响应的聚合结果。这是非常强大以及高效的，你可以使用简单的API来运行查询和多个聚合，并一次性获取两个（或任意一个）操作的结果，避免了网络的往返。

首先，该示例按状态对所有的账户进行分组，然后返回按计数降序排序的默认前10个默认状态：

```
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}
'
```

在SQL中，上面的聚合操作在概念上接近：

```
SELECT state, COUNT(*) FROM bank GROUP BY state ORDER BY COUNT(*) DESC
```

响应如下（部分）：

```
{
  "took": 29,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : 0.0,
    "hits" : [ ]
  },
  "aggregations" : {
    "group_by_state" : {
      "doc_count_error_upper_bound": 20,
      "sum_other_doc_count": 770,
      "buckets" : [ {
        "key" : "ID",
        "doc_count" : 27
      }, {
        "key" : "TX",
        "doc_count" : 27
      }, {
        "key" : "AL",
        "doc_count" : 25
      }, {
        "key" : "MD",
        "doc_count" : 25
      }, {
        "key" : "TN",
        "doc_count" : 23
      }, {
        "key" : "MA",
        "doc_count" : 21
      }, {
        "key" : "NC",
        "doc_count" : 21
      }, {
        "key" : "ND",
        "doc_count" : 21
      }, {
        "key" : "ME",
        "doc_count" : 20
      }, {
        "key" : "MO",
        "doc_count" : 20
      } ]
    }
  }
}
```

我们看到，ID中有27个账户，TX中有27个账户，AL中有25个账户，等等。

注意，我们设置`size=0`隐藏搜索匹配的结果，因为我们只想在响应中看到聚合的结果。

基于之前的聚合，以下示例按状态计算平均账户余额（仅针对按降序排序的前10个洲）

```
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```

注意我们如何在group_by_state聚合内嵌入average_balance聚合。这是所有聚合的常见模式。你可以随意在聚合中内嵌聚合，以提取你需要从数据中获得的透视摘要。

基于之前的聚合，将平均余额按递减排序。

```
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword",
        "order": {
          "average_balance": "desc"
        }
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
'
```

下面的例子展示了我们如何按年纪归类(20-29,30-39,40-49)来分组，再按性别分组，最后获取账户余额、每个年纪类、每个性别的分组。

```
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "group_by_age": {
      "range": {
        "field": "age",
        "ranges": [
          {
            "from": 20,
            "to": 30
          },
          {
            "from": 30,
            "to": 40
          },
          {
            "from": 40,
            "to": 50
          }
        ]
      },
      "aggs": {
        "group_by_gender": {
          "terms": {
            "field": "gender.keyword"
          },
          "aggs": {
            "average_balance": {
              "avg": {
                "field": "balance"
              }
            }
          }
        }
      }
    }
  }
}
'
```

还有很多其他聚合功能，我们在这里不会详细介绍。如果你想进一步做实验，请查看`aggregations reference guide`。
