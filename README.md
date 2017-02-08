# Elasticsearch 文档

* [Getting Started]
  * [Basic Concepts](Getting started/Basic Concepts.md)
  * [Exploring Your Data]
    * [Executing Aggregations](Getting started/Executing Aggregations.md)
* [API Conventions]
  * [Common options](API Conventions/Common options.md)
* [Document APIs]
  * [Reading and Writing documents](Document APIs/Reading and Writing documents.md)
  * [Index API](Document APIs/Index API.md)
* [Search APIs]
  * [Search](Search APIs/Search.md)
  * [Suggesters](Search APIs/Suggesters.md)
    * [Term suggester](Search APIs/Suggesters/Term suggester.md)
    * [Phrase Suggester](Search APIs/Suggesters/Phrase Suggester.md)
    * [Completion Suggester](Search APIs/Suggesters/Completion Suggester.md)
    * [Context Suggester](Search APIs/Suggesters/Context Suggester.md)
  * [Request Body Search]
    * [Query](Search APIs/Request Body Search/Query.md)
    * [From/Size](Search APIs/Request Body Search/From_Size.md)
    * [Sort](Search APIs/Request Body Search/Sort.md)
    * [Named Queries](Search APIs/Request Body Search/Named Queries.md)
* [Indices APIs]
  * [Refresh](Indices/Refresh.md)
* [Query DSL]
  * [Query and filter context](Query DSL/Query and filter context.md)
  * [Match All Query](Query DSL/Match All Query.md)
  * [Full text queries](Query DSL/Full text queries.md)
    * [Match Query](Query DSL/Full text queries/Match Query.md)
    * [Match Phrase Query](Query DSL/Full text queries/Match Phrase Query.md)
    * [Match Phrase Prefix Query](Query DSL/Full text queries/Match Phrase Prefix Query.md)
    * [Multi Match Query](Query DSL/Full text queries/Multi Match Query.md)
    * [Common Terms Query](Query DSL/Full text queries/Common Terms Query.md)
  * [Term level queries](Query DSL/Term level queries.md)
    * [Term Query](Query DSL/Term level queries/Term Query.md)
    * [Terms Query](Query DSL/Term level queries/Terms Query.md)
    * [Range Query](Query DSL/Term level queries/Range Query.md)
  * [Compound queries]
    * [Constant Score Query](Query DSL/Compound queries/Constant Score Query.md)
    * [Bool Query](Query DSL/Compound queries/Bool Query.md)
    * [Dis Max Query](Query DSL/Compound queries/Dis Max Query.md)
  * [Minimum Should Match](Query DSL/Minimum Should Match.md)
* [mapping](mapping/mapping.md)
  * [Field datatypes](mapping/Field datatypes.md)
    * [Array datatpe](mapping/Field datatypes/Array datatype.md)
    * [Date datatype](mapping/Field datatypes/Date datatype.md)
    * [Keyword datatype](mapping/Field datatypes/Keyword datatype.md)
    * [Nested datatype](mapping/Field datatypes/Nested datatype.md)
    * [Object datatype](mapping/Field datatypes/Object datatype.md)
    * [String datatype](mapping/Field datatypes/String datatype.md)
    * [Text datatype](mapping/Field datatypes/Text datatype.md)
  * [Meta-Fields](mapping/meta-fields.md)
    * [_all field](mapping/Meta-fields/_all field.md)
    * [_id field](mapping/Meta-fields/_id field.md)
    * [_source field](mapping/Meta-fields/_source field.md)
    * [_type field](mapping/Meta-fields/_type field.md)
    * [_uid field](mapping/Meta-fields/_uid field.md)
  * [Mapping parameters](mapping/Mapping parameters.md)
    * [analyzer](mapping/Mapping parameters/analyzer.md)
  	* [boost](mapping/Mapping parameters/boost.md)
    * [coerce](mapping/Mapping parameters/coerce.md) 
    * [copy_to](mapping/Mapping parameters/copy_to.md)
    * [doc_values](mapping/Mapping parameters/doc_values.md)
    * [fielddata](mapping/Mapping parameters/fielddata.md)
    * [format](mapping/Mapping parameters/format.md)
    * [ignore_malformed](mapping/Mapping parameters/ignore_malformed.md)
    * [include_in_all](mapping/Mapping parameters/include_in_all.md)
    * [index](mapping/Mapping parameters/index.md)
    * [index_options](mapping/Mapping parameters/index_options.md)
    * [fields](mapping/Mapping parameters/fields.md)
    * [null_value](mapping/Mapping parameters/null_value.md)
    * [properties](mapping/Mapping parameters/properties.md)
    * [search_analyzer](mapping/Mapping parameters/search_analyzer.md)
    * [store](mapping/Mapping parameters/store.md)
  * [Dynamic Mapping](mapping/dynamic mapping.md)
