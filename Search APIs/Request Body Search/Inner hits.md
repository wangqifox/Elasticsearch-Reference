# Inner hits

`parent/child`和`nested`功能允许返回不同范围中匹配的文档。在`parent/child`情况中，根据子文档的匹配返回父文档或者根据父文档的匹配返回子文档。在`nested`情况下，根据嵌套对象的匹配返回文档。

在这两种情况下，文档中不同范围匹配的返回数据被隐藏了。在很多情况下，我们需要知道内部嵌套的对象或者具有父子关系文档的确切信息。`inner hits`就是用来解决这个问题的。`inner hits`返回搜索匹配的结果以及不同范围中的附加嵌套对象。

`Inner hits`通过在`nested`,`has_child`,`has_parent`查询和过滤中定义`inner_hits`来使用。结构看起来如下：

```
"<query>" : {
    "inner_hits" : {
        <inner_hits_options>
    }
}
```

一旦支持`inner_hits`的查询中定义了`inner_hits`，每个搜索结果都会包含`inner_hits`JSON对象：

```
"hits": [
     {
        "_index": ...,
        "_type": ...,
        "_id": ...,
        "inner_hits": {
           "<inner_hits_name>": {
              "hits": {
                 "total": ...,
                 "hits": [
                    {
                       "_type": ...,
                       "_id": ...,
                       ...
                    },
                    ...
                 ]
              }
           }
        },
        ...
     },
     ...
]
```

## 选项

`inner hits`支持以下选项：

|选项|说明|
|---|----|
|from|在返回的常规搜索匹配中，每个inner_hits的第一个匹配值的偏移量|
|size|`inner_hits`返回值的最大数量。默认返回前三个匹配值|
|sort|`inner_hits`的排序依据。默认情况下根据评分来排序|
|name|用于返回值中定义`inner hit`的名字。当一个搜索请求中定义了多个`inner hits`时很有用。默认值取决于内部命中定义的查询。对于`has_child`查询和过滤器来说这是子类型，对于`has_parent`查询和过滤器来说这是父类型，对于嵌套查询来说这是嵌套路径。|

`inner hits`也支持以下文档特性：

- `Highlighting`
- `Explain`
- `Source filtering`
- `Script fields`
- `Doc value fields`
- `Include versions`

## 嵌套的`inner hits`

嵌套的`inner_hits`可以用于将嵌套的内部对象作为搜索命中的内部命中。

```
curl -XPUT 'localhost:9200/test?pretty' -H 'Content-Type: application/json' -d'
{
  "mappings": {
    "doc": {
      "properties": {
        "comments": {
          "type": "nested"
        }
      }
    }
  }
}
'
curl -XPUT 'localhost:9200/test/doc/1?refresh&pretty' -H 'Content-Type: application/json' -d'
{
  "title": "Test title",
  "comments": [
    {
      "author": "kimchy",
      "text": "comment text"
    },
    {
      "author": "nik9000",
      "text": "words words words"
    }
  ]
}
'
curl -XPOST 'localhost:9200/test/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "nested": {
      "path": "comments",
      "query": {
        "match": {"comments.text" : "words"}
      },
      "inner_hits": {} 	// 1
    }
  }
}
'
```

- 1 定义在嵌套查询中的`inner hit`。不需要定义其他的选项。

上面的搜索请求生成的一个返回示例：

```
{
  ...,
  "hits": {
    "total": 1,
    "max_score": 0.9651416,
    "hits": [
      {
        "_index": "test",
        "_type": "doc",
        "_id": "1",
        "_score": 0.9651416,
        "_source": ...,
        "inner_hits": {
          "comments": { 			// 1
            "hits": {
              "total": 1,
              "max_score": 0.9651416,
              "hits": [
                {
                  "_nested": {
                    "field": "comments",
                    "offset": 1
                  },
                  "_score": 0.9651416,
                  "_source": {
                    "author": "nik9000",
                    "text": "words words words"
                  }
                }
              ]
            }
          }
        }
      }
    ]
  }
}
```

- 1 搜索请求中定义的`inner hit`使用的名字。可以通过`name`选项自定义名字

上面的例子中`_nested`元数据是重要的，因为它定义了`inner hit`来自哪个内部嵌套对象。`field`定义了嵌套命中来源的对象数组，以及`_source`中相对于它位置的`offset`。这是因为对`inner_hits`中命中对象的真实位置进行排序和评分通常与嵌套内部对象定义的位置是不同的。

默认情况下，`inner_hits`中的命中对象也返回`_source`，但是已经经过修改了。通过`_source`的过滤功能，可以返回或禁用元数据的一部分。如果嵌套级别上定义了`stored`字段，这些数据也可以通过`fields`返回。

一个重要的默认特性是`inner_hits`内部命中数据返回的`_source`是相对于`_nested`元数据的。所以，上面的例子中嵌套的命中数据值返回了评论部分，而不是评论所包含的顶层文档的整个元数据。

## 嵌套的内部命中以及`_source`

