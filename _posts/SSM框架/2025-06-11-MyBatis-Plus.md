---
title: MyBatis Plus
categories:
  - SSM框架
tags:
  - SSM
  - Spring-Boot
  - MyBatis
  - MyBatis-Plus
  - East-Trans
  - MapStruct
location:
  - 黄金时代
abbrlink: 'mybatis_plus'
permalink: 'mybatis_plus'
date: 2025-06-11 16:41:00
updated: 2025-06-11 16:41:00
---

> 摘要：MyBatis-Plus 是基于 MyBatis 框架的一个**增强**工具，主要目的是**简化 MyBatis 的开发过程**，提供更加简洁、方便的 CRUD 操作。

<!-- more -->

---

## 目录

[TOC]

## MyBatis Plus

MyBatis-Plus 是基于 MyBatis 框架的一个**增强**工具，主要目的是**简化 MyBatis 的开发过程**，提供更加简洁、方便的 CRUD 操作。

- 在保留 MyBatis 强大功能的基础上，通过**封装和优化**一些常见操作来**提高开发效率**。

### 功能特性

提供了许多开箱即用的功能，部分核心特性包括：

1. **无侵入设计**：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑。不会改变 MyBatis 原有的 API 和使用方式，可以自由选择 MyBatis 和 MyBatis-Plus 的功能。
2. **强大的 CRUD 操作**：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求
    - **自动生成 CRUD 代码**：通过 `BaseMapper` 和 `ServiceImpl` 接口，提供了一系列 CRUD 操作的方法，如 `insert`、`delete`、`update` 和 `select`，减少了**重复的** SQL 编写工作。
3. **条件构造器**：如 `QueryWrapper`，可以通过**链式编程**方式轻松构建复杂的查询条件。
4. 分页查询、性能优化、以及支持多种数据。
5. **损耗小**：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作
6. **支持 Lambda 形式调用**：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错
7. **支持主键自动生成**：支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题
8. **内置代码生成器**：采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用
9. **内置分页插件**：基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 List 查询
    - **分页插件支持多种数据库**：支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer 等多种数据库

参考：[mybatis plus 常用知识汇总（保姆级教程！~） ](https://www.cnblogs.com/xxctx/p/18404560)

### 快速开始

#### 引入依赖

在 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-12-mybatis/lab-12-mybatis-plus/pom.xml) 文件中，引入相关依赖。

- 相比来说，将 `mybatis-spring-boot-starter` 替换成 `mybatis-plus-boot-starter` 。

```xml
<!-- 实现对 MyBatis Plus 的自动化配置 -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.2.0</version>
</dependency>
```

#### Java 配置类

创建 [`Application.java`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-12-mybatis/lab-12-mybatis-plus/src/main/java/cn/iocoder/springboot/lab12/mybatis/Application.java) 类，配置 `@MapperScan` 注解，扫描对应 Mapper 接口所在的包路径。

```java
// MyBatisPlusConfig.java

@MapperScan(basePackages = "cn.iocoder.springboot.lab12.mybatis.mapper")
public class MyBatisPlusConfig {
}
```

#### 应用配置文件

