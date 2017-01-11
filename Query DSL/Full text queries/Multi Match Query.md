# Multi Match Query

`multi_math`查询建立在`match`查询之上，用来做多字段查询。

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
  "query": {
    "multi_match" : {
      "query":    "this is a test", 	// 1
      "fields": [ "subject", "message" ] 	// 2
    }
  }
}'
```

- 1 查询语句
- 2 查询的字段

## 字段和每个字段的`boost`

字段可以通过通配符来指定：

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
  "query": {
    "multi_match" : {
      "query":    "Will Smith",
      "fields": [ "title", "*_name" ] 	// 1
    }
  }
}'
```

- 1 查询`title`, `first_name`, `last_name`字段

可以使用插入符号(^)来便是字段的`boost`：

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
  "query": {
    "multi_match" : {
      "query" : "this is a test",
      "fields" : [ "subject^3", "message" ] 	// 1
    }
  }
}'
```

- 1 `subject`字段重要度是`message`字段的三倍

## `multi_match`查询的类型

`multi_match`查询的方式内部是根据`type`参数的，可以被设置成：

|参数|说明|
|---|----|
|best_fields|(默认)查询匹配任意字段的文档，但是使用匹配度最高的`_score`|
|most_fields|查询匹配任意字段的文档，结合每个字段的`_score`|
|cross_fields|使用相同的分析器处理字段，就像它们是一个大字段。在任意字段中查找每个单词|
|phrase|在每个字段执行`match_phrase`，结合每个字段的`_score`|
|phrase_prefix|在每个字段执行`match_phrase_prefix`，结合每个字段的`_score`|

## `best_fields`

当你想要在同一个字段上索引匹配度最高的多个单词时，`best_fields`类型最有用。比如，在一个字段中的`brown fox`比`brown`在一个字段`fox`在另一个字段要更有意义。

`best_fields`类型给每个字段都生成一个`match`查询，然后将它们包裹在`dis_max`查询中，来发现匹配度最高的字段。比如这个查询：

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
  "query": {
    "multi_match" : {
      "query":      "brown fox",
      "type":       "best_fields",
      "fields":     [ "subject", "message" ],
      "tie_breaker": 0.3
    }
  }
}'
```

会被当成这样来执行：

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
  "query": {
    "dis_max": {
      "queries": [
        { "match": { "subject": "brown fox" }},
        { "match": { "message": "brown fox" }}
      ],
      "tie_breaker": 0.3
    }
  }
}'
```

通常`best_fields`类型会使用匹配度最高的字段的分值，但是如果指定了`tie_breaker`，就会按照下面的步骤来计算分数：

- 先获得匹配度最高的字段的分值
- 给其他匹配的字段加上`tie_breaker * _score`

正如`match`查询中所示，`best_fields`也接受`analyzer`, `boost`, `operator`, `minimum_should_match`, `fuzziness`, `lenient`, `prefix_length`, `max_expansions`, `rewrite`, `zero_terms_query`, `cutoff_frequency`。

### `operator`和`minimum_should_match`

`best_fields`和`most_fields`类型都是以字段为中心的，它们会为每个字段生成一个`match`查询。这意味着`operator`和`minimum_should_match`参数会被分别应用到每个字段，这可能不是你想要的。

比如：

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
  "query": {
    "multi_match" : {
      "query":      "Will Smith",
      "type":       "best_fields",
      "fields":     [ "first_name", "last_name" ],
      "operator":   "and" 	// 1
    }
  }
}'
```

- 1 所有的分词都必须出现

这个查询会以如下的形式执行：

```
(+first_name:will +first_name:smith) | (+last_name:will  +last_name:smith)
```

换句话说，一个文档的字段只有匹配所有的分词才算是匹配。

`cross_fields`是更好的解决方案。

### `most_fields`

当以不同方式查询包含相同文本的多个字段时，`most_fields`类型最有用。比如，主字段可以包含同义词、词干和没有变音符号的分词。第二字段可以包含原始分词，第三字段可以包含shingles。通过组合三个字段的分数，我们可以在主字段中匹配尽可能多的文档，使用第二和第三字段将最相似的结果推送到列表的顶部。

查询：

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
  "query": {
    "multi_match" : {
      "query":      "quick brown fox",
      "type":       "most_fields",
      "fields":     [ "title", "title.original", "title.shingles" ]
    }
  }
}'
```

会以这样的方式来执行：

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title":          "quick brown fox" }},
        { "match": { "title.original": "quick brown fox" }},
        { "match": { "title.shingles": "quick brown fox" }}
      ]
    }
  }
}'
```

每个`match`查询子句的分数相加，然后除以`match`子句的数量。

正如`match`查询中所示，`best_fields`也接受`analyzer`, `boost`, `operator`, `minimum_should_match`, `fuzziness`, `lenient`, `prefix_length`, `max_expansions`, `rewrite`, `zero_terms_query`, `cutoff_frequency`。

## `phrase`和`phrase_prefix`

`phrase`和`phrase_prefix`类型的行为和`best_fields`相似，但是它们使用`match_phrase`或`match_phrase_prefix`查询而不是`match`查询。

查询：

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
  "query": {
    "multi_match" : {
      "query":      "quick brown f",
      "type":       "phrase_prefix",
      "fields":     [ "subject", "message" ]
    }
  }
}'
```

会以这样的方式来执行：

```
curl -XGET 'localhost:9200/_search?pretty' -d'
{
  "query": {
    "dis_max": {
      "queries": [
        { "match_phrase_prefix": { "subject": "quick brown f" }},
        { "match_phrase_prefix": { "message": "quick brown f" }}
      ]
    }
  }
}'
```

正如`Match`查询所示，`phrase`和`phrase_prefix`类型接受`analyzer`, `boost`, `lenient`, `slop`, `zero_terms_query`。 `phrase_prefix`还接受`max_expansions`。

> `fuzziness`参数不能用在`phrase`或`phrase_prefix`类型。

## `cross_fields`

查询多个结构化字段的文档时，`cross_fields`类型很有用。比如，当对`first_name`和`last_name`字段查询`Will Smith`时，最好的匹配应该是`Will`在一个字段`Smith`在另一个字段。

。。。