* [Analysis](Analysis/Analysis.md)
  * [Anatomy of an analyzer](Analysis/Anatomy of an analyzer.md)
  * [Testing analyzer](Analysis/Testing analyzer.md)
  * [Analyzers](Analysis/Analyzers.md)
    * [Configuring built-in analyzers](Analysis/Analyzers/Configuring built-in analyzers.md)
    * [Standard Analyzer](Analysis/Analyzers/Standard Analyzer.md)
    * [Simple Analyzer](Analysis/Analyzers/Simple Analyzer.md)
    * [Whitespace Analyzer](Analysis/Analyzers/Whitespace Analyzer.md)
    * [Stop Analyzer](Analysis/Analyzers/Stop Analyzer.md)
    * [Keyword Analyzer](Analysis/Analyzers/Keyword Analyzer.md)
    * [Pattern Analyzer](Analysis/Analyzers/Pattern Analyzer.md)
    * [Language Analyzer](Analysis/Analyzers/Language Analyzer.md)
    * [Fingerprint Analyzer](Analysis/Analyzers/Fingerprint Analyzer.md)
    * [Custom Analyzer](Analysis/Analyzers/Custom Analyzer.md)
  * [Tokenizers](Analysis/Tokenizers.md)
    * [Standard Tokenizer](Analysis/Tokenizers/Standard Tokenizer.md)
    * [Letter Tokenizer](Analysis/Tokenizers/Letter Tokenizer.md)
    * [Lowercase Tokenizer](Analysis/Tokenizers/Lowercase Tokenizer.md)
    * [Whitespace Tokenizer](Analysis/Tokenizers/Whitespace Tokenizer.md)
    * [UAX URL Email Tokenizer](Analysis/Tokenizers/UAX URL Email Tokenizer.md)
    * [Classic Tokenizer](Analysis/Tokenizers/Classic Tokenizer.md)
    * [Thai Tokenizer](Analysis/Tokenizers/Thai Tokenizer.md)
    * [NGram Tokenizer](Analysis/Tokenizers/NGram Tokenizer.md)
    * [Edge NGram Tokenizer](Analysis/Tokenizers/Edge NGram Tokenizer.md)
    * [Keyword Tokenizer](Analysis/Tokenizers/Keyword Tokenizer.md)
    * [Pattern Tokenizer](Analysis/Tokenizers/Pattern Tokenizer.md)
    * [Path Hierarchy Tokenizer](Analysis/Tokenizers/Path Hierarchy Tokenizer.md)
  * [Token Filters](Analysis/Token Filters.md)
    * [Standard Token Filter](Analysis/Token Filters/Standard Token Filter.md)
    * [Stop Token Filter](Analysis/Token Filters/Stop Token Filter.md)
  * [Character Filters](Analysis/Character Filters.md)
    * [HTML Strip Char Filter](Analysis/Character Filters/HTML Strip Char Filter.md)
    * [Mapping Char Filter](Analysis/Character Filters/Mapping Char Filter.md)
    * [Pattern Replace Char Filter](Analysis/Character Filters/Pattern Replace Char Filter.md)



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
