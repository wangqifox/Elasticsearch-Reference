# 刷新

刷新API可以显式地刷新一个或多个索引，使得上次刷新后执行的所有操作可用于搜索。（近）实时能力取决于使用的索引引擎。比如，内部引擎需要调用刷新，但是默认情况下周期性地调度刷新。

```
curl -XPOST 'localhost:9200/twitter/_refresh?pretty'
```

## 多个索引

刷新API可以通过单个调用应用于多个索引，甚至应用于`_all`索引。

```
curl -XPOST 'localhost:9200/kimchy,elasticsearch/_refresh?pretty'
curl -XPOST 'localhost:9200/_refresh?pretty'
```
