---
title: Elasticsearch 分布式搜索引擎
categories:
  - 微服务
tags:
  - 微服务
  - 分布式
  - Elasticsearch
  - ELK
location:
  - 黄金时代
abbrlink: 'elasticsearch'
permalink: 'elasticsearch'
date: 2025-09-08 16:41:00
updated: 2025-09-08 16:41:00
---

> 摘要：可以用来实现**搜索**、**日志**统计、分析、系统监控等功能。

<!-- more -->

---

### 目录

[TOC]

### Elasticsearch

> 分布式全文搜索引擎

### 简介

`Elasticsearch` 是一个基于Lucene的搜索服务器。

- 一个开源的**分布式**全文搜索引擎，基于**restful web接口**。
- 可以用来实现**搜索**、**日志**统计、分析、系统监控等功能。
- 广泛运用于云计算中，能够达到实时搜索，具有稳定、可靠、快速的特点。
- 端口：默认端口号为 `9200/9300`，`Kibana` 默认端口为 `5601`。

> 参考：[ES分布式搜索引擎(ES从入门到精通一篇就够了) ](https://www.cnblogs.com/buchizicai/p/17093719.html)

#### MySQL、Elasticsearch

- MySQL：擅长**事务类型**操作，可以确保数据的安全和一致性；对安全性要求较高的写操作。使用**正向索引**。
- Elasticsearch：擅长**海量数据**的搜索、分析、计算；对**查询性能**要求较高的搜索需求。使用**倒排索引**。

###### SQL 和 ES DSL 对应关系

| MySQL                   | Elasticsearch Query DSL | 说明                                                         |
| ----------------------- | ----------------------- | ------------------------------------------------------------ |
| SQL                     | DSL                     | 是elasticsearch提供的JSON风格的请求语句，用来操作elasticsearch，实现CRUD。 |
| 数据库                  | Index（索引）           | 就是文档的集合。                                             |
| Table                   | `Type`（类型）          | 是索引的**逻辑类别分区**，通常为具有一组**公共字段**的文档类型 |
| Row                     | Document（文档）        | 就是一条条的数据，文档都是JSON格式。                         |
| Column                  | Field（字段）           | 就是JSON文档中的字段。                                       |
| id？                    | Term（词条）            |                                                              |
| Schema（s给ma，表结构） | Mapping（映射）         | 是索引中文档的约束，例如字段类型约束。                       |

###### 相关概念

- `Near Realtime`（**近实时**）：Elasticsearch是一个近乎实时的搜索平台，这意味着从**索引文档**到**可搜索文档**之间只有一个轻微的延迟（通常是一秒钟）。
- `Cluster`（**集群**）：是一个或多个**节点的集合**，它们一起保存整个数据，并提供跨所有节点的**联合索引**和搜索功能。每个群集都有自己的唯一群集名称，节点通过名称加入群集。
- `Node`（节点）：指属于集群的**单个Elasticsearch实例**，存储数据并参与集群的**索引和搜索**功能。可以将节点配置为按集群名称加入特定集群，默认情况下，每个节点都设置为加入一个名为`elasticsearch`的群集。
- `Index`（**索引**）：索引是一些具有相似特征的**文档集合**，类似于MySql中**数据库**的概念。
    - `Type`（**类型**）：类型是索引的**逻辑类别分区**，通常为具有一组**公共字段**的文档类型，类似MySql中表的概念。`注意`：在6.0.0及更高的版本中，一个索引只能包含一个类型。
    - `Shards`（**分片**）：当索引存储大量数据时，可能会超出单个节点的硬件限制，为了解决这个问题，Elasticsearch提供了将**索引细分**为分片的概念。分片机制赋予了索引**水平扩容**的能力、并允许**跨分片分发和并行化**操作，从而提高性能和吞吐量。
    - `Replicas`（副本）：在可能出现故障的网络环境中，需要有一个**故障切换机制**，Elasticsearch提供了将索引的分片复制为一个或多个副本的功能，副本在某些节点失效的情况下提供高可用性。
- `Document`（**文档**）：文档是可被索引的基本信息单位，以`JSON`形式表示，类似于MySQL中**行记录**（`Row`）的概念。
    - 文档中往往包含很多的**字段（Field）**，类似于MySQL数据库中的列。

#### 倒排索引

MySQL **正向索引**：如给**表中的id**创建索引，基于 `title ` 做**模糊查询**时，只能**全表扫描**/逐行扫描，随着数据量增加，查询效率也会越来越低。当数据量达到**数百万**时，就是一场灾难。

倒排索引中两个非常重要的概念：

- **文档**（`Document`）
- **词条**（`Term`）：对文档数据或用户搜索数据，利用某种**算法分词**，得到的具备含义的词语。如：我是中国人，就可以分为：我、是、中国人、中国、国人这样的几个词条。

**创建倒排索引**是对正向索引的一种特殊处理，流程如下：

1. 存储时：
    1. 将每一个**文档**的数据利用**算法分词**，得到一个个**词条**；
    2. 创建**词条列表**：每行数据包括**词条**、词条所在**文档id**、位置等信息；
    3. 因为词条唯一性，可以给词条创建**倒排索引**（相当于 SQL 中的表），例如hash表结构索引。
2. 查询时：（利用分词器）先**分词**得到词条，再根据词条去（~~词条列表~~）**倒排索引**中匹配（查询对应的文档id），根据文档id去**正向索引**中查询文档存入结果集。

> **全文检索（full text）查询**：利用分词器对用户输入内容分词，然后去**倒排索引库**中匹配。

**IK分词器**有几种模式？

1. `ik_smart`：智能切分，粗粒度
2. `ik_max_word`：最细切分，细粒度

<img src="../assets/2729274-20230205171803916-704919285.png" alt="img" style="zoom:60%;" />

### DSL 命令 / RestAPI

Elasticsearch 提供了基于JSON的**DSL**（`Domain Specific Language`）来**定义查询**。常见的查询类型包括：

1. **查询所有**：查询出所有数据，一般测试用。例如：`match_all`
2. **全文检索（full text）查询**：利用分词器对用户输入内容分词，然后去**倒排索引库**中匹配。例如：
    - `match_query`、`multi_match_query`
3. **精确查询**：根据精确词条值查找数据，一般是查找**keyword**、数值、日期、boolean等类型字段。例如：
    - `ids`、`range`、`term`（分词）：
4. **地理查询**：根据经纬度查询。例如：
    - `geo_distance`、`geo_bounding_box`
5. **复合（compound）查询**：复合查询可以将上述各种查询条件组合起来，合并查询条件。例如：
    - `bool`、`function_score`

#### 索引库的CRUD

- 创建索引库：PUT /索引库名
- 查询索引库：GET /索引库名
- 删除索引库：DELETE /索引库名
- 修改索引库（添加字段）：PUT /索引库名/_mapping

#### 文档操作

- 创建文档：POST /{索引库名}/_doc/文档id
- 查询文档：GET /{索引库名}/_doc/文档id
- 删除文档：DELETE /{索引库名}/_doc/文档id
- 修改文档：
    - 全量修改：**PUT** /{索引库名}/**_doc**/文档id，直接覆盖原来的文档
    - **增量修改**：**POST** /{索引库名}/**_update**/文档id { "doc": {字段}}，修改文档中的部分字段

```
# 查看所有节点
GET _cat/nodes

# 查看book索引数据
GET book/_search
{
    "query": {
    "match": {
      "content": "chenqionghe"
    }
  }
}

# 添加一条数据
POST book/_doc 
{
  "page":8,
  "content": "chenqionghe喜欢运动，绳命是如此的精彩，绳命是多么的辉煌"
}

# 更新数据
PUT book/_doc/iSAz4XABrERdg9Ao0QZI
{
  "page":8,
  "content":"chenqionghe喜欢运动，绳命是剁么的回晃；绳命是入刺的井猜"
}

# 删除数据
POST book/_delete_by_query
{
  "query": {
    "match": {
      "page": 8
    }
  }
}

# 批量插入数据
POST book/_bulk
{ "index":{} }
{ "page":22 , "content": "Adversity, steeling will strengthen body.逆境磨练意志，锻炼增强体魄。"}
{ "index":{} }
{ "page":23 , "content": "Reading is to the mind, such as exercise is to the body.读书之于头脑，好比运动之于身体。"}
{ "index":{} }
{ "page":24 , "content": "Years make you old, anti-aging.岁月催人老，运动抗衰老。"}
{ "index":{} }
```

### SQL 查询

在`Kibana`的Console中输入如下命令：

```
POST /_sql?format=txt
{
  "query": "SELECT account_number,address,age,balance FROM account LIMIT 10"
}
```

#### 将SQL转化为DSL

当需要使用Query DSL时，也可以先使用SQL来查询，然后通过`Translate API`转换即可。

例如翻译以下查询语句：

```
POST /_sql/translate
{
  "query": "SELECT account_number,address,age,balance FROM account WHERE age>32 LIMIT 10"
}
```

### ES 与 MySQL 数据同步

#### 同步调用：

1. `hotel-demo`对外提供接口，用来修改`elasticsearch`中的数据；
2. 酒店管理服务在完成数据库操作后，直接调用`hotel-demo`提供的接口。

<img src="../assets/2729274-20230205174204697-1782713458.png" alt="img" style="zoom:50%;" />

#### 异步通知：

- `hotel-admin`对`mysql`数据库数据完成增、删、改后，发送MQ消息；
- `hotel-demo`**监听MQ**，接收到消息后完成`elasticsearch`数据修改。

<img src="../assets/2729274-20230205174208679-699617903.png" alt="img" style="zoom: 50%;" />

