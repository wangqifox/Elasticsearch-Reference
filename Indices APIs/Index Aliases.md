# Index Aliases

Elasticsearch的API针对特定的索引接受一个索引名，适当的时候接受多个索引。索引别名API为索引赋予一个别名，所有的API会自动把别名转换成真是的索引名。一个别名可以映射到一个或多个索引，当指定时，别名会自动扩展到别名索引。别名也可以与搜索时自动应用的过滤器和路由值相关联。别名不能与索引具有相同的名称。

下面的例子将索引`test1`与别名`alias1`相关联：

```
curl -XPOST 'localhost:9200/_aliases?pretty' -H 'Content-Type: application/json' -d'
{
    "actions" : [
        { "add" : { "index" : "test1", "alias" : "alias1" } }
    ]
}
'
```

删除相同的别名：

```
curl -XPOST 'localhost:9200/_aliases?pretty' -H 'Content-Type: application/json' -d'
{
    "actions" : [
        { "remove" : { "index" : "test1", "alias" : "alias1" } }
    ]
}
'
```

重命名别名就是简单地使用相同的API执行`remove`和`add`操作。该操作是自动的，不用担心别名不指向索引的短暂时刻：

```
curl -XPOST 'localhost:9200/_aliases?pretty' -H 'Content-Type: application/json' -d'
{
    "actions" : [
        { "remove" : { "index" : "test1", "alias" : "alias1" } },
        { "add" : { "index" : "test2", "alias" : "alias1" } }
    ]
}
'
```

将别名关联到多个索引就是简单地执行多个`add`操作：

```
curl -XPOST 'localhost:9200/_aliases?pretty' -H 'Content-Type: application/json' -d'
{
    "actions" : [
        { "add" : { "index" : "test1", "alias" : "alias1" } },
        { "add" : { "index" : "test2", "alias" : "alias1" } }
    ]
}
'
```

多个索引可以使用`indices`数组语法通过一个操作来指定：

```
curl -XPOST 'localhost:9200/_aliases?pretty' -H 'Content-Type: application/json' -d'
{
    "actions" : [
        { "add" : { "indices" : ["test1", "test2"], "alias" : "alias1" } }
    ]
}
'
```

为了在单个操作中指定多个别名，存在相应的`aliases`数组语法。

对于上面的例子来说，也可以使用glob模式将一个别名与多个共享一部分相同名字的索引相关联：

```
curl -XPOST 'localhost:9200/_aliases?pretty' -H 'Content-Type: application/json' -d'
{
    "actions" : [
        { "add" : { "index" : "test*", "alias" : "all_test_indices" } }
    ]
}
'
```

在该例子中，别名是一个point-in-time别名，将当前所有匹配的索引整合起来。不过如果有匹配的索引增加或者删除了，别名不会自动更新。

对一个指向多个索引的别名进行索引是错误的。

也可以在一个操作中使用别名交换索引：

```
curl -XPUT 'localhost:9200/test?pretty' // 1
curl -XPUT 'localhost:9200/test_2?pretty'   // 2
curl -XPOST 'localhost:9200/_aliases?pretty' -H 'Content-Type: application/json' -d'
{
    "actions" : [
        { "add":  { "index": "test_2", "alias": "test" } },
        { "remove_index": { "index": "test" } }     // 3
    ]
}
'
```

- 1 我们错误增加的一个索引
- 2 我们应该增加的索引
- 3 `remove_index`就像`Delete Index`

## Filtered Aliases

带有过滤器的别名提供了一种创建相同索引的不同“视图”的简单方法。过滤器可以使用Query DSL来定义，并应用于所有搜索，计数，按查询删除和更多类似于此别名的操作。

为了创建带过滤器的别名，首先我们需要确保字段已经存在于映射中：

```
curl -XPUT 'localhost:9200/test1?pretty' -H 'Content-Type: application/json' -d'
{
  "mappings": {
    "type1": {
      "properties": {
        "user" : {
          "type": "keyword"
        }
      }
    }
  }
}
'
```

现在我们可以创建一个在字段`user`上使用过滤器的别名：

```
curl -XPOST 'localhost:9200/_aliases?pretty' -H 'Content-Type: application/json' -d'
{
    "actions" : [
        {
            "add" : {
                 "index" : "test1",
                 "alias" : "alias2",
                 "filter" : { "term" : { "user" : "kimchy" } }
            }
        }
    ]
}
'
```

## Routing

可以将路由值关联到别名。该特性可以和带过滤器的别名一起使用来避免一些不必要的分片操作。

下面的命令创建了一个指向索引`test`的别名`alias1`。`alias1`创建之后，所有对该别名的操作都自动修改为使用值1进行路由：

