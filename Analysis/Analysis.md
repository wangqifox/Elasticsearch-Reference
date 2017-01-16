# Analysis

分析过程将文本（比如任意email的正文）转化为词元或者分词，将它们添加到倒排索引中以便搜索。分析通过分析器来执行，既可以使用内建的分析器或者是每个索引中定义的分析器。

## 索引阶段分析

比如，在索引阶段，内建的`english`分析器将下面这段文字：

```
"The QUICK brown foxes jumped over the lazy dog!"
```

拆分成下面的分词，添加到倒排索引中：

```
[ quick, brown, fox, jump, over, lazi, dog ]
```

## 指定索引阶段的分析器

每个文本字段在映射时可以指定自己的分析器：

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "mappings": {
    "my_type": {
      "properties": {
        "title": {
          "type":     "text",
          "analyzer": "standard"
        }
      }
    }
  }
}
'
```

在索引阶段，如果没有指定分析器，它会在索引设置中查找名叫`default`的分析器。如果找不到，默认使用`standard analyzer`。

## 搜索阶段分析

全文查询比如`match`查询在搜索阶段对查询字符串应用相同的分析过程，将查询字符串转化为与倒排索引中存储的格式相同的分词。

比如，用户查询:

```
"a quick fox"
```

相同的分析器将该字符串拆分为以下分词：

```
[ quick, fox ]
```

查询字符串中使用的确切分词没有出现在原始文本中，因为我们在原始文本和查询字符串中使用了相同的分析器，查询字符串中的分词匹配原始文本中的分词，这意味着该查询匹配我们的示例文档。

## 指定搜索阶段分析器

通常在索引阶段和搜索阶段会使用相同的分析器，全文查询比如`match`查询会使用映射来查找每个字段所使用的分析器。

每个字段所使用的分析器通过以下步骤来决定：

- 查询中指定的`analyzer`
- 映射中指定的`search_analyzer`参数
- 映射中指定的`analyzer`参数
- 索引设置中名为`default_search`的分析器
- 索引设置中名为`default`的分析器
- `standard`分析器
