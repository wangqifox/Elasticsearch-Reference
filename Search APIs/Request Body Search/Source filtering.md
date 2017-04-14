# Source filtering

用来控制命中的搜索结果中`_source`字段如何返回。

默认情况下，返回`_source`字段的内容，除非你使用了`stored_fields`参数或者禁用了`_source`字段。

你可以使用`_source`参数关闭`_source`检索:

将`_source`设置为`false`关闭`_source`检索：

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "_source": false,
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
'
```

`_source`也接受一个或多个通配符来控制返回`_source`的哪部分：

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "_source": "obj.*",
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
'
```

或者：

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "_source": [ "obj1.*", "obj2.*" ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
'
```

最后，为了完全控制，你可以使用`includes`和`excludes`模式：

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "_source": {
        "includes": [ "obj1.*", "obj2.*" ],
        "excludes": [ "*.description" ]
    },
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
'
```
