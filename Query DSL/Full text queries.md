# 全文查询

高级别的全文查询通常被用来在文本字段（比如email正文）上执行全文查询。他们了解如何分析查询的字段，并在执行之前将每个字段的分析器（或`search_analyzer`）应用于查询字符串。

全文查询都包括：

## `match`查询

执行全文查询的标准查询，包括模糊匹配、短语查询、临近查询。

## `match_phrase`查询

和`match`查询相似，但是用来匹配精确的短语，或者单词临近匹配。

## `match_phrase_prefix`查询

穷人的`search-as-you-type`。和`match_phrase`查询相似，但是对最后一个词进行通配符搜索。

## `multi_match`查询

多字段版本的`match`查询

## `common_terms`查询

更专门的查询，偏好不常见的单词

## `query_string`查询

支持紧凑的Lucene查询字符串语法，允许您在单个查询字符串中指定AND | OR | NOT条件和多字段搜索。 仅限专家用户。

## `simple_query_string`

一个更简单，更健壮的query_string语法版本，适合直接暴露给用户。