```
curl -XPOST 'localhost:9200/_aliases?pretty' -H 'Content-Type: application/json' -d'
{
    "actions" : [
        {
            "add" : {
                 "index" : "test",
                 "alias" : "alias1",
                 "routing" : "1"
            }
        }
    ]
}
'
```

也可以为搜索和索引操作指定不同的路由值：

如上面的例子所示，搜索路由可以包含多个使用逗号分隔的值。索引路由只能包含单个值。

如果使用路由别名的搜索操作同时有路由参数，使用搜索别名路由和参数中指定的路由的交集。例如下面的命令会使用`2`作为路由值：

```
curl -XGET 'localhost:9200/alias2/_search?q=user:kimchy&routing=2,3&pretty'
```

如果使用索引路由别名的索引操作也具有父路由，则父路由将被忽略。

## Add a single alias

还可以使用endpoint来增加别名

```
PUT /{index}/_alias/{name}
```

其中：

|参数|说明|
|----|---|
|index|别名指向的索引。可以是它们中任意一个`* | _all | glob pattern | name1, name2, …`|
|name|别名的名称。这是必需的|
|routing|和别名相关联的路由，可选|
|filter|和别名相关联的过滤器，可选|

你也可以使用复数形式的`_aliases`

示例：

增加基于时间的别名

```
curl -XPUT 'localhost:9200/logs_201305/_alias/2013?pretty'
```

增加一个用户别名

首先创建一个索引，对`user_id`字段增加一个映射：

```
curl -XPUT 'localhost:9200/users?pretty' -H 'Content-Type: application/json' -d'
{
    "mappings" : {
        "user" : {
            "properties" : {
                "user_id" : {"type" : "integer"}
            }
        }
    }
}
'
```

然后对特定的用户增加别名：

```
curl -XPUT 'localhost:9200/users/_alias/user_12?pretty' -H 'Content-Type: application/json' -d'
{
    "routing" : "12",
    "filter" : {
        "term" : {
            "user_id" : 12
        }
    }
}
'
```

## 索引创建时的别名

别名也可以在索引创建的时候指定：

```
curl -XPUT 'localhost:9200/logs_20162801?pretty' -H 'Content-Type: application/json' -d'
{
    "mappings" : {
        "type" : {
            "properties" : {
                "year" : {"type" : "integer"}
            }
        }
    },
    "aliases" : {
        "current_day" : {},
        "2016" : {
            "filter" : {
                "term" : {"year" : 2016 }
            }
        }
    }
}
'
```

## 删除别名

endpoint是`/{index}/_alias/{name}`

其中：

|参数|说明|
|---|---|
|index|`* | _all | glob pattern | name1, name2, …`|
|name|`* | _all | glob pattern | name1, name2, …`|

也可以选择复数形式的`_aliases`：

```
curl -XDELETE 'localhost:9200/logs_20162801/_alias/current_day?pretty'
```

## 检索已存在的别名

获取索引别名API允许通过别名和索引名称进行过滤。此api重定向到主服务器并获取请求的索引别名（如果可用）。这个api只能连续寻找索引别名。

可能的选项：

|选项|说明|
|---|---|
|index|要获取别名的索引名称。通过通配符支持部分名称，也可以使用逗号分隔多个索引名称。还可以使用索引的别名。|
|alias|在响应中返回的别名的名称。和索引选项一样，此选项支持通配符和选项，指定多个别名以逗号分隔|
|ignore_unavailable|如果指定的索引名称不存在要做的。如果设置为`true`，那么这些索引将被忽略。|

endpoint是`/{index}/_alias/{alias}`

例子：

获取索引用户所有的别名：

```
curl -XGET 'localhost:9200/logs_20162801/_alias/*?pretty'
```

响应：

```
{
 "logs_20162801" : {
   "aliases" : {
     "2016" : {
       "filter" : {
         "term" : {
           "year" : 2016
         }
       }
     }
   }
 }
}
```

获取任意索引中名称为2016的别名：

```
curl -XGET 'localhost:9200/_alias/2016?pretty'
```

响应：

```
{
  "logs_20162801" : {
    "aliases" : {
      "2016" : {
        "filter" : {
          "term" : {
            "year" : 2016
          }
        }
      }
    }
  }
}
```

获取以20开头的所有别名：

```
curl -XGET 'localhost:9200/_alias/20*?pretty'
```

响应：

```
{
  "logs_20162801" : {
    "aliases" : {
      "2016" : {
        "filter" : {
          "term" : {
            "year" : 2016
          }
        }
      }
    }
  }
}
```

还有一个HEAD变体的get索引别名api来检查索引别名是否存在。索引别名存在api支持与get索引别名api相同的选项。例子：

```
curl -XHEAD 'localhost:9200/_alias/2016?pretty'
curl -XHEAD 'localhost:9200/_alias/20*?pretty'
curl -XHEAD 'localhost:9200/logs_20162801/_alias/*?pretty'
```