嵌套文档没有`_source`字段，因为文档的元数据被保存在根文档的`_source`字段中。为了只包含嵌套文档的元数据，根文档的源数据经过解析，在`inner hit`中只包含嵌套文档相关的源数据。对每个匹配的嵌套文档做这样的操作对于执行搜索请求在时间上是有影响的，特别是当`size`和`inner hit`的`size`比默认值设置的更大。为了避免嵌套`inner hits`抽取源数据时的代价，可以禁止包含元数据，仅仅依赖`stored`字段。

```
curl -XPUT 'localhost:9200/test?pretty' -H 'Content-Type: application/json' -d'
{
  "mappings": {
    "doc": {
      "properties": {
        "comments": {
          "type": "nested",
          "properties": {
            "text": {
              "type": "text",
              "store": true
            }
          }
        }
      }
    }
  }
}
'
curl -XPUT 'localhost:9200/test/doc/1?refresh&pretty' -H 'Content-Type: application/json' -d'
{
  "title": "Test title",
  "comments": [
    {
      "author": "kimchy",
      "text": "comment text"
    },
    {
      "author": "nik9000",
      "text": "words words words"
    }
  ]
}
'
curl -XPOST 'localhost:9200/test/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "nested": {
      "path": "comments",
      "query": {
        "match": {"comments.text" : "words"}
      },
      "inner_hits": {
        "_source" : false,
        "stored_fields" : ["comments.text"]
      }
    }
  }
}
'
```

## 嵌套对象字段和`inner hits`的层次

如果映射的嵌套对象字段有多个层次，每个层次可以通过点号路径来访问。例如，有一个`comments`嵌套字段，包含一个`votes`嵌套字段，并且`votes`应随根命中返回，则可以定义以下路径：

```
curl -XPUT 'localhost:9200/test?pretty' -H 'Content-Type: application/json' -d'
{
  "mappings": {
    "doc": {
      "properties": {
        "comments": {
          "type": "nested",
          "properties": {
            "message": {
              "type": "text",
              "store": true
            },
            "votes": {
              "type": "nested"
            }
          }
        }
      }
    }
  }
}
'
curl -XPUT 'localhost:9200/test/doc/1?refresh&pretty' -H 'Content-Type: application/json' -d'
{
  "title": "Test title",
  "comments": [
    {
      "author": "kimchy",
      "text": "comment text",
      "votes": []
    },
    {
      "author": "nik9000",
      "text": "words words words",
      "votes": [
        {"value": 1 , "voter": "kimchy"},
        {"value": -1, "voter": "other"}
      ]
    }
  ]
}
'
curl -XPOST 'localhost:9200/test/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "nested": {
      "path": "comments.votes",
        "query": {
          "match": {
            "comments.votes.voter": "kimchy"
          }
        },
        "inner_hits" : {}
    }
  }
}
'
```

返回如下：

```
{
  ...,
  "hits": {
    "total": 1,
    "max_score": 0.6931472,
    "hits": [
      {
        "_index": "test",
        "_type": "doc",
        "_id": "1",
        "_score": 0.6931472,
        "_source": ...,
        "inner_hits": {
          "comments.votes": { 
            "hits": {
              "total": 1,
              "max_score": 0.6931472,
              "hits": [
                {
                  "_nested": {
                    "field": "comments",
                    "offset": 1,
                    "_nested": {
                      "field": "votes",
                      "offset": 0
                    }
                  },
                  "_score": 0.6931472,
                  "_source": {
                    "value": 1,
                    "voter": "kimchy"
                  }
                }
              ]
            }
          }
        }
      }
    ]
  }
}
```

简介引用只支持嵌套`inner hits`

## 父子`inner hits`

父子关系的文档可以使用`inner_hits`来包含父文档或子文档

```
curl -XPUT 'localhost:9200/test?pretty' -H 'Content-Type: application/json' -d'
{
  "settings": {
    "mapping.single_type": false
  },
  "mappings": {
    "my_parent": {},
    "my_child": {
      "_parent": {
        "type": "my_parent"
      }
    }
  }
}
'
curl -XPUT 'localhost:9200/test/my_parent/1?refresh&pretty' -H 'Content-Type: application/json' -d'
{
  "test": "test"
}
'
curl -XPUT 'localhost:9200/test/my_child/1?parent=1&refresh&pretty' -H 'Content-Type: application/json' -d'
{
  "test": "test"
}
'
curl -XPOST 'localhost:9200/test/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "has_child": {
      "type": "my_child",
      "query": {
        "match": {
          "test": "test"
        }
      },
      "inner_hits": {}    		// 1
    }
  }
}
'
```

- 1 `inner hit`定义和嵌套例子中的一样

上面的搜索请求返回的例子：

```
{
  ...,
  "hits": {
    "total": 1,
    "max_score": 1.0,
    "hits": [
      {
        "_index": "test",
        "_type": "my_parent",
        "_id": "1",
        "_score": 1.0,
        "_source": ...,
        "inner_hits": {
          "my_child": {
            "hits": {
              "total": 1,
              "max_score": 0.18232156,
              "hits": [
                {
                  "_type": "my_child",
                  "_id": "1",
                  "_score": 0.18232156,
                  "_routing": "1",
                  "_parent": "1",
                  "_source": {
                    "test": "test"
                  }
                }
              ]
            }
          }
        }
      }
    ]
  }
}
```
