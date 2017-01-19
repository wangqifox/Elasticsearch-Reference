# Mapping字符过滤器

`mapping`字符过滤器接受一个键值映射。当遇见一个与键相同的字符时，将它替换为键关联的值。

匹配是贪婪的，在匹配点会匹配尽量长的的字符串。替换允许是空字符串。

## 匹配

`mapping`字符过滤器接受以下参数：

|参数|说明|
|---|----|
|mappings|映射列表，每个元素的格式如下:`key=>value`|
|mappings_path|路径，绝对路径或者相对于`config`目录的相对路径，文件中每行都是`key=>value`这样格式的映射，文件编码为`UTF-8`|

`mappings`或者`mappings_path`参数必须提供。

## 示例配置

该例子中，我们配置`mapping`字符过滤器来将阿拉伯数字替换为等价的拉丁数字：

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "keyword",
          "char_filter": [
            "my_char_filter"
          ]
        }
      },
      "char_filter": {
        "my_char_filter": {
          "type": "mapping",
          "mappings": [
            "٠ => 0",
            "١ => 1",
            "٢ => 2",
            "٣ => 3",
            "٤ => 4",
            "٥ => 5",
            "٦ => 6",
            "٧ => 7",
            "٨ => 8",
            "٩ => 9"
          ]
        }
      }
    }
  }
}
'
curl -XPOST 'localhost:9200/my_index/_analyze?pretty' -d'
{
  "analyzer": "my_analyzer",
  "text": "My license plate is ٢٥٠١٥"
}
'
```

产生以下分词：

```
[ My license plate is 25015 ]
```

键和值可以是字符串。以下的例子将`:)`和`:(`替换为等价的文本：

```
curl -XPUT 'localhost:9200/my_index?pretty' -d'
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "standard",
          "char_filter": [
            "my_char_filter"
          ]
        }
      },
      "char_filter": {
        "my_char_filter": {
          "type": "mapping",
          "mappings": [
            ":) => _happy_",
            ":( => _sad_"
          ]
        }
      }
    }
  }
}
'
curl -XPOST 'localhost:9200/my_index/_analyze?pretty' -d'
{
  "analyzer": "my_analyzer",
  "text": "I\u0027m delighted about it :("
}
'
```

产生以下分词：

```
[ I'm, delighted, about, it, _sad_ ]
```

