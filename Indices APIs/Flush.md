# Flush

flush API允许通过API刷新一个或多个索引。索引的刷新过程基本上通过将数据刷新到索引存储并清除内部事务日志来释放内存。默认情况下，Elasticsearch使用内存启发式方法来根据需要自动触发刷新操作，以清除内存。

```
curl -XPOST 'localhost:9200/twitter/_flush?pretty'
```

## 请求参数

flush API接受以下请求参数：

|参数|说明|
|----|---|
|wait_if_ongoing|如果设置为`true`，如果另一个flush操作已经在运行了，flush操作会阻塞直到可以运行。默认是`false`，如果另一个flush操作已经在运行，flush操作会在分片级别抛出异常。|
|force|flush操作是否应该强制执行，即使是没有必要的，比如没有修改被提交到索引中。如果事务日志id在没有修改提交的情况下也要增加，那么这个设置就是有用的。（此设置可视为内部设置）|

## 多个索引

flush API可以在一次调用中应用于多个索引，甚至是`_all`索引。

```
curl -XPOST 'localhost:9200/kimchy,elasticsearch/_flush?pretty'
curl -XPOST 'localhost:9200/_flush?pretty'
```
