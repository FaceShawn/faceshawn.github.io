---
title: Sharding Sphere
categories:
  - 微服务
tags:
  - 微服务
  - 分布式
  - 高并发
  - 高性能
  - 高可用
  - Sharding-Sphere
  - 数据库
location:
  - 黄金时代
abbrlink: 'sharding_sphere'
permalink: 'sharding_sphere'
date: 2025-08-29 16:41:00
updated: 2025-08-29 16:41:00
---

> 摘要：用于分库分表。

<!-- more -->

---

## 目录

[TOC]

## ShardingSphere

Apache ShardingSphere 是一款分布式的数据库生态系统。

## 分类

包含两大产品：

### ShardingSphere-Proxy

`ShardingSphere-Proxy` **服务端**分库分表：通过 JDBC 驱动层透明代理实现**读写分离**。被定位为透明化的**数据库代理端**，提供封装了数据库二进制协议的服务端版本，用于完成对异构语言的支持。

- 代理层介于应用程序与数据库间，每次请求都需要做一次转发，请求会存在额外的时延。
- 这种方式对于应用非常友好，应用基本零改动，和语言无关，可以通过连接共享减少连接数消耗。
- 向应用程序完全透明，可直接当做 MySQL/PostgreSQL 使用；
- 兼容 MariaDB 等基于 MySQL 协议的数据库，以及 openGauss 等基于 PostgreSQL 协议的数据库；
- 适用于任何兼容 MySQL/PostgreSQL 协议的的客户端，如：MySQL Command Client, MySQL Workbench,Navicat 等。

### ShardingSphere-JDBC

`ShardingSphere-JDBC` **客户端**分库分表：经常简称之为 sharding-jdbc 。定位为轻量级 Java 框架，在 Java 的 JDBC 层提供的额外服务。使用**客户端直连**数据库，以 jar 包形式提供服务，无需额外部署和依赖，可理解为增强版的 JDBC 驱动，完全兼容 JDBC 和各种 ORM 框架。

- 适用于任何基于 JDBC 的 ORM 框架，如：JPA, Hibernate, Mybatis, Spring JDBC Template
    或直接使用JDBC；
- 支持任何第三方的数据库连接池，如：DBCP, C3P0, BoneCP, HikariCP 等；
- 支持任意实现 JDBC 规范的数据库，目前支持 MySQL，PostgreSQL，Oracle，SQLServer 以及任何可使用JDBC访问的数据库。

<img src="../assets/37480a85a746dcb7a469efb4d6acb24d.jpg" alt="img" style="zoom: 40%;" /><img src="../assets/3206512c7b6a0a08fe6e3af6891b03ae.jpg" alt="img" style="zoom:40%;" />

### 对比

当在 Proxy 和 JDBC 两种模式选择时，可以参考下表对照：

|            | JDBC         | Proxy            |
| :--------- | :----------- | :--------------- |
| 数据库     | 任意         | MySQL/PostgreSQL |
| 连接消耗数 | 高           | **低**           |
| 异构语言   | 仅Java       | 任意             |
| 性能       | **损耗低**   | 损耗略高         |
| 中心化     | **无中心化** | 中心化           |
| 静态入口   | 无           | **有**           |

越来越多的公司都在生产环境使用了 sharding-jdbc ，最核心的原因就是：**简单**（原理简单，易于实现，方便运维）。

### 混合部署架构

另外，在产品图中，Governance Center也是其中重要的部分。作用有点类似于微服务架构中的配置中心，可以使用第三方服务统一管理分库分表的配置信息，当前建议使用的第三方服务是Zookeeper，同时也支持**Nacos**，Etcd等其他第三方产品。

由于ShardingJDBC和ShardingProxy都支持通过Governance Center，将配置信息交个第三方服务管理，因此，也就自然支持了通过Governance Center进行整合的混合部署架构。

<img src="../assets/fa70b65c70979f4c7ff071438c89c263.png" alt="在这里插入图片描述" style="zoom: 50%;" />

## 功能

1. **SQL 路由**：根据 SQL 类型（SELECT/WRITE）自动路由到主库或从库。
2. **负载均衡**：支持轮询、随机、权重等算法分配**读请求**到多个从库。
3. **主从同步**：依赖 MySQL 原生主从复制机制保障数据一致性。
4. **故障转移**：配置心跳检测实现从库故障自动剔除。

