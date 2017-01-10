# From / Size

使用`from`和`size`参数来进行分页。`from`参数定义了你要提取的第一个结果的偏移量。`size`参数定义了要返回的最大匹配数。

`from`和`size`可以作为请求参数来设置，也可以在请求体中来设置。`from`默认是`0`，`size`默认是`10`。

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
    "from" : 0, "size" : 10,
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}'
```

注意`from + size`不能比`index.max_result_window`更大，这个值默认是`10000`。`Scroll`或者`Search After`API是查询大量数据更有效的办法。
