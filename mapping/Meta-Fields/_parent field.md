# _parent 字段

通过标记一个映射是另一个映射的父映射，可以建立相同的索引中文档之间的父子关系：

```
curl -XPUT 'localhost:9200/my_index?pretty' -H 'Content-Type: application/json' -d'
{
  "mappings": {
    "my_parent": {},
    "my_child": {
      "_parent": {
        "type": "my_parent" 		// 1
      }
    }
  }
}
'
curl -XPUT 'localhost:9200/my_index/my_parent/1?pretty' -H 'Content-Type: application/json' -d'		// 2
{
  "text": "This is a parent document"
}
'
curl -XPUT 'localhost:9200/my_index/my_child/2?parent=1&pretty' -H 'Content-Type: application/json' -d'		// 3
{
  "text": "This is a child document"
}
'
curl -XPUT 'localhost:9200/my_index/my_child/3?parent=1&refresh=true&pretty' -H 'Content-Type: application/json' -d'		// 4
{
  "text": "This is another child document"
}
'
curl -XGET 'localhost:9200/my_index/my_parent/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "has_child": { 		// 5
      "type": "my_child",
      "query": {
        "match": {
          "text": "child document"
        }
      }
    }
  }
}
'
```

- 1 `my_parent`类型是`my_child`类型的父类型
- 2 索引父文档
- 3 4 索引两个子文档，指定父文档的ID
- 5 寻找所有包含匹配查询子文档的父文档

详细信息请查看`has_child`, `has_parent`, `children`, `inner hits`

`_parent`字段的值可以在聚合和脚本中访问，也可以在`parent_id`查询中访问:

```
curl -XGET 'localhost:9200/my_index/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "parent_id": { 				// 1
      "type": "my_child",
      "id": "1"
    }
  },
  "aggs": {
    "parents": {
      "terms": {
        "field": "_parent", 	// 2
        "size": 10
      }
    }
  },
  "script_fields": {
    "parent": {
      "script": {
         "inline": "doc['_parent']" 		// 3
      }
    }
  }
}
'
```

- 1 查询`_parent`字段的id（也见`has_parent`查询及`has_child`查询）
- 2 在`_parent`字段聚合（也见`children`聚合）
- 3 脚本中访问`_parent`字段

## Parent-child的限制

- 父子的类型必须是不同的，父子关系不可以建立在相同类型的文档上
- `_parent.type`的设置只能指向尚不存在的类型。这意味着一个类型创建之后不能变成父类型
- 父文档和子文档必须在同一个分片中索引。`parent`的ID被用来作为子文档的路由值，为了保证子文档和父文档索引在同一个分片中。这意味着对子文档进行`getting`,`deleting`,`updating`操作的时候，必须提供父文档的值

## 全局序数

父子文档使用`global ordinals`来加速连接。对分片的任意修改都需要重建全局序数。分片中存储越多的父文档id，重建时耗费的时候就越多。

默认情况下，全局序数的建立是同步的。当索引有所改变，`_parent`字段的全局序数在刷新的时候会立即重建。这会显著增加刷新的时间。不过大多数情况下，这是正确的权衡，否则当第一次父子查询或聚合发生的时候全局序数就会重建。这会对查询造成显著的延迟，并且这通常会更糟，因为在发生许多写入时，可能会在单个刷新间隔内重新创建`_parent`字段的多个全局序数。

当`parent/child`使用不频繁，但是写入很频繁的情况下，禁用同步加载是有意义的。

```
curl -XPUT 'localhost:9200/my_index?pretty' -H 'Content-Type: application/json' -d'
{
  "mappings": {
    "my_parent": {},
    "my_child": {
      "_parent": {
        "type": "my_parent",
        "eager_global_ordinals": false
      }
    }
  }
}
'
```

全局序数使用的堆数量检查如下所示：

```
# Per-index
curl -XGET 'localhost:9200/_stats/fielddata?human&fields=_parent&pretty'
# Per-node per-index
curl -XGET 'localhost:9200/_nodes/stats/indices/fielddata?human&fields=_parent&pretty'
```