[`resources/application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-12-mybatis/lab-12-mybatis-plus/src/main/resources/application.yaml) 配置文件。

- 将 `mybatis` 替换成 `mybatis-plus` 配置项目。实际上，如果老项目在用 `mybatis-spring-boot-starter` 的话，直接将 `mybatis` 修改成 `mybatis-plus` 即可。
- 相比 `mybatis` 配置项来说，`mybatis-plus` 增加了更多配置项，也因此无需在配置 [`mybatis-config.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-12-mybatis/lab-12-mybatis-xml/src/main/resources/mybatis-config.xml) 配置文件。更多的 MyBatis-Plus 配置项，可以看看 [MyBatis-Plus 使用配置](https://mybatis.plus/config/#mapperlocations) 。
- 配置 `logging` 的原因是，方便看到 MyBatis-Plus 自动生成的 SQL 。生产环境下，记得关闭。

```yaml
spring:
  # datasource 数据源配置内容
  datasource:
    url: jdbc:mysql://47.112.193.81:3306/testb5f4?useSSL=false&useUnicode=true&characterEncoding=UTF-8
    driver-class-name: com.mysql.jdbc.Driver
    username: testb5f4
    password: F4df4db0ed86@11

# mybatis-plus 配置内容
mybatis-plus:
  configuration:
    map-underscore-to-camel-case: true # 虽然默认为 true ，但是还是显示去指定下。
  global-config:
    db-config:
      id-type: auto # ID 主键自增
      logic-delete-value: 1 # 逻辑已删除值(默认为 1)
      logic-not-delete-value: 0 # 逻辑未删除值(默认为 0)
  mapper-locations: classpath*:mapper/*.xml
  type-aliases-package: cn.iocoder.springboot.lab12.mybatis.dataobject

# logging
logging:
  level:
    # dao 开启 debug 模式，mybatis 输入 sql
    cn:
      iocoder:
        springboot:
          lab12:
            mybatis:
              mapper: debug
```

### 自动生成

> 创建数据库表

[三种方式](https://blog.csdn.net/SoulNone/article/details/126445011)不用纠结，对于使用的人来说不分优劣，只分好用不好用。

1. **AutoGenerator**：官方 Java 配置代码。可以快速生成 `Entity、Mapper、Mapper XML、Service、Controller`等各个模块的代码，极大的提升了开发效率。

    - 自定义模板，使用的是Velocity模板引擎（也可使用Freemarker）

    - [MybatisPlus 新代码生成器](https://baomidou.com/guides/new-code-generator/)

    - [MybatisPlus 代码生成器及配置注释](https://www.cnblogs.com/buchizicai/p/16606917.html)

        > 这里讲解的是新版 （mybatis-plus 3.5.1+版本），旧版不兼容

    - [代码生成器配置](https://baomidou.com/reference/new-code-generator-configuration)

2. **MyBatis Plus 插件**：简洁易用

3. ~~MyBatisX-Generator 插件~~

4. Mybatis-plus Code Generator 插件

5. **EasyCode 插件**，最全面

#### AutoGenerator

自带的，通过 Java 程序自动生成。

```java
FastAutoGenerator.create("url", "username", "password")
        .globalConfig(builder -> builder
                .author("Baomidou")
                .outputDir(Paths.get(System.getProperty("user.dir")) + "/src/main/java")
                .commentDate("yyyy-MM-dd")
        )
        .packageConfig(builder -> builder
                .parent("com.baomidou.mybatisplus")
                .entity("entity")
                .mapper("mapper")
                .service("service")
                .serviceImpl("service.impl")
                .xml("mapper.xml")
        )
        .strategyConfig(builder -> builder
                .entityBuilder()
                .enableLombok()
        )
        .templateEngine(new FreemarkerTemplateEngine())
        .execute();
```

##### 生成方式

代码生成器目前支持两种生成方式：

1. **DefaultQuery (元数据查询)**
    - **优点：** 根据**通用接口**读取数据库元数据相关信息，对数据库通用性较好。
    - **缺点：** 依赖数据库厂商驱动实现。
    - **备注：** 默认方式，部分类型处理可能不理想。
2. ~~SQLQuery (SQL查询)~~
    - **优点：** 需要根据数据库编写对应表、主键、字段获取等查询语句。
    - **缺点：** 通用性不强，同数据库厂商不同版本可能会存在兼容问题（例如，H2数据库只支持1.X版本）。
    - **备注：** 后期不再维护。

如果是已知数据库（无版本兼容问题），请继续按照原有的SQL查询方式继续使用，示例代码如下：

```java
// MYSQL 示例 切换至SQL查询方式,需要指定好 dbQuery 与 typeConvert 来生成
FastAutoGenerator.create("url", "username", "password")
                .dataSourceConfig(builder ->
                        builder.databaseQueryClass(SQLQuery.class)
                                .typeConvert(new MySqlTypeConvert())
                                .dbQuery(new MySqlQuery())
                )

                // Other Config ...
```

元数据查询目前有如下问题：

1. 不支持使用 NotLike 的方式反向生成表。

2. 无法读取**表注释**，解决方法：

    - MySQL链接增加属性 `remarks=true&useInformationSchema=true`
    - Oracle链接增加属性 `remarks=true` 或者 `remarksReporting=true`（某些驱动版本）
    - SqlServer：驱动不支持

3. 部分 PostgreSQL 类型处理不佳（如 json、jsonb、uuid、xml、money 类型），解决方法：

    - 转换成自定义的类型配合自定义 TypeHandler 来处理。
    - 扩展 typeConvertHandler 来处理（3.5.3.3 后增加了 typeName 获取）。

4. MySQL 下 **tinyint 字段**转换问题：

    - 当字段长度为 1 时，无法转换成 **Boolean 字段**，建议在指定数据库连接时添加 `&tinyInt1isBit=true`。
- 当字段长度大于 1 时，默认转换成 **Byte**，如果想继续转换成 Integer，可使用如下代码：

```java
FastAutoGenerator.create("url", "username", "password")
        .dataSourceConfig(builder ->
                builder.typeConvertHandler((globalConfig, typeRegistry, metaInfo) -> {
                    // 兼容旧版本转换成Integer
                    if (JdbcType.TINYINT == metaInfo.getJdbcType()) {
                        return DbColumnType.INTEGER;
                    }
                    return typeRegistry.getColumnType(metaInfo);
                })
        );
```

##### 依赖

由于代码生成器用到了模板引擎，请自行引入喜好的模板引擎。

MyBatis-Plus Generator 支持如下模板引擎：

- VelocityTemplateEngine(Default)
- FreemarkerTemplateEngine
- BeetlTemplateEngine
- EnjoyTemplateEngine

如果还想使用或适配其他模板引擎，可自行继承 `AbstractTemplateEngine` 并参考其他模板引擎实现自定义。

```xml
    <!--mysql数据库驱动-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.35</version>
    </dependency>
	<!--通过注解消除实际开发中的样板式代码-->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>

    <!--mybatis-plus启动器-->
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.3.0</version>
    </dependency>
    <!--代码生成器-->
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-generator</artifactId>
        <version>3.3.0</version>
    </dependency>

    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger-ui</artifactId>
        <version>2.9.2</version>
    </dependency>
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger2</artifactId>
        <version>2.9.2</version>
    </dependency>
```

##### 使用

可以通过以下两种形式使用代码生成器。

###### 快速生成

在 CodeGenerator 中的 main 方法中直接添加生成器代码，并进行相关配置，然后直接运行即可生成代码。

```java
// CodeGenerator.java
public static void main(String[] args) {

    FastAutoGenerator.create("url", "username", "password")
        .globalConfig(builder -> {
            builder.author("baomidou") // 设置作者
                .enableSwagger() // 开启 swagger 模式
                .outputDir("D://"); // 指定输出目录
        })

        .dataSourceConfig(builder ->
         	builder.typeConvertHandler((globalConfig, typeRegistry, metaInfo) -> {
                int typeCode = metaInfo.getJdbcType().TYPE_CODE;
                if (typeCode == Types.SMALLINT) {
                    // 自定义类型转换
                    return DbColumnType.INTEGER;
                }
                return typeRegistry.getColumnType(metaInfo);
            })
        )

        .packageConfig(builder ->
        	builder.parent("com.baomidou.mybatisplus.samples.generator") // 设置父包名
                   .moduleName("system") // 设置父包模块名
                   .pathInfo(Collections.singletonMap(OutputFile.xml, "D://")) // 设置mapperXml生成路径
         )

        .strategyConfig(builder ->
        	builder.addInclude("t_simple") // 设置需要生成的表名
                   .addTablePrefix("t_", "c_") // 设置过滤表前缀
         )

        .templateEngine(new FreemarkerTemplateEngine()) // 使用Freemarker引擎模板，默认的是Velocity引擎模板

        .execute();
}
```

###### ~~交互式生成~~

交互式生成在运行之后，会提示输入相应的内容，等待配置输入完整之后就自动生成相关代码。

如果需要更多例子可查看 test 包下面的 samples。

- [H2CodeGeneratorTest](https://github.com/baomidou/generator/blob/develop/mybatis-plus-generator/src/test/java/com/baomidou/mybatisplus/generator/samples/H2CodeGeneratorTest.java)
- [FastAutoGeneratorTest](https://github.com/baomidou/generator/blob/develop/mybatis-plus-generator/src/test/java/com/baomidou/mybatisplus/generator/samples/FastAutoGeneratorTest.java)

##### 配置

请移步至 [代码生成器配置](https://baomidou.com/reference/new-code-generator-configuration/) 查看。

###### BaseDO 类

1. baseEntity：用来写一些公共字段。例如：create_time（创建时间）、creator（创建人）、update_time、updator、logic_delete 等。
2. baseController：用来写基础公共的控制。

```java
package com..common.dao;
import java.io.Serializable;

//public class BaseEntity implements Serializable {
public abstract class BaseDO implements Serializable, TransPojo {
}
```

###### BaseMapper

#### MyBatisPlus 插件

选中Other菜单，会出现Config Database（配置数据库）和Code Generator（代码生成）

- 生成 Controller、Service、Entity

#### MyBatisX-Generator 插件

- mapper 和 xml 可以**来回跳转**
- mybatis.xml、mapper.xml 提示
- mapper 和 xml 支持类似 jpa 的自动提示（参考 MybatisCodeHelperPro）
- 集成 mybatis 生成器 GUI（从免费的 mybatis 插件复制）
    - 直接用数据库连接，不用另外配置。
    - Lombok 注解不是特别友好

#### MyBatisplus Code Generator

Mybatisplus 代码生成器，介绍了各种代码生成器的优点。本着约定大于配置的理念。

关键词：Mybatis Plus、Maven、Spring Boot、Lombok、Mysql、Freemarker、XMind、Excel 等。

提供两种形式：Windows 桌面工具 exe 和 IDEA 插件。

打开方式：工具 -> Mybatisplus 代码生成器 或 `Ctrl + Alt + 0`。

功能：

1. 一键生成 Mybatis Plus 代码，傻瓜式功能选择。
2. Freemarker **代码自定义模板**配置。
3. 工程化：支持 Maven 和 Spring Boot 工程化生成。
4. 多数据源管理。
5. 配置可以是数据持久化，用户操作记忆。
6. 一键导出数据库表的**思维导图**。
7. 一键导出 **Excel 设计文档**到数据库表。

#### EasyCode 插件

EasyCode是基于IntelliJ IDEA Ultimate版开发的一个代码生成插件，主要通过**自定义模板**（基于velocity）来生成各种想要的代码。

- 通常用于生成Entity、Dao、Service、Controller。
- 如果动手能力强还可以用于生成HTML、JS、PHP等代码。理论上来说只要是**与数据有关的代码**都是可以生成的。

- 直接用数据库连接，不用另外配置。
- **自定义模板**：点击 File->Settings->Easy Code->Template Setting。
    - 比如想在生成的 dao 层代码中，额外添加一个不需要任何条件，获取所有数据的getAll()方法（默认的生成模版中没有这个方法）。
    - 需要**编写自定义模版**。
- 支持导出配置。

### 实体类

- 实体类放在 `dal.dataobject` 包下，以 DO 结尾；mapper 数据库访问类放在 `dal.mysql` 包下，以 Mapper 结尾。

#### BaseDO

[BaseDO](https://github.com/YunaiV/yudao-cloud/blob/master/yudao-framework/yudao-spring-boot-starter-mybatis/src/main/java/cn/iocoder/yudao/framework/mybatis/core/dataobject/BaseDO.java)是所有数据库实体的**父类**：

- `abstract class`：
- `createTime` + `creator` 字段，创建人相关信息。
- `updater` + `updateTime` 字段，创建人相关信息。
- 增加了 `deleted` 字段，并添加了`@TableLogic`注解，设置该字段为**逻辑删除**的标记。
- LocalDateTime 日期时间

```java
// 抽象类
@Data
@JsonIgnoreProperties(value = "transMap") // 由于 Easy-Trans 会添加 transMap 属性，避免 Jackson 在 Spring Cache 反序列化报错
public abstract class BaseDO implements Serializable, TransPojo {

    /**
     * 创建时间，插入时调用 DefaultDBFillFieldHandler.inserFill 自动填充
     */
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createTime;
    /**
     * 最后更新时间，插入 or 更新时
     */
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updateTime;
    
    /**
     * 创建者，目前使用 AdminUserDO / MemberUserDO / SysUser 的 id 编号
     * 使用 String 类型的原因是，未来可能会存在非数值的情况，留好拓展性。
     */
    @TableField(fill = FieldFill.INSERT)
    private String creator;
    /**
     * 更新者
     */
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private String updater;
    
    /**
     * 是否删除
     */
    @TableLogic
    private Boolean deleted;

     /**
     * 把 creator、createTime、updateTime、updater 都清空，
     * 避免前端直接传递 creator 之类的字段，直接就被更新了
     */
    public void clean(){
        this.creator = null;
        this.createTime = null;
        this.updater = null;
        this.updateTime = null;
    }
}
```

对应的 SQL 字段如下：

```sql
`creator` 		varchar(64) 	CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci DEFAULT '' COMMENT '创建者',
`create_time` 	datetime 		NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
`updater` 		varchar(64) 	CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci DEFAULT '' COMMENT '更新者',
`update_time` 	datetime 		NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
`deleted` 		bit(1) 			NOT NULL DEFAULT b'0' COMMENT '是否删除',
```

#### UserDO

创建 [DO/UserDO.java](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-12-mybatis/lab-12-mybatis-plus/src/main/java/cn/iocoder/springboot/lab12/mybatis/dataobject/UserDO.java) 类 。相比 [「2.5 UserDO」](https://www.iocoder.cn/Spring-Boot/MyBatis/#) 来说，主要差别有：

- `extends BaeDO`：继承基类
- 实体类的注释要完整，特别是哪些字段是关联（**外键**）、枚举、冗余等。

```java
// UserDO.java
@Data
@EqualsAndHashCode(callSuper = true)
@TableName(value = "user")
public class UserDO extends BaeDO {

    /**
     * 用户编号
     */
    private Long id;

    private String username;
    private String password;

    // ... 无需 setting/getting 方法
}
```

对应的创建表的 SQL 如下：

```sql
CREATE TABLE `user` (
  	`id` 			int(11) 	NOT NULL 		AUTO_INCREMENT COMMENT '用户编号',
  	`username` 		varchar(64) COLLATE utf8mb4_bin 	DEFAULT NULL COMMENT '账号',
  	`password` 		varchar(32) COLLATE utf8mb4_bin 	DEFAULT NULL COMMENT '密码',

    `creator` 		varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci 	DEFAULT '' COMMENT '创建者',
    `create_time` 	datetime 	NOT NULL 	DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    `updater` 		varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci DEFAULT '' COMMENT '更新者',
    `update_time` 	datetime 	NOT NULL 	DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    `deleted` 		bit(1) 		NOT NULL 	DEFAULT b'0' COMMENT '是否逻辑删除。0-未删除；1-删除',
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_username` (`username`)
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;
```

##### ProductSkuDO

- property：DO 静态类，对应 SQL `properties varchar(512) null comment '属性数组，JSON 格式',`

```java
package cn..module.product.dal.dataobject.sku;

/**
 * 商品 SKU DO
 */
@TableName(value = "product_sku", autoResultMap = true)
@KeySequence("product_sku_seq") // 用于 Oracle、PostgreSQL、Kingbase、DB2、H2 数据库的主键自增。如果是 MySQL 等数据库，可不写。
@Data
@EqualsAndHashCode(callSuper = true)
@ToString(callSuper = true)
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class ProductSkuDO extends BaseDO {

    /**
     * 商品 SKU 编号，自增
     */
    @TableId
    private Long id;
    /**
     * SPU 编号
     *
     * 关联 {@link ProductSpuDO#getId()}
     */
    private Long spuId;
    /**
     * 属性数组，JSON 格式
     */
    @TableField(typeHandler = JacksonTypeHandler.class)
    private List<Property> properties;
    /**
     * 商品价格，单位：分
     */
    private Integer price;
    
    ...

    // ========== 营销相关字段 =========

    // ========== 统计相关字段 =========
    /**
     * 商品销量
     */
    private Integer salesCount;

    /**
     * 商品属性
     */
    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public static class Property {

        /**
         * 属性编号
         * 关联 {@link ProductPropertyDO#getId()}
         */
        private Long propertyId;
        /**
         * 属性名字
         * 冗余 {@link ProductPropertyDO#getName()}
         *
         * 注意：每次属性名字发生变化时，需要更新该冗余
         */
        private String propertyName;

        /**
         * 属性值编号
         * 关联 {@link ProductPropertyValueDO#getId()}
         */
        private Long valueId;
        /**
         * 属性值名字
         * 冗余 {@link ProductPropertyValueDO#getName()}
         *
         * 注意：每次属性值名字发生变化时，需要更新该冗余
         */
        private String valueName;

    }
}
```



#### 自动映射规则

主要涉及如何将数据库表和字段自动映射到 Java 实体类及其属性。

1. `@TableName`：表名与实体类名的映射
2. `@TableId`：主键的自动映射
3. `@TableField`：字段名与属性名的映射

#### @TableName

增加了 [`@TableName`](https://mybatis.plus/guide/annotation.html#tablename) 注解：设置了 UserDO 对应的表名是 `user` 。毕竟，要使用 MyBatis-Plus 给自动生成 CRUD 操作。

- 默认规则：MyBatis-Plus 默认使用实体类名作为数据库表名的前缀。比如，如果你的实体类名为 `User`，那么它会映射到名为 `user` 的数据库表。

- 自定义规则：可以使用 `@TableName` 注解来指定自定义的表名。例如：

    ```java
    @TableName("sys_user")
    public class User {
        // 属性和方法
    }
    ```

##### @KeySequence

> 在数据库设计中，主键的生成方式多种多样，而序列（Sequence）是一种常见的生成主键的方式。

用于标识实体类中的主键字段，并指定使用哪个数据库的序列来生成主键。

- 通过在实体类字段上添加 `@KeySequence` 注解，可以简单地实现**基于序列的主键生成**，无需手动处理序列的获取和使用。

- **`value` 属性：** 用于指定使用的**数据库序列**的名称。

```java
@TableName("infra_api_access_log")
@KeySequence(value = "infra_api_access_log_seq") // 用于 Oracle、PostgreSQL、Kingbase、DB2、H2 数据库的主键自增。如果是 MySQL 等数据库，可不写。
@Data
@AllArgsConstructor
public class ApiAccessLogDO extends BaseDO {
```

#### @TableId

##### 主键

`id` 主键编号，推荐使用 Long 型自增，原因是：

- 自增，保证数据库是按顺序写入，性能更加优秀。
- Long 型，避免未来业务增长，超过 Int 范围。

对应的 SQL 字段如下：

```sql
`id` bigint NOT NULL AUTO_INCREMENT COMMENT '编号',
```

项目的 `id` **默认**采用数据库自增的策略，如果希望使用 Snowflake **雪花算法**，可以修改 `application.yaml` 配置文件，将配置项 `mybatis-plus.global-config.db-config.id-type` 修改为 `ASSIGN_ID`。

##### @TableId

@TableId：专门用在**主键上**的注解，如果数据库中的主键字段名和实体中的属性名，不一样且不是驼峰之类的对应关系。

- 可以在实体中表示主键的属性上加@**Tableid**注解，并指定value属性值为表中主键的字段名，即可以对应上。

在 MyBatis-Plus 中，`@TableId` 注解用于标识实体类中的主键字段。

- 有两个主要属性：`value` 和 `type`。分别用于指定字段名和主键生成策略。

##### `@TableId` 注解属性

1. **`value`**:

    - **用途**：指定数据库表中的主键列名。它的值应该是数据库表中实际的列名。

    - 示例：如果数据库表中的主键列名是`user_id`，则在实体类中可以这样配置：

        ```java
        @TableId(value = "user_id", type = IdType.ASSIGN_UUID)
        private String userId;
        ```

2. **`type`**:

    - **用途**：指定主键生成策略。MyBatis-Plus 提供了多种主键生成策略，`IdType` 枚举类定义了这些策略。

    - **常用的生成策略**：

        - `IdType.AUTO`：数据库自动生成（通常是自增长 ID）。
        - `IdType.INPUT`：用户输入 ID（即需要手动设置）。
        - `IdType.ASSIGN_ID`：由 MyBatis-Plus 生成的 ID（通常是 UUID）。
        - `IdType.ASSIGN_UUID`：**生成 UUID**（字符串类型的唯一 ID）。

    - **示例**：如果想使用 UUID 作为主键，可以使用 `IdType.ASSIGN_UUID`：

        ```java
        @TableId(value = "user_id", type = IdType.ASSIGN_UUID)
        private String userId;
        ```

#### @TableField

> 表字段填充，is fill

- 默认规则：字段名和属性名默认是直接映射的。例如，数据库中的 `user_name` 字段会映射到实体类中的 `userName` 属性。

- 驼峰命名规则：MyBatis-Plus 默认启用了驼峰命名转换。即，数据库中的下划线命名（如 `user_name`）会被自动转换为 Java 属性的驼峰命名（如 `userName`）。

- 自定义字段映射：可以使用`@TableField`注解来指定自定义的字段名。例如：

    ```java
    @TableField("user_name")
    private String userName;
    ```

`@TableField`的其他用法：

| 值        | 描述                                    | 备注                                                         |
| --------- | --------------------------------------- | ------------------------------------------------------------ |
| value     | 数据表中的字段名                        | 驼峰命名方式，该值可无                                       |
| update    | 预处理 set 字段自定义注入               |                                                              |
| condition | 预处理 WHERE 实体**条件**自定义运算规则 |                                                              |
| el        | 详看注释说明                            |                                                              |
| **exist** | 是否为**数据库表**字段                  | 默认 true 存在，false 不存在                                 |
| strategy  | 字段验证                                | 默认 非 null 判断，查看 com.baomidou.mybatisplus.enums.FieldStrategy |
| **fill**  | 字段**自动填充**标记                    | FieldFill, 配合**自动填充**使用 。DEFAULT、INSERT、UPDATE、INSERT_UPDATE。 |

##### value 属性字段映射

比如说数据库中字段为`last_name`，而实体类的属性为`lastName`。

前提是在**全局策略配置**中将**驼峰命名**关闭。

```xml
<property name="dbColumnUnderline" value="false"></property>
```

- 关于MyBatisPlus中进行通用CRUD全局策略配置参照：https://blog.csdn.net/BADAO_LIUMANG_QIZHI/article/details/89425049

这时就可以在**实体类上**添加：

```java
@TableField(value="last_name")
```

##### sql语句的关键词

如果表中字段存在 **sql语句的关键词** ，比如 `desc` , 那么需要按照下面的写法, 否则mybatis plus 拼接含有 `desc字段` 的sql语句时会报错。

```java
@TableField("`desc`") 
private String desc;
```

##### select = false

`@TableField(select = false)`

作用：用于指定某个字段在执行 SQL 查询时**不参与查询**，即在 SELECT 语句中不包含该字段。

用途：通常用于那些只需要在数据库操作中存在，但不需要在查询结果中显示的字段。

- 例如，**逻辑删除**字段、内部使用的字段等。

```java
@TableField(select = false)
private Integer age;
```

##### exist 属性

`@TableField(exist = false)`

作用：用于指定某个字段不对应数据库中的任何字段，即在数据库表中**不存在该字段**。

用途：通常用于那些**只在 Java 实体中存在**的字段，但在数据库表中没有相应的字段。例如，用于计算或临时存储的数据。

```java
@TableField(exist = false)
private Integer age;
```

又比如在实体类中有一个属性为remark，但是在数据库中**没有这个字段**，

但是在执行插入操作时，给实体类的remark属性赋值了，那么可以通过在实体类的remark属性上添加：

```java
@TableField(exist=false)
 private String remark;
```

##### fill 属性自动填充

[DefaultDBFieldHandler (opens new window)](https://github.com/YunaiV/yudao-cloud/blob/master/yudao-framework/yudao-spring-boot-starter-mybatis/src/main/java/cn/iocoder/yudao/framework/mybatis/core/handler/DefaultDBFieldHandler.java)基于 MyBatis 自动填充机制，实现 **BaseDO 通用字段**的自动设置。

![DefaultDBFieldHandler 自动填充](../assets/mybatis_02.png)

##### typeHandler 字段类型处理器

MyBatis Plus 提供 **TypeHandler 字段类型处理器**，用于 **JavaType 与 JdbcType** 之间的转换。

- `@Builder`：
- 使用时，需要设置实体的 `@TableName` 注解的 `@autoResultMap = true`。
- `@TableField`

###### 复杂字段类型转换

常用的字段类型处理器有：

- [JacksonTypeHandler](https://github.com/baomidou/mybatis-plus/blob/a3e121c27cd26cb7c546dfb88190f3b1f574dc38/mybatis-plus-extension/src/main/java/com/baomidou/mybatisplus/extension/handlers/JacksonTypeHandler.java)：通用的 Jackson 实现 JSON 字段类型处理器。如 JavaBean 转为 JSON。

![字段处理器的示例](../assets/mybatis_13.png)

###### 字段加密

[EncryptTypeHandler (opens new window)](https://github.com/YunaiV/yudao-cloud/blob/master/yudao-framework/yudao-spring-boot-starter-mybatis/src/main/java/cn/iocoder/yudao/framework/mybatis/core/type/EncryptTypeHandler.java)，基于 [Hutool AES (opens new window)](https://plus.hutool.cn/apidocs6/org/dromara/hutool/crypto/symmetric/AES.html)实现字段的加密与解密。

例如说，[数据源配置 (opens new window)](https://github.com/YunaiV/yudao-cloud/blob/master/yudao-module-infra/yudao-module-infra-server/src/main/java/cn/iocoder/yudao/module/infra/dal/dataobject/db/DataSourceConfigDO.java)的 `password` 密码需要实现**加密存储**，则只需要在该字段上添加 **EncryptTypeHandler 处理器**。

示例代码如下：

```java
@TableName(value = "infra_data_source_config", autoResultMap = true) // 添加 autoResultMap = true
public class DataSourceConfigDO extends BaseDO {

    // ... 省略其它字段
    /**
     * 密码
     */
    @TableField(typeHandler = EncryptTypeHandler.class) // 添加 EncryptTypeHandler 处理器
    private String password;

}
```

另外，在 `application.yaml` 配置文件中，可使用 `mybatis-plus.encryptor.password` 设置加密密钥。

字段加密后，只允许使用**精准**匹配，无法使用模糊匹配。

```java
@Test // 测试使用 password 查询，可以查询到数据
public void testSelectPassword() {
    // mock 数据
    DataSourceConfigDO dbDataSourceConfig = randomPojo(DataSourceConfigDO.class);
    dataSourceConfigMapper.insert(dbDataSourceConfig);// @Sql: 先插入出一条存在的数据

    // 调用
    DataSourceConfigDO result = dataSourceConfigMapper.selectOne(DataSourceConfigDO::getPassword,
            EncryptTypeHandler.encrypt(dbDataSourceConfig.getPassword())); // 重点：需要使用 EncryptTypeHandler 去加密查询字段！！！
}
```

#### 逻辑删除

所有表通过 `deleted` 字段来实现逻辑删除，值为 0 表示未删除，值为 1 表示已删除。

- 可见 `application.yaml` 配置文件的 `logic-delete-value: 1` 和 `logic-not-delete-value: 0 ` 配置项。当然，也可以通过注解的 `value` 和 `delval` 来定义未删除和删除。

- 具体关于 MyBatis-Plus 的逻辑删除功能，看下 [逻辑删除](https://mybatis.plus/guide/logic-delete.html) 部分的文档。

![逻辑删除的配置](../assets/mybatis_03.png)

##### 自动拼接 `WHERE deleted = 0`

① 所有 SELECT 查询，都会自动拼接 `WHERE deleted = 0` 查询条件，过滤已经删除的记录。

- 如果要 SELECT 被删除的记录，只能通过在 XML 或者 `@SELECT` 来手写 SQL 语句。例如：

```java
@Mapper
public interface RoleMapper extends BaseMapperX<RoleDO> {
    @Select("SELECT id FROM system_role WHERE update_time > #{maxUpdateTime} LIMIT 1")
    RoleDO selectExistsByUpdateTimeAfter(Date maxUpdateTime);
}
```

##### 额外增加 **`delete_time` 字段**

② 建立**唯一索引**时，需要额外增加 **`delete_time` 字段**（初始值为0，表示未被逻辑删除），**添加到唯一索引**字段中，避免唯一索引冲突。例如说，`system_users` 使用 `username` 作为唯一索引：

- 未添加前：先**逻辑删除**了一条 `username = yudao` 的记录，然后又插入了一条 `username = yudao` 的记录时，会报**索引冲突**的异常。
- 已添加后：先逻辑删除了一条 `username = yudao` 的记录**并更新 `delete_time`** 为当前时间，然后又插入一条 `username = yudao` 并且 `delete_time` 为 0 的记录，不会导致唯一索引冲突。

```java
// 插入字典类型
DictTypeDO dictType = BeanUtils.toBean(createReqVO, DictTypeDO.class);
dictType.setDeletedTime(LocalDateTimeUtils.EMPTY); // 唯一索引，避免 null 值
dictTypeMapper.insert(dictType);


public class LocalDateTimeUtils {

    /**
     * 空的 LocalDateTime 对象，主要用于 DB 唯一索引的默认值
     */
    public static LocalDateTime EMPTY = buildTime(1970, 1, 1);
    
    // 创建指定时间
    public static LocalDateTime buildTime(int year, int month, int day) {
        return LocalDateTime.of(year, month, day, 0, 0, 0);
    }
}
```



#### Easy-Trans 数据翻译

easy-trans是一款用于做**数据翻译**的代码辅助插件，利用mybatis plus/jpa/等ORM框架的**能力自动查表**，让开发者可以快速的把**id/字典码** 翻译为前端需要展示的数据；

- 能减少sql的注入。

> 为什么实现 `{@link TransPojo}` 接口？
>
> * 因为使用 Easy-Trans `TransType.SIMPLE` 模式，集成 MyBatis Plus 查询。

##### 适用场景

1. 我有一个id，但是需要给客户展示他的 title/name 但是又不想自己手动做**表关联查询**

2. 我有一个**字典码 sex** 和 一个**字典值0** 希望能翻译成 **男** 给客户展示。

3. 我有一组 user id 比如 1，2,3 希望能展示成 张三,李四,王五 给客户

4. 我有一个**枚举**，枚举里有一个title字段，想给前端展示title的值 给客户

5. 我有一个**唯一键**(比如手机号，身份证号码，但是非其他表id字段)，但是需要给客户展示他的title/name 但是又不想自己手动做**表关联查询**

##### easy trans 支持的五种类型

1. 字典翻译(`TransType.DICTIONARY`)：需要使用者把字典信息刷新到DictionaryTransService 中进行缓存，使用字典翻译的时候取缓存数据源

2. **简单翻译**(`TransType.SIMPLE`)：比如有userId需要userName或者userPo给前端，原理是组件**使用MybatisPlus/JPA的API**自动进行查询，把结果放到**TransMap**中。
    - 无需自己实现数据源(推荐)，适用于根据id翻译name/title等 。此数据源需要配合`easy_trans_mybatis_plus_extend`或者`easy_trans_jpa_extend`一起使用。

3. 跨微服务翻译(`TransType.RPC`)：比如订单和用户是2个微服务，但是要在订单详情里展示订单的创建人的用户名，需要用到RPC翻译。

    1. 原理是订单微服务使用**restTemplate**调用用户服务的一个统一的接口，把需要翻译的id**传过去**，
    2. 然后用户微服务使用MybatisPlus/JPA的API自动进行查询把结果给订单微服务，然后订单微服务拿到数据后进行翻译。
    3. 当然使用者只是需要一个注解，这些事情都是由组件自动完成的。

4. 自动翻译(`TransType.AUTO`)：还是id翻译name场景，但是使用者如果想组件**调用自己写的方法**而不通过Mybatis Plus/JPA 的API进行数据查询，就可以使用AutoTrans。

5. 枚举翻译`(TransType.ENUM`)：比如要**把SEX.BOY 翻译为男**，可以用枚举翻译。

##### 依赖

```xml
<dependency>
    <groupId>com.fhs-opensource</groupId>
    <artifactId>easy-trans-spring-bootstarter</artifactId>
    <version>2.0.12</version>
</dependency>
```

##### 配置

```yaml
# VO 转换（数据翻译）相关
easy-trans:
  is-enable-global: false # 【默认禁用，对性能确认压力大】启用全局翻译（拦截所有 SpringMVC ResponseBody 进行自动翻译 )。如果对于性能要求很高可关闭此配置，或通过 @IgnoreTrans 忽略某个接口
```

##### 用法

1. 实现 AutoTransable 接口
2. 实现 TransPojo 接口：代表这个类需要**被翻译**或者被当作翻译的**数据源**。
3. 在需要翻译的字段上**添加 @Trans 注解**即可。

###### AutoTransable 接口

只有实现了这个接口的才能自动翻译。

为什么要赋值粘贴到 -common 包下？

 * 因为 AutoTransable 属于 easy-trans-service 下，无法方便的在 -module-xxx-api 模块下使用

```java

/**
 * @since  2020-05-19 10:26:15
 */
public interface AutoTransable<V extends VO> {

    /**
     * 根据 ids 查询数据列表
     * @param ids 编号数组
     * @return 数据列表
     */
    default List<V> selectByIds(List<? extends Object> ids){
        return new ArrayList<>();
    }
    ....
}
```



###### @Trans 注解

`@Trans(type = TransType.SIMPLE,target = Users.class,fields = "userName")`

1. type表示翻译类型 -- 简单翻译

2. target表示要翻译出来的结果字段**在哪个表中**（对应的实体类）

3. fields表示对应翻译的**是哪个字段**

4. ref----将翻译出来的数据映射到本实体类的**某个属性**上

    refs----将翻译出来的数据映射到本实体类的**多个**属性上

    alias----别名，解决翻译结果字段名**重名**问题

准备一张设备表device和一张用户表users，其中device表中的 **user_id 字段**关联了users表中的id字段。

- 这里的target表示将 `userId` 翻译为 `Users` 表的field字段`"userName", "phone"`。

```java
// Device.java

@Data
//实现TransPojo  接口，代表这个类需要被翻译或者被当作翻译的数据源
public class Device extends BaseEntity implements TransPojo {
    
    private Long id;
    private String deviceName;
    
	//SIMPLE 翻译，用于关联其他的表进行翻译，userName和phone 为 Users 的字段
    @Trans(type = TransType.SIMPLE, target = Users.class, fields = {"userName", "phone"})
    private Long userId;
 
}
```

User 表

```java
// User.java

@Data
//实现TransPojo  接口，代表这个类需要被翻译或者被当作翻译的数据源
public class Users extends BaseEntity implements TransPojo {
    
    private Long id;
 
    private String userName;
    private String phone;
}
```

###### @TransMethodResult

@TransMethodResult 注解：用于将翻译结果**映射**到结果集中。

- 一般情况下，由于easy-trans框架是将**结果集**映射到前端的，所以当**后端需要得到结果集**进行查询，导出等操作时值为null，所以需要在**调用方法时**就将结果映射，需要使用该接口。

```java
//DeviceController.java
    
	@Autowired
    private DeviceService deviceService;
    
    @GetMapping
    @TransMethodResult  //用于将翻译的结果映射到transMap中展示
    public R<List<Device>> getDeviceList() {
        //list()是Mybais plus提供的方法
        List<Device> deviceList = deviceService.list();
    }
```

###### 查询结果

如下：

- 发现userId已经成功地被翻译成了userName和phone，并将翻译的结果封装在了transMap中。

<img src="../assets/da017bcad2731458a0a8e9f070ec0ecc.png" alt="a401f84cf39d1d3b3b7e5db51ef8ebb5.png" style="zoom: 80%;" />

###### 平级模式

如果想让userName、phone在json中和userId同级展示，可以使用平铺模式：

在application.xml中开局平铺模式

```yaml
easy-trans:
  is-enable-tile: true #启用平铺模式
```

- 此时再看结果，发现 userName、phone和userId是同级

### Mapper 接口

- 默认配置下，MyBatis Mapper XML 需要写在各 `yudao-module-xxx-server` 模块的 `resources/mapper` 目录下。
- 简单的**单表查询**，优先在 Mapper 中通过 `default` 方法实现。
- 不要在 Controller、Service 中，**直接**进行 MyBatis Plus 操作。建议封装到对应的 Mapper 中，这样会更加简洁干净可管理。否则会导致：
    1. 会导致 Service 中的代码**越来越乱**，无法聚焦业务逻辑。逻辑里遍布了各种查询，无法**统一管理**实际有哪些查询条件。
    2. Service 会存在很多**相同且重复**的 SELECT 查询逻辑，无法更好的实现 SELECT 查询的**复用**。
- Mapper 的 SELECT 查询方法的命名，采用 Spring Data 的 ["Query methods" (opens new window)](https://docs.spring.io/spring-data/jpa/reference/jpa/query-methods.html)策略，方法名使用 `selectBy查询条件` 规则。

|      | 示例                             |
| ---- | -------------------------------- |
| 错误 | ![img](../assets/mybatis_05.png) |
| 正确 | ![img](../assets/mybatis_06.png) |

#### BaseMapper 接口

在 [`cn.iocoder.mybatis.mapper`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-12-mybatis/lab-12-mybatis-plus/src/main/java/cn/iocoder/springboot/lab12/mybatis/mapper) 包路径下，创建 [UserMapper](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-12-mybatis/lab-12-mybatis-plus/src/main/java/cn/iocoder/springboot/lab12/mybatis/mapper/UserMapper.java) 接口。

`UserMapper` （通过继承 `BaseMapper<User>`），可以**自动生成**常规的 CRUD 操作，立即拥有了所有的 **CRUD 操作方法**。可以在服务层中**直接使用**这些方法，无需再编写任何 SQL 语句。

- 另外，[BaseMapperX (opens new window)](https://github.com/YunaiV/yudao-cloud/blob/master/yudao-framework/yudao-spring-boot-starter-mybatis/src/main/java/cn/iocoder/yudao/framework/mybatis/core/mapper/BaseMapperX.java)接口，继承 MyBatis Plus 的 **BaseMapper 接口**，提供**更强的 CRUD 操作能力**。

- Mybatis Plus 通过 lambda 表达式获取数据库对应的列名。如，`UserDO::getName`。

> 更多 BaseMapper 提供的接口方法，可看看 [《MyBatis-Plus 文档 —— CRUD 接口》](https://mybatis.plus/guide/crud-interface.html#mapper-crud-接口) 。

BaseMapper 提供的常用方法有：

1. 四个 CRUD 方法：
    1. `#insert(UserDO user)`
    2. `#updateById(UserDO user)`
    3. `#deleteById(@Param("id") Integer id)`
    4. `#selectById(@Param("id") Integer id)`：
2. 条件查询：

`QueryWrapper` 是 MyBatis-Plus 提供的一个工具类，用于**构建查询条件**。

- `selectList` 方法根据条件查询所有符合条件的记录。

```java
// 构建查询条件
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.eq("username", "john_doe");  // 设置条件：用户名为 john_doe
 
// 执行查询
List<User> users = userMapper.selectList(queryWrapper);
 
// 输出查询结果
for (User u : users) {
    System.out.println(u.getUsername());
}
```

##### BaseMapperX 接口

> 在 MyBatis Plus 的 BaseMapper 的基础上拓展，提供更多的能力。

封装的方法有：

- selectPage()
    - selectJoinPage()
- selecctOne()
    - selectFirstOne()
- selectCount()
- selectList()
- insertBatch()：批量插入，适合大量数据插入
- updateBatch()
- delete()
    - deleteBatch()

如，selectOne 方法，使用指定条件，查询单条记录。

1. 在 `BaseMapperX `中封装 `QueryWrapper`；
2. `MemberUserMapper` 继承 `BaseMapperX`；

```java
public interface BaseMapperX<T> extends MPJBaseMapper<T> {
    
	default T selectOne(String field, Object value) {
        return selectOne(new QueryWrapper<T>().eq(field, value));
    }
    
    // 使用 LambdaQueryWrapper 调用，如 selectOne(MemberUserDO::getMobile, mobile)
    default T selectOne(SFunction<T, ?> field, Object value) {
        return selectOne(new LambdaQueryWrapper<T>().eq(field, value));
    }
}

@Mapper
public interface MemberUserMapper extends BaseMapperX<MemberUserDO> {

    default MemberUserDO selectByMobile(String mobile) {
        return selectOne(MemberUserDO::getMobile, mobile);
    }

    default List<MemberUserDO> selectListByNicknameLike(String nickname) {
        return selectList(new LambdaQueryWrapperX<MemberUserDO>()
                .likeIfPresent(MemberUserDO::getNickname, nickname));
    }
}
```



常见条件查询有：

#### SelectOne

1. 对于`#selectByUsername(@Param("username") String username)`方法，使用了`QueryWrapper<T>`构造相对灵活的条件，这样一些**动态 SQL** 就无需在 XML 中编写。
    - 建议 1 ：使用 QueryWrapper 拼接动态条件（如用`#selectList(Wrapper<T> queryWrapper)` 等方法）。
    - 建议 2 ：因为 QueryWrapper 暂时不支持一些类似 `<if />` 等 MyBatis 的 OGNL 表达式，可以通过继承 QueryWrapper 类，封装 [QueryWrapperX](https://github.com/YunaiV/onemall/blob/master/common/common-framework/src/main/java/cn/iocoder/common/framework/mybatis/QueryWrapperX.java) 类。
    - 更多 `QueryWrapper `提供的拼接方法，可以看 [《MyBatis-Plus 文档 —— 条件构造器》](https://mybatis.plus/guide/wrapper.html#abstractwrapper) 。
2. 对于`#selectPageByCreateTime(IPage<UserDO> page, @Param("createTime") Date createTime)`方法，是额外添加的，用于演示 MyBatis-Plus 提供的分页插件。
    - 更多 IPage 的内容，可以看 [《MyBatis-Plus 文档 —— 分页插件》](https://mybatis.plus/guide/page.html) 。

```java
// UserMapper.java

@Repository
public interface UserMapper extends BaseMapper<UserDO> {

    default UserDO selectByUsername(@Param("username") String username) {
        return selectOne(new QueryWrapper<UserDO>().eq("username", username));
    }

    // 实际也可以使用 MyBatis-Plus 的 QueryWrapper 很方便的实现，
    // 这里仅仅是为了演示在 MyBatis-Plus 混合使用 XML 。
    List<UserDO> selectByIds(@Param("ids") Collection<Integer> ids);

    default IPage<UserDO> selectPageByCreateTime(IPage<UserDO> page, @Param("createTime") Date createTime) {
        return selectPage(page,
                new QueryWrapper<UserDO>().gt("create_time", createTime));
    }
}

@Mapper
public interface ProductSkuMapper extends BaseMapperX<ProductSkuDO> {

    @Select("SELECT * FROM product_sku WHERE id = #{id}")
    ProductSkuDO selectByIdIncludeDeleted(@Param("id") Long id);
    
    ...
```

##### UserMapper.xml

在 [`resources/mapper`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-12-mybatis/lab-12-mybatis-plus/src/main/resources/mapper) 路径下，创建 [`UserMapper.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-12-mybatis/lab-12-mybatis-plus/src/main/resources/mapper/UserMapper.xml) 配置文件。

- 是不是一下子，瘦了！

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.iocoder.springboot.lab12.mybatis.mapper.UserMapper">

    <sql id="FIELDS">
        id, username, password, create_time
    </sql>

    <select id="selectByIds" resultType="UserDO">
        SELECT
            <include refid="FIELDS" />
        FROM users
        WHERE id IN
            <foreach item="id" collection="ids" separator="," open="(" close=")" index="">
                #{id}
            </foreach>
    </select>

</mapper>
```

##### 简单测试

创建 [UserMapperTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-12-mybatis/lab-12-mybatis-plus/src/test/java/cn/iocoder/springboot/lab12/mybatis/mapper/UserMapperTest.java) 测试类，来测试一下简单的 UserMapper 的每个操作。

- 多了一个分页的单元测试方法。

```java
// UserMapperTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class UserMapperTest {

    @Autowired
    private UserMapper userMapper;

    @Test
    public void testInsert() {
        // UUID.randomUUID()
        UserDO user = new UserDO().setUsername(UUID.randomUUID().toString())
                .setPassword("nicai").setCreateTime(new Date())
                .setDeleted(0); // 一般情况下，是否删除，可以全局枚举下。
        userMapper.insert(user);
    }

    @Test
    public void testUpdateById() {
        UserDO updateUser = new UserDO().setId(1)
                .setPassword("wobucai");
        userMapper.updateById(updateUser);
    }

    @Test
    public void testDeleteById() {
        userMapper.deleteById(2);
    }


    @Test
    public void testSelectByUsername() {
        userMapper.selectByUsername("yunai");
    }

    @Test
    public void testSelectByIds() {
        List<UserDO> users = userMapper.selectByIds(Arrays.asList(1, 3));
        System.out.println("users：" + users.size());
    }

    @Test
    public void testSelectPageByCreateTime() {
        IPage<UserDO> page = new Page<>(1, 10);
        Date createTime = new Date(2018 - 1990, Calendar.FEBRUARY, 24); // 临时 Demo ，实际不建议这么写
        page = userMapper.selectPageByCreateTime(page, createTime);
        System.out.println("users：" + page.getRecords().size());
    }

}
```

#### selectCount

[`#selectCount(...)` (opens new window)](https://github.com/YunaiV/yudao-cloud/blob/master/yudao-framework/yudao-spring-boot-starter-mybatis/src/main/java/cn/iocoder/yudao/framework/mybatis/core/mapper/BaseMapperX.java#L46-L56)方法，使用指定条件，查询记录的数量。

![selectCount 示例](../assets/mybatis_10.png)

#### selectList

[`#selectList(...)` (opens new window)](https://github.com/YunaiV/yudao-cloud/blob/master/yudao-framework/yudao-spring-boot-starter-mybatis/src/main/java/cn/iocoder/yudao/framework/mybatis/core/mapper/BaseMapperX.java#L58-L76)方法，使用指定条件，查询多条记录。

<img src="../assets/mybatis_011.png" alt="mybatis_011" style="zoom:90%;" />



#### selectByIds

> select(List)ByIds

- 通过 Collection ids 获取 List。

```java
//ProductSkuServiceImpl.java

// 更新 SPU 库存
List<ProductSkuDO> skus = productSkuMapper.selectByIds(
    convertSet(updateStockReqDTO.getItems(), ProductSkuUpdateStockReqDTO.Item::getId));
Map<Long, Integer> spuStockIncrCounts = ProductSkuConvert.INSTANCE.convertSpuStockMap(
    updateStockReqDTO.getItems(), skus);
productSpuService.updateSpuStock(spuStockIncrCounts);
```

#### selectPage 分页

> 见下

#### insertBatch

[`#insertBatch(...)`](https://github.com/YunaiV/yudao-cloud/blob/master/yudao-framework/yudao-spring-boot-starter-mybatis/src/main/java/cn/iocoder/yudao/framework/mybatis/core/mapper/BaseMapperX.java#L78-L88)方法，遍历数组，逐条插入数据库中，适合**少量**数据插入，或者对**性能要求不高**的场景。 

为什么不使用 insertBatchSomeColumn 批量插入？

- 只支持 MySQL 数据库。其它 Oracle 等数据库使用会报错，可见 [InsertBatchSomeColumn (opens new window)](https://github.com/baomidou/mybatis-plus/blob/a3e121c27cd26cb7c546dfb88190f3b1f574dc38/mybatis-plus-extension/src/main/java/com/baomidou/mybatisplus/extension/injector/methods/InsertBatchSomeColumn.java)说明。
- 未支持多租户。插入数据库时，多租户字段不会进行自动赋值。

![insertBatch 示例](../assets/mybatis_12.png)

##### 批量插入

绝大多数场景下，推荐使用 MyBatis Plus 提供的 IService 的 [`#saveBatch()` (opens new window)](https://github.com/baomidou/mybatis-plus/blob/34ebdf6ee6/mybatis-plus-extension/src/main/java/com/baomidou/mybatisplus/extension/service/IService.java#L66-L74)方法。示例 [PermissionServiceImpl](https://github.com/YunaiV/yudao-cloud/blob/master/yudao-module-system/yudao-module-system-server/src/main/java/cn/iocoder/yudao/module/system/service/permission/PermissionServiceImpl.java#L200-L230)如下：

- XxxBatchInsertMapper

![saveBatch 示例](../assets/mybatis_14.png)



#### MPJBaseMapper 连表 JOIN

MyBatis Plus Join 的基础接口，提供连表 Join 能力。

```java
public interface MPJBaseMapper<T> extends BaseMapper<T>, JoinMapper<T> {
}
```

##### 多表查询

尽量避免数据库的**连表（多表）查询**，而是采用**多次查询 + Java 内存拼接**的方式替代。

```java
@Tag(name = "管理后台 - 用户")
@RestController
@RequestMapping("/system/user")
@Validated
public class UserController {

    @Resource
    private AdminUserService userService;
    @Resource
    private DeptService deptService;
    
	@GetMapping("/page")
    @Operation(summary = "获得用户分页列表")
    @PreAuthorize("@ss.hasPermission('system:user:query')")
    public CommonResult<PageResult<UserRespVO>> getUserPage(@Valid UserPageReqVO pageReqVO) {
        // 获得拼接需要的数据，获得用户分页列表
        PageResult<AdminUserDO> pageResult = userService.getUserPage(pageReqVO);
        if (CollUtil.isEmpty(pageResult.getList())) {
            return success(new PageResult<>(pageResult.getTotal()));
        }
        // 拼接结果返回，拼接数据
        Map<Long, DeptDO> deptMap = deptService.getDeptMap(
                convertList(pageResult.getList(), AdminUserDO::getDeptId));
        return success(new PageResult<>(UserConvert.INSTANCE.convertList(pageResult.getList(), deptMap),
                pageResult.getTotal()));
    }
}
```

![UserController 示例](../assets/mybatis_20.png)

### 条件构造器

> Mybatis Plus 通过 lambda 表达式获取数据库对应的列名。如，`UserDO::getName`。

MyBatis-Plus 提供了强大的条件构造器，使得在查询数据库时可以灵活地构建条件，而无需手动编写复杂的 SQL 语句。

- 主要通过 `Wrapper` 接口、及其常用实现类 `QueryWrapper` 和 `LambdaQueryWrapper` 来**实现条件查询**。

#### **`Wrapper` 接口**

`Wrapper` 是 MyBatis-Plus 提供的**条件构造器接口**，用于构建动态 SQL。

- 有多个实现类，其中最常用的是 `QueryWrapper` 和 `LambdaQueryWrapper`。
- 常用于复杂查询，比如 **selectPage 查询**方法。

#### `QueryWrapper`

`QueryWrapper` 是 MyBatis-Plus 提供的一个**通用**条件构造器，用于以**非 Lambda 表达式**的方式构建查询条件。

常用方法：

- **eq**: 等于
- **ne**: 不等于
- **gt**、ge: 大于、大于等于
- **lt**、le: 小于、 小于等于
- **in**: 在指定范围内
- **inSql**: 允许使用**子查询的结果集**作为 IN 条件的范围
- **between**: 在两者之间
- **like**: 模糊查询
- **or**、and: 或、并且条件
- **isNull**、isNotNull: 判断字段是否为 NULL
- **orderByAsc**、orderByDesc: 升序排序、 降序排序

示例：

```java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper
    .eq("name", "张三")  // name 等于 张三。 eq也可以写三个参数, 第一个参数是boolean, false 表示这个条件不起作用, true 表示起作用
    .ge("age", 18)       // age 大于等于 18
    .like("email", "gmail.com")  // email 包含 gmail.com
    .orderByDesc("create_time"); // 按 create_time 降序排列
 
List<User> users = userMapper.selectList(queryWrapper);

//查询学生
QueryWrapper<Student> queryWrapper = new QueryWrapper();
queryWrapper.lambda().eq(Student::getName, “老王”);
```

#### `LambdaQueryWrapper`

`LambdaQueryWrapper` 是 `QueryWrapper` 的 **Lambda 版本**，用于在构建条件时**避免使用字符串**来指定字段，增加了类型安全性。

- 使用字段的 **Lambda 表达式**来构建条件。
- 通过**方法引用**的方式来使用实体**字段名**，避免直接写数据库表字段名时的错写名字。

三种方式：

1. `LambdaQueryWrapper<T>`方式
2. `QueryWrapper<实体>().lambda()`方式
3. `Wrappers.<实体>lambdaQuery()`方式

示例：

```java
LambdaQueryWrapper<User> lambdaQuery = new LambdaQueryWrapper<>();
lambdaQuery
    .eq(User::getName, "张三")  // name 等于 张三
    .ge(User::getAge, 18)       // age 大于等于 18
    .like(User::getEmail, "gmail.com")  // email 包含 gmail.com
    .orderByDesc(User::getCreateTime); // 按 create_time 降序排列
 
List<User> users = userMapper.selectList(lambdaQuery);
```

![LambdaQueryWrapper 条件构造器](../assets/mybatis_08.png)

##### LambdaQueryWrapperX

继承 MyBatis Plus 的条件构造器，拓展了 [LambdaQueryWrapperX (opens new window)](https://github.com/YunaiV/yudao-cloud/blob/master/yudao-framework/yudao-spring-boot-starter-mybatis/src/main/java/cn/iocoder/yudao/framework/mybatis/core/query/LambdaQueryWrapperX.java)和 [QueryWrapperX (opens new window)](https://github.com/YunaiV/yudao-cloud/blob/master/yudao-framework/yudao-spring-boot-starter-mybatis/src/main/java/cn/iocoder/yudao/framework/mybatis/core/query/QueryWrapperX.java)类（重写父类方法，方便链式调用）。

- 主要是增加 **xxxIfPresent 方法**，用于判断值不存在的时候，不要拼接到条件中。
- `apply()`：
- setSql()：

```java
default Long selectCountByTagId(Long tagId) {
    return selectCount(new LambdaQueryWrapperX<MemberUserDO>()
            .apply("FIND_IN_SET({0}, tag_ids)", tagId));
}

/**
     * 更新用户积分（增加）
     *
     * @param id        用户编号
     * @param incrCount 增加积分（正数）
     */
    default void updatePointIncr(Long id, Integer incrCount) {
        Assert.isTrue(incrCount > 0);
        LambdaUpdateWrapper<MemberUserDO> lambdaUpdateWrapper = new LambdaUpdateWrapper<MemberUserDO>()
                .setSql(" point = point + " + incrCount)
                .eq(MemberUserDO::getId, id);
        update(null, lambdaUpdateWrapper);
    }
```

![xxxIfPresent 方法](../assets/mybatis_15.png)

具体的使用示例如下：

![LambdaQueryWrapperX 使用示例](../assets/mybatis_16.png)

#### MPJLambdaWrapper

> Join QueryWrapper

#### `UpdateWrapper` 和 `LambdaUpdateWrapper`

这两个类分别是用于构建**更新条件**的构造器，功能与 `QueryWrapper` 和 `LambdaQueryWrapper` 类似，但用于 `UPDATE` 操作。

示例：

```java
UpdateWrapper<User> updateWrapper = new UpdateWrapper<>();
updateWrapper
    .eq("name", "张三")
    .set("age", 30);  // 将年龄更新为 30
 
userMapper.update(null, updateWrapper);
```

具体用法可参考 [Mybatis plus 官网](https://baomidou.com/guides/wrapper/)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/e0823466837e4cd2b861882830c2fb9d.png)

### 分页实现

- 前端：基于 Element UI 分页组件 [Pagination(opens new window)](https://element.eleme.io/#/zh-CN/component/pagination)
- 后端：基于 MyBatis Plus 分页功能，二次封装。

以 [系统管理 -> 租户管理 -> 租户列表] 菜单为例子，讲解它的**分页 + 搜索**的实现。

#### 前端分页实现

##### Vue 界面

界面 [`tenant/index.vue` (opens new window)](https://github.com/yudaocode/yudao-ui-admin-vue2/blob/master/src/views/system/tenant/index.vue)相关的代码如下：

```html
<template>
    <!-- 搜索工作栏 -->
    <el-form :model="queryParams" ref="queryForm" size="small" :inline="true" v-show="showSearch" label-width="68px">
      <el-form-item label="租户名" prop="name">
        <el-input v-model="queryParams.name" placeholder="请输入租户名" clearable @keyup.enter.native="handleQuery"/>
      </el-form-item>
      <el-form-item label="联系人" prop="contactName">
        <el-input v-model="queryParams.contactName" placeholder="请输入联系人" clearable @keyup.enter.native="handleQuery"/>
      </el-form-item>
      <el-form-item label="联系手机" prop="contactMobile">
        <el-input v-model="queryParams.contactMobile" placeholder="请输入联系手机" clearable @keyup.enter.native="handleQuery"/>
      </el-form-item>
      <el-form-item label="租户状态" prop="status">
        <el-select v-model="queryParams.status" placeholder="请选择租户状态" clearable>
          <el-option v-for="dict in this.getDictDatas(DICT_TYPE.COMMON_STATUS)"
                       :key="dict.value" :label="dict.label" :value="dict.value"/>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="el-icon-search" @click="handleQuery">搜索</el-button>
        <el-button icon="el-icon-refresh" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>
    
    <!-- 列表 -->
    <el-table v-loading="loading" :data="list">
        <!-- 省略每一列... -->
    </el-table>
    
    <!-- 分页组件 -->
    <pagination v-show="total > 0" :total="total" :page.sync="queryParams.pageNo" :limit.sync="queryParams.pageSize" 
                @pagination="getList"/>

</template>

<script>
import { getTenantPage } from "@/api/system/tenant";

export default {
	name: "Tenant",
	components: {},
	data() {
      // 遮罩层
      return {
        // 遮罩层
        loading: true,
        // 显示搜索条件
        showSearch: true,
        // 总条数
        total: 0,
        // 租户列表
        list: [],
        // 查询参数
        queryParams: {
          pageNo: 1,
          pageSize: 10,
          // 搜索条件
          name: null,
          contactName: null,
          contactMobile: null,
          status: undefined,
        },
      }
	},
	created() {
	  this.getList();
	},
	methods: {
	  /** 查询列表 */
	  getList() {
	    this.loading = true;
	    // 处理查询参数
	    let params = {...this.queryParams};
		// 执行查询
	    getTenantPage(params).then(response => {
		  this.list = response.data.list;
		  this.total = response.data.total;
		  this.loading = false;
		});
      },
      /** 搜索按钮操作 */
      handleQuery() {
        this.queryParams.pageNo = 1;
        this.getList();
      },
      /** 重置按钮操作 */
      resetQuery() {
        this.resetForm("queryForm");
        this.handleQuery();
      }
    }
}
</script>
```

##### API 请求

请求 [`system/tenant.js` (opens new window)](https://github.com/yudaocode/yudao-ui-admin-vue2/blob/master/src/api/system/tenant.js)相关的代码如下：

```javascript
import request from '@/utils/request'

// 获得租户分页
export function getTenantPage(query) {
  return request({
    url: '/system/tenant/page',
    method: 'get',
    params: query
  })
}
```

#### 后端分页实现

> 后端：基于 MyBatis Plus 分页功能，二次封装。

##### Controller 接口

在 [TenantController (opens new window)](https://github.com/YunaiV/yudao-cloud/blob/master/yudao-module-system/yudao-module-system-server/src/main/java/cn/iocoder/yudao/module/system/controller/admin/tenant/TenantController.java#L75-L81)类中，定义 `/admin-api/system/tenant/page` 接口。

- Request 分页请求，使用 [TenantPageReqVO (opens new window)](https://github.com/YunaiV/yudao-cloud/blob/master/yudao-module-system/yudao-module-system-server/src/main/java/cn/iocoder/yudao/module/system/controller/admin/tenant/vo/tenant/TenantPageReqVO.java)类，它继承 PageParam 类
- Response 分页结果，使用 PageResult 类，每一项是 [TenantRespVO (opens new window)](https://github.com/YunaiV/yudao-cloud/blob/master/yudao-module-system/yudao-module-system-server/src/main/java/cn/iocoder/yudao/module/system/controller/admin/tenant/vo/tenant/TenantRespVO.java)类

```java
@Tag(name = "管理后台 - 租户")
@RestController
@RequestMapping("/system/tenant")
public class TenantController {

    @Resource
    private TenantService tenantService;

    @GetMapping("/page")
    @Operation(summary = "获得租户分页")
    @PreAuthorize("@ss.hasPermission('system:tenant:query')")
    public CommonResult<PageResult<TenantRespVO>> getTenantPage(@Valid TenantPageReqVO pageVO) {
        PageResult<TenantDO> pageResult = tenantService.getTenantPage(pageVO);
        return success(TenantConvert.INSTANCE.convertPage(pageResult));
    }
}


@GetMapping("/page")
@Operation(summary = "获得交易订单分页")
@PreAuthorize("@ss.hasPermission('trade:order:query')")
public CommonResult<PageResult<TradeOrderPageItemRespVO>> getOrderPage(TradeOrderPageReqVO reqVO) {
    // 查询订单
    PageResult<TradeOrderDO> pageResult = tradeOrderQueryService.getOrderPage(reqVO);
    if (CollUtil.isEmpty(pageResult.getList())) {
        return success(PageResult.empty());
    }

    // 查询用户信息
    Set<Long> userIds = CollUtil.unionDistinct(convertList(pageResult.getList(), TradeOrderDO::getUserId),
                                               convertList(pageResult.getList(), TradeOrderDO::getBrokerageUserId, Objects::nonNull));
    Map<Long, MemberUserRespDTO> userMap = memberUserApi.getUserMap(userIds);
    // 查询订单项
    List<TradeOrderItemDO> orderItems = tradeOrderQueryService.getOrderItemListByOrderId(
        convertSet(pageResult.getList(), TradeOrderDO::getId));
    // 最终组合
    return success(TradeOrderConvert.INSTANCE.convertPage(pageResult, orderItems, userMap));
}

// TradeOrderQueryServiceImpl.java
@Override
public List<TradeOrderItemDO> getOrderItemListByOrderId(Collection<Long> orderIds) {
    if (CollUtil.isEmpty(orderIds)) {
        return Collections.emptyList();
    }
    return tradeOrderItemMapper.selectListByOrderId(orderIds);
}

// TradeOrderItemMapper.java
default List<TradeOrderItemDO> selectListByOrderId(Collection<Long> orderIds) {
    return selectList(TradeOrderItemDO::getOrderId, orderIds);
}

// BaseMapperX.java
default List<T> selectList(SFunction<T, ?> field, Collection<?> values) {
    if (CollUtil.isEmpty(values)) {
        return CollUtil.newArrayList();
    }
    return selectList(new LambdaQueryWrapper<T>().in(field, values));
}
```

###### 分页参数 PageParam

分页请求，需要继承 [PageParam (opens new window)](https://github.com/YunaiV/yudao-cloud/blob/master/yudao-framework/yudao-common/src/main/java/cn/iocoder/yudao/framework/common/pojo/PageParam.java)类。

```java
@Schema(description="分页参数")
@Data
public class PageParam implements Serializable {

    private static final Integer PAGE_NO = 1;
    private static final Integer PAGE_SIZE = 10;

    @Schema(description = "页码，从 1 开始", required = true, example = "1")
    @NotNull(message = "页码不能为空")
    @Min(value = 1, message = "页码最小值为 1")
    private Integer pageNo = PAGE_NO;

    @Schema(description = "每页条数，最大值为 100", required = true, example = "10")
    @NotNull(message = "每页条数不能为空")
    @Min(value = 1, message = "每页条数最小值为 1")
    @Max(value = 100, message = "每页条数最大值为 100")
    private Integer pageSize = PAGE_SIZE;

}
```

###### 分页请求 VO

分页请求VO，~~分页条件~~，在子类中进行定义。以 TenantPageReqVO 举例：

```java
@Schema(description = "管理后台 - 租户分页 Request VO")
@Data
@EqualsAndHashCode(callSuper = true)
@ToString(callSuper = true)
public class TenantPageReqVO extends PageParam {

    @Schema(description = "租户名", example = "芋道")
    private String name;

    @Schema(description = "联系人", example = "芋艿")
    private String contactName;

    @Schema(description = "联系手机", example = "15601691300")
    private String contactMobile;

    @Schema(description = "租户状态（0正常 1停用）", example = "1")
    private Integer status;

    @DateTimeFormat(pattern = FORMAT_YEAR_MONTH_DAY_HOUR_MINUTE_SECOND)
    @Schema(description = "创建时间")
    private LocalDateTime[] createTime;
}
```

###### 分页结果 PageResult

分页结果 [PageResult (opens new window)](https://github.com/YunaiV/yudao-cloud/blob/master/yudao-framework/yudao-common/src/main/java/cn/iocoder/yudao/framework/common/pojo/PageResult.java)类，代码如下：

- 分页结果的数据 `list` 的每一项，通过自定义 **VO 类**，例如说 [TenantRespVO (opens new window)](https://github.com/YunaiV/yudao-cloud/blob/master/yudao-module-system/yudao-module-system-server/src/main/java/cn/iocoder/yudao/module/system/controller/admin/tenant/vo/tenant/TenantRespVO.java)类。

```java
@Schema(description = "分页结果")
@Data
public final class PageResult<T> implements Serializable {

    @Schema(description = "数据", required = true)
    private List<T> list;

    @Schema(description = "总量", required = true)
    private Long total;
}
```

##### Mapper

针对 MyBatis Plus 分页查询的二次分装，在 [BaseMapperX](https://github.com/YunaiV/yudao-cloud/blob/master/yudao-framework/yudao-spring-boot-starter-mybatis/src/main/java/cn/iocoder/yudao/framework/mybatis/core/mapper/BaseMapperX.java) 中实现，目的是使用**项目自己的分页封装**：

- 【入参】查询前，将项目的**分页参数 [PageParam](https://github.com/YunaiV/yudao-cloud/blob/master/yudao-framework/yudao-common/src/main/java/cn/iocoder/yudao/framework/common/pojo/PageParam.java)**，转换成 MyBatis Plus 的 **IPage 对象**。
- 【出参】查询后，将 MyBatis Plus 的分页结果 IPage，转换成项目的分页结果 [PageResult](https://github.com/YunaiV/yudao-cloud/blob/master/yudao-framework/yudao-common/src/main/java/cn/iocoder/yudao/framework/common/pojo/PageResult.java)。

在 BaseMapperX 中：

![BaseMapperX 实现](../assets/mybatis_01.png)

具体的使用示例，可见 [TenantMapper](https://github.com/YunaiV/yudao-cloud/blob/master/yudao-module-system/yudao-module-system-server/src/main/java/cn/iocoder/yudao/module/system/dal/mysql/tenant/TenantMapper.java)类中，定义 **selectPage 查询方法**。

- 完整实战，可见 [《开发指南 —— 分页实现》](https://cloud.iocoder.cn/page-feature) 文档。
- 常用 LambdaQueryWrapperX 实现。

```java
@Mapper
public interface TenantMapper extends BaseMapperX<TenantDO> {

    default PageResult<TenantDO> selectPage(TenantPageReqVO reqVO) {
        return selectPage(reqVO, new LambdaQueryWrapperX<TenantDO>()
        	.likeIfPresent(TenantDO::getName, reqVO.getName()) // 如果 name 不为空，则进行 like 查询
            .likeIfPresent(TenantDO::getContactName, reqVO.getContactName())
            .likeIfPresent(TenantDO::getContactMobile, reqVO.getContactMobile())
            .eqIfPresent(TenantDO::getStatus, reqVO.getStatus()) // 如果 status 不为空，则进行 = 查询
            .betweenIfPresent(TenantDO::getCreateTime, reqVO.getBeginCreateTime(), reqVO.getEndCreateTime()) // 如果 create 不为空，则进行 between 查询
            .orderByDesc(TenantDO::getId)); // 按照 id 倒序
    }
}
```

###### IPage

IPage＜实体＞转 IPage＜Vo＞

```java
/**
 * 根据用户姓名分页查询用户
 *
 * @param userQo
 * @return
 */
@Override
public IPage<UserVo> selectPageAll(UserQo userQo) {
    int page = userQo.getPage();
    int limit = userQo.getLimit();

    IPage<User> userIPage = cpWalletLogMapper.selectPage(new Page<>(page, limit),
            new LambdaQueryWrapper<User>()
                    .like(User::getName, userQo.getUserName())
                    .orderByAsc(User::getId)
    );

    return  userIPage .convert(User -> ConvertUtils.beanCopy(User, UserVo.class));
}
```

### 高级查询

基于 `userMapper.selectMaps(queryWrapper)`。

#### 分组查询

示例 ：按 `age` 分组并统计人数

假设想统计各个年龄段的人数，可以使用如下代码：

```java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.select("age", "COUNT(*) as count");
queryWrapper.groupBy("age");
 
List<Map<String, Object>> result = userMapper.selectMaps(queryWrapper);
```

生成的 SQL：

```sql
SELECT age, COUNT(*) as count FROM user GROUP BY age;
```

#### 聚合查询

示例 ：按 `age` 分组并过滤统计结果（`HAVING`）

如果只想统计人数大于 1 的年龄段，可以添加 `HAVING` 条件：

```java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.select("age", "COUNT(*) as count");
queryWrapper.groupBy("age");
queryWrapper.having("COUNT(*) > 1");
 
List<Map<String, Object>> result = userMapper.selectMaps(queryWrapper);
```

生成的 SQL：

```sql
SELECT age, COUNT(*) as count FROM user GROUP BY age HAVING COUNT(*) > 1;
```

#### 排序查询

##### 示例 1：按多字段排序，并指定是否为空

假设想按 `age` 升序排序，并希望将 `name` 为空的记录排在前面，可以使用如下代码：

```java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.orderByAsc("age").orderByAsc("name", true);
 
List<User> result = userMapper.selectList(queryWrapper);
```

生成的 SQL：

```sql
SELECT * FROM user ORDER BY age ASC, name ASC;
```

##### 示例 2：按 `age` 升序和 `name` 降序组合排序

假设想先按**年龄升序**排序，再按**姓名降序**排序，可以使用如下代码：

```java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.orderByAsc("age").orderByDesc("name");
 
List<User> result = userMapper.selectList(queryWrapper);
```

生成的 SQL：

```sql
SELECT * FROM user ORDER BY age ASC, name DESC;
```

#### 逻辑查询

`func` 方法是 MyBatis-Plus 提供的一个非常灵活的功能，它允许将一段**自定义的逻辑**包装到查询条件中。

- 这对于需要根据不同的条件来**动态构建**查询的场景特别有用。

##### 1. `func` 方法的基本用法

`func` 方法接收一个 `Consumer`，参数是 `QueryWrapper`（或 `LambdaQueryWrapper`）的一个实例。

- 可以在这个 `Consumer` 中编写自定义的逻辑，并根据不同的条件来**动态地**添加或修改查询条件。

语法结构：

```java
queryWrapper.func(wrapper -> {
    // 在这里编写自定义逻辑
    if (condition) {
        wrapper.eq("column", value);
    } else {
        wrapper.ne("column", value);
    }
});
```

主要参数：

- **`Consumer<QueryWrapper>` 或 `Consumer<LambdaQueryWrapper>`**：这是一个函数式接口，允许传入一个 Lambda 表达式或方法引用。可以在这个接口的 `accept` 方法中实现自己的逻辑。

实际应用场景：

- 假设有一个用户查询接口，允许用户根据不同的条件来过滤结果，例如按 `id` 或按 `name`。

- 可以使用 `func` 来根据用户输入动态地构建查询条件。

```java
LambdaQueryWrapper<User> lambdaQueryWrapper = new LambdaQueryWrapper<>();
lambdaQueryWrapper.func(wrapper -> {
    if (userInput != null && userInput.isValid()) {
        wrapper.eq(User::getName, userInput.getName());
    } else {
        wrapper.ne(User::getId, 1);
    }
});
List<User> users = userMapper.selectList(lambdaQueryWrapper);
```

- 示例解释：
    - 如果 `userInput` 非空且有效，则查询条件为 `name = userInput.getName()`。
    - 否则，查询条件为 `id != 1`。

##### 2. `and` 和 `or` 的使用

在 MyBatis-Plus 中，`and` 和 `or` 用于在构建查询条件时处理多条件的逻辑运算。它们允许在查询中组合多个条件，以实现复杂的查询逻辑。

###### `and` 方法

`and` 方法：用于将多个查询条件通过逻辑“与” (`AND`) 连接在一起。将多个条件组合成一个大的 `AND` 条件，从而要求所有这些条件都必须满足。

示例

假设有一个 `User` 表，想查询年龄大于 20 且名字为 "Jack" 的用户。可以使用以下代码：

```java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.gt("age", 20).and(wrapper -> wrapper.eq("name", "Jack"));
 
List<User> users = userMapper.selectList(queryWrapper);
```

生成的 SQL：

```sql
SELECT * FROM user WHERE age > 20 AND name = 'Jack';
```

在这个例子中，`and` 方法的作用是将 `.gt("age", 20)` 和 `.eq("name", "Jack")` 这两个条件通过 `AND` 组合在一起。

###### `or` 方法

`or` 方法：用于将多个查询条件通过逻辑“或” (`OR`) 连接在一起。它将多个条件组合成一个大的 `OR` 条件，只要其中一个条件满足，就会返回符合的结果。

示例

假设有一个 `User` 表，想查询年龄大于 20 或名字为 "Jack" 的用户。可以使用以下代码：

```java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.gt("age", 20).or(wrapper -> wrapper.eq("name", "Jack"));
 
List<User> users = userMapper.selectList(queryWrapper);
```

生成的 SQL：

```sql
SELECT * FROM user WHERE age > 20 OR name = 'Jack';
```

在这个例子中，`or` 方法的作用是将 `.gt("age", 20)` 和 `.eq("name", "Jack")` 这两个条件通过 `OR` 组合在一起。

###### 组合使用 `and` 和 `or`

还可以结合使用 `and` 和 `or` 方法，以构建更复杂的查询。

- 例如，如果你想查询年龄大于 20 且（名字为 "Jack" 或邮箱为 "test@example.com"）的用户，可以使用以下代码：

```java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.gt("age", 20)
            .and(wrapper -> wrapper.eq("name", "Jack")
                                    .or().eq("email", "test@example.com"));
 
List<User> users = userMapper.selectList(queryWrapper);
```

生成的 SQL：

```sql
SELECT * FROM user WHERE age > 20 AND (name = 'Jack' OR email = 'test@example.com');
```

在这个例子中，`and` 和 `or` 方法的结合使用允许在 `age > 20` 的基础上，增加一个组合条件 `(name = 'Jack' OR email = 'test@example.com')`。

#### 其他查询

###### `apply` 方法

`apply` 方法：允许直接在 `QueryWrapper` 中**插入自定义的 SQL 片段**。会被添加到 **WHERE 子句的末尾**。这允许在现有的查询条件基础上，添加更复杂的条件或函数。

基本用法：

```java
QueryWrapper<User> wrapper = new QueryWrapper<>();
wrapper.apply("DATE_FORMAT(create_time, '%Y-%m-%d') = {0}", "2024-09-01");
```

在这个示例中：

- `apply` 方法接受一个 SQL 片段作为第一个参数，并可以通过 `{}` 占位符来插入参数。
- 这里使用了 `DATE_FORMAT` 函数来**格式化 `create_time` 字段**，并将其与特定的日期进行比较。

###### `last` 方法

`last` 方法：用于在生成的 SQL 查询的**末尾**添加额外的 SQL 片段。

- 通常用于添加额外的 SQL 语句，如 `ORDER BY`, `LIMIT`, `OFFSET` 等，这些操作是在生成的 SQL 的最后部分进行的。

基本用法：

```java
QueryWrapper<User> wrapper = new QueryWrapper<>();
wrapper.last("LIMIT 5");
```

在这个示例中：

- `last` 方法添加了一个 `LIMIT 5` 子句到查询的末尾，用于限制结果集的返回行数。

示例：假设有一个 `User` 表，并且想要查询所有 `age` 大于 20 的用户，并且结果按照 `id` 降序排列，并且只返回前 10 条记录。

- 可以使用 `apply` 和 `last` 方法来实现这个需求：

```java
QueryWrapper<User> wrapper = new QueryWrapper<>();
wrapper.gt("age", 20) // age > 20
       .orderByDesc("id") // 按 id 降序排序
       .last("LIMIT 10"); // 限制返回结果为前 10 条
```

### 服务层

`ServiceImpl` 和 `IService` 是 MyBatis-Plus 中用于服务层（Service Layer）的两个重要接口和类，它们**帮助简化和规范了**与数据库交互的业务逻辑。

#### IService 接口

`IService` 是 MyBatis-Plus 提供的一个通用服务接口。

- 定义了一些常见的 CRUD（Create, Read, Update, Delete）操作，并将这些操作抽象成方法。
- 这意味着，当使用 `IService` 接口时，无需自己手动编写这些常见的数据库操作方法。

一些常用方法：

- `boolean save(T entity)`: 保存一个实体类对象到数据库。
- `boolean removeById(Serializable id)`: 根据 ID 删除数据。
- `boolean updateById(T entity)`: 根据 ID 更新数据。
- `T getById(Serializable id)`: 根据 ID 查询数据。
- `List<T> list()`: 查询所有数据。
- `Page<T> page(Page<T> page)`: 分页查询数据。

#### ServiceImpl 类

`ServiceImpl` 是 MyBatis-Plus 提供的一个基础实现类，实现了 `IService` 接口中的方法。

- `ServiceImpl` 通常是被继承的，提供了具体的数据库操作方法的实现。
- 开发者只需在自己定义的服务实现类中继承 `ServiceImpl` 类，就可以获得默认的 CRUD 功能。

假设有一个用户表 `User`，并为其定义了一个实体类 `User` 和一个 Mapper 接口 `UserMapper`。

- 可以定义一个服务接口 `UserService` 和一个服务实现类 `UserServiceImpl`。

##### 定义服务接口：

```java
public interface UserService extends IService<User> {
    // 可以定义一些自定义的服务方法
}
```

##### 定义服务实现类：

```java
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
    // 可以重写 ServiceImpl 中的方法，或者定义更多的业务逻辑
}
```

在这个例子中，`UserServiceImpl` 继承了 `ServiceImpl<UserMapper, User>` 并实现了 `UserService` 接口。

- 通过这种方式，`UserServiceImpl` 类可以直接使用 `ServiceImpl` 提供的基本 CRUD 方法。

Mapper 和 Service 中有很多的方法，具体用法可以参考 [Mybatis plus 官网](https://baomidou.com/guides/data-interface/#service-interface)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/fee94c9857eb465b879d42431bb586f5.png)

#### 事务处理

在使用注解定义 SQL 查询时，事务管理通常在**服务层**进行。

- 可以使用 Spring 的 `@Transactional` 注解来管理事务。

```java
@Service
public class UserService {
    @Autowired
    private UserMapper userMapper;
 
    @Transactional
    public void createUser(User user) {
        userMapper.insert(user);
        // 其他逻辑...
    }
}
```



## MapStruct

### 简介

#### 选型

MyBtatis从数据库中查询的数据映射到domain的实体类上，有时候需要将domain的**实体类**映射给前端的VO类，用于展示。

因此，可以借助框架或是工具来实现对象的转换，例如说：

- Hutool 里的`BeanUtils.copyProperties()`：
    - 但是只能转换类中**字段名和类型**都一样的字段。
    - 而且 由于采用的是**反射**，实际上当重复调用时**效率比较低**。（实际测试在生成 次数为1000000时需要1.6秒，而使用MapStruct仅需要69毫秒）。
- Spring BeanUtils
- Apache BeanUtils
- Dozer
- Orika
- **MapStruct**：通过创建一个 MapStruct Mapper 接口，并定义一个转换接口方法，后续交给 MapStruct 自动生成对象转换的代码即可。
- ModelMapper
- JMapper

#### MapStruct

> MapStruct解决的问题：手动创建bean映射器非常耗时。 该库可以自动生成Bean映射器类。

MapStruct是一个开源的基于Java的代码生成器，用于创建实现 Java Bean之间转换的扩展映射器。

- 使用 MapStruct，只需要（通过`@Mapper`、`@Mapping` 注解）**创建接口**，而该库会通过**注解**在编译过程中**自动创建**具体的映射实现，
- 大大减少了通常需要手工编写的样板代码的数量。

用于各个对象**实体间的相互转换**。

- 例如数据库底层实体 转为页面对象，Model 转为 **DTO**，DTO 转为其他中间对象、VO 等等，相关转换代码为**编译时自动产生**的新文件和代码。
- 大部分属性都是相同的，只有少部分的不同。
- 两个对象之间**相同属性名的**会被自动转换。指定特殊情况时，需要通过注解在抽象方法上说明**不同属性之间的**转换。

转换方法一般均为抽象方法，所以这一类文件一般使用 **接口类**，或者抽象类均可，官方的介绍一般均使用了接口类文件来完成。

- 参考：[MapStruct使用指南](https://juejin.cn/post/6956190395319451679)

#### 优点

- 使用**纯 Java 方法**代替 Java 反射机制快速执行。
- 编译时**类型安全**：只能映射彼此的对象和属性，不能映射一个 Order 实体到一个 Customer DTO 中等等。
- 如果无法映射实体或属性，则在编译时**清除错误报告**。

### 依赖

```xml
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct-processor</artifactId>
    <version>1.2.0.Final</version>
 </dependency>
```

### 映射

#### 基本映射用法

加入 MapStruct 的转换相关的注解。

1. **`Convert` 接口类**上**加 `@Mapper`**：当前类认为是要执行 MapStruct 相关操作的类。标记这个接口作为一个映射接口，并且是编译时MapStruct处理器的入口。
    - 注意这里定义的是抽象类，实际上使用接口类也可以。
2. 在接口中定义了`convert()`、`toDto()`方法：该方法接收一个`Doctor`实例为参数，并返回一个`DoctorDto`实例。MapStruct 会把 `Doctor`实例映射到一个`DoctorDto`实例。
    1. 默认情况下，当源对象与 目标对象拥有一样的属性时会自动转换。
    2. 通过添加参数，MapStruct 会自动在返回的 POJO 实例中加入。
    3. 名称不一样时，在方法上**加 `@Mapping`**：为指定某些特殊映射的注解，
        - `source` 为入参，**源对象的属性名**， `target `为目标对象的属性。二者没有顺序之分。
3. 通过` XxxMapper.INSTANCE.` 入口调用方法：如果要将`Doctor`实例映射到一个`DoctorDto`实例，可以这样写：

```java
DoctorDto doctorDto = DoctorMapper.INSTANCE.toDto(doctor);
```

##### `@Mapper`



##### `@Mapping`

- `qualifiedByName` 属性，可以自定义转换方法。

```java
@Named("convertAreaIdToAreaName")
default String convertAreaIdToAreaName(Integer areaId) {
    return AreaUtils.format(areaId);
}

// 将地区编号转为地区名
@Mapping(source = "areaId", target = "areaName", qualifiedByName = "convertAreaIdToAreaName")
DeliveryPickUpStoreSimpleRespVO convert02(DeliveryPickUpStoreDO bean);
```

##### Convert 接口

```java
@Mapper
public interface UserMapper { // UserMapperConvert

    UserMapper INSTANCE = Mappers.getMapper(UserMapper.class);

    @Mapping(source = "describe", target = "des")
    PersonVO transToViewObject(PersionDTO persionDTO);
    
    //DTO中没有openid，通过参数在VO中加入
    AppAuthLoginRespVO convert(OAuth2AccessTokenRespDTO bean, String openid);
}

@Mapper
public interface DoctorMapper {
    
    DoctorMapper INSTANCE = Mappers.getMapper(DoctorMapper.class);
    
    //不同字段间映射，不同属性名称
    //Doctor中的specialty字段对应于DoctorDto类的 specialization 。
    @Mapping(source = "doctor.specialty", target = "specialization")
    //多个源类
    @Mapping(source = "education.degreeName", target = "degree")
    DoctorDto toDto(Doctor doctor); 
    DoctorDto convertToDto(Doctor doctor)
}
```

##### 多个源类

`@Mapping` 注解还支持多个对象转换为一个对象。示例如下图：

![ 复杂示例](../assets/structmap32.png)

##### 构建

当构建/编译应用程序时，MapStruct插件会识别出DoctorMapper接口并为其**生成一个实现类**。

- 这段代码中创建了一个`DoctorMapper`类型的实例`INSTANCE`，在生成对应的实现代码后，这就是我们调用的“入口”。
    - `INSTANCE`：是为了在外面调用该方法， 接口中的属性**默认为静态属性**所以可以直接调用到。
    - 自动生成的**接口的实现**可以通过Mapper的**class对象**获取。按照惯例，接口中会声明一个成员变量**INSTANCE**，从而让客户端可以访问Mapper接口的实现。

```java
public class DoctorMapperImpl implements DoctorMapper {
    @Override
    public DoctorDto toDto(Doctor doctor) {
        if ( doctor == null ) {
            return null;
        }
        DoctorDtoBuilder doctorDto = DoctorDto.builder();

        doctorDto.id(doctor.getId());
        doctorDto.name(doctor.getName());

        return doctorDto.build();
    }
}
```

`DoctorMapperImpl`类中包含一个`toDto()`方法，将`Doctor`属性值映射到`DoctorDto`的属性字段中。

**注意**：可能注意到了上面实现代码中的`DoctorDtoBuilder`。因为builder代码往往比较长，为了简洁起见，这里省略了builder模式的实现代码。

- 如果类中包含Builder，MapStruct会尝试使用它来创建实例；
- 如果没有的话，MapStruct将通过`new`关键字进行实例化

#### 子对象映射

多数情况下，POJO中不会*只*包含基本数据类型，其中往往会包含其它类。比如说，一个`Doctor`类中会有多个患者类。

- 通过 `@Mapping` 注解指定。

```java
@Mapper
public interface PatientMapper {
    PatientMapper INSTANCE = Mappers.getMapper(PatientMapper.class);
    PatientDto toDto(Patient patient);
}

@Mapper(uses = {PatientMapper.class})
public interface DoctorMapper {

    DoctorMapper INSTANCE = Mappers.getMapper(DoctorMapper.class);

    @Mapping(source = "doctor.patientList", target = "patientDtoList")
    @Mapping(source = "doctor.specialty", target = "specialization")
    DoctorDto toDto(Doctor doctor);
}
```

#### ~~更新现有实例~~

有时，我们希望用DTO的最新值更新一个模型中的属性，对目标对象（例子中是`DoctorDto`）使用`@MappingTarget`注解，就可以更新现有的实例。

### 数据类型转换

#### 数据类型映射

MapStruct支持`source`和`target`属性之间的数据类型转换。还提供了基本类型及其相应的包装类之间的自动转换。

自动类型转换适用于：

- 基本类型及其对应的包装类之间。比如， `int` 和 `Integer`， `float` 和 `Float`， `long` 和 `Long`，`boolean` 和 `Boolean` 等。
- 任意基本类型与任意包装类之间。如 `int` 和 `long`， `byte` 和 `Integer` 等。
- 所有基本类型及包装类与`String`之间。如 `boolean` 和 `String`， `Integer` 和 `String`， `float` 和 `String` 等。
- 枚举和`String`之间。
- Java大数类型(`java.math.BigInteger`， `java.math.BigDecimal`) 和Java基本类型(包括其包装类)与`String`之间。
- 其它情况详见[MapStruct官方文档](https://link.juejin.cn?target=https%3A%2F%2Fmapstruct.org%2Fdocumentation%2Fstable%2Freference%2Fhtml%2F%23implicit-type-conversions)。

##### 数字格式转换

在进行日期转换的时候，可以通过`dateFormat`标志指定日期的格式。

除此之外，对于数字的转换，也可以使用`numberFormat`指定显示格式：

#### 枚举映射

为了在这些枚举项之间建立桥梁，可以使用`@ValueMappings`注解，可以包含多个`@ValueMapping`注解。

- 这里，将`source`设置为三个具体枚举项之一，并将`target`设置为`CARD`。

#### 集合映射

##### List映射

```java
@Mapper
public interface DoctorMapper {
    List<DoctorDto> map(List<Doctor> doctor);
}
```

##### ~~Set和Map映射~~

##### ~~集合映射策略~~

##### ~~目标集合实现类型~~

### 进阶操作

#### 依赖注入

到目前为止，我们一直在通过`getMapper()`方法访问生成的映射器：

```java
DoctorMapper INSTANCE = Mappers.getMapper(DoctorMapper.class);
```

但是，如果使用的是Spring，只需要简单修改映射器配置，就可以像常规依赖项一样**注入映射器**。

修改 `DoctorMapper` 以支持Spring框架：

```java
@Mapper(componentModel = "spring")
public interface DoctorMapper {}
```

在`@Mapper`注解中添加`(componentModel = "spring")`：是为了告诉MapStruct，在生成映射器实现类时，希望它能支持通过Spring的依赖注入来创建。

- 这样，生成的 `DoctorMapperImpl` 会带有 `@Component` 注解，
- 就不需要在接口中添加 `INSTANCE` 字段了。

```java
@Component
public class DoctorMapperImpl implements DoctorMapper {}
```

只要被标记为`@Component`，Spring就可以把它作为一个bean来处理，就可以在其它类（如控制器）中通过`@Autowire`注解来使用它：

```java
@Controller
public class DoctorController() {
    @Autowired
    private DoctorMapper doctorMapper;
}
```

如果你不使用Spring，MapStruct也支持[Java CDI](https://link.juejin.cn?target=https%3A%2F%2Fdocs.oracle.com%2Fjavaee%2F6%2Ftutorial%2Fdoc%2Fgiwhl.html)：

```java
@Mapper(componentModel = "cdi")
public interface DoctorMapper {}
```

#### 添加默认值

`@Mapping` 注解有两个很实用的标志就是常量 `constant` 和默认值 `defaultValue` 。

- 无论`source`如何取值，都将始终使用常量值；
-  如果`source`取值为`null`，则会使用默认值。

#### ~~添加表达式~~

#### ~~添加自定义方法~~

#### ~~创建自定义映射器~~

##### @BeforeMapping、@AfterMapping

为了进一步控制和定制化，可以定义 `@BeforeMapping` 和 `@AfterMapping`方法。

- 显然，这两个方法是在每次映射之前和之后执行的。
- 也就是说，在最终的实现代码中，会在两个对象真正映射之前和之后添加并执行这两个方法。

#### ~~映射异常处理~~

#### ~~映射配置~~

##### ~~继承配置~~

##### ~~继承逆向配置~~





