# Elasticsearch 文档中文翻译

* [Getting Started]
  * [Basic Concepts](Getting%20Started/Basic%20Concepts.md)
  * [Exploring Your Data]
    * [Executing Aggregations](Getting%20started/Executing%20Aggregations.md)
* [API Conventions]
  * [Common options](API%20Conventions/Common%20options.md)
* [Document APIs]
  * [Reading and Writing documents](Document%20APIs/Reading%20and%20Writing%20documents.md)
  * [Index API](Document%20APIs/Index%20API.md)
* [Search APIs]
  * [Search](Search%20APIs/Search.md)
  * [Suggesters](Search%20APIs/Suggesters.md)
    * [Term suggester](Search%20APIs/Suggesters/Term%20suggester.md)
    * [Phrase Suggester](Search%20APIs/Suggesters/Phrase%20Suggester.md)
    * [Completion Suggester](Search%20APIs/Suggesters/Completion%20Suggester.md)
    * [Context Suggester](Search%20APIs/Suggesters/Context%20Suggester.md)
  * [Request Body Search]
    * [Query](Search%20APIs/Request%20Body%20Search/Query.md)
    * [From/Size](Search%20APIs/Request%20Body%20Search/From_Size.md)
    * [Sort](Search%20APIs/Request%20Body%20Search/Sort.md)
    * [Named Queries](Search%20APIs/Request%20Body%20Search/Named%20Queries.md)
* [Indices APIs]
  * [Refresh](Indices/Refresh.md)
* [Query DSL]
  * [Query and filter context](Query%20DSL/Query%20and%20filter%20context.md)
  * [Match All Query](Query%20DSL/Match%20All%20Query.md)
  * [Full text queries](Query%20DSL/Full%20text%20queries.md)
    * [Match Query](Query%20DSL/Full%20text%20queries/Match%20Query.md)
    * [Match Phrase Query](Query%20DSL/Full%20text%20queries/Match%20Phrase%20Query.md)
    * [Match Phrase Prefix Query](Query%20DSL/Full%20text%20queries/Match%20Phrase%20Prefix%20Query.md)
    * [Multi Match Query](Query%20DSL/Full%20text%20queries/Multi%20Match%20Query.md)
    * [Common Terms Query](Query%20DSL/Full%20text%20queries/Common%20Terms%20Query.md)
  * [Term level queries](Query%20DSL/Term%20level%20queries.md)
    * [Term Query](Query%20DSL/Term%20level%20queries/Term%20Query.md)
    * [Terms Query](Query%20DSL/Term%20level%20queries/Terms%20Query.md)
    * [Range Query](Query%20DSL/Term%20level%20queries/Range%20Query.md)
  * [Compound queries]
    * [Constant Score Query](Query%20DSL/Compound%20queries/Constant%20Score%20Query.md)
    * [Bool Query](Query%20DSL/Compound%20queries/Bool%20Query.md)
    * [Dis Max Query](Query%20DSL/Compound%20queries/Dis%20Max%20Query.md)
  * [Minimum Should Match](Query%20DSL/Minimum%20Should%20Match.md)
* [mapping](mapping/mapping.md)
  * [Field datatypes](mapping/Field%20datatypes.md)
    * [Array datatpe](mapping/Field%20datatypes/Array%20datatype.md)
    * [Date datatype](mapping/Field%20datatypes/Date%20datatype.md)
    * [Keyword datatype](mapping/Field%20datatypes/Keyword%20datatype.md)
    * [Nested datatype](mapping/Field%20datatypes/Nested%20datatype.md)
    * [Object datatype](mapping/Field%20datatypes/Object%20datatype.md)
    * [String datatype](mapping/Field%20datatypes/String%20datatype.md)
    * [Text datatype](mapping/Field%20datatypes/Text%20datatype.md)
  * [Meta-Fields](mapping/meta-fields.md)
    * [_all field](mapping/Meta-fields/_all%20field.md)
    * [_id field](mapping/Meta-fields/_id%20field.md)
    * [_source field](mapping/Meta-fields/_source%20field.md)
    * [_type field](mapping/Meta-fields/_type%20field.md)
    * [_uid field](mapping/Meta-fields/_uid%20field.md)
  * [Mapping parameters](mapping/Mapping%20parameters.md)
    * [analyzer](mapping/Mapping%20parameters/analyzer.md)
  	* [boost](mapping/Mapping%20parameters/boost.md)
    * [coerce](mapping/Mapping%20parameters/coerce.md) 
    * [copy_to](mapping/Mapping%20parameters/copy_to.md)
    * [doc_values](mapping/Mapping%20parameters/doc_values.md)
    * [fielddata](mapping/Mapping%20parameters/fielddata.md)
    * [format](mapping/Mapping%20parameters/format.md)
    * [ignore_malformed](mapping/Mapping%20parameters/ignore_malformed.md)
    * [include_in_all](mapping/Mapping%20parameters/include_in_all.md)
    * [index](mapping/Mapping%20parameters/index.md)
    * [index_options](mapping/Mapping%20parameters/index_options.md)
    * [fields](mapping/Mapping%20parameters/fields.md)
    * [null_value](mapping/Mapping%20parameters/null_value.md)
    * [properties](mapping/Mapping%20parameters/properties.md)
    * [search_analyzer](mapping/Mapping%20parameters/search_analyzer.md)
    * [store](mapping/Mapping%20parameters/store.md)
  * [Dynamic Mapping](mapping/dynamic%20mapping.md)