<img src="../assets/1740542496552-dbf53244-b6f0-4d1d-9145-edeeeced0bb5.png" alt="img" style="zoom: 40%;" />

## 基本原理

在后端开发中，JDBC 编程是最基本的操作。不管 ORM 框架是 Mybatis 还是 Hibernate ，亦或是 spring-jpa ，他们的**底层实现**是 JDBC 的模型。

sharding-jdbc 的本质上就是**实现 JDBC 的核心接口**。

![img](../assets/up-3804866a3a45cca3f342d05298c1befde0c.png)



![img](../assets/up-6202a9c68de22a1431dc3620769d3252145.png)

| 接口              | 实现类                    |
| :---------------- | :------------------------ |
| DataSource        | ShardingDataSource        |
| Connection        | ShardingConnection        |
| Statement         | ShardingStatement         |
| PreparedStatement | ShardingPreparedStatement |
| ResultSet         | ShardingResultSet         |

下图展示了 Prxoy 和 JDBC 两种模式的核心流程。

<img src="../assets/up-d92c89ad3afda090af1efd2a5180fcbec55.png" alt="img" style="zoom:50%;" />

1. **SQL 解析**：分为词法解析和语法解析。 先通过词法解析器将 SQL 拆分为一个个不可再分的单词。再使用语法解析器对 SQL 进行理解，并最终提炼出解析上下文。解析上下文包括表、选择项、排序项、分组项、聚合函数、分页信息、查询条件以及可能需要修改的占位符的标记。

2. **执行器优化**：合并和优化分片条件，如 OR 等。

3. **SQL 路由**：根据**解析上下文**匹配用户配置的**分片策略**，并生成**路由路径**。目前支持分片路由和广播路由。

4. **SQL 改写**：将 SQL 改写为在真实数据库中可以正确执行的语句。SQL 改写分为正确性改写和优化改写。

5. **SQL 执行**：通过**多线程**执行器**异步**执行。

6. **结果归并**：将多个执行结果集归并以便于通过统一的 JDBC 接口输出。结果归并包括**流式归并**、**内存归并**和使用装饰者模式的**追加归并**这几种方式。

## 核心概念

- 虚拟库： ShardingSphere的核心就是提供一个具备分库分表功能的虚拟库，他是一个ShardingSphereDatasource实例。应用程序只需要像操作单数据源一样访问这个实例即可。
    - 真实库： 实际保存数据的数据库。都被包含在ShardingSphereDatasource实例当中，由ShardingSphere决定未来需要使用哪个真实库。
- 逻辑表： 应用程序直接操作的逻辑表。
    - 真实表： 实际保存数据的表。与逻辑表表名不需要一致，但是需要有相同的表结构，可以分布在不同的真实库中。应用可以维护一个逻辑表与真实表的对应关系，所有的真实表默认也会映射成为ShardingSphere的虚拟表。
- 分布式主键生成算法： 给逻辑表生成唯一主键。由于逻辑表的数据是分布在多个真实表当中的，所有，单表的索引就无法保证逻辑表的ID唯一性。
    - ShardingSphere集成了几种常见的基于单机生成的分布式主键生成器。比如SNOWFLAKE，COSID_SNOWFLAKE**雪花算法**可以生成单调递增的long类型的数字主键，还有UUID，NANOID可以生成字符串类型的主键。
    - 当然，ShardingSphere也支持应用自行扩展主键生成算法。比如基于Redis，Zookeeper等第三方服务，自行生成主键。
- 分片策略： 表示逻辑表要如何分配到真实库和真实表当中，分为**分库策略和分表策略**两个部分。
    - 分片策略由**分片键和分片算法**组成。
        1. **分片键**：是进行数据水平拆分的关键字段。如果没有分片键，ShardingSphere将只能进行全路由，SQL执行的性能会非常差。
        2. **分片算法**：则表示根据分片键如何寻找对应的真实库和真实表。简单的分片策略可以使用Groovy表达式直接配置，当然，ShardingSphere也支持自行扩展更为复杂的分片算法。

## 实战案例

> 实战过程中，需要配置数据源信息，逻辑表对应的真实节点和分库分表策略（**分片字段**和**分片算法**）

