---
title: Spring Boot 整合 Swagger UI 接口文档
categories:
  - SSM框架
tags:
  - SSM
  - Spring-Boot
  - Swagger-UI
location:
abbrlink: 'swagger_ui'
permalink: 'swagger_ui'
date: 2025-06-14 13:42:12
updated: 2025-06-14 11:56:00
---

> 摘要：可动态地根据**注解**生成在线API文档。是一套基于 OpenAPI 规范构建的开源工具，可帮助设计、构建、记录及使用 Restful 接口。

<!-- more -->

## 目录

[TOC]

## 整合 Swagger-UI 接口文档

##### 接口文档简介

Swagger-UI（音S歪戈）：可动态地根据**注解**生成在线API文档。是一套基于 OpenAPI 规范构建的开源工具，可帮助设计、构建、记录及使用 Restful 接口。

作用： 前后端分离的情况下，只需少量注解即可生成一份自带 UI 界面的 **Rest API 文档**，包括接口需要的参数及返回值，还可直接对 API **调试**。

##### 整合 Swagger-UI 的步骤

1. 添加项目依赖；
2. 添加 Swagger-UI 的 Java 配置文件；
3. 给 XxxController 添加 Swagger 注解；
4. 修改 MyBatis Generator 注释的生成规则，运行代码生成器重新生成 model 实体类，给字段加上注解 `@ApiModelProperty` 来取代原来的方法注释，（~~mapper 接口及其xml实现文件~~）。
    1. 在 Generator 生成类中，读取并解析 `generatorConfig.xml` 配置文件，
    2. 指定**自定义注释生成器** `CommentGenerator` 类中的方法，
    3. 使用 `field.addJavaDocLine()`，给实体类的字段添加注释、Swagger 注解 `@ApiModelProperty` 来取代原来的方法注释。
5. 访问Swagger-UI接口文档地址：http://localhost:8080/swagger-ui/index.html

##### 配置文件

`config/SwaggerConfig.java`

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig extends BaseSwaggerConfig {
    @Override
    public SwaggerProperties swaggerProperties() {
        return SwaggerProperties.builder()
            .apiBasePackage("com.macro.mall.controller")
            .title("mall后台系统")
            .description("mall后台相关接口文档")
            .contactName("macro")
            .version("1.0")
            .enableSecurity(true)
            .build();
    }
}
```

##### 常用注解

1. `@EnableSwagger2`：打开 Swagger-UI，用于生成相关文档信息；
2. `@Api`：修饰 Controller 类；
3. `@ApiOperation`：修饰 Controller 类中的接口方法；
4. `@ApiParam`：修饰接口参数；
5. `@ApiModelProperty`：修饰**实体类的属性**，用于实体类是请求参数或返回结果时；
6. `@Schema` 注解：通常与Swagger或OpenAPI规范一起使用，用于为API模型（如[请求体](https://so.csdn.net/so/search?q=请求体&spm=1001.2101.3001.7020)、响应体等）中的属性或整个模型**提供元数据描述**。这些描述信息对于生成API文档、客户端代码以及理解API的结构和用法非常有帮助。

##### `CommentGenerator` 自定义注释生成器

```java
/**
 * 自定义注释生成器
 */
public class CommentGenerator extends DefaultCommentGenerator {
	public void addFieldComment(...) {
        ...
        // 给model的字段添加@ApiModelProperty注解来取代原来的方法注释
        field.addJavaDocLine("@ApiModelProperty(value=\"" + remarks + "\")" );
    }
    // 使其能在import中导入@ApiModelProperty，
    // 否则需手动导入该类，在需生成大量实体类时非常麻烦
    public void addJavaFileComment(...) {
    ...
    }
}
```

参考：

- [springboot-guide Swagger ](https://snailclimb.gitee.io/springboot-guide/#/./docs/basis/swagger)
- [从零入门 ！Spring Security With JWT（含权限验证）后端部分代码](https://github.com/Snailclimb/spring-security-jwt-guide)



## Swagger 注解

#### 基本信息注解

可用于：类、接口。

- `@OpenAPIDefinition`：用于定义整个 API 文档的基本信息。
    - `info`：指定 `@Info` 注解的对象，用于描述 API 文档的基本信息。

- `@Info`：用于定义 API 文档的基本信息。

- `@Contact`：用于定义 API 文档中的联系人信息。

- `@License`：用于定义 API 文档中的许可证信息。

#### 分组注解

- `@Tag`：用于给 API 分组，用途类似于为 API 文档**添加标签**。
    - 可用于：方法、类、接口。

```java
@FeignClient(name = ApiConstants.NAME)
@Tag(name = "RPC 服务 - 文件")
public interface FileApi {

    String PREFIX = ApiConstants.PREFIX + "/file";
    ....
}
```

#### 请求方法注解

可用于：方法。

以下注解用于描述 API 的请求方法：

- `@Operation`：用于描述 API 的操作。
- `@RequestBody`：用于描述操作的请求体。
- `@ApiResponse`：用于描述操作的响应结果。
- `@Content`：用于描述请求体或响应的内容。
- `@Parameter`：用于描述操作的**输入参数**。
    - `@Parameters`
- `@Schema`：用于描述数据模型的属性。
    - 可用于：方法、类、接口。

```java
@Parameters({
    @Parameter(name = "dataSourceConfigId", description = "数据源配置的编号", required = true, example = "1"),
    @Parameter(name = "name", description = "表名，模糊匹配", example = "yudao"),
    @Parameter(name = "comment", description = "描述，模糊匹配", example = "芋道")
})
```

#### 路径注解

可用于：方法的参数。

以下注解用于描述 API 的路径：

- `@Path`：用于定义路径参数。
    - 可用于：方法。
- `@PathVariable`：用于描述路径参数。
- `@RequestParam`：用于描述查询参数。
- `@RequestBody`：用于描述请求体。

#### 响应注解

可用于：方法。

以下注解用于描述 API 的响应结果：

- `@ApiResponse`：用于描述响应结果。
- `@Content`：用于描述响应结果的内容。
- `@Schema`：用于描述数据模型的**属性**。
    - 可用于：方法、类、接口。

```JAVA
@Schema(description = "RPC 服务 - API 错误日志创建 Request DTO")
@Data
public class ApiErrorLogCreateReqDTO {

    @Schema(description = "链路追踪编号", example = "89aca178-a370-411c-ae02-3f0d672be4ab")
    private String traceId;

    @Schema(description = "用户编号", requiredMode = Schema.RequiredMode.REQUIRED, example = "1024")
    private Long userId;
    
    ....
}
```









