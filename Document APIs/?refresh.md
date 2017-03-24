# ?refresh

`Index`, `Update`, `Delete`, `Bulk` API支持设置`refresh`来控制该请求作出的修改何时对搜索可见。以下是允许的值：

- 空字符串和`true`

	操作发生后，立即刷新相关的主分片和副本分片（非整个索引），更新过的文档可以立即出现在搜索结果中。这个操作应该经过仔细的思考和验证，因为从索引和搜索角度来看，都会导致性能低下。

- `wait_for`

	返回前等待请求作出的修改因`refresh`操作而对搜索可见。这种情况下不强制立即刷新，而是等待刷新操作。Elasticsearch周期性地自动刷新经过修改的分片，时间间隔通过`index.refresh_interval`来设置，默认为1秒。该设置是动态的，调用`Refresh`接口或者将`refresh`设置为`true`也会引发刷新操作，导致设置了`refresh=wait_for`的请求返回。

- `false`(默认)

    相关的操作都不会刷新。该请求返回后，作出的修改会在某个时间点对搜索可见。

## 选择何种设置

除非你有充分的理由等待修改变得可见，使用`refresh=false`或者因为是默认的，在URL中不管`refresh`参数。这是最简单和最快速的选择。

如果你必须在请求的同时使修改对搜索可见，那么你必须在增加Elasticsearch负载(true)和等待请求返回(wait_for)之间做出权衡。对于这些决定有一些内容需要知道。

- 对索引做出的修改越多，`wait_for`相比于`true`节省的工作越多。这种情况下，索引只在间隔`index.refresh_interval`之后做出改变
- `true`创建了低效的索引结构（微小的segments），之后必须合并成更加高效的索引结构（更大的segments）。这意味着`true`的代价在索引阶段创建微小的segment、在搜索阶段搜索微小的segment、在合并阶段创建更大的segment。
- 不要连续地开启多个`refresh=wait_for`的请求。应该包装成一个单独的设置为`refresh=wait_for`的bulk请求，Elasticsearch会并行执行这些请求，当它们都结束后返回。
- 如果refresh的间隔设置为-1，禁止自动刷新，设置了`refresh=wait_for`的请求会一直等待下去，直到某些行为触发了刷新。将`index.refresh_interval`设置为小于默认值比如200ms可以使`refresh=wait_for`更快地返回，不过仍然会产生低效的segment。
- `refresh=wait_for`只影响请求本身，但是`refresh=true`会引发立即刷新从而影响其他正在执行的请求。通常来说，如果你不希望打断一个正在执行的系统，那么`refresh=wait_for`修改地更少。

## `refresh=wait_for`会强制刷新

当已经有`index.max_refresh_listeners`（默认是1000）个请求在分片中等待刷新，则下一个设置了`refresh=wait_for`的请求和设置了`refresh=true`的请求行为一致：它会强制刷新。这保证当设置了`refresh=wait_for`的请求返回时，它做出的改变对搜索可见，防止阻塞请求使用未经检验的资源。当一个请求因为超出了监听槽的数量而触发了强制刷新，返回中会包含`forced_refresh:true`。

bulk请求在每个分片中只占用一个槽，和修改分片的次数无关。

## 示例

以下会创建一个文档，立即刷新以对搜索可见：

```
curl -XPUT 'localhost:9200/test/test/1?refresh&pretty' -H 'Content-Type: application/json' -d'
{"test": "test"}
'
curl -XPUT 'localhost:9200/test/test/2?refresh=true&pretty' -H 'Content-Type: application/json' -d'
{"test": "test"}
'
```

以下会创建一个文档，不会做刷新以对搜索可见：

```
curl -XPUT 'localhost:9200/test/test/3?pretty' -H 'Content-Type: application/json' -d'
{"test": "test"}
'
curl -XPUT 'localhost:9200/test/test/4?refresh=false&pretty' -H 'Content-Type: application/json' -d'
{"test": "test"}
'
```

以下会创建一个文档，等待直到对搜索可见：

```
curl -XPUT 'localhost:9200/test/test/4?refresh=wait_for&pretty' -H 'Content-Type: application/json' -d'
{"test": "test"}
'
```
