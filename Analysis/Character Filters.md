# 字符过滤器

字符过滤器用来预处理字符流，然后传递给分词器。

字符过滤器以流的方式接受原始文档，通过增加、删除、改变字符来变换字符流。比如，字符过滤器可以将印度-阿拉伯数字（٠‎١٢٣٤٥٦٧٨‎٩）转变为等价的阿拉伯-拉丁数字，或者将HTML元素比如`<b>`去除。

Elasticsearch拥有一系列内建的字符过滤器，可以用来构建自定义的分析器。

## HTML strip Character Filter

`html_strip`字符过滤器将HTML元素（比如`<b>`）去除，然后对HTML体进行编码

## Mapping Character Filter

`mapping`字符过滤器将指定的字符串指定为另一个字符串

## Pattern Replace Character Filter

`pattern_replace`字符过滤器替换匹配正则表达式的字符
