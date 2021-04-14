---
title: ES Basic Usage
date: 2021-04-09 21:03:23
categories: 
- ElasticSearch
tags:
- 初级
- ES
---

帮助新手快速上手ElasticSearch

<!-- more -->

## 索引(Index)

索引(Index)是特定结构的文档(Document)的集合.

通过mapping来定义文档的schema, 包含:
- 指定字段类型
- 是否可索引(index), 默认为true
- 是否配置多字段(fields), 针对text字段, 默认会添加keyword字段
- 是否支持新增字段的自动识别(Dynamic Mapping), 默认为true

### 字段类型

- Date
- Numeric
- Boolean
- text/keyword
- 地理, geo_point/geo_sharp

## 文档(Document)

文档是ES中可被搜索的最小单元.

文档包含一系列元数据:
- _index, 文档所在的索引名
- _id, 文档的唯一id
- _version, 版本信息, 用于并发控制
- _score, 相关性打分, 影响搜索的查准率和查全率
- _source, 保存文档的原始数据

文档的常用操作:
- 创建
- 读取
- 修改
- 删除

## 文本类型的字段

文本类型:
- text, 用于全文检索
- keyword, 用于精确匹配

针对text字段可以设置特定的analyzer, 用于对该字段的值进行解析.

## Analyzer




## 搜索