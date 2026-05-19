---
title: SSM 框架、Spring MVC
categories:
  - SSM框架
tags:
  - SSM
  - Spring-MVC
  - Spring
  - Spring-Boot
  - MyBatis
  - RESTful
  - HTTP
  - Tomcat
  - HttpServetRequest
  - Hutool
location:
  - 黄金时代
abbrlink: 'ssm'
permalink: 'ssm'
date: 2021-09-01 13:42:12
updated: 2025-05-28 11:56:00
---

> 摘要：Spring + Spring MVC + Mybatis。Java Web，Spring MVC，包括 MVC 设计模式、工作原理、前后端分离、接收请求参数（REST 常用注解）、参数校验、统一响应和异常，Hutool、Guava 工具类。

<!-- more -->

## 目录

[TOC]

## SSM 框架

> 介绍项目时，用一个[业务流程](#业务流程)说说 spring mvc 如何做的。

SSM 框架集是 `Spring + Spring MVC + Mybatis ` 框架的整合：`Spring` 实现**业务对象**管理，`Spring MVC` 负责请求转发和视图管理，`Mybatis` 作为数据对象的持久化引擎。

- 是标准的 MVC 模式，将整个系统划分为 `model/DAO` 层、`View` 层、`Controller` 层、`Service` 层.
- 是目前比较主流的 Java EE 企业级框架，适用于搭建各种大型的企业级应用系统。

### 模块框架演化

1. MVC 模式：应用分层开发 ——> Spring MVC
    1. Model：**@Entity + @NamedQuery 写 SQL** ——> JPA 数据接口层（extends JpaRepository + @Repository 注解）
    2. View：
    3. Controller：
2. SSM：
    1. Spring（实现**业务对象 **Bean 管理）——> Spring Boot
    2. Spring MVC：负责请求转发和视图管理。
        - Controller：
            - ——> Service 层 + ServiceImpl：（剥离重复的业务逻辑，@Autowired 注入 Bean）
            - RESTful + @RestController ——> CommonResult + ErrorCode + Exception + Hibernate Validator 参数校验
        - View ——> 对 View 细化，前后端分离（后端**关注数据和逻辑**，前端关注界面与交互）
        - Model：DAO 数据持久层 ——> DAL 数据访问层：
            - Mapper 接口
            - POJO：DO + VO + DTO，Lombok（实体类 Getter/Settor 注解）
            - **Redis**
    3. MyBatis（作为数据对象的持久化引擎） ——> MyBatis Plus ——> MapStruct + Convert
        - **ORM**：对象–关系映射，用于建立 Java Object **实体类**和数据库表之间的映射，从而达到**操作**实体类就相当于操作数据库表的目的。
        - MyBatis 实现 Mapper 接口
            1. 全注解实现 Mapper（SQL **不够灵活**，每次都要修改 SQL）
            2. XML 实现 Mapper。工作量大。（MyBatis Generator 生成）
            3. MyBatis Plus 增强（BaseMapper + LambdaQueryWrapper 条件构造器） ——> MapStruct  + **Convert**（ DO 转 VO） + Easy Trans 翻译枚举属性
3. Spring Boot 整合框架 ——> Spring Cloud ——> Spring Cloud Alibaba
    - 整合 Spring Security ——> Cookie、Session、Token、JWT——> OAuth2 + RBAC 授权
    - 整合 Swagger2 ——> Swagger3 ——> Knife4j
    - 整合 ELK
    - ——> ~~WebSocket~~
4. 单体应用 ——> **多模块** ——> 分布式架构 ——> SOA ——> 微服务架构：
    - API：Feign 
        - Gateway、Nacos、Config
        - 分布式链路追踪
    - 高并发、高性能、高可用集群：
        - 负载均衡：Nginx、CDN、Feign 客户端负载均衡
        - **消息队列**
        - 限流降级熔断
    - 数据库： 读写分离、分库分表

### 简化的开发步骤

1. 需求分析：功能需求和非功能需求（稳定性、性能）；
2. 系统设计：系统**架构**示意图：普通用户、管理员，前端（输入输出），后台，MyBatis 数据持久化，微服务，配置等；
    1. 系统概要设计：系统功能模块（划分）图：系统-子系统-功能模块，各自的描述、输入输出接口设计（格式、入参和出参/返回值）。
        - 接口文档现在都是 [Swagger-UI](#接口文档)，注解标注在 Controller；
    2. 数据库表结构设计：
        1. **逻辑设计/详细设计** : 通过将 E-R 图转换成表，实现从 E-R 模型到关系模型的转换（转为 POJOs 及 DAO 接口），并应用三大范式优化；
        2. **物理结构设计** : 为设计的数据库选择合适的存储结构和存取路径，做具体的技术选型。
        3. 数据库**实现** : 包括 SQL 编程（创建数据库表）、测试和试运行。
        4. 数据库的**运行和维护** : log 日志，根据新需求进行建表、索引优化、大表拆分。
    3. 功能模块详细设计：模块流程图（Controller 调用 Sercive 实现业务逻辑）。
3. 系统实现；
4. 单元测试，回归测试，覆盖率测试。

#### 系统业务架构示意图

项目技术架构图：

业务架构图：

应用架构图：

<img src="../assets/mall系统架构示意图.jpg" alt="系统架构图" style="zoom: 37%;" />

#### ~~系统功能模块图~~



#### E-R 模型图

<img src="../assets/image-20220925110316119.png" alt="image-20220925110316119" style="zoom:100%;" />

<img src="../assets/image-20220925111724175.png" alt="image-20220925111724175" style="zoom:100%;" />

### 项目架构

<img src="../assets/image-20250805140940884.png" alt="img" style="zoom:60%;" />

在[《阿里巴巴 Java 开发手册》](https://github.com/alibaba/p3c/blob/master/阿里巴巴Java开发手册（华山版）.pdf)中，推荐分层如下图：

<img src="../assets/ef0d24cfaecdbe703ad646e09e697454" alt="应用分层" style="zoom:80%;" />

#### 分层目录

项目的目录结构展示了Maven所约定了源代码的位置，只需配置很少的信息就可以自动完成编译，测试和打包等工作。

<img src="../assets/arch_screen_02.png" alt="img" style="zoom:70%;" /><img src="../assets/250px-Maven_CoC.svg.png" alt="img" style="zoom:100%;" />

Java Web 项目中的 SSM 目录结构，同时也遵循 maven 的目录规范：

##### `mall-admin/src/`

`mall-admin/src/test/`：编写测试用例

`mall-admin/src/main/`：自定义的代码

1. `java/com.macro.mall/`：
    1. `MallAdminApplication.java`：程序主入口；
    2. `common/`：存放通用类
        1. `api/`：~~参数定义~~、`CommonResult` 通用返回结果、IErrorCode、ResultCode；
            1. `BaseController/CommonResult` 类：用于**统一封装返回对象**，供所有的 controller 继承，主要提供了返回成功或失败对象的几种方法。
        2. `baseconfig/`：BaseRedisConfig、BaseSwaggerConfig；
        3. `config/`：存放 **Java 配置类**，如 MyBatis、Swagger、Redis、Spring Security、Oss 配置类等。
        4. `util/`：`RequestUtil`、`SessionUtil`、`JwtTokenUtil`、`WebResponseUtil`等各种工具类。
    3. `component/`：存放通用组件、AOP 切面，如日志、异常、权限管理、搜索等
        1. `domain/`：SwaggerProperties、WebLog 日志封装类、搜索结果信息类；
        2. `service/`：RedisService 操作。
            1. `RedisCacheAspect`：Redis 缓存切面，防止Redis宕机影响正常业务逻辑；
            2. 简单搜索，综合搜索、筛选、排序；
        3. `log/` ：WebLogAspect Controller层的统一日志处理**切面**；
        4. `exception/`：GlobalExceptionHandler 全局异常处理**切面**；
        5. `security/`：`DynamicSecurityFilter` 类等；
        6. `search/`：搜索切面。
    4. ~~`mbg/`~~：最好单独一个独立项目
        1. ` mapper/XxxMapper.java`：
        2. `model/`：
    5. `dao/XxxDao.java`：存放自定义的 mapper 接口；
        1. `dto/`：存放自定义的数据传输对象，如请求参数、返回结果。
        2. `bo/`：
    6. `controller/`：
    7. `service/`：
        - `serviceImpl/`：
2. `resources/` ：
    1. `dao/XxxDao.xml`：自定义的 **xml 映射文件**；
    2. `application.yml`：[Spring 的配置文件](#Spring配置)，具体包括 datasource 数据库连接配置（含 Druid 数据池配置）、MyBatis 配置、Redis配置、logging 和 logstash 配置、JWT 配置、Secure 配置等。
    3. **View 视图层/表示层/展示层**：与控制层结合比较紧密，需要二者结合起来协同开发。主要负责前端 JSP 页面的显示。~~负责处理 HTTP 请求，将 JSON 参数转换为对象。~~
    4. WEB-INF：

##### `mall-mbg/src/main/`

> MyBatis Generator 生成器模块，生成 MBG 代码

1. `java/com.macro.mall/`：
    1. ` mapper/XxxMapper.java`： **mapper 接口/映射器**，存放（MyBatis Generator 生成的）通用 mapper 接口，对应  `resources/` 下的 xml 映射文件；常用的**单表查询**接口、和 **`Example`类的条件查询**等。参数为 model、id 等，供业务逻辑层调用。
    2. `model/`：**Entity 实体层 / model / Bean**，存放生成的 **model 实体**/ POJO 类；对业务对象与数据库的行进行相互转换，用对象**映射**数据表，二者一一对应，本质是数据表的**对象化**。是一种 ORM 对象关系映射。一般只在 DAO 层与 Service 层之间传输。
    3. `CommentGenerator.java`：自定义注释生成类，使用 `field.addJavaDocLine()` 定义注释的**生成规则**，给实体类的字段添加注释、添加 **Swagger 注解** `@ApiModelProperty` 来取代原来的方法注释。
    4. `Generator.java`： **MyBatis 代码生成器**，程序主入口，读取并解析 `resources/generatorConfig.xml` 配置文件，用来运行生成 mbg 包中的代码。
2. `resources/` ：
    1. `com.macro.mall.mapper/XxxMapper.xml`：通用的 **xml 映射文件**，
    2. `generatorConfig.xml`： MyBatis Generator 生成器的配置文件，可配置数据库连接，指定生成 model、mapper 接口及 mapper.xml 的路径，可通过 `<commentGenerator>`标签来指定添加 **Swagger 注解**等。详细配置见 [Mybatis Generator 配置](#Mybatis Generator 配置)。
    3. `generator.properties`：配置 JDBC 数据库连接的参数。

##### ~~`mall-search/src/main/`~~

> 搜索模块，整合到上面的主要结构中；

1. `java/com.macro.mall.search/`：
    1. `config/`：存放 **Java 配置类**，如 MyBatis、Swagger 配置类。
    2. `controller/`：简单搜索，综合搜索、筛选、排序；
    3. `dao/XxxDao.java`：存放自定义的 mapper 接口；
    4. `domain/`
    5. `repository/`
    6. `service/`
    7. `MallSearchApplication.java`

##### ~~`mall-security/src/main/`~~

> 权限管理模块

1. `java/com.macro.mall.security/`：
    1. `aspect/`：RedisCacheAspect Redis 缓存切面，防止Redis宕机影响正常业务逻辑；
    2. `component/`：
        - `DynamicSecurityFilter` 类
    3. `config/`：RedisConfig、SecurityConfig；
    4. `util/`：JwtTokenUtil；

##### ~~`mall-portal/src/main/`~~

> 微服务模块

#### POJO 分层领域模型

<img src="../assets/5618351-edf45bb66a4f47a3.png" alt="img" style="zoom: 80%;" />

![UTOOLS1582426816862.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/2/23/1706ffcfd8d1255b~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp)

`POJO`（Plain Ordinary Java Object）: 指只有 setter / getter / toString 的简单类，包括 DO / DTO / BO / VO 等。

- 一个POJO持久化以后就是`PO`
- 直接用它传递、传递过程中就是`DTO`
- 直接用来对应**表示层**就是`VO`

分类：

1. `DO`（Data/**Domain** Object）：**数据库映射对象**，数据/领域对象，由 DAO 层向上（Service 层 / ServiceImpl 实现层）传输的，从现实世界中抽象出的有形或无形的**业务实体**。
    - ~~`PO`~~（Persistent Object）、**Entity**：持久化对象，与持久层（通常是关系型数据库表）数据结构一 一对应的 POJO 类，由 DAO 层向上（Service 层）传输的 **数据源对象**。最形象的理解就是一个PO就是数据库中的一条记录。 好处是可以把一条记录作为一个对象处理，可以方便的转为其它对象。
3. `BO`（Business Object）：业务对象，由 Service 层向上（Controller 层）传输的，主要作用是把业务逻辑封装为一个对象。通常由 DO **转化、整合**而来，可包含**多个 DO 的属性**，也可只包含一个 DO 的部分属性。
3. `DTO`（Data Transfer Object）：数据传输对象，用于展示层（前端）与**服务层**间的数据传输对象。存放**参数类型定义类**。DTO为系统与外界交互的模型对象，可以将DTO对象转化为**BO对象**或者是普通的**entity对象**，让service层去处理。
    - `BeanUtils.copyProperties(umsAdminParam, umsAdmin);`，Service 中将 DTO 对象 `UmsAdminParm `赋值给 DO 实体类对象 `UmsAdmin`。
    - 比如添加用户时，只需要传过来用户的姓名和年龄，后端接受到数据后，将添加创建时间和更新时间和默认密码三个字段，然后保存到数据库。
4. `VO`（View Object）：视图对象，通常是 Web 向**模板渲染引擎层**传输的对象。用于**展示层**，把某个指定页面（或组件）的所有数据封装起来。
5. ~~`Query`~~：数据查询对象，各层接收上层的查询请求。注意超过 2 个参数的查询封装，禁止使用 Map 类传输。

#### DTO 数据传输对象

用于 **Service 层**与 **Web 层**间、前后端间的传输。**存放自定义的传输对象**，如**请求参数**类型定义和返回结果。

- `POJO `对象无法满足业务需求；
- 面向表现层（界面UI），通过 UI 的需求来定义 `DTO`。实现了表现层与 `Model `间的解耦，**表现层不引用 Model**；如果开发过程中模型改变了而界面没变，就只需改 `Model `而不需去改表现层。要维护 `DTO` 与 `Model `间的映射关系。

#### DAO 数据访问 / 持久层

用于处理数据业务，存放自定义的 `Mapper` 映射器**接口**（`public interface UmsAdminDao{}`），由对应 `resources/dao`下的映射文件（`XxxDao.xml`）实现。MyBatis 可动态实现接口的方法，所以不需要 `daoImpl`。

直接进行数据库的读写操作，返回与数据库表一一对应的**数据对象 DO**，即 `Domain/Entity` 数据库映射对象。

- DAO 层的数据源配置、以及有关数据库连接的参数都在 Spring 配置文件中进行配置。

~~repository 层~~：通过对 Entity 层的封装提供 CRUD 接口 。继承 CrudRepository、PagingAndSortingRepository、JpaRepository 接口，或用[对应JPA各层常用注解](#数据库（事务问题）)。

二者非常类似，但从架构设计上讲有着本质的区别：侧重点完全不同。

- Repository 是相对对象而言，蕴含着真正的OO概念，即一个**数据仓库角色**，负责所有对象的持久化管理；
- DAO 则是相对数据库而言，DAO**直接负责**数据库的存取工作。

##### MyBatis

> 两个配置指定的路径相同。

- Spring Boot 配置类配置 MyBatis，用来指定 mapper 映射器接口的 xml 实现所在的**项目目录**。
- MyBatis 配置类中通过`@MapperScan`，用来指定 mapper 映射器接口的 xml 实现所在的**包路径**。

##### Mybatis Generator 简介

MyBatis 的代码生成器，可根据数据库生成 `model、domain` 实体类，常用的**单表查询** `XxxMapper` 接口、和**`Example`类的条件查询**、及对应的 `XxxMapper.xml`；

#### Controler 控制层

存放控制器：类上常用用[`@Controller`](#@Controller)定义控制层，`@RestController` = `@Controller` + `@ResponseBody`；

1. 用于路由解析，**对外暴露 REST API 接口**；负责**接收并转发**请求、参数校验；
    1. 方法上常用 [@RequestXxx](#REST常用注解) 系列注解，如`@RequestMapping` 用来拦截和转发用户请求、`@RequestParam` 参数绑定、`@RequestBody` 用于获取 HTTP 请求体中的数据；
    2. 返回对象常用 `@ResponseBody` 返回响应结果。
    3. `@Valid` 等用于搭配 [参数校验](#整合HibernateValidator)（具体通过在 **POJOs 对象**的字段上加注解实现）；
    4. 根据 UI 和接口的入参和出参，编写 Controller 类中的方法。
2. **控制业务流程**、页面访问控制和交互（组织不同层面），**指定视图**并将响应结果发送给客户端；异常处理（给方法加上**切面**来统一处理异常）。
3. 调用 Service 层**处理请求**和**响应事件**（“事件”包括用户的行为和数据的改变），将 Service 层返回的 `BO/DO` 转化为 `DTO/VO`（**封装成统一返回对象**返回给调用方，如果返回数据用于**前端模版渲染**则返回 `VO`，否则一般返回 `DTO`）。

    - 如将 gender 属性中的 0 转化“男”，1 转化为“女”等。[一个完整后端请求的 4 个组成部分、请求参数校验、统一响应、统一异常](https://www.zhihu.com/question/434640634/answer/2640841148)。
    - Service 层的代码抛出异常，可以给controller层的方法加上**切面**来统一处理异常。
        - `@ControllerAdvice` 注解：用来定义 Controller 层的切面，所有 `@Controller` 注解的类中的方法执行都会进入该切面，
        - 同时可使用 `@ExceptionHandler` 对不同的[异常进行捕获和处理](#异常)；对于捕获的异常，进行日志记录，并封装返回对象。[设计之道－controller层的设计](https://www.jianshu.com/p/654f4589eb8e)

#### Service 业务逻辑层

- 添加 Service 接口及实现：用于处理事件，实现具体的业务逻辑，如事务控制等。处于中间层的位置，既调用 DAO 层提供的接口处理业务逻辑，又提供接口给 Controller 层。 每个模型都有一个 Service 接口，分别封装各自的业务处理方法。
- 返回**数据对象 DO 或业务对象 BO**。外部调用（HTTP、RPC）方法也在这一层，对于外部调用来说，Service 一般会将外部调用返回的 DTO 转化为 BO。

## Spring、Spring MVC、Spring Boot

Spring，Spring MVC，Spring Boot 之间什么关系?

1. `Spring`：是一个**轻量级开源框架**，用于简化 ~~Java EE~~ 企业级应用程序开发。核心是 IoC（**控制反转**）和 **AOP**（面向切面编程）。还提供了事务管理等功能。
    - 可以用于构建**任何类型**的 Java 应用程序，包括 Web 应用程序、桌面应用程序、和批处理应用程序等。
2. `Spring MVC`：Spring MVC 是构建在 Spring 之上的 **Web 框架**，用于**构建 Web 应用程序**。提供了一个基于 Model、View、Controller 模式的 Web 框架，用于处理 Web 请求和响应。
    - 通过将请求**映射**到相应的处理器方法，并使用**视图**来呈现响应，使得构建Web应用程序变得简单和灵活。
3. `Spring Boot`：是一个用于**简化 Spring 应用程序开发**的框架，用于快速构建基于 Spring 的 Web 应用程序。Spring Boot **依赖于** Spring 框架和 Spring MVC。提供了**自动配置**、开箱即用等功能。
    - 内置了许多常用的**第三方库和框架**，简化了配置和部署过程。还提供了一组用于开发Web应用程序、RESTful 服务和微服务的起步**依赖项**。

## Java Web

> Spring 整合 Web

### 服务器

**Web 服务器**：指能为**发出请求**的浏览器（客户端）**提供文档**的程序。只需支持HTTP协议、HTML文档格式及URL。与客户端的网络浏览器配合。因为Web服务器主要支持的协议就是HTTP，所以通常情况下HTTP服务器和WEB服务器是相等的。

- `N’ginx`：轻量级的 Web （静态资源）服务器、反向代理服务器，内存占用少，启动极快，高并发能力强，在互联网项目中广泛应用。
- Apache HTTP 服务器：用 C 语言实现的 HTTP Web 服务器。

**应用程序服务器**：Web服务器传送(serves)页面使浏览器可以浏览，而应用程序服务器提供的是客户端应用程序可以调用(call)的方法(methods)。

- `Tomcat`：是一个开源 `Servlet容器 / Web容器`，负责处理客户端请求，并返回响应。简单来说，Tomcat 主要实现了 2 个核心功能：
    1. 处理 `Socket` 连接，负责网络字节流、`Request` 和 `Response` 对象的**转化**。用来管理、运行、支持 Java Servlet、JSP（JavaServer Pages ）、Java 表达式语言、 Java WebSocket 技术 。
    2. 加载和管理 `Servlet`，及具体处理 `Request` 请求。
- `Jetty`、`Undertow`：

**Web 应用服务器**：目前常用的混合软件解决方案。

Servlet 容器（**重点理解**）：是一个（基于 Java）**运行在服务器端**的 Web 组件 / 接口，用于交互式的浏览和修改数据，生成动态 Web 内容。 

- 按照 Servlet 规范编写的 Java 类，被编译为平台独立的字节码，可被动态地加载到支持 Java 的 Web 服务器中运行。
- 执行过程。

### Filter 过滤器

> 见统一异常文档。

### Tomcat

<img src="../assets/tomcat_server_response.png" alt="在这里插入图片描述" style="zoom: 50%;" />

Tomcat 运行机制：Tomcat 服务器接受客户请求并做出响应的过程。

Spring Boot 在启动 SpringMVC 时，会默认初始化一个内嵌的 Tomcat ，监听 8080 端口的请求。

### HTTP 请求处理流程

1. **客户端**（通常都是浏览器）访问 **Web 服务器**，发送 HTTP 请求。
    - Web 服务器：也叫 HTTP 服务器、web容器，其需要提供web应用运行所需的环境，接收客户端的Http请求
2. Web 服务器接收到请求后，传递给 **Servlet 容器**。
    -  Servlet 容器：也称Servlet引擎，为Servlet的运行提供环境支持，可以理解为 **Tomcat** 或其他服务器。
3. **Servlet 容器**（根据 url 等路径决定）加载对应 Servlet 实例；调用 Servlet 的 `service()` 方法返回 **response 对象**，向其传递表示请求和响应的对象。
    1. 如果Serlvet没有被实例化，则创建该Servlet的一个实例（调用init方法）；
    2. 或从线程池中取出一个空闲线程（产生 Servlet 实例）；
4. Servlet容器根据用户的HTTP请求，创建一个**`ServletRequest`请求对象**（封装了 HTTP 请求信息、见 [HttpServletRequest](#接收请求参数)）和一个可以对HTTP请求进行响应的`ServletResponse`对象（类似于寄信，并在信中说明回信的地址），
5. 然后调用`HttpServlet`中重写的**`service(ServletRequest req, ServletResponse res)`方法**，
    1. 并在这个方法中，将这两个对象**向下转型**，得到**`HttpServletRequest`**和`HttpServletResponse`两个对象，
    2. 然后将客户端的请求转发到`HttpServlet`中的`service(HttpServletRequest req, HttpServletResponse resp)`。
6. `HttpServletResponse`：服务端处理完Http的请求后，将处理结果作为**Http响应**返回给客户端。

参考：[HttpServletRequest和@Requestparam、@RequestBody、直接实体接收请求参数的区别与示例](https://blog.csdn.net/qq_41358574/article/details/120422160)

### WebSocket

> WebSocket 框架，支持多节点的广播。见 WebSocket 文档。

提供 **Token 认证**、WebSocket **集群广播**、Message 监听。

## Web 相关的封装组件

Web 相关的封装组件：

1. web：针对 **SpringMVC** 的基础封装 ：
    1. YudaoWebAutoConfiguration API 前缀和跨域
    2. 参数校验
    3. 统一响应和异常处理（GlobalResponseBodyHandler、GlobalExceptionHandler、ApiRequestFilter）。
2. encrypt：HTTP API 加密组件：支持 Request 和 Response 的加密、解密
3. API log 访问日志
4. Swagger 接口文档
5. XSS 安全、漏洞防护
6. 数据脱敏
7. Jackson 序列化

### HTTP API 加解密配置

```java
package cn..framework.encrypt.config;

/**
 * HTTP API 加解密配置
 *
 */
@ConfigurationProperties(prefix = "yudao.api-encrypt")
@Validated
@Data
public class ApiEncryptProperties {

    /**
     * 是否开启
     */
    @NotNull(message = "是否开启不能为空")
    private Boolean enable;

    /**
     * 请求头（响应头）名称
     *
     * 1. 如果该请求头非空，则表示请求参数已被「前端」加密，「后端」需要解密
     * 2. 如果该响应头非空，则表示响应结果已被「后端」加密，「前端」需要解密
     */
    @NotEmpty(message = "请求头（响应头）名称不能为空")
    private String header = "X-Api-Encrypt";

    /**
     * 对称加密算法，用于请求/响应的加解密
     *
     * 目前支持
     * 【对称加密】：
     *      1. {@link cn.hutool.crypto.symmetric.SymmetricAlgorithm#AES}
     *      2. {@link cn.hutool.crypto.symmetric.SM4#ALGORITHM_NAME} （需要自己二次开发，成本低）
     * 【非对称加密】
     *      1. {@link cn.hutool.crypto.asymmetric.AsymmetricAlgorithm#RSA}
     *      2. {@link cn.hutool.crypto.asymmetric.SM2} （需要自己二次开发，成本低）
     *
     * @see <a href="https://help.aliyun.com/zh/ssl-certificate/what-are-a-public-key-and-a-private-key">什么是公钥和私钥？</a>
     */
    @NotEmpty(message = "对称加密算法不能为空")
    private String algorithm;

    /**
     * 请求的解密密钥
     *
     * 注意：
     * 1. 如果是【对称加密】时，它「后端」对应的是“密钥”。对应的，「前端」也对应的也是“密钥”。
     * 2. 如果是【非对称加密】时，它「后端」对应的是“私钥”。对应的，「前端」对应的是“公钥”。（重要！！！）
     */
    @NotEmpty(message = "请求的解密密钥不能为空")
    private String requestKey;

    /**
     * 响应的加密密钥
     *
     * 注意：
     * 1. 如果是【对称加密】时，它「后端」对应的是“密钥”。对应的，「前端」也对应的也是“密钥”。
     * 2. 如果是【非对称加密】时，它「后端」对应的是“公钥”。对应的，「前端」对应的是“私钥”。（重要！！！）
     */
    @NotEmpty(message = "响应的加密密钥不能为空")
    private String responseKey;
}
```

### ~~API 日志~~



### Swagger

> 见 Swagger 文档。

基于 Swagger + Knife4j 实现 API 接口文档。

* Swagger 自动配置类，基于 OpenAPI + Springdoc 实现。
    1. Springdoc 文档地址：<a href="https://github.com/springdoc/springdoc-openapi">仓库</a>
    2. Swagger 规范，于 2015 更名为 OpenAPI 规范，本质是一个东西
* 构建 Authorization 认证请求头参数

### XSS

> 跨站脚本攻击。见攻击技术和漏洞防护文档。

创建 XssFilter Bean，解决 Xss 安全问题。

* XSS 过滤 jackson 反序列化器。
* 在反序列化的过程中，会对字符串进行 XSS 过滤。

### 脱敏

> 见数据脱敏文档。

### JackSon 框架

Jackson可以轻松的将**Java对象**转换成**json对象和xml文档**，同样也可以将json、xml转换成Java对象。

- 相比json-lib框架，Jackson所依赖的jar包较少，简单易用并且性能也要相对高些。
- Jackson解析的速度算是同类框架中最快的，同时也是Spring MVC中内置使用的 **json 解析方式**。

#### 注解

核心 Jackson 注解由 **jackson-annotations 核心组件**定义，它不依赖于任何其他包。

- **@JsonProperty：**用于在 Java 对象的属性和 JSON 字段之间建立映射关系。通过该注解，可以自定义属性在序列化和反序列化过程中所对应的 JSON 字段的名称。
- **@JsonAlias：**在反序列化时可以让Bean的属性接**收多个json字段的名称**。
- @JsonAutoDetect：用于控制序列化和反序列化过程中Java对象中哪些属性可以被JSON格式化库处理。默认情况下，只有公有的getters才能被自动识别并进行JSON序列化。因此，若Java对象需要在序列化和反序列化过程中使用非公有的成员变量，需添加@JsonAutoDetect注解来告诉JACKSON库何时自动序列化和反序列化成员变量。
- **@JsonInclude：**是为实体类在接口序列化返回值时增加规则的注解。
    - 例如，一个接口需要**过滤掉返回值为null的字段**，即值为null的字段不返回，可以在实体类中增加如下注解 `@JsonInclude(JsonInclude.Include.NON_NULL)`
- **@JsonUnwrapped：**表明属性应该以**扁平化**的形式进行序列化，即目标属性将不会序列化为 JSON 对象，但其属性将序列化为包含它的对象的属性。
- @JsonView：两个URL，一个为获取用户详情信息包含用户的所有字段（用户名，密码），一个为获取用户信息字段（只要用户名），怎么处理呢？@JsonView可以十分方便的解决以上问题。
    - 使用接口声明多个视图
    - 在对象属性的get方法上指定视图
- @JacksonInject：在使用 JSON格式进行反序列化的时候，经常有这样一些需求。从客户端或者其他渠道获取了一个JSON格式的数据对象，该对象包含若干个属性。但是在将JSON字符串反序列化时，需要给它加上一些默认数据，比如：
    - responseTime数据响应时间，赋值为当前时间即可；
        数据反序列化的操作人，赋值为系统当前用户等
- **@JsonValue（get方法、属性字段）：一个类只能用一个**，加上这个注解时，该类的对象序列化时就会**只返回**这个字段的值做为序列化的结果。
    - 比如一个枚举类的**get方法**上加上该注解，那么在序列化这个枚举类的对象时，返回的就是枚举对象的这个属性，而不是这个枚举对象的序列化json串。
- **@JsonPropertyOrder：**用于指定实体生成 json 时的属性**顺序**，一般用得不多
- **@JsonRawValue：**能够按**原样序列化**属性。属性值**不会被转义或者加引号**（或者说，会自动去掉转义，多余的引号）。属性值已经是一个 JSON String，或者属性值已经被加了引号时很有用。
- @JsonRootName：用来指定root wrapper的名字。注意，只有当WRAP_ROOT_VALUE开启时，此注解才生效。
    - 比如：一般序列化 User是这样的格式 { "id": 1, "name": "yooo" }，但是我想添加一个root wrapper，挂载在"User"节点下，
- @JsonSerialize（用于字段或get方法上）：主要用于定制 Java 对象到 JSON 的序列化过程。通过在 Java 类或属性上使用 @JsonSerialize注解，开发者可以指定一个自定义的序列化器，从而掌握对象如何被转换为 JSON 。这种灵活性使得开发者能够更好地应对各种复杂的序列化需求。使用场景：处理敏感信息、处理日期格式、处理特殊类型数据或者逻辑
- **@JsonDeserialize（**用于字段或set方法上**）：**json反序列化注解，用于字段或set方法上，作用于setter()方法，将json数据反序列化为java对象。使用方法同@JsonSerialize类似。
- `@JsonUnwrapped`：扁平化对象。

##### 日期

- `@JsonFormat`：是一个**时间格式化**注解。用来格式化 JSON 数据。
    - 比如存储在mysql中的数据是date类型的，当读取出来封装在实体类中时，就会变成英文时间格式，而不是yyyy-MM-dd HH:mm:ss这样的中文时间，因此需要用注解来格式化时间。

```java
@JsonFormat(shape=JsonFormat.Shape.STRING, pattern="yyyy-MM-dd'T'HH:mm:ss.SSS'Z'", timezone="GMT")
private Date date;
```

##### 双向引用

`@JsonBackReference和@JsonManagedReference，以及@JsonIgnore`均是为了解决对象中存在**双向引用**导致的**无限递归**（infinite recursion）问题。这些标注均可用在**属性**或**对应的get、set方法**中。  

- **@JsonBackReference和@JsonManagedReference**：通常配对使用，用在父子关系中。
    - 在**序列化**（serialization，即将对象转换为json数据）时，
        1. `@JsonBackReference`标注的属性会**被忽略**（即结果中的json数据不包含该属性的内容），作用相当于**@JsonIgnore**。
        2. `@JsonManagedReference`标注的属性（也可以没有此注解）则会**被序列化**。
    - 但在**反序列化**（deserialization，即json数据转换为对象）时，
        1. 如果没有`@JsonManagedReference`，则不会自动注入@JsonBackReference标注的属性（被忽略的父或子）；
        2. 如果有`@JsonManagedReference`，则会自动注入@JsonBackReference标注的属性。  

- **@JsonIgnore**：用于属性上。在json序列化时将java bean中的一些**属性忽略掉**，序列化和反序列化都受影响。一般标记在属性或者方法上，返回的json数据即**不包含该属性**。
    - 直接忽略某个属性，以断开无限递归，序列化或反序列化均忽略。
    - 当然如果标注在get、set方法中，则可以分开控制，**序列化**对应的是get方法，反序列化对应的是set方法。
    - 在父子关系中，当**反序列化**时，@JsonIgnore不会**自动注入**被忽略的属性值（父或子），这是它跟`@JsonBackReference和@JsonManagedReference`最大的区别。  
- **@JsonIgnoreProperties：**此注解是**类**注解，作用是json序列化时将java bean中的一些属性忽略掉，序列化和反序列化都受影响。
    - `@JsonIgnoreProperties({"userRoles"})`：作用在类上，用于过滤掉特定字段，不返回或不解析；
- **@JsonIgnoreType：**该类作为别的类的属性时，该属性忽略序列化和反序列化。
- @JacksonAnnotationsInside：需要**自定义注解**来标注类属性，来达到批量解析类属性的目的，而且并且我们希望**被这个注解标志的属性不被 Json 序列化**（不希望返回给前端）。通常情况下会采用**@JsonIgnore**

##### Setter、Getter、Creator

- @JsonAnySetter：可以将序列化时未识别到的属性都扔进一个map里。

    @JsonAnyGetter：将map里的属性都平铺输出。

- @JsonSetter 和 @JsonGetter：@JsonSetter只能用于setter方法，@JsonGetter只能用户getter方法，只用二者中的一个标签时，与@JsonProperty作用一模一样。

    - 这样一种情景：对于一个java的pojo类的一个属性，我们希望序列化时是一个属性名，反序列化时是另一个属性名。

- @JsonCreator（静态的工厂方法、构造方法上）：加了@JsonCreator注解，该类的对象在反序列化时，就走有@JsonCreator注解的方法来反序列化，原先无参+set的过程失效。

    - 注意：反序列化时，默认会调用实体类的无参构造来实例化一个对象，然后使用setter来初始化属性值。如果我们想走别的方法来实例化对象，就需要@JsonCreator方法

#### 导入依赖

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.0</version>
</dependency>
```

#### 配置文件

这里介绍两种方式，第一种采用**配置类**的方式进行自定义Jackson的配置，例如

```java
@Configuration
public class MapperConfig {
    @Bean
    public Jackson2ObjectMapperBuilder jackson2ObjectMapperBuilder(){
        return new Jackson2ObjectMapperBuilder()
            // 配置日期格式
            .dateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"))
            // 配置时区
            .timeZone(TimeZone.getTimeZone("Asia/Shanghai"))
            // 配置序列化时包含哪些属性
            .serializationInclusion(JsonInclude.Include.NON_NULL)
            // 配置禁用一些特性
            .featuresToDisable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
            // 配置启用一些特性
            .featuresToEnable(DeserializationFeature.ACCEPT_EMPTY_STRING_AS_NULL_OBJECT);
    }
}
```

第二种，采用配置文件的方式进行自定义Jackson的配置，例如

```yaml
spring:
  jackson:
    #日期格式化
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8
    #设置空如何序列化
    default-property-inclusion: non_null    
    serialization:
      #格式化输出 
      indent_output: true
      #忽略无法转换的对象
      fail_on_empty_beans: false
    deserialization:
      #允许对象忽略json中不存在的属性
      fail_on_unknown_properties: false
    parser:
      #允许出现特殊字符和转义符
      allow_unquoted_control_chars: true
      #允许出现单引号
      allow_single_quotes: true
```

##### 自定义 Jackson 的配置大全

```yaml
spring:
  jackson:
    # 设置属性命名策略,对应jackson下PropertyNamingStrategy中的常量值，SNAKE_CASE-返回的json驼峰式转下划线，json body下划线传到后端自动转驼峰式
    property-naming-strategy: SNAKE_CASE
    # 全局设置@JsonFormat的格式pattern
    date-format: yyyy-MM-dd HH:mm:ss
    # 当地时区
    locale: zh_CN
    # 设置全局时区
    time-zone: GMT+8
    # 常用，全局设置pojo或被@JsonInclude注解的属性的序列化方式
    default-property-inclusion: NON_NULL #不为空的属性才会序列化,具体属性可看JsonInclude.Include
    
    # 常规默认,枚举类SerializationFeature中的枚举属性为key，值为boolean设置jackson序列化特性,具体key请看SerializationFeature源码
    visibility:
      #属性序列化的可见范围
      getter: non_private
      #属性反序列化的可见范围
      setter: protected_and_public
      #静态工厂方法的反序列化
      CREATOR: public_only
      #字段
      FIELD: public_only
      #布尔的序列化
      IS_GETTER: public_only
      #所有类型(即getter setter FIELD）不受影响,无意义
      NONE: public_only
      #所有类型(即getter setter FIELD）都受其影响（慎用）
      ALL: public_only
      
    serialization:
      #反序列化是否有根节点
      WRAP_ROOT_VALUE: false
      #是否使用缩进，格式化输出
      
    # 枚举类DeserializationFeature中的枚举属性为key，值为boolean设置jackson反序列化特性,具体key请看DeserializationFeature源码
    deserialization:

    # 枚举类MapperFeature中的枚举属性为key，值为boolean设置jackson ObjectMapper特性
    # ObjectMapper在jackson中负责json的读写、json与pojo的互转、json tree的互转,具体特性请看MapperFeature,常规默认即可
    mapper:

    parser:
    
    generator:
```



## Spring MVC

> 本质上相当于传统的 Servlet 和 Struts 中的 action

是一个基于 **MVC 设计模式**的轻量级 Web 开发框架，是 Spring 框架的一部分。 

MVC：`Model-View-Controler` 的简称，即模型—视图—控制器。解耦。

Spring 中通过注解来定义各层，如 `@Controller` 定义控制层、`@Service` 定义 service 业务逻辑层。

> 详情见[项目架构](#项目架构)中。

> [Spring MVC 常用基本注解](#声明 Bean 的注解)

### MVC 设计模式演化

`Model 1` 时代：

1. `Model` 数据模型层：负责数据逻辑（业务规则）的处理和实现**数据操作/存取**。相当于JavaBean。
2. `View` 视图层：负责**用户交互**（格式化数据展示给用户，并接受用户输入）、数据验证等功能。不进行任何业务逻辑处理。
3. `Controller` 控制层：负责**接受请求**并调用相应的模型去**处理请求**，然后指定相应视图来显示处理结果。

`Model 2` 时代，早期 Java Web MVC 开发模式：

1. JavaBean + JSP；
2. JavaBean + JSP + Servlet
   1. Java Bean（Model，即 dao 和 bean）
   2. JSP（View）
   3. Servlet（Controller，可在 JSP 中实现）

`Spring MVC` 时代：

1. 持久层（Dao 数据库操作 / Entity 实体类 / Model）：可整合 JdbcTemplate、Hibernate 和 MyBatis 等技术。
2. 表现层（View）：提供与 Spring MVC、Struts2 框架的整合。
3. 控制层（Controller）：
4. 业务逻辑层（Service）：`service + serviceImpl` 。处理业务逻辑；处理请求和响应事件，“事件”包括**用户的行为**和**数据的改变**；管理事务和记录日志等。

#### Spring MVC VS Struts2

1. `Spring MVC` 基于方法开发，会将 URL 信息与 Controller 类的某个方法绑定并进行映射，请求参数作为该方法的形参，生成 Handler 对象，只包含一个 method 方法；`Struts2` 基于类开发，**Action 类**中所有方法的请求参数都是**成员变量**，方法越多，类越乱。
2. `Spring MVC` 支持单例开发模式；而 `Struts2` 由于只能通过**类的成员变量**接受参数，无法用单例模式。

不同：

1. 入口不同：Struts2 为 filter 过滤器；SpringMVC 为一个 Servlet，即前端控制器；
2. 开发方式不同：Struts2 基于类开发，传递参数通过类的属性，只能设置为多例；SpringMVC 基于方法开发（一个 url 对应一个方法），请求参数传递到方法形参，可为单例也可为多例（建议单例）；
3. 请求方式不同：Struts2 值栈存储请求和响应的数据，通过OGNL存取数据；SpringMVC 通过参数解析器将 request 请求内容解析，给方法形参赋值，将数据和视图封装成 ModelAndView 对象，最后又将其中的模型数据通过 request 域传输到页面，jsp 视图解析器默认使用的是 jstl。

### 工作原理/流程说明

<img src="../assets/springmvc_work_flow.jpg" alt="img" style="zoom:100%;" />

#### 核心组件

1. **`DispatcherServlet`**：**核心的中央处理器**、**前端控制器**/分发器/**调度器**，负责接收请求、分发，并给予客户端响应。
2. **`HandlerMapping`**：**处理器映射器**，**根据 URL 去匹配**查找能处理的 `Handler` ，并会将请求涉及到的**拦截器**和 `Handler` 一起封装。
3. **`HandlerAdapter`**：**处理器适配器**，根据 `HandlerMapping` 找到的 `Handler` ，**适配执行**对应的 `Handler`；
4. **`Handler`**：**请求处理器**，处理实际请求的处理器。
5. **`ViewResolver`**：**视图解析器**，根据 `Handler` 返回的逻辑视图 / 视图 / **`ModelAndView` 对象**，解析并渲染真正的视图，并传递给 `DispatcherServlet` 响应客户端。

#### 工作原理

传统开发模式（JSP，Thymeleaf 等）的工作原理：

1. 客户端（浏览器）发送请求到 **`Di'spatcherServlet` 前端控制器**/分发器/**调度器**；
2. `DispatcherServlet` （根据请求信息）查询 **`HandlerMapping` 处理器映**射；`HandlerMapping` 根据 URL 去匹配查找能处理的 `Handler`（也就是平常说的 `Controller` 控制器） ，并会将请求涉及到的拦截器和 `Handler` 一起封装。
3. ~~返回 `Handler` 处理器对象；~~
4. 调用 **`HandlerA'dapter` 适配器**，解析请求到对应的具体 Handler（即 **Controller**）；
5. `HandlerAdapter` 适配器调用**真正的处理器** `Handler` 处理请求和相应的业务逻辑；Controller 调用 Service 业务逻辑层处理后返回结果；
6. 处理器处理完业务后，会返回一个 **`ModelAndView` 对象**（Model 是返回的数据对象，View 是逻辑上的 View）；
7. ~~返回 `ModelAndView` 对象给前端控制器；~~
8. 请求 `ViewResolver` 视图解析器解析视图，根据逻辑 View 查找实际的 View；
9. **视图渲染**：`DispaterServlet` 把返回的 Model 传给 View；
10. JSP 渲染视图；
11. 将 View 响应给浏览器。

### 前后端分离

然而现在主流的开发方式是**前后端分离**，这种情况下 Spring MVC 的 `View` 概念发生了一些变化。

由于 `View` 通常由前端框架（Vue, React 等）来处理，后端不再负责渲染页面，而是只负责提供数据，因此：

- 后端通常不再返回具体的视图，而是返回**纯数据**（通常是 JSON 格式），由前端负责渲染和展示。
- `View` 的部分往往不需要设置，Spring MVC 的控制器方法只需要返回数据，不再返回 `ModelAndView`，而是直接返回数据，Spring 会自动将其转换为 JSON 格式。相应的，`ViewResolver` 也将不再被使用。

##### @RestController

> 见 Spring Boot 文档。

`@RestController` 注解，添加在类上，是 `@Controller` 和 `@ResponseBody` 的组合注解。

- 表示直接使用接口方法的返回结果。默认情况下，使用 JSON 作为序列化方式。
- **默认会返回 JSON 格式**的数据，而不是试图解析视图。无需使用 InternalResourceViewResolver 解析视图，返回 HTML 结果。

目前主流的架构，都是 前后端分离 的架构，后端只需要提供 API 接口，仅仅返回数据。

- 而视图部分的工作，全部交给前端来做。
- 也因此，项目中 99.99% 使用 `@RestController` 注解。
- 如果使用的是 `@Controller`，可以结合 `@ResponseBody` 注解来返回 JSON。

### 接收请求参数

HTTP 请求中接收前端参数的三种常用方式

1. `HttpServletRequest`：
    - 后端用于响应的有`HttpServletResponse`。
2. `@RequestParam`：具体见下面 [REST 常用注解](#REST 常用注解)。
3. `@RequestBody`：具体见下面 [REST 常用注解](#REST 常用注解)。

取参类型区别：

- `@RequestParam` 和 `@RequestBody` 都是从 `HttpServletRequest request` 中取参的，而 `@PathVariable` 是映射 URI 请求参数中的**占位符**到目标方法的参数。

- `@RequestParam` 可以获取（Content-Type 的**默认值**） `application/x-www-form-urlencoded` 以及 `application/json` 这两种类型的参数，
- 但是 `@RequestBody` 是用来获取**非默认类型**的数据，比如 `application/json`、`application/xml` 等。

#### HttpServletRequest

##### 获取 request 对象

获取 HttpServletRequest 对象的四种方法：

1. 传参：Controller 中**加参数** `HttpServletRequest request` 来获取 request 对象；

    - 实现原理是，在Controller方法开始处理请求时，Spring会将request对象赋值到方法参数中。此时request对象相当于局部变量，毫无疑问是线程安全的。
    - 缺点：冗余太多。

2. IoC：**自动注入**（成员变量）来获取 request 对象；优点有：

    1. 注入不局限于 Controller 中，还可以在任何Bean中注入，包括Service、Repository及普通的Bean。

    2. 注入的对象不限于request，还可以注入其他`scope`为`request`或`session`的对象，如`response`对象、`session`对象等；并保证线程安全。

    3. 大大减少了代码冗余。

        ```java
        @Autowire
        HttpServletRequest request;
        
        //?
        @Autowire
        HttpSession session;
        ```

3. **基类**中自动注入（推荐）；

    ```java
    public class BaseController {
        @Autowired
        protected HttpServletRequest request;     
    }
    ```

4. 通过 `RequestContextHolder` 的静态方法获取 request 对象及 response 等相关对象。

    - 优点：可以在非Bean中直接获取。
    - `RequestContextHolder`：持有上下文的Request容器。

    ```java
    //获取当前请求对象
    ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
    HttpServletRequest request = attributes.getRequest();
    
    //or
    @SuppressWarnings("PatternVariableCanBeUsed")
    public static HttpServletRequest getRequest() {
        RequestAttributes requestAttrs = RequestContextHolder.getRequestAttributes();
        if (!(requestAttrs instanceof ServletRequestAttributes)) {
            return null;
        }
        ServletRequestAttributes servletRequestAttrs = (ServletRequestAttributes) requestAttrs;
        return servletRequestAttrs.getRequest();
    }
    ```

##### 使用 request 对象

`HttpServletRequest` 接口可以获得的参数更多，可以获得客户端的**请求行和请求头、请求体**信息。

当访问Servlet时，会在请求消息的请求行中，包含请求方法、请求资源名、请求路径等信息，为了获取这些信息，定义了一系列用于获取请求行的方法。

1. 获得请求行：`String getMethod()` GET/POST等。
2. 获得**请求头**：`String getHeader(String name)`等。
3. 获得请求体：`String getParameter(String name)`等。
4. `getServletPath()`
5. 获取 Session：`getSession()`。

```java
Public static void login(User user, HttpServletRequest request) {
    HttpSession session = request.getSession();
    Session.setAttribute(User, user);
}

/**
  * 获得租户编号，从 header 中
  * 考虑到其它 framework 组件也会使用到租户编号，所以不得不放在 WebFrameworkUtils 统一提供
  * @param request 请求
  * @return 租户编号
  */
public static Long getTenantId(HttpServletRequest request) {
    String tenantId = request.getHeader(HEADER_TENANT_ID);
    return NumberUtil.isNumber(tenantId) ? Long.valueOf(tenantId) : null;
}

/**
     * 获得当前用户的类型
     * 注意：该方法仅限于 web 相关的 framework 组件使用！！！
     *
     * @param request 请求
     * @return 用户编号
     */
public static Integer getLoginUserType(HttpServletRequest request) {
    if (request == null) {
        return null;
    }
    // 1. 优先，从 Attribute 中获取
    Integer userType = (Integer) request.getAttribute(REQUEST_ATTRIBUTE_LOGIN_USER_TYPE);
    if (userType != null) {
        return userType;
    }
    // 2. 其次，基于 URL 前缀的约定
    if (request.getServletPath().startsWith(properties.getAdminApi().getPrefix())) {
        return UserTypeEnum.ADMIN.getValue();
    }
    if (request.getServletPath().startsWith(properties.getAppApi().getPrefix())) {
        return UserTypeEnum.MEMBER.getValue();
    }
    return null;
}
```

#### HttpServletResponse

`HttpServletResponse`：服务端处理完Http的请求后，根据HttpServletResponse对象将处理结果作为Http响应返回给客户端。

#### HttpHeaders

是一个用于表示HTTP请求或响应头的类，~~属于 `java.net.http` 包，从Java 11开始引入~~。

- 这个类提供了一种方便的方式来操作**HTTP消息头**，包括添加、删除和获取头字段的值。
- 是Spring框架中的一个常用类，用于表示**HTTP请求或响应中**的头部信息。提供了一系列方法来操作头部信息，例如获取所有头部信息、获取特定头部信息、添加头部信息等。
- 在Spring框架中，HttpHeaders类通常位于org.springframework.http包中 。

以下是 HttpHeaders 类的一些关键特性和用法：

- 不可变性：HttpHeaders 实例是不可变的，这意味着一旦创建，就不能更改其内容。这有助于确保线程安全性。
- 构建器模式：HttpHeaders 的实例是通过构建器模式创建的，这允许你以一种灵活和链式的方式设置头字段。
- 头字段操作：可以使用 set、add、remove 和 clear 等方法来操作头字段。
- 遍历头字段：可以通过迭代器或使用 names 和 values 方法来遍历头字段。
- 获取头字段值：可以使用 firstValue 或 allValues 方法来获取头字段的值。
- 条件请求：HttpHeaders 类还支持创建条件请求，例如 If-Modified-Since 和 If-None-Match。

```java
 /**
     * 快递 100 API 请求
     *
     * @param url 请求 url
     * @param req 对应请求的请求参数
     * @param respClass 对应请求的响应 class
     * @param <Req> 每个请求的请求结构 Req DTO
     * @param <Resp> 每个请求的响应结构 Resp DTO
     */
private <Req, Resp> Resp httpRequest(String url, Req req, Class<Resp> respClass) {
    // 请求头
    HttpHeaders headers = new HttpHeaders();
    headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
    // 请求体
    String param = JsonUtils.toJsonString(req);
    String sign = generateReqSign(param, config.getKey(), config.getCustomer()); // 签名
    MultiValueMap<String, String> requestBody = new LinkedMultiValueMap<>();
    requestBody.add("customer", config.getCustomer());
    requestBody.add("sign", sign);
    requestBody.add("param", param);
    log.debug("[httpRequest][请求参数({})]", requestBody);

    // 发送请求
    HttpEntity<MultiValueMap<String, String>> requestEntity = new HttpEntity<>(requestBody, headers);
    ResponseEntity<String> responseEntity = restTemplate.exchange(url, HttpMethod.POST, requestEntity, String.class);
    log.debug("[httpRequest][的响应结果({})]", responseEntity);
    // 处理响应
    if (!responseEntity.getStatusCode().is2xxSuccessful()) {
        throw exception(EXPRESS_API_QUERY_ERROR);
    }
    return JsonUtils.parseObject(responseEntity.getBody(), respClass);
}
```

#### REST 常用注解

常用于 Spring MVC 的 [Controller 层](#Controler控制层)。

- `REST`：代表着**抽象状态转移**，根据 HTTP 协议从客户端发送数据到服务端，如：服务端的一本书以 XML 或 JSON 格式传递到客户端；常用于 Controller 控制器。
- `REST API`：利用 HTTP 中 get、post、put/patch、delete 及其他 HTTP 方法构成 REST 中数据资源的增删改查操作。

##### @RequestMapping VS @GetMapping

都用于处理常见的 HTTP 请求类型：

- `@RequestMapping(value = "/user", method = RequestMethod.GET, produces = "application/json; charset=utf-8")`：可标注在 Controller 控制器**类上**和方法上，用于映射 HTTP 请求。
    1. `produces`属性：指定返回值类型和字符编码；`consumes`属性：指定处理请求的提交内容类型（Content-Type），例如 `application/json, text/html, image/jpeg`。
    2. 标注在 Controller 控制器类上时：会应用到控制器的所有方法上；一般用于**配置 URI 请求前缀**，即该 Controller 下的所有请求都需加上此前缀，请求方式可用方法上的注解指定；
    3. 标注在请求方法上时：二者等价。
- `@GetMapping("/user")/@PostMapping()@PutMapping()/@DeleteMapping()/@PatchMapping()`：仅可标注在方法上。

##### @RequestParam VS @PathVariable

都用于前后端传值：

- `@RequestParam` ：用于从 URL **获取查询参数**。最适合 Web 应用程序。如果 URL 中不存在查询参数，则可指定默认值。（`/teachers?type=web`）
    1. `defaultValue `：如果本次请求没有携带这个参数，或参数为空，那么就会启用默认值；
    2. `name/value`：绑定本次参数的名称，要跟URL上面的一样；value 是name属性的一个别名:
    3. `required `：这个参数不是必须的，如果为 true，不传参数**会报错**。
- `@PathVariable`：用于从 URI 中**获取路径参数**。最适合 RESTful Web 服务。可在一个方法中定义多个。（`/klasses/123456/`）
    1. `name/value`：绑定参数的名称，**默认不传递时，绑定为同名的形参。** 赋值但名称不一致时则报错；
    2. `required`：这个参数不是必须的，如果为 true，不传参数会报错。
    3. 使用时需要注意两点：参数接收类型使用基本类型；如果标明参数名称，则参数名称必须和URL中参数名称一致。

区别：

1. 都用于提取方法参数；都用于 GET、POST请求；
2. 获取参数值的方式不同，`@RequestParam` 从请求携带的**参数**中获取参数（`/teachers?type=web`），而 `@PathVariable` 从**请求的 URI** 中获取（`/klasses/123456/`）。

```java
// 如果请求的 url 是：/klasses/123456/teachers?type=web
// 则获取到的数据就是：klassId=123456, type=web
@GetMapping("/klasses/{klassId}/teachers")
public List<Teacher> getKlassRelatedTeachers(
	@PathVariable("klassId") Long klassId,
	@RequestParam(value = "type", defaultValue = "10000", required = false) String klassType ) {
	...
}
```

###### 不使用 @PathVariable

对于 SpringMVC 提供的 `@PathVariable` 路径参数，并没有在项目中使用，主要原因如下：

1. 封装的权限框架，基于 URL 作为**权限标识**，暂时是不支持带有路径参数的 URL 。
2. 基于 URL 进行告警，而带有路径参数的 URL ，“相同” URL 实际对应的是不同的 URL ，导致无法很方便的实现按照单位时间**请求错误次数**告警。
3. `@PathVariable` 路径参数的 URL ，会带来一定的 SpringMVC 的**性能下滑**。具体可以看看 [《SpringMVC RESTful 性能优化》](https://tech.imdada.cn/2015/12/23/springmvc-restful-optimize/) 文章。

##### @RequestBody

`@RequestBody`：将传入的 **HTTP 请求体**与注解的方法参数中的**对象**绑定， 要求传递一个 `JSON` 格式的字符串。响应体另见[@ResponseBody](#@Controller)。

- 不能用于GET请求；
- 用于读取 Request 请求的 body 部分、`Content-Type` 为 `application/json` 格式的数据，接收到数据后会自动将数据**绑定到 Java 对象**上去。系统会将请求的 body 中的 json 字符串转换为 Java 对象。

使用 @RequestBody 需要满足如下条件：

1. Content-Type 为 application/json，确保传递的是 JSON 数据；
2. 参数转化的配置必须统一，否则无法接收数据，比如 json、request 混用等。

```java
@ApiOperation(value = "登录以后返回token")
@RequestMapping(value = "/login", method = RequestMethod.POST)
@ResponseBody
public CommonResult login(@Validated @RequestBody UmsAdminLoginParam umsAdminLoginParam) {
	String token = adminService.login(umsAdminLoginParam.getUsername(), umsAdminLoginParam.getPassword());
	...
}
```

##### @RequestParam VS @RequestBody

一个请求方法只可有一个 `@RequestBody`，在一个请求中只能用一次；但可有多个 `@RequestParam` 和 `@PathVariable`。

前端在不明确指出 `Content-Type` 时，默认为 `application/x-www-form-urlencoded` 格式，在这种格式下，后端：

- 使用 `@RequestParam` 可以直接获取指定的参数，可以获取默认以及 `application/json` 这两种类型的参数，但是一旦前端传递的是 JSON 数据，那么使用 @RequestParam 取不到值，还报错。

- 使用 `@RequestBody` 是用来获取**非默认类型**的数据，比如 `application/json`、`application/xml` 等。

##### @RequestXxx 系列

1. `@RequestHeader`：用于获取有关 HTTP 请求头的详细信息。将此注解用作**方法参数**。注解的可选元素是**名称，必填，值，defaultValue。** 可在一种方法中多次使用。
2. `@RequestAttribute`：将方法参数绑定到请求属性。提供了从控制器方法方便地访问请求属性的方法。可访问服务器端填充的对象。

##### @Required

`@Required`：用于 Bean 设置方法。表示在配置时用**必需的属性**填充带注解的 Bean，否则将引发异常 `BeanInitilizationException` 。

##### @CrossOrigin

> 见计算机网络文档

用于解决跨域问题。

##### @ModelAttribute

作用是将请求参数绑定到**Model对象**。

- 被@ModelAttribute注释的方法会在Controller每个方法执行前被执行（如果在一个Controller映射到多个URL时，要谨慎使用）。

 如，获取POST请求的FORM表单数据

```html
<!--jsp页面-->
<form action ="<%=request.getContextPath()%>/demo/addUser5" method="post">
    用户名:<input type="text" name="username"/><br/>
    密码:<input type="password" name="password"/><br/>
    <input type="submit" value="提交"/> <input type="reset" value="重置"/>
</form>
```

Controller 中：

```java
/**
* 5、使用@ModelAttribute注解获取POST请求的FORM表单数据
* @param user
* @return
*/
@RequestMapping(value="/addUser5", method=RequestMethod.POST)
public String addUser5(@ModelAttribute("user") UserModel user) {
    System.out.println("username is:"+user.getUsername());
    System.out.println("password is:"+user.getPassword());
    return "demo/index";
}
```

##### 5 种常见的请求类型

1. `GET`：请求从服务器获取特定资源，如 GET /users（获取所有学生）；
2. `POST`：在服务器上创建一个新的资源，如 POST /users（创建学生）；
3. `PUT`：更新服务器上的资源，如 PUT /users/12（更新编号为 12 的学生）；
4. `DELETE`：从服务器删除指定的资源，如 DELETE /users/12（删除编号为 12 的学生）；
5. `PATCH`：更新服务器上的资源（可看作是**部分更新**），较少用。

```java
@GetMapping("/users")
//<==>@RequestMapping(value = "/users", method = RequestMethod.GET)
public ResponseEntity<List<User> > getAllUsers() {
	return userRepository.findAll();
}

@PostMapping("/users")
public ResponseEntity<User> createUser(@Valid @RequestBody UserCreateRequest userCreateRequest) {
	return userRespository.save(userCreateRequest);
}

@PutMapping("/users/{userId}")
public ResponseEntity<User> updateUser(@PathVariable(value = "userId") Long userId, @Valid @RequestBody UserUpdateRequest userUpdateRequest) {
  ......
}

@DeleteMapping("/users/{userId}")
public ResponseEntity deleteUser(@PathVariable(value = "userId") Long userId){
  ......
}

@PatchMapping("/profile")
public ResponseEntity updateStudent(@RequestBody StudentUpdateRequest stuUpdateRequest) {
	stuRepository.updateDetail(stuUpdateRequest);
	return ResponseEntity.ok().build();
}
```

### 参数校验、统一响应和异常

> 参考参数校验、统一响应和异常处理文档

### 业务流程举例

> 介绍项目时，用一个业务流程说说 spring mvc 如何做的。

### 测试接口

方法有：

1. Postman、
2. CURL
3. 浏览器，手工模拟请求后端 API 接口。
4. IDEA HTTP Client 插件 .http 模拟请求
5. @Test 单元测试

### AJAX

SpringMVC 中的转发和重定向：

- 转发： `return: "hello" `

- 重定向 ：`return: "redirect:hello.jsp"`

通过 **JackSon 框架**把 Java 里的对象直接转换成 js 可识别的 JSON 对象，在接受 AJAX 方法里直接返回Object，List 等，方法前需要加上注解 `@ResponseBody`。

### Axios

前端 HTTP 框架



## Hutool 工具类

Hutool 是一个小而全的 Java工具类库，通过**静态方法封装**，降低相关API的学习成本，提高工作效率，使Java拥有函数式语言般的优雅，让Java语言也可以“甜甜的”。

- 是项目中**“util”包**友好的替代，节省了开发人员对项目中**常用工具类和方法**的封装时间，使开发专注于业务，同时可以最大限度的避免封装不完善带来的bug

- 该工具类主要对 文件、流、加密解密、转码、正则、线程、XML等 JDK方法进行封装，组成各种Util工具类，同时提供以下组件：
- [官方参考文档](https://hutool.cn/docs/#/)

### 包含组件

一个Java基础工具类，对文件、流、加密解密、转码、正则、线程、XML等JDK方法进行封装，组成各种Util工具类，同时提供以下组件：

> 可以根据需求对每个模块单独引入，也可以通过引入`hutool-all`方式引入所有模块。

| 模块               | 介绍                                                         |
| ------------------ | ------------------------------------------------------------ |
| hutool-aop         | JDK动态代理封装，提供非IOC下的**切面**支持                   |
| hutool-bloomFilter | **布隆过滤**，提供一些Hash算法的布隆过滤                     |
| hutool-cache       | 简单缓存实现                                                 |
| **hutool-core**    | 核心，包括**Bean操作、日期、各种Util**等                     |
| hutool-cron        | 定时任务模块，提供类Crontab表达式的定时任务                  |
| hutool-crypto      | **加密解密**模块，提供对称、非对称和摘要算法封装             |
| hutool-db          | JDBC封装后的数据操作，基于ActiveRecord思想                   |
| hutool-dfa         | 基于DFA模型的多关键字查找                                    |
| hutool-extra       | 扩展模块，对第三方封装（模板引擎、邮件、Servlet、二维码、Emoji、FTP、分词等） |
| hutool-http        | 基于HttpUrlConnection的Http客户端封装                        |
| hutool-log         | 自动识别日志实现的日志门面                                   |
| hutool-script      | 脚本执行封装，例如Javascript                                 |
| hutool-setting     | 功能更强大的Setting配置文件和Properties封装                  |
| hutool-system      | 系统参数调用封装（JVM信息等）                                |
| hutool-json        | JSON实现                                                     |
| hutool-captcha     | 图片验证码实现                                               |
| hutool-poi         | 针对POI中Excel和Word的封装                                   |
| hutool-socket      | 基于Java的NIO和AIO的Socket封装                               |
| hutool-jwt         | JSON Web Token (JWT)封装实现                                 |

#### 依赖

【Hutool 5.x 支持 JDK8+，JDK7 使用 Hutool 4.x版本】

```xml
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>4.6.3</version>
</dependency>
```



### ConvertUtil

[类型转换工具类-Convert](https://www.bookstack.cn/read/hutool/52e25ad985b3cc80.md)：封装了针对Java常见类型的转换，用于简化类型转换。

- 常见类型转换：大部分方法为toXXX，参数为Object，可以实现将任意可能的类型转换为指定类型。同时支持第二个参数**defaultValue**用于在转换失败时返回一个默认值。
    - `Convert.toLong(id)`
- `Convert.convert(Class<T>, Object)`方法：可以将任意类型转换为指定类型，Hutool中预定义了许多类型转换，例如转换为URI、URL、Calendar等等，
- 半角和全角转换、16进制（Hex）、Unicode和字符串转换、编码转换、时间单位转换、金额大小写转换、原始类和包装类转换。

### JavaBean 和 Class

该类别主要对 对象、类、类加载、类的类型判断、反射 等相关功能进行了封装（ [API 文档](https://apidoc.gitee.com/dromara/hutool/) ）

- ResourceUtils：
- `ClassPathResource` ：获取 classPath 下的文件，在 Tomcat 等容器下，classPath 一般是WEB-INF/classes；用于日志。

- ClassLoaderUtil：ClassLoader 工具类
- ClassUtil：类工具类
- `ObjectUtil`：对象工具类
- TypeUtil：针对Type的工具类封装
- `ReflectUtil`：反射工具类
- `ReferenceUtil`： 引用工具类
- JAXBUtil：JAXB（Java Architecture for XML Binding），根据XML Schema产生Java对象
- SerializeUtil：类序列化工具类
- ModifierUtil：类修饰符工具类
- ServiceLoaderUtil：SPI机制中的服务加载工具类
- [泛型类型工具-TypeUtil](https://www.bookstack.cn/read/hutool/567f1944215621a9.md)

介绍链接：[Java - HuTool 使用 ReflectUtil、ClassUtil等工具类（二）](https://blog.csdn.net/weixin_42272869/article/details/124809368?csdn_share_tail={"type"%3A"blog"%2C"rType"%3A"article"%2C"rId"%3A"124809368"%2C"source"%3A"weixin_42272869"}&ctrtid=4Rj2n)

```java
/**
     * 获得自身的代理对象，解决 AOP 生效问题
     *
     * @return 自己
     */
    private TradeOrderQueryServiceImpl getSelf() {
        return SpringUtil.getBean(getClass());
    }
```

##### BeanUtil

`BeanUtil` JavaBean 工具类：提供对**Java反射**和自省API的包装。主要目的是利用反射机制对JavaBean的属性进行处理。用于 Map 与 JavaBean 对象的互相转换及对象属性的拷贝。

1. `BeanUtil.copyProperties(Object source, Object target)`：将 source 中的值赋给 target，名称不同的属性不进行处理，需手动处理。
    1. 如 构造入参 PO；
    2. 如 浅拷贝方法 `BeanUtils.copyProperties(umsAdminParam, umsAdmin);`，Service 中将 DTO 对象 `UmsAdminParm `赋值给 DO 实体类对象 `UmsAdmin`。
    3. 支持浅拷贝或深拷贝。
2. `toBean()`：此处并不是传Bean对象，而是Bean类，Hutool会自动调用**默认构造方法**创建对象。

```java
FileDO file = BeanUtil.toBean(createReqVO, FileDO.class);

@Override
@TenantIgnore // 访问令牌校验时，无需传递租户编号；主要解决上传文件的场景，前端不会传递 tenant-id
public CommonResult<OAuth2AccessTokenCheckRespDTO> checkAccessToken(String accessToken) {
    OAuth2AccessTokenDO accessTokenDO = oauth2TokenService.checkAccessToken(accessToken);
    return success(BeanUtils.toBean(accessTokenDO, OAuth2AccessTokenCheckRespDTO.class));
}
```

##### ObjectUtil

- `isAllNotEmpty(values)`：
- `defaultIfNull(levelId, 0L)`：

```java
ObjectUtil.notEqual(orderItem.getUserId(), userId);

// 校验是否是活动订单
if (ObjUtil.isAllEmpty(seckillActivityId, combinationActivityId, combinationHeadId, bargainRecordId)) {
    return true;
}
```

##### ReflectUtil

```java
// 执行方法
ReflectUtil.invoke(user, "getEmail")
```

#### 文件 / 路径 / 系统相关

包括像文件中内容转义、XML文件处理、URL处理、系统属性相关工具类

- EscapeUtil：转义和反转义工具类
- XmlUtil：XML工具类
- HashUtil：Hash算法工具类
- SystemPropsUtil：系统属性工具
- URLUtil：URL统一资源定位符相关工具类
- PageUtil：分页工具类
- RuntimeUtil：系统运行时工具类

介绍链接：[Java - HuTool 使用 EscapeUtil、XmlUtil等工具类（四）](https://blog.csdn.net/weixin_42272869/article/details/124830106?csdn_share_tail={"type"%3A"blog"%2C"rType"%3A"article"%2C"rId"%3A"124830106"%2C"source"%3A"weixin_42272869"}&ctrtid=01vBF)

### 数据类型

#### 基本类型 转换

- BooleanUtil ：Boolean 类型相关工具类
- ByteUtil：Byte 类型相关工具类
- CharUtil：字符工具类
- [EnumUtil](https://www.bookstack.cn/read/hutool/dad076e947f5795b.md)：枚举工具类

#### 数字

- NumberUtil：数字工具类
- CharsetUtil：字符集工具类
- HexUtil：十六进制工具类
- RadixUtil：进制转换工具类
- [随机工具-RandomUtil](https://www.bookstack.cn/read/hutool/377f64112be7197a.md)
    - `RandomUtil.randomString(CHARACTERS, length)`
    - `RandomUtil.randomNumbers(6)`

```java
/**
     * 生成指定长度的随机字符串
     *
     * @param length 长度
     * @return {@link String}
     */
    public static String generateRandomText(int length) {
        return RandomUtil.randomString(CHARACTERS, length);
    }

/**
     * 计算百分比金额
     *
     * @param price        金额
     * @param rate         百分比，例如说 56.77% 则传入 56.77
     * @param scale        保留小数位数
     * @param roundingMode 舍入模式
     */
public static BigDecimal calculateRatePrice(Number price, Number rate, int scale, RoundingMode roundingMode) {
    return NumberUtil.toBigDecimal(price)
        .multiply(NumberUtil.toBigDecimal(rate)) // 乘以
        .divide(BigDecimal.valueOf(100), scale, roundingMode); // 除以 100
}
```

#### 日期时间

`DateUtil` 日期时间工具类：定义了一些常用的日期时间操作方法；

[日期时间](https://www.bookstack.cn/read/hutool/date.md) DateUtil：提供针对JDK中Date和Calendar对象的封装

- **[`DateUtil` ](https://www.bookstack.cn/read/hutool/8168b022b2c31abe.md)**针对日期时间操作提供一系列静态方法。封装主要是日期和字符串之间的**转换**，以及提供对日期的定位（一个月前等）。
- `DateTime` 提供类似于Joda-Time中日期时间对象的封装。对于Date对象，为了便捷，使用了一个**DateTime类**来代替之，继承自Date对象，
    - 主要的便利在于，覆盖了toString()方法，返回yyyy-MM-dd HH:mm:ss形式的字符串，方便在输出时的调用（例如日志记录等），提供了众多便捷的方法对日期对象操作。

##### LocalDateTimeUtil

从Hutool的5.4.x开始，Hutool加入了针对JDK8+日期API的封装，此工具类的功能包括`LocalDateTime`和`LocalDate`的解析、格式化、转换等操作。

详细接口参考：[时间工具类-- LocalDateTimeUtil (修正版)](https://blog.csdn.net/qq_41463655/article/details/106007147)

###### between

between(LocalDateTime.now(), accessTokenDO.getExpiresTime(), ChronoUnit.SECONDS)：返回

```java
public void set(OAuth2AccessTokenDO accessTokenDO) {
    String redisKey = formatKey(accessTokenDO.getAccessToken());
    // 清理多余字段，避免缓存
    accessTokenDO.setUpdater(null).setUpdateTime(null).setCreateTime(null).setCreator(null).setDeleted(null);
    long time = LocalDateTimeUtil.between(LocalDateTime.now(), accessTokenDO.getExpiresTime(), ChronoUnit.SECONDS);
    if (time > 0) {
        stringRedisTemplate.opsForValue().set(redisKey, JsonUtils.toJsonString(accessTokenDO), time, TimeUnit.SECONDS);
    }
}
```

##### 日期转换

```java
String dateStr = "2020-01-23T12:23:56";
DateTime dt = DateUtil.parse(dateStr);

// Date对象转换为LocalDateTime
LocalDateTime of = LocalDateTimeUtil.of(dt);

// 时间戳转换为LocalDateTime
of = LocalDateTimeUtil.ofUTC(dt.getTime());
```

##### 日期字符串解析

```java
// 解析ISO时间
LocalDateTime localDateTime = LocalDateTimeUtil.parse("2020-01-23T12:23:56");

// 解析自定义格式时间
localDateTime = LocalDateTimeUtil.parse("2020-01-23", DatePattern.NORM_DATE_PATTERN);
```

解析同样支持`LocalDate`：

```java
LocalDate localDate = LocalDateTimeUtil.parseDate("2020-01-23");

// 解析日期时间为LocalDate，时间部分舍弃
localDate = LocalDateTimeUtil.parseDate("2020-01-23T12:23:56", DateTimeFormatter.ISO_DATE_TIME);
```

##### 日期格式化

```java
LocalDateTime localDateTime = LocalDateTimeUtil.parse("2020-01-23T12:23:56");

// "2020-01-23 12:23:56"
String format = LocalDateTimeUtil.format(localDateTime, DatePattern.NORM_DATETIME_PATTERN);
```

##### 日期偏移

```java
final LocalDateTime localDateTime = LocalDateTimeUtil.parse("2020-01-23T12:23:56");

// 增加一天
// "2020-01-24T12:23:56"
LocalDateTime offset = LocalDateTimeUtil.offset(localDateTime, 1, ChronoUnit.DAYS);
```

如果是减少时间，offset第二个参数传负数即可：

```java
// "2020-01-22T12:23:56"
offset = LocalDateTimeUtil.offset(localDateTime, -1, ChronoUnit.DAYS);
```

##### 计算时间间隔

```java
LocalDateTime start = LocalDateTimeUtil.parse("2019-02-02T00:00:00");
LocalDateTime end = LocalDateTimeUtil.parse("2020-02-02T00:00:00");

Duration between = LocalDateTimeUtil.between(start, end);

// 365
between.toDays();
```

##### 一天的开始和结束

```java
LocalDateTime localDateTime = LocalDateTimeUtil.parse("2020-01-23T12:23:56");

// "2020-01-23T00:00"
LocalDateTime beginOfDay = LocalDateTimeUtil.beginOfDay(localDateTime);

// "2020-01-23T23:59:59.999999999"
LocalDateTime endOfDay = LocalDateTimeUtil.endOfDay(localDateTime);
```

#### StrUtil 字符串

`StrUtil` 字符串工具类：定义了一些常用的字符串操作方法；

##### `hasText(val)`

有文本。

##### hasBlank、hasEmpty

如果**一旦有空的**就返回true，常用于判断好多字段是否有空的（例如web表单数据）。

- 区别是`hasEmpty`只判断是否为null或者空字符串（""），`hasBlank`则会把不可见字符也算做空，`isEmpty`和`isBlank`同理。

```java
if (StrUtil.isBlank(orderItem.getPicUrl())) {
    orderItem.setPicUrl(spu.getPicUrl());
}
```

#####  removePrefix、removeSuffix

去掉字符串的前缀后缀，例如文件名的扩展名。

- 还有忽略大小写的`removePrefixIgnoreCase`和`removeSuffixIgnoreCase`。

```java
String fileName = StrUtil.removeSuffix("pretty_girl.jpg", ".jpg")  //fileName -> pretty_girl
```

##### sub() 方法

String 的 subString方法**越界**啥的都会报异常，还得自己判断。

- 把各种情况判断都加进来了，而且index的位置还支持负数，-1表示最后一个字符（这个思想来自于[Python](https://www.python.org/)），
- 还有就是如果不小心把第一个位置和第二个位置搞反了，也会自动修正（例如想截取第4个和第2个字符之间的部分也是可以）

```java
String str = "abcdefgh";
String strSub1 = StrUtil.sub(str, 2, 3);  //strSub1 -> c
String strSub2 = StrUtil.sub(str, 2, -3); //strSub2 -> cde
String strSub3 = StrUtil.sub(str, 3, 2);  //strSub2 -> c
```

##### format()

```java
public static String formatPrice(Integer price) {
    return String.format("%.2f", price / 100d);
}

StrUtil.format("砍价活动：省 {} 元", TradePriceCalculatorHelper.formatPrice(discountPrice));
```

### 集合容器

#### [ArrayUtil](https://www.bookstack.cn/read/hutool/50db4cabc87b5968.md) 数组工具类

- `ArrayUtil.firstMatch()`

```java
public static SmsSceneEnum getCodeByScene(Integer scene) {
    return ArrayUtil.firstMatch(sceneEnum -> sceneEnum.getScene().equals(scene),
                                values());
}
```

##### PrimitiveArrayUtil

> 原始类型数组工具类

#### [CollUtil](https://www.bookstack.cn/read/hutool/85a7389837bd401f.md) 集合工具

定义了一些常用的集合操作；

- `CollUtil.isNotEmpty(list)`
- `CollUtil.toList()`
- `CollUtil.containsAny(coll1, coll2)`：若 coll2 有 coll1，则返回 true。交集
- ListUtil.empty()：相当于 new 空的 List。

```java
//如果有交集，说明有权限
if (CollUtil.containsAny(menuRoleIds, roleIds)) {
    return true;
}

// 2.2 满减送活动
RewardActivityMatchRespDTO rewardActivity = 
    CollUtil.findOne(rewardActivityMap,
                     activity -> CollUtil.contains(activity.getSpuIds(), spuId));
spuVO.setRewardActivity(BeanUtils.toBean(rewardActivity,
                  	 AppTradeProductSettlementRespVO.RewardActivity.class));

List<ArticleDO> articles = articleMapper.selectListByTitle(title);
return CollUtil.getLast(articles); // 返回列表中的最后一个元素
```

##### CollectionUtils

- convertMap：

```java
@Override
public void validateCategoryList(Collection<Long> ids) {
    if (CollUtil.isEmpty(ids)) {
        return;
    }
    // 获得商品分类信息
    List<ProductCategoryDO> list = productCategoryMapper.selectByIds(ids);
    // for forEach
    Map<Long, ProductCategoryDO> categoryMap = CollectionUtils.convertMap(list, ProductCategoryDO::getId);
    // 校验
    ids.forEach(id -> {
        // 校验分类是否存在
        ProductCategoryDO category = categoryMap.get(id);
        ...
    });
}
```

#### [ListUtil](https://plus.hutool.cn/apidocs/cn/hutool/core/collection/ListUtil.html)



#### **[MapUtil](https://www.bookstack.cn/read/hutool/fa3d273651700cb0.md) 工具**

用于创建 Map 对象及判空；

> 介绍链接: [Java - HuTool 使用 ArrayUtil、StrUtil等工具类（一）](https://blog.csdn.net/weixin_42272869/article/details/124754171?spm=1001.2014.3001.5501)

- getLong()

```java
Long count = MapUtil.getLong(map, "count", 0L);
```

### 信息验证

在项目开发过程中，会涉及到一些信息的有效性验证，例如电话号码、身份证号、社会信用代码，对身份敏感的数据需要脱敏等，下面的工具类就是我们常用的一些工具类的封装

- PhoneUtil：电话号码工具类
- `Validator.isEmail(userMail)`
- IdcardUtil：身份证相关工具类
- IdUtil：**唯一ID**生成器工具类
- ReUtil：正则相关工具类
- ZipUtil：压缩工具类
- CreditCodeUtil：统一社会信用代码（GB32100-2015）工具类
- CoordinateUtil：坐标系转换相关工具类
- DesensitizedUtil：脱敏工具类
- [网络工具-NetUtil](https://www.bookstack.cn/read/hutool/bd698900f58f95bf.md)
- [图形验证码（Hutool-captcha）](https://www.bookstack.cn/read/hutool/773ebbbf4d030326.md)：

介绍链接：[Java - HuTool 使用 PhoneUtil、ReUtil等工具类（三）](https://blog.csdn.net/weixin_42272869/article/details/124822098?csdn_share_tail={"type"%3A"blog"%2C"rType"%3A"article"%2C"rId"%3A"124822098"%2C"source"%3A"weixin_42272869"}&ctrtid=IWQ9X)

#### IdUtil

> 见分布式、微服务框架文档。

唯一ID生成器的工具类，涵盖了：

1. UUID
2. ObjectId（MongoDB）
3. Snowflake（Twitter）

```java
// 生成密码
String password = IdUtil.fastSimpleUUID();
```

### 文件格式

`JSONUtil`：

- 常用方式：`String json1 = JSONUtil.toJsonStr(实体类);`：将 Java 对象转为 JSON。
- 较原始且不好用：`String json2 = JSONUtil.parse(实体类).toJSONString(0);`

```java
String callbackData = BinaryUtil.toBase64String(JSONUtil.parse(callback).toString().getBytes("utf-8"));

// 将redis返回的 String value 格式化为 DO 对象
JsonUtils.parseObject(stringRedisTemplate.opsForValue().get(redisKey), OAuth2AccessTokenDO.class);
```



[XML工具-XmlUtil](https://www.bookstack.cn/read/hutool/e41e0b0a699544fb.md)

[压缩工具-ZipUtil](https://www.bookstack.cn/read/hutool/a56da94bbb16617b.md)

[IO流相关](https://www.bookstack.cn/read/hutool/io.md)

### 未经过分类

[工具类](https://www.bookstack.cn/read/hutool/tools.md)：此包中为未经过分类的一些工具类。

- ~~[Escape工具-EscapeUtil](https://www.bookstack.cn/read/hutool/b91f19c931503e63.md)~~
- [Hash算法-HashUtil](https://www.bookstack.cn/read/hutool/4e2278edfa70fef4.md)
- [URL工具-URLUtil](https://www.bookstack.cn/read/hutool/5122006c1ce039fe.md)
- [分页工具-PageUtil](https://www.bookstack.cn/read/hutool/eaafedba59607164.md)
- [剪贴板工具-ClipboardUtil](https://www.bookstack.cn/read/hutool/d923a0a14060def3.md)
- [命令行工具-RuntimeUtil](https://www.bookstack.cn/read/hutool/936a34d4b083c6ec.md)
- [正则工具-ReUtil](https://www.bookstack.cn/read/hutool/1316caf9fe4aa391.md)

- [语言特性](https://www.bookstack.cn/read/hutool/feature.md)
- [Codec编码](https://www.bookstack.cn/read/hutool/codec.md)
- [文本操作](https://www.bookstack.cn/read/hutool/text.md)
- [注解工具-AnnotationUtil](https://www.bookstack.cn/read/hutool/6db01cdbb910c883.md)
- [比较器](https://www.bookstack.cn/read/hutool/dc6a3e5f8e5ae899.md)
- [异常](https://www.bookstack.cn/read/hutool/exceptions.md)
- [数学相关-MathUtil](https://www.bookstack.cn/read/hutool/5f276f91bbc1d925.md)
- [线程和并发 - 线程工具-ThreadUtil](https://www.bookstack.cn/read/hutool/e46eae82b27baa7c.md)
- [图片](https://www.bookstack.cn/read/hutool/image.md)

#### 其它组件

- [配置文件(Hutool-setting）](https://www.bookstack.cn/read/hutool/setting.md)
- [日志（Hutool-log）](https://www.bookstack.cn/read/hutool/logs.md)
- [缓存（Hutool-cache）](https://www.bookstack.cn/read/hutool/cache.md)
- [加密解密（Hutool-crypto）](https://www.bookstack.cn/read/hutool/crypto.md)：
- [DFA查找（Hutool-dfa）](https://www.bookstack.cn/read/hutool/dfa.md)
- [数据库（Hutool-db）](https://www.bookstack.cn/read/hutool/db.md)
- [HTTP客户端（Hutool-http）](https://www.bookstack.cn/read/hutool/http.md)：
- [定时任务（Hutool-cron）](https://www.bookstack.cn/read/hutool/crontab.md)
- [扩展（Hutool-extra）](https://www.bookstack.cn/read/hutool/extra.md)
- [布隆过滤（Hutool-bloomFilter](https://www.bookstack.cn/read/hutool/1dbb5bac320e9439.md)：
- [切面（Hutool-aop）](https://www.bookstack.cn/read/hutool/aop.md)
- [脚本（Hutool-script）](https://www.bookstack.cn/read/hutool/script.md)
- [Office文档操作（Hutool-poi）](https://www.bookstack.cn/read/hutool/office.md)
- [系统属性调用-SystemUtil](https://www.bookstack.cn/read/hutool/277cdf75daebb48c.md)

## Guava 工具类

Guava是一个功能非常强大的工具类，提供了很多实用的封装方法，让代码变得简约。

### 优点

1. 高效设计良好的API，被Google的开发者设计，实现和使用
2. 遵循高效的java语法实践
3. 使代码更刻度，简洁，简单
4. 节约时间，资源，提高生产力

### 核心库

1. 原生类型支持 [primitives support]
2. 字符串处理 [string processing]
3. 集合 [collections]
4. 缓存 [caching]
5. 通用注解 [common annotations]
6. 并发库 [concurrency libraries]
7. I/O 等等。

### 字符串处理

#### 拼接

Joiner 可以快速地把多个字符串或字符串数组连接成为用特殊符号连接的字符串。

```java
List<String> list = Lists.newArrayList("a", "b", "c");
String value = Joiner.on("-").skipNulls().join(list);
System.out.println(value);
//输出为： a-b-c
```

#### 分割

### 集合

#### 集合创建

各种以S结尾的**工厂类**简化了集合的创建。在创建泛型实例时，使代码更加简洁。

```java
List<String> list =Lists.newArrayList();
List<String> list2 = Lists.newArrayList("a","b","c");
Set<Integer> set = Sets.newHashSet();
```

类型推导的功能，在Java 7中已经得到支持。

#### Maps

##### newLinkedHashMapWithExpectedSize

初始化一个**大小合适**的map集合，避免在向集合添加元素时，因为大小不合适而resize。

- 每次resize都得执行以下步骤：再次去**分配空间**，再次去计算所有元素的**hashcode**，再次根据hashcode计算数组的**分配位置**，然后数组拷贝。

- 这样就可以大大提升 在使用hashmap时候的性能。和不必要的空间浪费。

```java
Map<String, Long> orderCount = Maps.newLinkedHashMapWithExpectedSize(5);
// 全部
orderCount.put("allCount", tradeOrderQueryService.getOrderCount(getLoginUserId(), null, null));

// 待付款（未支付）
orderCount.put("unpaidCount", 
               	tradeOrderQueryService.getOrderCount(getLoginUserId(),TradeOrderStatusEnum.UNPAID.getStatus(), null)
);
```

### Cache

Cache 是一个全内存的本地缓存实现，提供了**线程安全**的实现机制。
cache的常用参数：

  　　1. 缓存最大大小的设置：CacheBuilder.maximumSize(long)
        　　2. 过期时间：expireAfterAccess(long, TimeUnit) expireAfterWrite(long, TimeUnit)
              　　3. 引用类型：CacheBuilder.weakKeys() CacheBuilder.weakValues() CacheBuilder.softValues()
                    　　4. 自动刷新cache：CacheBuilder.refreshAfterWrite
          　　5. 过期时间：CacheBuilder.expireAfterWrite

### 线程

com.google.common.util.concurrent目录下是各种线程工具类。

- ListenableFuture：顾名思义就是可以监听的Future，它是对java原生Future的扩展增强。
- MoreExecutors：该工具类，提供了很多静态方法。其中listeningDecorator方法初始化
- ListeningExecutorService方法，使用此实例submit方法即可初始化ListenableFuture对象。
- ListeningExecutorService的invokeAny继承自Jdk原生类，率先返回线程组中首个执行完毕的。
- ListeningExecutorService的invokeAll并行执行线程组，等待所有线程执行完毕，适用于批量处理。

### 算法

#### 布隆过滤器

用于判断一个元素是否在一个超大的集合中。

- **哈希表**也能用于判断元素是否在集合中，但是布隆过滤器只需要哈希表的**1/8或1/4的空间复杂度**就能完成同样的功能。