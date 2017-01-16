# 分词器

分词器接受字符流，将其拆分为独立的分词（通常是单独的单词），然后输出分词流。比如，`whitespace`分词器在遇见空白符时将文本拆分为分词。它将文本"Quick brown fox!"转化为`[Quick, brown, fox!]`

分词器还负责记录每个分词的顺序或者位置（用作短语和单词近似查询）以及分词所表示的原始单词所在的起始和结束的字符偏移（用作搜索片段的高亮）。

Elasticsearch有许多内建的分词器，可以用来构建自定义的分析器。

## 面向单词的分词器

以下的分词器通常用来将全文拆分成独立的单词：

### Standard Tokenizer（标准分词器）

标准分词器在单词的分界将文本切分为分词，使用Unicode文本分割算法。它删除大部分的标点符号。对大部分语言来说是最好的选择。

### Letter Tokenizer（字母分词器）

字母分词器遇到非字母的字符时将文本切分为分词。

### Lowercase Tokenizer（小写分词器）

小写分词器和字母分词器相似，当遇到非字母时将文本切分为分词，但是它将所有的分词都转化为小写。

### Whitespace Tokenizer（空白分词器）

空白分词器遇到任意空白符时，将文本切分为分词。

### UAX URL Email Tokenizer（UAX URL Email分词器）

`uax_url_email`分词器和标准分词器相似，但是它可以识别URLs和email地址，并将它们作为单个分词。

### Classic Tokenizer（经典分词器）

经典分词器是一个基于英语语法的分词器。

### Thai Tokenizer（泰语分词器）

泰语分词器将泰语文本切分为单词

## Partial Word Tokenizers（部分词分词器）

这些分词器将文本或词分解成小片段，用于部分单词匹配。

### N-Gram Tokenizer

`ngram`分词器当遇见一系列指定的字符（如空白符或标点符号）时将文本切分为单词，然后返回每个单词的n-grams：连续字母的滑动窗口，比如`quick -> [qu, ui, ic, ck]`。

### Edge N-Gram Tokenizer

`edge_ngram`分词器当遇见一系列指定的字符（如空白符或标点符号）时将文本切分为单词，然后返回每个单词的n-grams，锚定在单词的起始位置，比如`quick -> [q, qu, qui, quic, quick]`

## 结构化文本分词器

以下分词器通常用于结构化数据，比如标识符、email地址、邮政编码、路径，而不是全文。

### Keyword Tokenizer（关键词分词器）

关键词分词器是一个“空”分词器，它接受文本然后将相同的文本作为单个分词输出。它可以结合分词过滤器比如`lowercase`来规范化分析后的分词。

### Pattern Tokenizer（模式分词器）

模式分词器使用正则表达式当匹配单词分隔符时将文本拆分为分词，或捕获匹配的文本作为分词。

### Path Tokenizer（路径分词器）

`path_hierarchy`分词器采用类似文件系统路径的分层值，在路径分隔符中分割，并将树中每一项都产生一个分词，比如`/foo/bar/baz -> [/foo, /foo/bar, /foo/bar/baz]`。