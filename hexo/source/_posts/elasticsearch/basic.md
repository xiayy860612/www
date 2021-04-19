---
title: ES Basic Usage
date: 2021-04-09 21:03:23
categories: 
- ElasticSearch
tags:
- 初级
- ES
---

帮助新手快速上手ElasticSearch, 主要针对以下几个问题:

- 如何进行建模
- 如何执行增删改查操作
- 如何进行统计计算

<!-- more -->

## 建模

ES中索引(Index)是特定结构的文档(Document)的集合, 
针对特定的需求, 需要定义为索引定义mapping, 即文档的schema.

定义文档的schema主要包含以下内容:
- 指定字段类型
- 是否可索引(index), 默认为true, true则字段可用于搜索, 否则无法用于搜索
- 是否配置多字段(fields), 针对text字段, 默认会添加keyword字段
- 是否支持新增字段的自动识别(Dynamic Mapping), 默认为true
- 为text字段设置分词器(Analyzer)

常用的字段类型
- Date
- Numeric
- Boolean
- 文本, text/keyword
- 地理, geo_point/geo_sharp

### 文本类型的字段

文本类型:
- text, 用于全文检索, 需要设置分词器(Analyzer), 用于对该字段的值进行解析
- keyword, 用于精确匹配查询

分词(Analysis)就是使用分词器(Analyzer)把文本转换为一系列单词(term/token)的过程;
它会在数据写入和查询时使用.

分词器由3部分组成:
- Character Filter, 针对原始文本进行处理
- Tokenizer, 按照规则切分单词
- Token Filter, 对切分后的单词进行处理

```plantuml
start
:input text content;
partition Analysis {
:use Character Filter;
:use Tokenizer;
:use Token Filter;
}
:get final token list;
end
```

可以使用***_analyze API***进行对分词器测试

## 文档(Document)

文档是ES中可被搜索的最小单元.

文档包含一系列元数据:
- _index, 文档所在的索引名
- _id, 文档的唯一id
- _version, 版本信息, 用于并发控制
- _score, 相关性打分, 影响搜索的查准率和查全率
- _source, 保存文档的原始数据

文档的常用操作:
- 创建文档
    + 索引(`PUT <index>/_doc/<id`>), 如果id不存在, 创建新文档; 
    否则先删除原有的, 然后再创建.
    + 创建指定id的文档(`PUT <index>/_create/<id>`), 如果id已经存在, 则创建失败
    + 新增文档(`POST <index>/_doc`), 不指定id, ES自动生成
- 读取(`GET <index>/_doc/<id>`)
- 修改(`POST <index>/_update/<id>`), 文档必须已经存在, 只会对相应字段做增量修改
- 删除(`DELETE <index>/_doc/<id>`)

## 搜索

ES可以通过以下几种方式进行搜索:
- URI Search
- [Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/query-dsl.html), 
推荐, 通过在request body中通过在query字段中输入搜索条件

ES提供了2中搜索上下文:
- Query Context, 提供相关性算分
- Filter Context, 不提供相关性算分
    + constant_score中的filter条件
    + bool查询中的must_not/filter条件

相关性算分用于说明搜索的结果的匹配程度, 算分越高, 结果匹配度越高, 并且结果文档越靠前.
ES在搜索时默认使用Query Context, 如果不需要相关性算分, 则需要显式使用Filter Context.

### Query DSL

- term查询, 用于精确匹配查询
    + term
    + exists
    + range
    + prefix
    + wildcard
    + regex
- full text查询, 用于全文检索, 需要配合分词器对搜索的输入进行解析
    + match
- bool查询, 用于多条件的查询
    + must, 必要条件, 并提供算分
    + filter, 必要条件, 不提供算分
    + should, 可选条件, 提供算分
    + must_not, 不包含条件, 不提供算分

## 聚合分析(Aggregation)

- Bucket, 对数据进行分组
- Metric, 对数据进行统计计算
- Pipeline, 对聚合结果再进行一次聚合分析

## Reference

- [Elasticsearch Guide](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)