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
location:
abbrlink: 'ssm'
permalink: 'ssm'
date: 2021-09-01 13:42:12
updated: 2025-05-28 11:56:00
---

> 摘要：Spring + Spring MVC + Mybatis。Spring MVC，包括 MVC、Java Web。

<!-- more -->

## 目录

[TOC]

## SSM 框架

> 至少在项目里做过。介绍项目时，用一个业务流程说说 spring mvc 如何做的。

SSM 框架集是 `Spring + Spring MVC + Mybatis ` 框架的整合，`Spring` 实现业务对象管理，`Spring MVC` 负责请求转发和视图管理，`Mybatis` 作为数据对象的持久化引擎。

- 是标准的 MVC 模式，将整个系统划分为 `model/DAO` 层、`View` 层、`Controller` 层、`Service` 层.
- 是目前比较主流的 Java EE 企业级框架，适用于搭建各种大型的企业级应用系统。

### 简化的开发步骤

1. 需求分析：功能需求和非功能需求（稳定性、性能）；
2. 系统设计：系统**架构**示意图：普通用户、管理员，前端（输入输出），后台，MyBatis 数据持久化，微服务，配置等；
    1. 系统概要设计：系统功能模块（划分）图：系统-子系统-功能模块，各自的描述、输入输出接口设计（格式、入参和出参/返回值）。接口文档现在都是 [Swagger-UI](#接口文档)，注解标注在 Controller；
    2. 数据库表结构设计：
        1. **逻辑设计/详细设计** : 通过将 E-R 图转换成表，实现从 E-R 模型到关系模型的转换（转为 POJOs 及 DAO 接口），并应用三大范式优化；
        2. **物理结构设计** : 为设计的数据库选择合适的存储结构和存取路径，做具体的技术选型。
        3. 数据库**实现** : 包括 SQL 编程（创建数据库表）、测试和试运行。
        4. 数据库的**运行和维护** : log 日志，根据新需求进行建表、索引优化、大表拆分。
    3. 功能模块详细设计：模块流程图（Controller 调用 Sercive 实现业务逻辑）。
3. 系统实现；
4. 单元测试，回归测试，覆盖率测试。

##### 系统业务架构示意图

项目技术架构图：

业务架构图：

应用架构图：

<img src="../assets/mall系统架构示意图.jpg" alt="系统架构图" style="zoom: 37%;" />

##### 系统功能模块图



##### E-R 模型图

<img src="../assets/image-20220925110316119.png" alt="image-20220925110316119" style="zoom:100%;" /><img src="../assets/image-20220925111724175.png" alt="image-20220925111724175" style="zoom:100%;" />

### 项目架构

<img src="../assets/arch_screen_02.png" alt="img" style="zoom:70%;" /><img src="../assets/image-20250805140940884.png" alt="img" style="zoom:60%;" />

#### 分层目录

项目的目录结构展示了Maven所约定了源代码的位置，只需配置很少的信息就可以自动完成编译，测试和打包等工作。

<img src="../assets/250px-Maven_CoC.svg.png" alt="img" style="zoom:80%;" />

Java Web 项目中的 SSM 目录结构，同时也遵循 maven 的目录规范：

##### `mall-admin/src/`

`mall-admin/src/test/`：编写测试用例。

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
- 直接用来对应表示层就是`VO`

分类：

1. `DO`（Data/**Domain** Object）：**数据库映射对象**，数据/领域对象，由 DAO 层向上（Service 层 / ServiceImpl 实现层）传输的，从现实世界中抽象出的有形或无形的**业务实体**。
    - ~~`PO`~~（Persistent Object）、**Entity**：持久化对象，与持久层（通常是关系型数据库表）数据结构一 一对应的 POJO 类，由 DAO 层向上（Service 层）传输的 **数据源对象**。最形象的理解就是一个PO就是数据库中的一条记录。 好处是可以把一条记录作为一个对象处理，可以方便的转为其它对象。
3. `BO`（Business Object）：业务对象，由 Service 层向上（Controller 层）传输的，主要作用是把业务逻辑封装为一个对象。通常由 DO **转化、整合**而来，可包含**多个 DO 的属性**，也可只包含一个 DO 的部分属性。
3. `DTO`（Data Transfer Object）：数据传输对象，用于展示层（前端）与服务层间的数据传输对象。存放**参数类型定义类**。DTO为系统与外界交互的模型对象，可以将DTO对象转化为**BO对象**或者是普通的**entity对象**，让service层去处理。
    - `BeanUtils.copyProperties(umsAdminParam, umsAdmin);`，Service 中将 DTO 对象 `UmsAdminParm `赋值给 DO 实体类对象 `UmsAdmin`。
    - 比如添加用户时，只需要传过来用户的姓名和年龄，后端接受到数据后，将添加创建时间和更新时间和默认密码三个字段，然后保存到数据库。
4. `VO`（View Object）：视图对象，通常是 Web 向**模板渲染引擎层**传输的对象。用于展示层，把某个指定页面（或组件）的所有数据封装起来。
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

- **Spring Boot 配置类**配置 MyBatis，用来指定 mapper 映射器接口的 xml 实现所在的**项目目录**。
- **MyBatis 配置类**中通过`@MapperScan`，用来指定 mapper 映射器接口的 xml 实现所在的**包路径**。

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

### Spring MVC

> 本质上相当于传统的 Servlet 和 Struts 中的 action

是一个基于 **MVC 设计模式**的轻量级 Web 开发框架，是 Spring 框架的一部分。 

MVC：`Model-View-Controler` 的简称，即模型—视图—控制器。解耦。

Spring 中通过注解来定义各层，如 `@Controller` 定义控制层、`@Service` 定义 service 业务逻辑层。

> 详情见[项目架构](#项目架构)中。

> [Spring MVC 常用基本注解](#声明 Bean 的注解)

##### MVC 设计模式

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

##### Spring MVC VS Struts2

1. `Spring MVC` 基于方法开发，会将 URL 信息与 Controller 类的某个方法绑定并进行映射，请求参数作为该方法的形参，生成 Handler 对象，只包含一个 method 方法；`Struts2` 基于类开发，**Action 类**中所有方法的请求参数都是**成员变量**，方法越多，类越乱。
2. `Spring MVC` 支持单例开发模式；而 `Struts2` 由于只能通过**类的成员变量**接受参数，无法用单例模式。

不同：

1. 入口不同：Struts2 为 filter 过滤器；SpringMVC 为一个 Servlet，即前端控制器；
2. 开发方式不同：Struts2 基于类开发，传递参数通过类的属性，只能设置为多例；SpringMVC 基于方法开发（一个 url 对应一个方法），请求参数传递到方法形参，可为单例也可为多例（建议单例）；
3. 请求方式不同：Struts2 值栈存储请求和响应的数据，通过OGNL存取数据；SpringMVC 通过参数解析器将 request 请求内容解析，给方法形参赋值，将数据和视图封装成 ModelAndView 对象，最后又将其中的模型数据通过 request 域传输到页面，jsp 视图解析器默认使用的是 jstl。

##### Spring MVC 的核心组件

1. **`DispatcherServlet`**：**核心的中央处理器**、**前端控制器**/分发器/**调度器**，负责接收请求、分发，并给予客户端响应。
2. **`HandlerMapping`**：**处理器映射器**，**根据 URL 去匹配**查找能处理的 `Handler` ，并会将请求涉及到的**拦截器**和 `Handler` 一起封装。
3. **`HandlerAdapter`**：**处理器适配器**，根据 `HandlerMapping` 找到的 `Handler` ，**适配执行**对应的 `Handler`；
4. **`Handler`**：**请求处理器**，处理实际请求的处理器。
5. **`ViewResolver`**：**视图解析器**，根据 `Handler` 返回的逻辑视图 / 视图 / **`ModelAndView` 对象**，解析并渲染真正的视图，并传递给 `DispatcherServlet` 响应客户端。

##### Spring MVC 工作原理/流程说明

<img src="../assets/springmvc_work_flow.jpg" alt="img" style="zoom:100%;" />

传统开发模式（JSP，Thymeleaf 等）的工作原理：

1. 客户端（浏览器）发送请求到 **`Di'spatcherServlet` 前端控制器**/分发器/**调度器**；
2. `DispatcherServlet` （根据请求信息）查询 `HandlerMapping` 处理器映射；`HandlerMapping` 根据 URL 去匹配查找能处理的 `Handler`（也就是我们平常说的 `Controller` 控制器） ，并会将请求涉及到的拦截器和 `Handler` 一起封装。
3. ~~返回 `Handler` 处理器对象；~~
4. 调用 `HandlerA'dapter` 适配器，解析请求到对应的具体 Handler（即 **Controller**）；
5. `HandlerAdapter` 适配器调用**真正的处理器** `Handler` 处理请求和相应的业务逻辑；Controller 调用 Service 业务逻辑层处理后返回结果；
6. 处理器处理完业务后，会返回一个 **`ModelAndView` 对象**（Model 是返回的数据对象，View 是逻辑上的 View）；
7. ~~返回 `ModelAndView` 对象给前端控制器；~~
8. 请求 `ViewResolver` 视图解析器解析视图，根据逻辑 View 查找实际的 View；
9. **视图渲染**：`DispaterServlet` 把返回的 Model 传给 View；
10. JSP 渲染视图；
11. 将 View 响应给浏览器。

然而现在主流的开发方式是**前后端分离**，这种情况下 Spring MVC 的 `View` 概念发生了一些变化。

由于 `View` 通常由前端框架（Vue, React 等）来处理，后端不再负责渲染页面，而是只负责提供数据，因此：

- 后端通常不再返回具体的视图，而是返回**纯数据**（通常是 JSON 格式），由前端负责渲染和展示。
- `View` 的部分往往不需要设置，Spring MVC 的控制器方法只需要返回数据，不再返回 `ModelAndView`，而是直接返回数据，Spring 会自动将其转换为 JSON 格式。相应的，`ViewResolver` 也将不再被使用。

怎么做到呢？

- 使用 `@RestController` 注解代替传统的 `@Controller` 注解，这样所有方法默认会返回 JSON 格式的数据，而不是试图解析视图。
- 如果你使用的是 `@Controller`，可以结合 `@ResponseBody` 注解来返回 JSON。

### Spring、Spring MVC、Spring Boot

Spring，Spring MVC，Spring Boot 之间什么关系?

1. `Spring`：是一个**轻量级开源框架**，用于简化 ~~Java EE~~ 企业级应用程序开发。核心是 IoC（**控制反转**）和 **AOP**（面向切面编程）。还提供了事务管理等功能。
    - 可以用于构建**任何类型**的 Java 应用程序，包括 Web 应用程序、桌面应用程序、和批处理应用程序等。
2. `Spring MVC`：Spring MVC 是构建在 Spring 之上的 **Web 框架**，用于**构建 Web 应用程序**。提供了一个基于 Model、View、Controller 模式的 Web 框架，用于处理 Web 请求和响应。
    - 通过将请求**映射**到相应的处理器方法，并使用**视图**来呈现响应，使得构建Web应用程序变得简单和灵活。
3. `Spring Boot`：是一个用于**简化 Spring 应用程序开发**的框架，用于快速构建基于 Spring 的 Web 应用程序。Spring Boot **依赖于** Spring 框架和 Spring MVC。提供了**自动配置**、开箱即用等功能。
    - 内置了许多常用的**第三方库和框架**，简化了配置和部署过程。还提供了一组用于开发Web应用程序、RESTful 服务和微服务的起步**依赖项**。

## Spring 整合 Web

#### Java Web

##### 服务器

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

##### Tomcat

<img src="../assets/tomcat_server_response.png" alt="在这里插入图片描述" style="zoom: 50%;" />

Tomcat 运行机制：Tomcat 服务器接受客户请求并做出响应的过程。

##### HTTP 请求处理流程

1. **客户端**（通常都是浏览器）访问 **Web 服务器**，发送 HTTP 请求。
    - Web 服务器：也叫 HTTP 服务器、web容器，其需要提供web应用运行所需的环境，接收客户端的Http请求
2. Web 服务器接收到请求后，传递给 **Servlet 容器**。
    -  Servlet 容器：也称Servlet引擎，为Servlet的运行提供环境支持，可以理解为 **Tomcat** 或其他服务器。
3. **Servlet 容器**（根据 url 等路径决定）加载对应 Servlet 实例；调用 Servlet 的 `service()` 方法返回 response 对象，向其传递表示请求和响应的对象。
    1. 如果Serlvet没有被实例化，则创建该Servlet的一个实例（调用init方法）；
    2. 或从线程池中取出一个空闲线程（产生 Servlet 实例）；
4. Servlet容器根据用户的HTTP请求，创建一个**`ServletRequest`请求对象**（封装了 HTTP 请求信息）和一个可以对HTTP请求进行响应的`ServletResponse`对象（类似于寄信，并在信中说明回信的地址），
5. 然后调用`HttpServlet`中重写的**`service(ServletRequest req, ServletResponse res)`方法**，
    1. 并在这个方法中，将这两个对象**向下转型**，得到**`HttpServletRequest`**和`HttpServletResponse`两个对象，
    2. 然后将客户端的请求转发到`HttpServlet`中的`service(HttpServletRequest req, HttpServletResponse resp)`。
6. `HttpServletResponse`：服务端处理完Http的请求后，将处理结果作为**Http响应**返回给客户端。

参考：[HttpServletRequest和@Requestparam、@RequestBody、直接实体接收请求参数的区别与示例](https://blog.csdn.net/qq_41358574/article/details/120422160)

#### 获取 HttpServletRequest 对象

获取 request 对象的四种方法：

1. 传参：Controller 中**加参数** `HttpServletRequest request` 来获取 request 对象；

    - 实现原理是，在Controller方法开始处理请求时，Spring会将request对象赋值到方法参数中。此时request对象相当于局部变量，毫无疑问是线程安全的。
    - 缺点：冗余太多。

2. IoC：**自动注入**（成员变量）来获取 request 对象；优点有：

    1. 注入不局限于 Controller 中，还可以在任何Bean中注入，包括Service、Repository及普通的Bean。

    2. 注入的对象不限于request，还可以注入其他`scope`为`request`或`session`的对象，如`response`对象、`session`对象等；并保证线程安全。

    3. 大大减少了代码冗余。

        ```
        @Autowire
        HttpServletRequest request;
        
        //?
        @Autowire
        HttpSession session;
        ```

3. **基类**中自动注入（推荐）；

    ```
    public class BaseController {
        @Autowired
        protected HttpServletRequest request;     
    }
    ```

4. 通过 `RequestContextHolder` 的静态方法获取 request 对象及 response 等相关对象。

    - 优点：可以在非Bean中直接获取。
    - `RequestContextHolder`：持有上下文的Request容器。

    ```
    //获取当前请求对象
    ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
    HttpServletRequest request = attributes.getRequest();
    ```

#### HTTP 请求中接收前端参数的方式

##### 三种常用的方式

1. `HttpServletRequest`：
    - 后端用于响应的有`HttpServletResponse`。
2. `@RequestParam`：具体见下面 [REST 常用注解](#REST 常用注解)。
3. `@RequestBody`：具体见下面 [REST 常用注解](#REST 常用注解)。

取参类型区别：

- `@RequestParam` 和 `@RequestBody` 都是从 `HttpServletRequest request` 中取参的，而 `@PathVariable` 是映射 URI 请求参数中的**占位符**到目标方法的参数。

- `@RequestParam` 可以获取（Content-Type 的**默认值**） `application/x-www-form-urlencoded` 以及 `application/json` 这两种类型的参数，
- 但是 `@RequestBody` 是用来获取**非默认类型**的数据，比如 `application/json`、`application/xml` 等。

##### HttpServletRequest

`HttpServletRequest` 接口可以获得的参数更多，它可以获得客户端的**请求行和请求头、请求体**信息。

当访问Servlet时，会在请求消息的请求行中，包含请求方法、请求资源名、请求路径等信息，为了获取这些信息，定义了一系列用于获取请求行的方法。

1. 获得请求行：`String getMethod()` GET/POST等。
2. 获得请求头：`String getHeader(String name)`等。
3. 获得请求体：`String getParameter(String name)`等。
4. 获取 Session：`getSession()`。

```
Public static void login(User user, HttpServletRequest request) {
    HttpSession session = request.getSession();
    Session.setAttribute(User, user);
}
```

##### HttpServletResponse

`HttpServletResponse`：服务端处理完Http的请求后，根据HttpServletResponse对象将处理结果作为Http响应返回给客户端。

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

```
// 如果请求的 url 是：/klasses/123456/teachers?type=web
// 则获取到的数据就是：klassId=123456, type=web
@GetMapping("/klasses/{klassId}/teachers")
public List<Teacher> getKlassRelatedTeachers(
	@PathVariable("klassId") Long klassId,
	@RequestParam(value = "type", defaultValue = "10000", required = false) String klassType ) {
	...
}
```

##### @RequestBody

`@RequestBody`：将传入的 **HTTP 请求体**与注解的方法参数中的**对象**绑定， 要求传递一个 `JSON` 格式的字符串。响应体另见[@ResponseBody](#@Controller)。

- 不能用于GET请求；
- 用于读取 Request 请求的 body 部分、`Content-Type` 为 `application/json` 格式的数据，接收到数据后会自动将数据**绑定到 Java 对象**上去。系统会将请求的 body 中的 json 字符串转换为 Java 对象。

使用 @RequestBody 需要满足如下条件：

1. Content-Type 为 application/json，确保传递的是 JSON 数据；
2. 参数转化的配置必须统一，否则无法接收数据，比如 json、request 混用等。

```
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

- 使用 `@RequestParam` 可以直接获取指定的参数，可以获取 默认以及 `application/json` 这两种类型的参数，但是一旦前端传递的是 JSON 数据，那么使用 @RequestParam 取不到值，还报错。

- 使用 `@RequestBody` 是用来获取**非默认类型**的数据，比如 `application/json`、`application/xml` 等。

##### @RequestXxx 系列

1. `@RequestHeader`：用于获取有关 HTTP 请求头的详细信息。将此注解用作**方法参数**。注解的可选元素是**名称，必填，值，defaultValue。** 可在一种方法中多次使用。
2. `@RequestAttribute`：将方法参数绑定到请求属性。提供了从控制器方法方便地访问请求属性的方法。可访问服务器端填充的对象。

##### @Required

`@Required`：用于 Bean 设置方法。表示在配置时用**必需的属性**填充带注解的 Bean，否则将引发异常 `BeanInitilizationException` 。

##### @CrossOrigin

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

```
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

#### AJAX、JSON

1. `@JsonIgnoreProperties({"userRoles"})`：作用在类上，用于过滤掉特定字段，不返回或不解析；

2. `@JsonIgnore`：用于属性上，同上；

3. `@JsonFormat`：用来格式化 JSON 数据。

    ```
    @JsonFormat(shape=JsonFormat.Shape.STRING, pattern="yyyy-MM-dd'T'HH:mm:ss.SSS'Z'", timezone="GMT")
    private Date date;
    ```

4. `@JsonUnwrapped`：扁平化对象。

SpringMVC 中的转发和重定向：

- 转发： `return: "hello" `

- 重定向 ：`return: "redirect:hello.jsp"`

通过 **JackSon 框架**把 Java 里的对象直接转换成 js 可识别的 JSON 对象，在接受 AJAX 方法里直接返回Object，List 等，方法前需要加上注解 @ResponseBody。

#### Axios

前端 HTTP 框架