- 另外搭建参考[shardingsphere-jdbc的基本使用及各种分片策略](https://blog.csdn.net/qq_39944028/article/details/131266440)

### 创建数据库

当企业用户创建一条采购订单 ， 会生成如下记录：

1. 订单基础表 ： t_ent_order 表 , 1条记录 ；
2. 订单详情表： t_ent_order_detail ，1条记录；
3. 订单明细表： t_ent_order_item , 多条记录 。

订单每年预估生成记录 1 亿条，数据量不大也不小，笔者参考原来神州专车的分库分表方式，订单数据采用了如下的分库分表策略：

- 订单基础表按照 ent_id (企业用户编号) 分库（四个分库），订单详情表保持一致。
- 订单明细表按照 ent_id (企业用户编号) 分库 (四个分库)，同时也要按照 ent_id (企业编号) 分表(八个分表)。

首先创建 4 个库，分别是：ds_0、ds_1、ds_2、ds_3 。

- 这四个分库，每个分库都包含 **订单基础表 ， 订单详情表 ，订单明细表** 。但是因为明细表需要分表，所以包含多张表。
- 在 doc 中 ，分别在 4个库里执行 init.sql 语句 ，执行后效果见图 。

<img src="../assets/up-c9971a2fa253252d45c65a54b4fd07eb1f6.png" alt="img" style="zoom: 50%;" />

### 使用shardingsphere jdbc

springboot 项目中配置依赖 ：

- 重点强调， 原来分库分表之前， 很多 springboot 工程依赖 druid ，必须要删除

```xml
<dependency>
    <groupId>org.apache.shardingsphere</groupId>
    <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
    <version>4.1.1</version>
</dependency>
```

配置文件中配置如下：

```yaml
shardingsphere:
  datasource:
    enabled: true
    names: ds0,ds1,ds2,ds3
    ds0:
      type: com.alibaba.druid.pool.DruidDataSource
      driverClassName: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://127.0.0.1:3306/ds_0?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=GMT%2B8&useTimezone=true
      username: root
      password: ilxw
    ds1:
      type: com.alibaba.druid.pool.DruidDataSource
      driverClassName: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://127.0.0.1:3306/ds_1?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=GMT%2B8&useTimezone=true
      username: root
      password: ilxw
    ds2:
      type: com.alibaba.druid.pool.DruidDataSource
      driverClassName: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://127.0.0.1:3306/ds_2?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=GMT%2B8&useTimezone=true
      username: root
      password: ilxw
    ds3:
      type: com.alibaba.druid.pool.DruidDataSource
      driverClassName: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://127.0.0.1:3306/ds_3?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=GMT%2B8&useTimezone=true
      username: root
      password: ilxw
  props:
    # 日志显示 SQL
    sql.show: true
  sharding:
    tables:
      # 订单表基础表
      t_ent_order:
        # 真实表
        actualDataNodes: ds$->{0..3}.t_ent_order
        # 分库策略
        databaseStrategy:
          complex:
            sharding-columns: id,ent_id
            algorithm-class-name: cn.javayongshardingsphere.jdbc.service.sharding.HashSlotAlgorithm
        # 分表策略
        tableStrategy:
          none:
      # 订单条目表
      t_ent_order_item:
        # 真实表
        actualDataNodes: ds$->{0..3}.t_ent_order_item_$->{0..7}
        # 分库策略
        databaseStrategy:
          complex:
            sharding-columns: id,ent_id
            algorithm-class-name: cn.javayongshardingsphere.jdbc.service.sharding.HashSlotAlgorithm
        # 分表策略
        tableStrategy:
          complex:
            sharding-columns: id,ent_id
            algorithm-class-name: cn.javayongshardingsphere.jdbc.service.sharding.HashSlotAlgorithm
      # 订单详情表
      t_ent_order_detail:
        # 真实表
        actualDataNodes: ds$->{0..3}.t_ent_order_detail
        # 分库策略
        databaseStrategy:
           complex:
              sharding-columns: id,ent_id
              algorithm-class-name: cn.javayongshardingsphere.jdbc.service.sharding.HashSlotAlgorithm
        # 分表策略
        tableStrategy:
            complex:
              sharding-columns: id,ent_id
              algorithm-class-name: cn.javayongshardingsphere.jdbc.service.sharding.HashSlotAlgorithm
    bindingTables:
      - t_ent_order,t_ent_order_detail
```

配置很简单，需要配置如下几点：

- 配置数据源，上面配置数据源是： ds0、ds1、ds2、ds3 ；
- 配置打印日志，也就是：sql.show ，在测试环境建议打开 ，便于调试；
- 配置哪些表需要分库分表 ，在 `shardingsphere.datasource.sharding.tables` 节点下面配置：

![img](../assets/up-4212b6ecc449d47123b5168313aad087ac4.png)

标红的是分库算法 ，下面的分表算法也是一样的。



上图中看到配置分片规则包含如下两点：

1. 真实节点

 对于应用来讲，查询的**逻辑表**是：t_ent_order_item 。

 它们在数据库中的真实形态是：`t_ent_order_item_0` 到 `t_ent_order_item_7`。

 真实数据节点是指数据分片的最小单元，由数据源名称和数据表组成。

 订单明细表的真实节点是：`ds$->{0..3}.t_ent_order_item_$->{0..7}` 。

2. 分库分表算法

配置分库策略和分表策略 , 每种策略都需要配置**分片字段**（ sharding-columns ）和**分片算法**。

## 分片算法

> 实现分布式主键直接路由到对应分片，则需要使用**基因法 & 自定义复合分片算法** 。

分片算法和阿里开源的数据库中间件 **cobar 路由算法**非常类似的。

假设现在需要将订单表平均拆分到4个分库 shard0 ，shard1 ，shard2 ，shard3 。

首先将 [0-1023] 平均分为4个区段：[0-255]，[256-511]，[512-767]，[768-1023]，然后对字符串（或子串，由用户自定义）做 hash， hash 结果对 1024 取模，最终得出的结果 **slot** 落入哪个区段，便路由到哪个分库。

<img src="../assets/up-95a591ba73b27acd967f4d4907722369a37.png" alt="img" style="zoom: 67%;" />

看起来分片算法很简单，但我们需要按照订单 ID 查询订单信息时依然需要路由四个分片，效率不高，那么如何优化呢 ？

答案是：**基因法** & **自定义复合分片算法**。

### 基因法

基因法：是指在订单 ID 中携带企业用户编号信息，可以在创建订单 **order_id** 时使用**雪花算法**，然后将 **slot** 的值保存在 **10位工作机器 ID** 里。

路由算法 ，可以和 **雪花算法** 天然融合在一起， 订单 **order_id** 使用雪花算法，我们可以将 **slot** 的值保存在 **10位工作机器ID** 里。

<img src="../assets/up-10106ff9ac00e5520ea047be17cb82077ce.png" alt="img" style="zoom: 67%;" />

通过订单 order_id 可以反查出  **slot** , 就可以定位该用户的订单数据存储在哪个分片里。

```java
Integer getWorkerId(Long orderId) {
 Long workerId = (orderId >> 12) & 0x03ff;
 return workerId.intValue();
}
```

### ID 生成器

1. 查询本地内存，判定是否可以从本地队列中获取 currentTime , seq 两个参数 ，若存在，直接组装；
2. 若不存在，调用 redis 的 INCRBY 命令 ，这里需要传递一个步长值，便于放一篇数据到本地内存里；
3. 将数据回写到本地内存 ；
4. 重新查询本地内存，本地队列中获取 currentTime , seq 两个参数 ，组装最后的结果，返回给生成器 。

<img src="../assets/up-a8ebd81abcd817a760b11c26cc3f5ce6ea6.png" style="zoom:67%;" />

下图展示了订单 ID 使用**雪花算法**的生成过程，生成的编号会携带企业用户 ID 信息。

![img](../assets/up-d8247d1c5da5b248fd08ce079ac448b760f.png)



### 自定义复合分片算法

解决了**分布式 ID** 问题，接下来的一个问题：sharding-jdbc 可否支持按照订单 ID ，企业用户 ID 两个字段来决定分片路由吗？

答案是：**自定义复合分片算法**。只需要实现 **ComplexKeysShardingAlgorithm** 类即可。

![img](../assets/up-831524520a49c17c07710b876f5f489ac64.png)

复合分片的算法流程非常简单：

1. 分片键中有主键值，则直接通过主键解析出路由分片；

2. 分片键中不存在主键值 ，则按照其他分片字段值解析出路由分片。

## 分库分表扩容

> 平滑扩容的核心是**全量同步**和**实时双向同步**，工程上有不少细节。

既然做了分库分表，如何实现**平滑扩容**也是一个非常有趣的话题。

### 当前分库分表情况

通过运维管理平台分析数据库实例使用情况（一般和运维一起），分析核心属性：表空间大小，表空间占比 ，数据容量，数据碎片率，表行数，平均行长。

数据库的瓶颈主要体现在：磁盘、CPU、内存、网络、连接数，而连接数主要是受 CPU 和 内存影响。

1. CPU 和内存可以通过升级配置来提升 ， 磁盘可以使用 SSD 提升写入速度 ，可以运维同学解决；

2. 磁盘 IO 在大量写入的情况下，写入性能会急剧下降 ，考虑分库；
3. 考虑到 MySQL **InnoDB  存储引擎** B+ tree 的特性 ，单表存储一般超过 1000万 ，IO 速度会下降，分表可以提升读取和写入速度；

归根到底，分库分表需要**业务增长以及成本**(服务器，人力投入)。 

### 数据迁移内容

见下图，假设原来订单数据有 4 个实例 ，每个实例一个数据库，每个数据库上包含 16 张表，现在需要把 4 个实例迁移到8个实例上，每个实例上一个数据库，每个数据库包含 64 张表 。

<img src="../assets/up-93a52112e8fedceaa745487cbb0fccf62f5.png" style="zoom:67%;" />

整个数据迁移工作包括 ：

1. 前期准备

2. 数据同步环节（ 历史数据全量同步、增量数据实时同步、rehash ）

3. 数据校验环节（ 全量校验、实时校验、校验规则配置 ）

4. 数据修复工具

### 前期准备

在数据同步之前，需要梳理迁移范围。

- 业务唯一主键 

    在进行数据同步前，需要先梳理所有表的唯一业务 ID，只有确定了唯一业务 ID 才能实现数据的同步操作。

    需要注意的是：

    业务中是否有使用数据库自增 ID 做为业务 ID 使用的，如果有需要业务先进行改造 。另外确保每个表是否都有唯一索引，

    一旦表中没有唯一索引，就会在数据同步过程中造成数据重复的风险，所以我们先将没有唯一索引的表根据业务场景增加唯一索引（有可能是联合唯一索引）。

- 迁移哪些表，迁移后的分库分表规则

    分表规则不同决定着 rehash 和数据校验的不同。需逐个表梳理是用户ID纬度分表还是非用户ID纬度分表、是否只分库不分表、是否不分库不分表等等。

### 数据同步

数据同步整体方案见下图，数据同步基于 binlog ，独立的中间服务做同步，对业务代码无侵入。

<img src="../assets/up-2f372c25c5a4c61829c98fc921638bedf8d.png" style="zoom: 80%;" />

#### 历史数据全量同步

> 也就是将旧库（ 4个实例，每个实例 16 张表）迁移到新库（ 8 个实例，每个实例 64 张表）。

通常的做法是：单独一个服务，使用游标的方式从旧库分片 select 语句，经过 rehash 后批量插入 （batch insert）到新库，需要配置jdbc 连接串参数 rewriteBatchedStatements=true 才能使批处理操作生效。

另外特别需要注意的是，历史数据也会存在不断的更新，如果先开启历史数据全量同步，则刚同步完成的数据有可能不是最新的。

- 所以这里的做法是，先开启**增量数据单向同步**（从旧库到新库），此时只是开启积压 kafka 消息并不会真正消费；
- 然后在开始**历史数据全量同步**，当历史全量数据同步完成后，在开启消费 kafka 消息进行增量数据同步（提高全量同步效率减少积压也是关键的一环），这样来保证迁移数据过程中的数据一致。

#### 增量数据单向同步

增量数据同步考虑到灰度切流稳定性、容灾 和 可回滚能力 ，采用实时双向同步方案，切流过程中一旦新库出现稳定性问题或者新库出现数据一致问题，可快速回滚切回旧库，保证数据库的稳定和数据可靠。

<img src="../assets/up-3de1ad6f4801271f1836cd59a9b2f056bfb.png" style="zoom:80%;" />

增量数据实时同步的大体思路 ：

1. 过滤循环消息：需要过滤掉循环同步的 binlog 消息 ;
2. 数据合并：同一条记录的多条操作只保留最后一条。为了提高性能，数据同步组件接到 kafka 消息后不会立刻进行数据流转，而是先存到本地阻塞队列，然后由本地定时任务每X秒将本地队列中的N条数据进行数据流转操作。此时N条数据有可能是对同一张表同一条记录的操作，所以此处只需要保留最后一条（类似于 redis aof 重写）;
3. update 转 insert ：数据合并时，如果数据中有 insert + update 只保留最后一条 update ，会执行失败，所以此处需要将update转为 insert 语句 ;
4. 按新表合并 ：将最终要提交的 N 条数据，按照新表进行拆分合并，这样可以直接按照新表纬度进行数据库批量操作，提高插入效率；

整个过程中有几个问题需要注意：

问题 1 ：怎么防止因异步消息无顺序而导致的数据一致问题 ？

首先 kafka 异步消息是存在顺序问题的，但是要知道的是 binlog 是顺序的，所以数据传输服务在对 kafka 消息投递时也是顺序的，此处要做的就是一个库保证只有一个消费者就能保障数据的顺序问题、不会出现数据状态覆盖，从而解决数据一致问题。

问题 2 ：是否会有丢消息问题，比如消费者服务重启等情况下 ？

这里没有采用自动提交 offset ，而是每次消费数据最终入库完成后，将 offset 异步存到一个 mysql 表中，如果消费者服务重启宕机等，重启后从 mysql 拿到最新的offset开始消费。这样唯一的一个问题可能会出现瞬间部分消息重复消费,但是因为上面介绍的binlog是顺序的, 所以能保证数据的最终一致。

问题3：update 转 insert 会不会丢字段 ？

binlog 是全字段发送,不会存在丢字段情况。

**双向同步时的 binlog 循环消费问题**

想象一下，业务写一条数据到旧实例的一张表，于是产生了一条 binlog ； 数据同步中间件接到 binlog 后，将该记录写入到新实例，于是在新实例也产生了一条 binlog ；此时 数据同步中间件又接到了该 binlog ......不断循环，消息越来越多，数据顺序也被打乱。

怎么解决该问题呢？ 我们采用数据染色方案，只要能够标识写入到数据库中的数据使数据同步中间件写入而非业务写入，当下次接收到该 binlog 数据的时候就不需要进行再次消息流转。

所以 数据同步中间件要求，每个数据库实例创建一个事务表，该事务表 tb_transaction 只有 id、tablename、status、create_time、update_time 几个字段，status 默认为 0。

再回到上面的问题，业务写一条数据到旧实例的一张表，于是产生了一条 binlog ； 数据同步中间件接到 binlog 后，如下操作：

``` SQL
# 开启事务，用事务保证一下sql的原子性和一致性
start transaction;
set autocommit = 0;
# 更新事务表status=1，标识后面的业务数据开始染色
update tb_transaction set status = 1 where tablename = ${tableName};
# 以下是业务产生binlog
insert xxx;
update xxx;
update xxx;
# 更新事务表status=0，标识后面的业务数据失去染色
update tb_transaction set status = 0 where tablename = ${tableName};
commit;
```

此时数据同步中间件将上面这些语句打包一起提交到新实例，新实例更新数据后也会生产对应上面语句的 binlog ；当数据同步中间件再次接收到 binlog 时，只要判断遇到 tb_transaction 表 status=1 的数据开始，后面的数据都直接丢弃不要，直到遇到status=0时，再继续接收数据，以此来保证数据同步中间件只会流转业务产生的消息。

![](../assets/up-55c4813079e817b17708d7e0d248fa73f9e.png)



##### 数据校验环节

数据校验模块由数据校验服务data-check模块来实现，主要是基于数据库层面的数据对比，逐条核对每一个数据字段是否一致，不一致的话会经过配置的校验规则来进行重试或者报警。

**全量校验**

以旧库为基准，查询每一条数据在新库是否存在，以及个字段是否一致。

以新库为基准，查询每一条数据在旧库是否存在，以及个字段是否一致。

**实时校验**

定时任务每5分钟校验，查询最近5+1分钟旧库和新库更新的数据，做diff。

差异数据进行二次、三次校验（由于并发和数据延迟存在），三次校验都不同则报警。

<img src="../assets/up-c1c75bd018103ff8753e8e2cb80889765b7.png" style="zoom: 67%;" />

##### 数据修复工具

经过数据校验，一旦发现数据不一致，则需要对数据进行修复操作。

数据修复有两种方案，一种是适用于大范围的数据不一致，采用重置kafka offset的方式，重新消费数据消息，将有问题的数据进行覆盖。

<img src="../assets/4257659cd39a4fcbb9377b292c7bbce4.png" alt="img" style="zoom: 50%;" />

第二种是适用于小范围的数据不一致，数据修复模块自动拉取数据校验data-check模块记录的sls日志，进行日志解析，生成同步语句，更新到目标库。

![img](../assets/d38386cc893e4f639ae56e962643ba45.png)

##### 灰度切换数据源

**整体灰度切流方案**

整体灰度方案：SP+用户纬度来实现，SP纬度：依靠灰度环境切量来做，用户纬度：依赖用户ID后四位百分比切流。

灰度切量的过程一定要配合停写（秒级），为什么要停写，因为数据同步存在一定延迟（正常毫秒级），而所有业务操作一定要保障都在一个实例上，否则在旧库中业务刚刚修改了一条数据，此时切换到新库如果数据还没有同步过来就是旧数据会有数据一致问题。所以步骤应该是：

1. 先停写
2. 观察数据全部同步完
3. 在切换数据源
4. 最后关闭停写，开始正常业务写入

**切流前准备——ABC验证**

虽然在切流之前，在测试环境进过了大量的测试，但是测试环境毕竟和生产环境不一样，生产环境数据库一旦出问题就可能是灭顶之灾，虽然上面介绍了数据校验和数据修复流程，但是把问题拦截在发生之前是做服务稳定性最重要的工作。

因此我们提出了ABC验证的概念，灰度环境ABC验证准备：

1. 新购买两套数据库实例，当前订单库为A，新买的两套为分别为B、C
2. 配置DTS从A单项同步到B（dts支持同构不需要rehash的数据同步），B做为旧库的验证库，C库做为新库
3. 用B和C作为生产演练验证
4. 当B和C演练完成之后，在将A和C配置为正式的双向同步

<img src="../assets/f48411309ee648e08240c7814b93a01d.png" alt="img" style="zoom:50%;" />

**灰度切流步骤**

具体灰度方案和数据源切换流程：

1. 代码提前配置好两套数据库分库分表规则。
2. 通过ACM配置灰度比例。
3. 代码拦截mybatis请求，根据用户id后四位取模，和ACM设置中设置的灰度比例比较，将新库标识通过ThreadLocal传递到分库分表组件。
4. 判断当前是否有灰度白名单，如命中将新库标识通过ThreadLocal传递到分库分表组件。
5. 分库分表组件根据ACM配置拿到新分库的分表规则，进行数据库读写操作。
6. 切量时会配合ACM配置灰度比例命中的用户进行停写。

## 测试接口

修改配置文件 application-test.yml ，配置好 MySQL 数据库 和 Redis 服务 。

启动 Main 函数：

![](../assets/jdbc4main.png)

启动过程中，会打印 shardingsphere jdbc 日志 。

![](../assets/startjdbcmain.gif)

启动成功之后，访问 swagger ui 地址：

>  http://localhost:9793/shardingsphere-jdbc-server/doc.html#/home

接下来，我们进行两个测试：新增订单和按照订单 ID 查询

**1、测试存储订单**

![](../assets/jdbc4save.png)

点击发送按钮，接口响应成功。

![](../assets/jdbc4console.gif)

我们插入1 条订单记录、1 条订单详情表进入 ds3 分片，并且 2 条订单条目表进入 ds3 分片的 t_ent_order_item_7 表。

**2、测试存储订单**

![](../assets/jdbc4queryorder.png)

参数名称是 orderId , 参数值：609335823493160961 ，点击发送按钮，接口响应成功 , 返回订单信息。

![](../assets/jdbc4queryconsole.png)

