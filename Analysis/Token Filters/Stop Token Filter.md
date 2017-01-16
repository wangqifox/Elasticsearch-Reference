# 停词分词过滤器

类型为`stop`的分词过滤器从分词流中删除停词。

停词分词过滤器类型接受以下的参数：

|参数|说明|
|---|----|
|stopwords|停词列表。默认是`_english_`停词|
|stopwords_path|停用词文件配置的路径（相对于`config`的路径，或绝对路径）。每个停词作为一行（用换行符分隔）。文件必须采用UTF-8编码|
|ignore_case|设置为`true`可以将所有单词首先转化为小写。默认为`false`。|
|remove_trailing|设置为`false`不忽略搜索中的最后一个分词，如果该分词是一个停词。这对于完成建议来说很有用。查询`green a`可以扩展到`green apple`，尽管通常会删除停词。默认是`true`。|

`stopwords`参数接受一个停词列表：

```
PUT /my_index
{
    "settings": {
        "analysis": {
            "filter": {
                "my_stop": {
                    "type":       "stop",
                    "stopwords": ["and", "is", "the"]
                }
            }
        }
    }
}
```

或者预定义的特定语言列表：

```
PUT /my_index
{
    "settings": {
        "analysis": {
            "filter": {
                "my_stop": {
                    "type":       "stop",
                    "stopwords":  "_english_"
                }
            }
        }
    }
}
```

Elasticsearch提供一下预定义的语言：

```
_arabic_, _armenian_, _basque_, _brazilian_, _bulgarian_, _catalan_, _czech_, _danish_, _dutch_, _english_, _finnish_, _french_, _galician_, _german_, _greek_, _hindi_, _hungarian_, _indonesian_, _irish_, _italian_, _latvian_, _norwegian_, _persian_, _portuguese_, _romanian_, _russian_, _sorani_, _spanish_, _swedish_, _thai_, _turkish_.
```

清空停词列表（禁用停词）使用：`_none_`