* [Analysis](Analysis/Analysis.md)
  * [Anatomy of an analyzer](Analysis/Anatomy%20of%20an%20analyzer.md)
  * [Testing analyzer](Analysis/Testing%20analyzer.md)
  * [Analyzers](Analysis/Analyzers.md)
    * [Configuring built-in analyzers](Analysis/Analyzers/Configuring%20built-in%20analyzers.md)
    * [Standard Analyzer](Analysis/Analyzers/Standard%20Analyzer.md)
    * [Simple Analyzer](Analysis/Analyzers/Simple%20Analyzer.md)
    * [Whitespace Analyzer](Analysis/Analyzers/Whitespace%20Analyzer.md)
    * [Stop Analyzer](Analysis/Analyzers/Stop%20Analyzer.md)
    * [Keyword Analyzer](Analysis/Analyzers/Keyword%20Analyzer.md)
    * [Pattern Analyzer](Analysis/Analyzers/Pattern%20Analyzer.md)
    * [Language Analyzer](Analysis/Analyzers/Language%20Analyzer.md)
    * [Fingerprint Analyzer](Analysis/Analyzers/Fingerprint%20Analyzer.md)
    * [Custom Analyzer](Analysis/Analyzers/Custom%20Analyzer.md)
  * [Tokenizers](Analysis/Tokenizers.md)
    * [Standard Tokenizer](Analysis/Tokenizers/Standard%20Tokenizer.md)
    * [Letter Tokenizer](Analysis/Tokenizers/Letter%20Tokenizer.md)
    * [Lowercase Tokenizer](Analysis/Tokenizers/Lowercase%20Tokenizer.md)
    * [Whitespace Tokenizer](Analysis/Tokenizers/Whitespace%20Tokenizer.md)
    * [UAX URL Email Tokenizer](Analysis/Tokenizers/UAX%20URL%20Email%20Tokenizer.md)
    * [Classic Tokenizer](Analysis/Tokenizers/Classic%20Tokenizer.md)
    * [Thai Tokenizer](Analysis/Tokenizers/Thai%20Tokenizer.md)
    * [NGram Tokenizer](Analysis/Tokenizers/NGram%20Tokenizer.md)
    * [Edge NGram Tokenizer](Analysis/Tokenizers/Edge%20NGram%20Tokenizer.md)
    * [Keyword Tokenizer](Analysis/Tokenizers/Keyword%20Tokenizer.md)
    * [Pattern Tokenizer](Analysis/Tokenizers/Pattern%20Tokenizer.md)
    * [Path Hierarchy Tokenizer](Analysis/Tokenizers/Path%20Hierarchy%20Tokenizer.md)
  * [Token Filters](Analysis/Token%20Filters.md)
    * [Standard Token Filter](Analysis/Token%20Filters/Standard%20Token%20Filter.md)
    * [Stop Token Filter](Analysis/Token%20Filters/Stop%20Token%20Filter.md)
  * [Character Filters](Analysis/Character%20Filters.md)
    * [HTML Strip Char Filter](Analysis/Character%20Filters/HTML%20Strip%20Char%20Filter.md)
    * [Mapping Char Filter](Analysis/Character%20Filters/Mapping%20Char%20Filter.md)
    * [Pattern Replace Char Filter](Analysis/Character%20Filters/Pattern%20Replace%20Char%20Filter.md)



```
Relational DB -> Databases -> Tables -> Rows -> Columns
Elasticsearch -> Indices -> Types -> Documents -> Fields
```

- document -> 文档
- index -> 索引
- type -> 类型
- token -> 表征
- filter -> 过滤器
- analyser -> 分析器
- mapping -> 映射
- field -> 字段
- shard -> 分片
