---
title: Spring Boot
categories:
  - SSM框架
tags:
  - SSM
  - Spring-Boot
  - ORM
  - MyBatis
  - Redis
  - ELK
  - Swagger-UI
  - Spring-Security
  - Hutool
  - Thymeleaf
location:
abbrlink: 'spring_boot'
permalink: 'spring_boot'
date: 2025-06-06 13:42:12
updated: 2025-06-06 11:56:00
---

> 摘要：Spring Boot 包括配置、自动装配、数据库等。Spring Boot 整合 Web、Hibernate、MyBatis、Redis、ELK、Swagger-UI、Security、Hutool 等。

<!-- more -->

## 目录

[TOC]

## Spring Boot

**Spring Boot**是一个开源 Java 框架，用于开发独立、产品等级的 Spring 应用程序，和节省开发人员工作量。Spring Boot使用[约定优于配置](https://zh.wikipedia.org/wiki/约定优于配置)设计模式，在Java平台帮助**最少化配置设定**，开发Spring为基础的应用程序。

- 大部分应用程序可以被预先配置，使用Spring团队的"专业意见"应用最好的设定，和使用Spring平台及第三方函式库。
- **约定优于配置**：也称作按约定编程，是一种软件设计范式，旨在减少软件开发人员需做决定的数量，获得简单的好处，而又不失灵活性。
    - 例如，在知名的Java[对象关系映射](https://zh.wikipedia.org/wiki/对象关系映射)框架[Hibernate](https://zh.wikipedia.org/wiki/Hibernate)的早期版本中，将类及其属性映射到数据库上需要是在XML文件中的描述，其中大部分信息都应能够按照约定得到，如将**类**映射到同名的**数据库表**，将**属性**分别映射到表上的**字段**。后续的版本抛弃了[XML](https://zh.wikipedia.org/wiki/XML)配置文件，而是使用这些恰当的约定，对于不符合这些约定的情形，可以使用[Java 标注](https://zh.wikipedia.org/wiki/Java_标注)来说明（参见下面提供的JavaBeans规范）。

参考列表：

- [Spring Boot 框架入门教程（快速学习版）](http://c.biancheng.net/spring_boot/)
- [Spring/Spring Boot 常用注解总结](https://javaguide.cn/system-design/framework/spring/spring-common-annotations.htm)

### 优点

简化 Spring 项目配置：

1. 管理依赖关系：提供一系列的 `starter` 项目对象模型（POMS）来**简化 Maven 配置**（引入依赖）；
2. **自动配置、开箱即用**：提供大量默认配置；通过`@Configuration`**注解和配置类**替换传统繁杂的 xml 配置文件，以 **JavaBean** 形式配置。**快速搭建**开发环境；
3. 方便集成**整合 Spring 生态系统**，如 Spring JDBC、Spring ORM、Spring Data、Spring Security 等；
4. 强大的数据库**事务管理**功能；
5. 部署方便：内置多种 **Web 容器**，如 [Tomcat](#Tomcat) （作为默认的嵌入式 HTTP 服务器/ `Servlet` 容器）、 Jetty、Undertow，可轻松开发和测试 Web 应用程序；直接运行**入口函数**即可启动项目/作为独立应用程序运行；
6. 项目可打包成 jar 文件，可用`java–jar xx.jar` 命令以 jar 包的形式独立运行；
7. 自带应用监控；
8. 集成 Junit，测试方便；

对比、区别见上 [Spring、Spring MVC、Spring Boot](#SSM 框架)。

### Spring 配置

#### 自动配置

SpringBoot的自动配置是一个运行时（更准确地说，是应用程序启动时）的过程，考虑了众多因素，才决定Spring配置应该用哪个，不该用哪个。

举个例子，当使用Spring整合MyBatis的时候，需要完成如下几个步骤：

   - 根据数据库连接配置，配置一个dataSource对象；
   - 根据dataSource对象和SqlMapConfig.xml文件（其中包含mapper.xml文件路径和mapper接口路径配置），配置一个sqlSessionFactory对象。

当我们使用SpringBoot整合MyBatis的时候，会自动创建dataSource和sqlSessionFactory对象，只需我们在`application.yml`和Java配置中添加一些自定义配置即可。

在`application.yml`中配置好数据库连接信息及mapper.xml文件路径。

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mall?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai
    username: root
    password: root

mybatis:
  mapper-locations:
    - classpath:mapper/*.xml
    - classpath*:com/**/mapper/*.xml
```

使用Java配置，配置好mapper接口路径。

```java
@Configuration
@MapperScan("com.macro.mall.tiny.mbg.mapper")
public class MyBatisConfig {
}
```

使用自动配置以后，我们整合其他功能的配置大大减少了，可以更加专注程序功能的开发了。

#### 配置文件

##### 配置文件格式

```
// 1. properties 格式
server.port = 8080
// 表示文档间隔
---

// 2. yml 格式
// 不支持 @PropertySource 注解导入配置
// spring-boot-starter-web 或 spring-boot-starter 都集成了 SnakeYAML 库，
// 引用任一个，Spring Boot 都会自动添加库到 classpath 下
server:
	port: 8080
// 以缩进来控制层级关系
// 键值对间必须有空格
// 大小写敏感
// 字符串不需加引号
// 双引号不转义, 单引号转义特殊字符，如"\n"为换行 
```

##### 配置绑定

配置绑定：读取（全局）配置文件中的属性值并绑定到 JavaBean 上，如数据库配置。用于**容器**中的组件。

1. `@ConfigurationProperties(prefix = "person") `：用在**类名**上，读取配置文件中（**指定前缀**为`"person"`）的所有配置数据，并与此 JavaBean/类（用`@Component` 等标注）中的所有属性绑定；支持松散语法绑定。

2. `@Value("${url}")`：用在**属性**上，只读取配置文件中的某一个配置，与当前属性**绑定**；只支持基本数据类型 + String 类型，支持 SpEL 表达式。**不推荐**。与 [Lombok 常用注解](#数据类型转换)重名；

    ```
    import org.springframework.beans.factory.annotation.Value;
    
    @Value("${aliyun.oss.accessKeyId}")
    private String ALIYUN_OSS_ACCESSKEYID;
    ```

3. `@PropertySource(value = "classpath:person.properties")`：读取指定位置的配置文件；如与 Spring Boot 无关的（与 person 相关的）**自定义配置**移动到 `src/main/resources/person.properties`中。不常用。

##### 导入配置文件

1. `@Import`： 允许从另一个 Java/XML 配置文件加载 Bean 定义。
2. `@ImportResource(locations = {"classpath:/beans.xml"})`：用于**主启动类**上，加载 Spring 配置文件，`src/main/resources/beans.xml`。
3. 推荐用全注解方式加载 Spring 配置：
    1. `@Configuration`：**定义配置类**，相当于 Spring 的配置文件；
    2. 配置类内可有一个或多个被 `@Bean` 注解的方法，~~会被 `AnnotationConfigApplicationContext` 或 `AnnotationConfigWebApplicationContext` 类扫描~~，构建 Bean 定义（相当于 Spring 配置文件中的 `<bean>` 标签），方法的返回值会以组件的形式添加到容器中，组件的 id 就是方法名。
    3. 优先级低于 `.properties/.yml` 配置文件。

##### 多环境配置/切换

1. 创建 `application-{profile}.properties` 文件，如：`application-dev.properties` 用于开发环境；
2. 在 `application.properties` 文件中添加 `spring.profiles.active=dev`；

```
// 命令行激活：将 Spring Boot 项目打包成 JAR 文件
// 命令行窗口，跳转到 JAR 文件所在目录，执行以下命令，
// 启动该项目，并激活开发环境的 Profile。
java -jar helloworld-0.0.1-SNAPSHOT.jar  --spring.profiles.active=dev
```

##### 配置文件优先级、加载顺序

Spring Boot 启动时会扫描以下位置的 `application.properties/application.yml` 文件，作为默认配置文件。由高到低4级配置文件：

- `file:./config/*/`
- 1级：`file:config/application.yml`：[最高] 工程路径config目录（jar包外的）
- 2级：`file:application.yml`：工程路径目录（项目根目录）
- 3级：`classpath:config/application.yml`：项目类路径config目录，`resources/config/application.yml`
- 4级：`classpath:application.yml` [最低]：项目类路径目录

> 注：file: 指当前项目根目录；

作用：

- 1级与2级留做系统**打包后**设置通用属性：1级常用于运维经理进行线上整体项目部署方案调控，2级服务于运维人员配置涉密线上环境。
- 3级和4级用于系统**开发阶段**设置通用属性：3级常用于项目经理进行整体项目属性调控，4级服务于开发人员本机开发与测试。

优先级：序号越小优先级越高。

1. 根目录下的优先级高于当前项目的类路径下的；
2. 先加载 `config/` 文件夹，子文件高于父文件夹；
3. 相同位置的 .properties 的**优先级**高于 .yml；
4. 带 `-{profile}` **多环境**的优先级高于不带的。

加载顺序：多层级配置文件间的属性，采用**叠加并覆盖**的形式作用于程序。

1. 存在相同的配置内容时，高优先级的内容会**覆盖**低优先级的内容；
2. 存在不同的配置内容时，配置内容**叠加**取并集。

##### classpath

`classpath`：指当前项目的类路径，常用 `classpath:文件名` 引用 `classpath` 路径下的文件。用来指示 JVM 搜索 `.class` 文件位置的环境变量。

1. 用 maven 构建（build）项目时，默认的 classpath 指向 `target/classes/`；
2. 用 maven 打包（package）项目时，默认的 classpath 指向 war 内部的 `WEB-INF/classes/`；

```
// springboot项目默认的classpath及工程编译后的位置
./src/main/java/ # 将.java文件按照包文件结构编译成.class存入target/classes/
./src/main/resources/ # 将static/、templates/目录按结构拷贝入target/classes/
./src/test/java/ # 将文件编译进target/test-classes/目录中
./target/

// 获取springboot项目默认的classpath
String classpath = ResourceUtils.getURL("classpath:").getPath();
```

#### 自定义配置类

##### 自定义Bean覆盖自动配置

虽然自动配置很好用，但有时候自动配置的Bean并不能满足你的需要，我们可以自己定义相同的Bean来覆盖自动配置中的Bean。

例如当我们使用Spring Security来保护应用安全时，由于自动配置并不能满足我们的需求，我们需要自定义基于WebSecurityConfigurerAdapter的配置。这里我们自定义了很多配置，比如将基于Session的认证改为使用JWT令牌、配置了一些路径的无授权访问，自定义了登录接口路径，禁用了csrf功能等。

```java
/**
 * SpringSecurity的配置
 */
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    private UmsAdminService adminService;
    @Autowired
    private RestfulAccessDeniedHandler restfulAccessDeniedHandler;
    @Autowired
    private RestAuthenticationEntryPoint restAuthenticationEntryPoint;
    @Autowired
    private IgnoreUrlsConfig ignoreUrlsConfig;

    @Override
    protected void configure(HttpSecurity httpSecurity) throws Exception {
        List<String> urls = ignoreUrlsConfig.getUrls();
        String[] urlArray = ArrayUtil.toArray(urls, String.class);
        httpSecurity.csrf()// 由于使用的是JWT，我们这里不需要csrf
                .disable()
                .sessionManagement()// 基于token，所以不需要session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .authorizeRequests()
                .antMatchers(HttpMethod.GET,urlArray) // 允许对于网站静态资源的无授权访问
                .permitAll()
                .antMatchers("/admin/login")// 对登录注册要允许匿名访问
                .permitAll()
                .antMatchers(HttpMethod.OPTIONS)//跨域请求会先进行一次options请求
                .permitAll()
                .anyRequest()// 除上面外的所有请求全部需要鉴权认证
                .authenticated();
        // 禁用缓存
        httpSecurity.headers().cacheControl();
        // 添加JWT filter
        httpSecurity.addFilterBefore(jwtAuthenticationTokenFilter(), UsernamePasswordAuthenticationFilter.class);
        //添加自定义未授权和未登录结果返回
        httpSecurity.exceptionHandling()
                .accessDeniedHandler(restfulAccessDeniedHandler)
                .authenticationEntryPoint(restAuthenticationEntryPoint);
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService())
                .passwordEncoder(passwordEncoder());
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public UserDetailsService userDetailsService() {
        //获取登录用户信息
        return username -> {
            AdminUserDetails admin = adminService.getAdminByUsername(username);
            if (admin != null) {
                return admin;
            }
            throw new UsernameNotFoundException("用户名或密码错误");
        };
    }

    @Bean
    public JwtAuthenticationTokenFilter jwtAuthenticationTokenFilter() {
        return new JwtAuthenticationTokenFilter();
    }

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

}
```

##### 自动配置微调

有时候我们只需要微调下自动配置就能满足需求，并不需要覆盖自动配置的Bean，此时我们可以在`application.yml`属性文件中进行配置。

比如微调下应用运行的端口。

```yaml
server:
  port: 8088
```

比如修改下数据库连接信息。

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mall?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai
    username: root
    password: root
```

##### 读取配置文件的自定义属性

有时候我们会在属性文件中自定义一些属性，然后在程序中使用。此时可以将这些自定义属性映射到一个属性类里来使用。

比如说我们想给Spring Security配置一个白名单，访问这些路径无需授权，我们可以先在`application.yml`中添添加如下配置。

```yaml
secure:
  ignored:
    urls:
      - /
      - /swagger-ui/
      - /*.html
      - /favicon.ico
      - /**/*.html
      - /**/*.css
      - /**/*.js
      - /swagger-resources/**
      - /v2/api-docs/**
```

之后创建一个属性类，使用`@ConfigurationProperties`注解配置好这些属性的前缀，再定义一个`urls`属性与属性文件相对应即可。

```java
/**
 * 用于配置白名单资源路径
 */
@Getter
@Setter
@Component
@ConfigurationProperties(prefix = "secure.ignored")
public class IgnoreUrlsConfig {

    private List<String> urls = new ArrayList<>();

}
```

### Spring Boot 自动装配原理

> 开箱即用

自动装配：通过注解或简单配置在 Spring Boot 帮助下实现某块功能。

##### Spring Factories 机制

1. Spring Boot 通过 `@EnableAutoConfiguration` **开启**自动装配，基于 [Spring Factories 机制](#IoC 容器)实现自动装配。
2. 通过 `SpringFactoriesLoader`（spring-core 包里） 自动扫描所有 Jar 包**类路径**下的 `META-INF/spring.factories ` 文件，**加载**其中的自动配置类（即通过 `@Conditional` 按需加载的配置类）实现自动装配。
3. 想要其生效必须引入 `spring-boot-starter-xxx` 包实现起步依赖。

#####  `@Conditional`

按需加载的配置类。

`@Conditional`注解的简化，省略了自己实现`Condition`接口：

- `@ConditionalOnBean(name = "dynamicSecurityService")`
- `@ConditionalOnMissingBean`
- `@ConditionalOnClass`
- `@ConditionalOnMissingClass`

##### `@SpringBootApplication`

Spring Boot **启动类上**的核心注解，实现自动装配的关键，由以下 3 个注解组合而成：

1. `@EnableAutoConfiguration`：开启 SpringBoot **自动配置**功能，简化配置编写。
    - 也可关闭，如关闭数据源自动配置：`@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class })`。
2. `@Configuration`：允许在上下文中注册额外的 Bean 或[导入其他配置类](#导入配置文件)。用于**声明 Spring 中的Java 配置类**，（通过简单调用同类中的其他 @Bean 方法来）配置/定义 Bean 间的依赖关系。同 `@Component`。
    - `@SpringBootConfiguration`：组合了 `@Configuration`，实现配置文件的功能；
3. `@ComponentScan`： 启用组件扫描，当声明组件时，会自动发现并注册为Spring应用上下文中的Bean。扫描 `@Component（@Service、@Controller）` 等注解标注的 Bean，**装配**（加载）到容器中，默认会扫描该类所在包及其子包下所有的类。

### Spring Boot Starter

只需在 Maven  pom.xml 配置中引入 starter 依赖，Spring Boot 就能自动扫描到要加载的信息并启动相应的**默认配置**。通过少量注解和简单配置就能用第三方组件提供的功能。


##### 库依赖

1. 一方库：本工程内部**子项目模块**依赖的库（jar 包）；
2. 二方库：**公司内部**发布到中央仓库，可供公司内其它应用依赖的库（jar 包）；
3. 三方库：公司之外的开源库（jar 包）。

##### GAV 定义规则

> 二方库依赖。

一般，包名根目录 `= groupId + artifactId`，唯一。

1. `groupId` 格式：`com.{公司/BU/组织域名}.业务线.[子业务线]`，最多 4 级。如，`com.taobao.jstorm` 或 `com.alibaba.dubbo.register`。
2. `artifactId` 格式：`产品线名/项目名-模块名`。语义不重复不遗漏，先到中央仓库查证一下。
3. `version`：主版本号.次版本号.修订号 
    1. 主版本号：产品方向改变，或大规模 API 不兼容，或**架构**不兼容升级；
    2. 次版本号：保持**相对**兼容性，增加**主要功能**特性，影响范围极小的 API 不兼容修改；
    3. 修订号：保持**完全**兼容性，修复 BUG、新增**次要功能**特性等。

##### 工作原理/启动过程

1. Spring Boot 在启动时，会去依赖的 starter 包中寻找 `resources/META-INF/spring.factories` 文件，根据文件中配置的 jar 包扫描项目所依赖的 jar 包；
2. 根据 `spring.factories` 配置加载 `AutoConfigure` 类；
3. 根据 `@Conditional` 注解的条件，进行自动配置并将 Bean 注入 Spring Context。

按照约定去读取 Spring Boot Starter 的配置信息，再根据配置信息对资源进行初始化，并注入到 Spring 容器中。

##### spring-boot-starter-parent

> 版本仲裁中心。

1. 默认 JDK 版本（Java 8）；
2. 默认字符集（UTF-8）；
3. 统一管理部分常用依赖，**版本仲裁**；
4. 资源过滤；
5. 默认插件配置；
6. 识别 `application.properties/.yml` 类型的配置文件。

##### 常用 starter

1. spring-boot-starter-**web**：提供了嵌入的 **Servlet 容器**、web开发需要的 servlet 与 jsp 支持及 **Spring MVC** 的依赖，并为 Spring MVC 提供了大量自动配置；
2. spring-boot-starter-**data-jpa**：数据库支持；
3. spring-boot-starter-**data-Redis**：redis 数据库；
4. spring-boot-starter-data-solr：代码质量检查？
5. spring-boot-devtools：LiveReload 自动刷新，将文件更新自动部署到服务器并重启。
6. **mybatis**-spring-boot-starter。

### Actuator

SpringBoot Actuator的关键特性是在应用程序里提供众多Web端点，通过它们了解应用程序运行时的内部状况。

#### 端点概览

Actuator提供了大概20个端点，常用端点路径及描述如下：

| 路径            | 请求方式 | 描述                                                        |
| --------------- | -------- | ----------------------------------------------------------- |
| /beans          | GET      | 描述应用程序上下文里全部的Bean，以及它们之间关系            |
| /conditions     | GET      | 描述自动配置报告，记录哪些自动配置生效了，哪些没生效        |
| /env            | GET      | 获取全部环境属性                                            |
| /env/{name}     | GET      | 根据名称获取特定的环境属性                                  |
| /mappings       | GET      | 描述全部的URI路径和控制器或过滤器的映射关系                 |
| /configprops    | GET      | 描述配置属性（包含默认值）如何注入Bean                      |
| /metrics        | GET      | 获取应用程序度量指标，比如JVM和进程信息                     |
| /metrics/{name} | GET      | 获取指定名称的应用程序度量值                                |
| loggers         | GET      | 查看应用程序中的日志级别                                    |
| /threaddump     | GET      | 获取线程活动的快照                                          |
| /health         | GET      | 报告应用程序的健康指标，这些值由HealthIndicator的实现类提供 |
| /shutdown       | POST     | 关闭应用程序                                                |
| /info           | GET      | 获取应用程序的定制信息，这些信息由info打头的属性提供        |

#### 查看配置明细

- 直接访问根端点，可以获取到所有端点访问路径，根端点访问地址：http://localhost:8088/actuator

```json
{
    "_links": {
        "self": {
            "href": "http://localhost:8088/actuator",
            "templated": false
        },
        "beans": {
            "href": "http://localhost:8088/actuator/beans",
            "templated": false
        },
        "caches-cache": {
            "href": "http://localhost:8088/actuator/caches/{cache}",
            "templated": true
        },
        "caches": {
            "href": "http://localhost:8088/actuator/caches",
            "templated": false
        },
        "health": {
            "href": "http://localhost:8088/actuator/health",
            "templated": false
        },
        "health-path": {
            "href": "http://localhost:8088/actuator/health/{*path}",
            "templated": true
        },
        "info": {
            "href": "http://localhost:8088/actuator/info",
            "templated": false
        },
        "conditions": {
            "href": "http://localhost:8088/actuator/conditions",
            "templated": false
        },
        "configprops": {
            "href": "http://localhost:8088/actuator/configprops",
            "templated": false
        },
        "env": {
            "href": "http://localhost:8088/actuator/env",
            "templated": false
        },
        "env-toMatch": {
            "href": "http://localhost:8088/actuator/env/{toMatch}",
            "templated": true
        },
        "loggers": {
            "href": "http://localhost:8088/actuator/loggers",
            "templated": false
        },
        "loggers-name": {
            "href": "http://localhost:8088/actuator/loggers/{name}",
            "templated": true
        },
        "heapdump": {
            "href": "http://localhost:8088/actuator/heapdump",
            "templated": false
        },
        "threaddump": {
            "href": "http://localhost:8088/actuator/threaddump",
            "templated": false
        },
        "metrics-requiredMetricName": {
            "href": "http://localhost:8088/actuator/metrics/{requiredMetricName}",
            "templated": true
        },
        "metrics": {
            "href": "http://localhost:8088/actuator/metrics",
            "templated": false
        },
        "scheduledtasks": {
            "href": "http://localhost:8088/actuator/scheduledtasks",
            "templated": false
        },
        "mappings": {
            "href": "http://localhost:8088/actuator/mappings",
            "templated": false
        }
    }
}
```

- 通过`/beans`端点，可以获取到Spring应用上下文中的Bean的信息，比如Bean的类型和依赖属性等，访问地址：http://localhost:8088/actuator/beans

```json
{
	"contexts": {
		"application": {
			"beans": {
				"sqlSessionFactory": {
					"aliases": [],
					"scope": "singleton",
					"type": "org.apache.ibatis.session.defaults.DefaultSqlSessionFactory",
					"resource": "class path resource [org/mybatis/spring/boot/autoconfigure/MybatisAutoConfiguration.class]",
					"dependencies": [
						"dataSource"
					]
				},
				"jdbcTemplate": {
					"aliases": [],
					"scope": "singleton",
					"type": "org.springframework.jdbc.core.JdbcTemplate",
					"resource": "class path resource [org/springframework/boot/autoconfigure/jdbc/JdbcTemplateConfiguration.class]",
					"dependencies": [
						"dataSource",
						"spring.jdbc-org.springframework.boot.autoconfigure.jdbc.JdbcProperties"
					]
				}
			}
		}
	}
}
```

- 通过`/conditions`端点，可以获取到当前应用的自动配置报告，`positiveMatches`表示生效的自动配置，`negativeMatches`表示没有生效的自动配置。

```json
{
	"contexts": {
		"application": {
			"positiveMatches": {
				"DruidDataSourceAutoConfigure": [{
					"condition": "OnClassCondition",
					"message": "@ConditionalOnClass found required class 'com.alibaba.druid.pool.DruidDataSource'"
				}]
			},
			"negativeMatches": {
				"RabbitAutoConfiguration": {
					"notMatched": [{
						"condition": "OnClassCondition",
						"message": "@ConditionalOnClass did not find required class 'com.rabbitmq.client.Channel'"
					}],
					"matched": []
				}
			}
		}
	}
}
```

- 通过`/env`端点，可以获取全部配置属性，包括环境变量、JVM属性、命令行参数和`application.yml`中的属性。

```json
{
	"activeProfiles": [],
	"propertySources": [{
			"name": "systemProperties",
			"properties": {
				"java.runtime.name": {
					"value": "Java(TM) SE Runtime Environment"
				},
				"java.vm.name": {
					"value": "Java HotSpot(TM) 64-Bit Server VM"
				},
				"java.runtime.version": {
					"value": "1.8.0_91-b14"
				}
			}
		},
		{
			"name": "applicationConfig: [classpath:/application.yml]",
			"properties": {
				"server.port": {
					"value": 8088,
					"origin": "class path resource [application.yml]:2:9"
				},
				"spring.datasource.url": {
					"value": "jdbc:mysql://localhost:3306/mall?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai",
					"origin": "class path resource [application.yml]:6:10"
				},
				"spring.datasource.username": {
					"value": "root",
					"origin": "class path resource [application.yml]:7:15"
				},
				"spring.datasource.password": {
					"value": "******",
					"origin": "class path resource [application.yml]:8:15"
				}
			}
		}
	]
}
```

- 通过`/mappings`端点，可以查看全部的URI路径和控制器或过滤器的映射关系，这里可以看到我们自己定义的`PmsBrandController`和`JwtAuthenticationTokenFilter`的映射关系。

```json
{
	"contexts": {
		"application": {
			"mappings": {
				"dispatcherServlets": {
					"dispatcherServlet": [{
						"handler": "com.macro.mall.tiny.controller.PmsBrandController#createBrand(PmsBrand)",
						"predicate": "{POST /brand/create}",
						"details": {
							"handlerMethod": {
								"className": "com.macro.mall.tiny.controller.PmsBrandController",
								"name": "createBrand",
								"descriptor": "(Lcom/macro/mall/tiny/mbg/model/PmsBrand;)Lcom/macro/mall/tiny/common/api/CommonResult;"
							},
							"requestMappingConditions": {
								"consumes": [],
								"headers": [],
								"methods": [
									"POST"
								],
								"params": [],
								"patterns": [
									"/brand/create"
								],
								"produces": []
							}
						}
					}]
				}
			},
			"servletFilters": [{
				"servletNameMappings": [],
				"urlPatternMappings": [
					"/*",
					"/*",
					"/*",
					"/*",
					"/*"
				],
				"name": "jwtAuthenticationTokenFilter",
				"className": "com.macro.mall.tiny.component.JwtAuthenticationTokenFilter"
			}]
		}
	}
}
```

#### 查看运行时度量

- 通过`/metrics`端点，可以获取应用程序度量指标，不过只能获取度量的名称；

```json
{
    "names": [
        "http.server.requests",
        "jvm.buffer.count",
        "jvm.buffer.memory.used",
        "jvm.buffer.total.capacity",
        "jvm.classes.loaded",
        "jvm.classes.unloaded",
        "jvm.gc.live.data.size",
        "jvm.gc.max.data.size",
        "jvm.gc.memory.allocated",
        "jvm.gc.memory.promoted",
        "jvm.gc.pause",
        "jvm.memory.committed",
        "jvm.memory.max",
        "jvm.memory.used",
        "jvm.threads.daemon",
        "jvm.threads.live",
        "jvm.threads.peak",
        "jvm.threads.states",
        "logback.events",
        "process.cpu.usage",
        "process.start.time",
        "process.uptime",
        "system.cpu.count",
        "system.cpu.usage"
    ]
}
```

- 需要添加指标名称才能获取对应的值，比如获取当前JVM使用的内存信息，访问地址：http://localhost:8088/actuator/metrics/jvm.memory.used

```json
{
    "name": "jvm.memory.used",
    "description": "The amount of used memory",
    "baseUnit": "bytes",
    "measurements": [
        {
            "statistic": "VALUE",
            "value": 3.45983088E8
        }
    ],
    "availableTags": [
        {
            "tag": "area",
            "values": [
                "heap",
                "nonheap"
            ]
        },
        {
            "tag": "id",
            "values": [
                "Compressed Class Space",
                "PS Survivor Space",
                "PS Old Gen",
                "Metaspace",
                "PS Eden Space",
                "Code Cache"
            ]
        }
    ]
}
```

- 通过`loggers`端点，可以查看应用程序中的日志级别信息，可以看出我们把`ROOT`范围日志设置为了INFO，而`com.macro.mall.tiny`包范围的设置为了DEBUG。

```json
{
	"levels": [
		"OFF",
		"ERROR",
		"WARN",
		"INFO",
		"DEBUG",
		"TRACE"
	],
	"loggers": {
		"ROOT": {
			"configuredLevel": "INFO",
			"effectiveLevel": "INFO"
		},
		"com.macro.mall.tiny": {
			"configuredLevel": "DEBUG",
			"effectiveLevel": "DEBUG"
		}
	}
}
```

- 通过`/health`端点，可以查看应用的健康指标。

```json
{
    "status": "UP"
}
```

#### 关闭应用

通过POST请求`/shutdown`端点可以直接关闭应用，但是需要将`endpoints.shutdown.enabled`属性设置为true才可以使用。 

```json
{
    "message": "Shutting down, bye..."
}
```

#### 定制Actuator

有的时候，我们需要自定义一下Actuator的端点才能满足我们的需求。

- 比如说Actuator有些端点默认是关闭的，我们想要开启所有端点，可以这样设置；

```yaml
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

- 比如说我们想自定义Actuator端点的基础路径，比如改为`/monitor`，这样我们我们访问地址就变成了这个：http://localhost:8088/monitor

```yaml
management:
  endpoints:
    web:
      base-path: /monitor
```

### 数据访问与 ORM 框架

参考：

- [Spring Boot JPA 基础：常见操作解析](https://snailclimb.gitee.io/springboot-guide/#/./docs/basis/springboot-jpa)
- [JPA 中非常重要的连表查询就是这么简单](https://snailclimb.gitee.io/springboot-guide/#/./docs/basis/springboot-jpa-lianbiao)

#### JDBC、ORM、JPA

<img src="../assets/v2-7f8d57795eeab6d98d3f956d2d050366_r.jpg" alt="img" style="zoom:50%;" />

1. **JDBC**（`Java DataBase Connectivity`）**API**：是 Java 连接数据库操作的（底层的、原生的）**统一接口标准**。
    - ~~JDBC 对 `Java `程序员而言是API，为数据库访问提供**统一接口标准**。~~由各个**数据库厂商**及第三方中间件厂商依照**JDBC规范**（为数据库连接）提供的标准方法。
    - 通过对应的 `DriverManager`（`MySQL JDBC Driver`、`Oracle JDBC Driver`等）去操作具体的数据库。
    - ~~**JdbcTemplate**~~：是Spring的一部分，是Spring**对JDBC的封装**，目的是使JDBC更加易于使用。处理了资源的建立和释放。帮助我们避免一些常见的错误，比如忘了总要关闭、连接。
2. **ORM**（`Object Relational Mapping`）**框架**：对象–关系映射，用于建立 Java Object **实体类**和数据库表之间的映射，从而达到**操作**实体类就相当于操作数据库表的目的。
    - ORM框架中贯穿着 JAVA 面向对象编程的思想；是对象持久化的核心。
    - 常见的 ORM 框架有：`Hibernate`、`spring data jpa`、`open jpa`。
3. **JPA**（`Java Persistence API`）**API**：Java 持久化 API，是 Java 应用程序访问 ORM 框架的**统一接口标准**/规范），实现用**同样的方式**访问不同的 ORM 框架。
4. **Spring Data JPA**：是`Spring`提供的一套简化`JPA`开发的框架。按照约定好的**方法命名规则**写`DAO`层接口，就可以在**不写接口实现**的情况下，实现对数据库的访问和操作，提供了CRUD、分页、排序、复杂查询等功能。
    - 可以理解为是对`JPA`规范的**再次封装**抽象。不是一个**完整JPA规范**的实现，只是一个代码抽象层，主要用于减少为各种持久层存储实现数据访问层所需的代码量。其底层依旧是 **`Hibernate` 框架**。

关系：

1. JDBC、ORM 框架、JPA 均属于持久化层，介于业务层和数据库之间。
2. jdbc 是**数据库**的统一接口标准；jpa是orm框架的统一接口标准。
3. jdbc更注重数据库，orm则更注重于java代码，但是实际上jpa实现的框架底层还是用jdbc去和数据库打交道。

参考：

- [JDBC、ORM、JPA、Spring Data JPA，傻傻分不清楚？](https://www.cnblogs.com/softwarearch/p/16396480.html)

##### Hibernate 与 MyBatis

1. **Hibernate**：是一个开源、**全自动**化的（标准的）ORM 框架，将 `POJO` 实体类与数据库表建立**映射关系**，可自动生成 SQL 语句并执行。对 JDBC 进行了非常轻量级的对象封装，封装了基本的 DAO 层操作，有较好的数据库移植性。
    - **面向对象**，着力于对象（与对象）间的关系，用于解决计算机逻辑问题，考虑的是对象的整个生命周期（包括对象的创建、持久化、状态的改变和行为等），持久化只是对象的一种状态。

2. **MyBatis**：是一个持久化、**半自动**化的`ORM`框架， ~~只支持将数据库**查询结果**映射到 `POJO` 实体类~~，而 `POJO` 到数据库的映射则需手动编写 `SQL` 语句实现。
    1. 是一个持久化框架，不是依照的`JPA` 规范。
    2. **面向关系模型**，着力于 `POJO` 与 `SQL` 间的映射关系，用于解决**数据的高效存取**问题。
      3. 可用简单的 XML 或注解来配置和映射原生信息，将接口和 `POJO` 映射成数据库中的记录。**主要依赖于** `SQL` 的编写与 `ResultMap` 查询结果集的映射，需额外维护。
      4. ~~支持定制化 `SQL` 优化、存储过程及高级映射。避免了几乎所有的 `JDBC` 代码和手动设置参数及获取结果集。~~

**区别**：查询关联对象（或关联集合对象）时，

1. `Hibernate` 可根据 ORM 对象关系模型直接获取，所以是**全自动**的。
2. 而 `MyBatis` 需**手动编写 SQL** 来完成，所以称为**半自动**的。

选型：

1. 进行**底层**编程，对**性能**要求高，直接用 `JDBC`；
2. 直接操作数据库表，没有过多的定制，用 `Hibernate`；
3. 灵活使用 SQL 语句，用 `MyBatis`；

##### JDBC 访问数据库

JDBC操作的几个关键环节：

1. 引入依赖，配置数据库连接；
2. 根据使用的DB类型不同，加载对应的 JdbcDriver 驱动；
3. 连接 DB；
4. 编写SQL语句；
5. 发送到DB中执行，并接收结果返回；
6. 对结果进行处理解析；
7. 释放过程中的连接资源。

如：

```
<!-- pom.xml 中引入依赖 -->
<!-- 1. 导入JDBC的场景启动器 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jdbc</artifactId>
</dependency>

<!-- 2. 导入mysql数据库驱动 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

`application.yml` 中：

```
# 3. 配置数据源/数据库连接
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/campusdate? \
        useUnicode=true&characterEncoding=UTF-8 \ # 表示用Unicode字符集，指定字符从数据库取出后和存入前的编码、解码格式
        &serverTimezone=Asia/Shanghai
        &useSSL=false \ # 表示在高版本禁用SSL
        &autoReconnect=true\  # 表示当数据库连接异常中断时自动重连；用来配置数据源连接池，控制重连特性
        &failOverReadOnly=false # 表示自动重连成功后，连接不设置为只读
	username: root
	password: root
	driver-class-name: com.mysql.jdbc.Driver ## **
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true # hibernate在操作时在控制台打印真实的sql语句，方便调试
	properties:
	  hibernate:
	    dialect: org.hibernate.dialect.MySQL5Dialect # 表示格式化输出的json字符串
	    #hbm2ddl:
	      #auto: update
## 自动建表（实体类维护数据表）方式
# create 表示每次重启项目（加载 hibernate）时，根据实体类重新生成新表结构，会导致数据丢失；
# create-drop 表示每次加载  hibernate 时，根据实体类生成新表，当 sessionFactory（项目）关闭时自动删除表结构；hsqldb, h2, derby等内嵌数据库默认设置
# update 表示第一次加载 hibernate 时，根据实体类自动建表；以后加载 hibernate 时，表结构随实体类自动更新，且保留数据库中的数据；
# validate 表示每次加载 hibernate 时，只验证实体类和数据表是否一致，不对数据库进行任何更改。
# none 表示啥都不做；其它非内嵌数据库默认设置
```

#### JPA 各层常用注解

[Repository（资源库）接口介绍](http://perfy315.iteye.com/blog/1460226)

控制层：

- `@Controller`
- `@Autowired`：属性上

业务层：

- `@Service`：[Pageable 接口、PageHelper 分页插件](#分页)

Repository 数据接口层：[Spring Boot 中 Repository 的使用](https://segmentfault.com/a/1190000012346333)

1. `@Repository`： 见[声明/注册 Bean 的注解](#声明/注册 Bean 的注解)；
2. `@CrudRepository`：继承了 `Repository` 接口，并新支持对实体类的增删改查等方法；
3. `@PagingAndSortingRepository`：继承了 `CrudRepository` 接口，并新支持分页、排序及根据条件查询等方法；
4. `@JpaRepository` 对应接口：包括 JPA 提供的增删改、分页查询及排序查询等。

```
@Repository
public interface PersonRepository extends JpaRepository<Person, Long> {
}
```

[Spring Boot 使用 JPA](http://www.imooc.com/wiki/springbootlesson/jpa.html) 测试类

##### JPA 常用注解（Entity 层）

[JPA 注解（一） id table entity](http://conkeyn.iteye.com/blog/602463)

Entity 数据持久层：

- `@Entity`：标注在类上，表示数据库持久化类，对应一个数据库实体。
    - `[name]`可选属性，默认为所标注的实体类名。因为用类反射机制 `Class.newInstance()` 方法创建实例的需要，至少有一个无参构造方法。也可标注抽象类。
- `@NamedQuery`：标注在接口的**自定义查询方法**上，指定要执行的查询语句；
- `@NamedQueries`：定义多个；
- `serialVersionUID`：适用于JAVA序列化机制。简单来说，通过判断类的`serialVersionUID`来验证版本一致。
    - 在进行反序列化时，JVM会把传来的字节流中的`serialVersionUID`与本地相应实体类的`serialVersionUID`进行比较。
    - 如果相同说明是一致的，可以进行反序列化，否则会出现反序列化版本一致的异常，即是`InvalidCastException`。

```
@Entity
@NamedQuery(name="UserEntity.findAll", query="SELECT u FROM UserEntity u")
public class UserEntity implements Serializable {}

// 在自己实现的DAO的xxxRepository接口里定义同名方法，先找是否有同名的NamedQuery，
// 如果有，则不按照接口定义的方法解析。
public List<UserModel> findByAge(int age);

@NamedQueries(value = { 
	@NamedQuery(name = User.QUERY_FIND_BY_LOGIN, query = "select u from User u where u." + User.PROP_LOGIN + " = :username"), 
	@NamedQuery(name = "getUsernamePasswordToken", query = "select new com.weibo.vo.TokenBO(u.username, u.password) from User u where u." + User.PROP_LOGIN + " = :username")
	}) 
```

- `@Query`：标注在（继承 `JpaRepository` 接口的）自定义查询方法上，指定要执行的查询语句；
- `@Modifying`：支持更新类的 Query 语句，配合 `@Transactional` [Spring 事务](#Spring 事务) 使用；

```
// like后的参数需在前、后加“%”
// nativeQuery=true表示指定本地查询
@Query(value="select * from tbl_user where name like %?1", nativeQuery=true)
public List<UserModel> findByUidOrAge(String name);

@Modifying
@Query(value = "update UserModel o set o.name=:newName where o.name like %:nn")
public int findByUidOrAge(@Param("nn") String name, @Param("newName") String newName);
```

- `@Table`：标注在类名前，设置数据库表名；
    - `name `属性：表示实体对应表名，默认为实体名；
    - `catalog `和 `schema`（思gay玛）属性：表示目录名或数据库名，根据不同的数据类型有所不同；
    - `uniqueConstraints` 属性：表示该实体所关联的唯一约束条件，可有多个唯一约束；默认没有，需配合`@UniqueContraint`用。

```
@Entity
@Table(name = "tb_contact", schema = "test", uniqueConstraints = {   
        @UniqueConstraint(columnNames = {"name", "email" }),  
        @UniqueConstraint(columnNames = {"col1", "col2" })  
}) 

public class ContactEO implements Serializable {}
```

- `@Temporal`：时间；
- `@Column`：表示持久化属性映射表中的字段。标注在 `getter()` 或属性前。
    1. `unique `属性：字段为**唯一标识**，默认为 false。也可用`@Table` 中的`@UniqueConstraint`，如上。
    2. `nullable `属性：字段可为 null，默认为 true（允许为 null）。
    3. `insertable `属性：在用 “INSERT” SQL 脚本插入数据时，需插入该字段的值。多用于只读属性，如主键和外键等。这些字段值通常是自动生成的。
    4. `updatable`，同上。
    5. `columnDefinition `属性：创建表时，字段对应创建的 SQL 语句，一般用于通过 Entity 生成表**定义**时。如`@Column(columnDefinition = "tinyint(1) default 1")`，设置**字段类型和默认值**。
    6. `table `属性：当映射多个表时，指定表中的字段。默认值为主表名。
    7. `length `属性：字段长度，当字段的类型为 varchar 时才有效，默认为255个字符。
    8. `precision` 和 `scale` 属性：精度；当字段类型为 double 时，precision 表示数值的总长度，scale 表示小数点所占的位数。

```
@Temporal(TemporalType.TIMESTAMP)
@Column(name = "create_time", unique = false)
private Date createTime;
```

- `@Id`：标注在**属性上**，表示该字段对应数据库中的列为主键。[数据类型转换](#数据类型转换)。能标识为主键的属性类型有：
    1. 基本数据类型及其对应的封装类：`Character、Byte、Short、Integer、Long`；
    2. 大数值类型：`java.math.BigInteger`；
    3. 字符串类型：`java.lang.String`；
    4. 时间日期型：
        1. `java.util.Date`
          2. ~~`java.sql.Date`~~
- `@GeneratedValue`：指定主键生成策略。
    - **`strategy`** 属性：表示生成主键的策略，有：
        1. GenerationType.**TABLE**：用一个特定的数据库表格来保存主键；
        2. GenerationType.**SEQUENCE**：用序列机制生成主键，不支持主键自增长，如 Oracle、PostgreSQL；
        3. GenerationType.**IDENTITY**：主键自增长，如 MySQL ；
        4. GenerationType.**AUTO**：默认，把主键生成策略交给持久化引擎，以上三种选一。
    - `generator `属性：为不同策略类型所对应的生成规则名。与`@GenericGenerator`搭配使用。
- `@TableGenerator`
- `@SequenceGenerator`

```
@Id
@GeneratedValue(strategy=GenerationType.AUTO)
private String id;
```

- **`@Transient`**：声明不与数据库映射的字段，**不需保存**进数据库 。
- `@Basic`
    - `fetch` 属性：表示获取值的方式，默认为 EAGER（即时/非延迟加载），LAZY（惰性/延迟加载）。
    - `optional` 属性：表示是否可为 null，不能用于 Java 基本数据类型。
- `@Lob`：声明为大字段。

```java
@Lob
// 指定 Lob 类型数据的获取策略
@Basic(fetch = FetchType.EAGER)
// columnDefinition 属性指定数据表对应的 Lob 字段类型
@Column(name = "content", columnDefinition = "LONGTEXT NOT NULL")
private String content;
```

- `@Enumerated(EnumType.STRING)`：枚举类型的字段；
- `@CreatedDate`、`@CreatedBy`：表示为创建**时间**字段、创建人，在实体被 insert 时会设置值；
    - `@LastModifiedDate`、`@LastModifiedBy`同理；
- `@EnableJpaAuditing`：开启 JPA 审计功能。

#### Lombok 常用注解

`Lombok` （音lao母报ke）插件：Java 语言增强库，**简化** `POJOs` 实体类**封装**。通过为实体类**添加注解**、来自动生成（并代替）通用方法；减少冗余代码。

- 用途：一般用于简单、属性比较多的 `POJOs ` 实体类、子实体类、`DTO `数据传输对象、**请求参数**或返回结果实体类。
- 缺点：提高阅读源码的门槛，继承父类时 `equals()` 可能会出错。

**原理**：在编译器**编译时**通过操作 AST（抽象语法树）改变字节码生成。即改变编写源码的方式，是**编译时**的特性，~~不像 Spring 的依赖注入一样是运行时的特性~~。

1. `@Data`：用在**类上**，相当于同时使用 `@Getter` / `@Setter`、`@ToString`、`@EqualsAndHashCode`、`@RequiredArgsConstructor` 这**5个注解**的组合，生成通常与简单`POJO`关联的所有样板和 `bean`；

    - 因为包含 `@EqualsAndHashcode()` 所以也有以下相同的问题。

2. `@EqualsAndHashCode(callSuper=false)`：用在**类上**，自动生成 `equals()` 方法和 `hashCode()` 方法，包括所有非静态变量和非 `transient` 的变量。

    1. **问题**：默认 `callSuper=false`。默认仅使用子类中定义的属性、不调用**父类**的方法，可能会导致**比较对象时**出错；
        1. 如，有多个类有相同的属性（如 id），把它们提取到父类中作为共同属性；
        2. 因为子类仅比较当前对象的属性、而不比较父类中的 `id`，所以子类对象各属性相等时**其父类 `id`** 可能并不相同。
        3. **lombok 自动生成**的`equals()` 和 `hashCode()` 方法返回 true，判定为相等，但实际上并不相等，从而导致出错。
    2. **修复方法**：
        1. 在使用 `@Data` 的同时，加上 `@EqualsAndHashcode(callSuper=true)`，覆盖 `@Data` 中的，设置让其生成的方法中调用父类方法。
        2. 可用 `@Getter @Setter @ToString` 代替 `@Data`、并自定义 `equals(Object other)` 和 `hashCode()`方法；比如有些类只需要判断主键id是否相等即可。
    3. 另外，Java 规定，重写 `equals()` 必须重写 `hashCode()` 方法；
    4. `@EqualsAndHashCode.Include `和 `@EqualsAndHashCode(onlyExplicitlyIncluded = true)`来精确指定要使用的字段或方法。

3. `@Value`：用在**类上**，和 `@Data`类似，区别在于所有**成员变量**默认定义为 `private final` 修饰，且不生成 `setter()` 方法；与 Spring 中用于绑定 `@Configuration` [配置](#配置绑定)的注解 `@Value` 重名，注意 import 路径；

4. `@Getter` / `@Setter`：用在**类/属性上**；

    1. 若字段的类型为`boolean`，则为`isXxx()`；
    2. 用在类上时，默认 `@Setter(AccessLevel.PUBLIC) `，指定访问级别有`PUBLIC, PROTECTED, PACKAGE, PRIVATE, NONE`。
    3. `@Getter(lazy=true)`：可替代经典的 `Double Check Lock` 样板代码。

5. `@ToString`：用在**类上**，自动覆写 `toString()` 方法，默认打印所有非静态字段；更常用 `@Override` 重写 `toString()` **方法**的方式实现；

    1. 类上用 `@ToString(exclude=”id”)`、字段上用 `@ToString.Exclude`：用于排除 id 属性；
    2. `@ToString(onlyExplicitlyIncluded = true)`，配合用 `@ToString.Include` 在字段上标记要包含的每个字段；
    3. `@ToString(callSuper=true, includeFieldNames=true)`：调用父类的 `toString()`方法，将父类实现的输出包含到当前输出中，包含所有属性。

6. `@NonNull`：用在**方法参数上**，如果为空，则抛出 NPE；

7. `@NoArgsConstructor, @RequiredArgsConstructor, @AllArgsConstructor`：用在**类上**，自动生成构造函数；

    1. 指定 `staticName = "of"`参数：生成返回类对象的静态工厂方法；
    2. `@NoArgsConstructor`：生成无参构造函数。
        1. 如果字段由 final 修饰，则将导致编译器错误，除非使用`@NoArgsConstructor(force = true)`，否则所有 final 字段都将初始化为 `0 / false / null`；
        2. 对于具有约束（如`@NonNull`）的字段，不会生成任何检查。
    3. `@RequiredArgsConstructor`：为每个**需特殊处理**的字段生成有**1个参数**的构造函数。
        1. 所有未初始化的 final 字段；
        2. 所有未声明其位置的、未标记为`@NonNull`的字段。
    4. `@AllArgsConstructor`：为每个字段生成有1个参数的构造函数。标有`@NonNull`的字段将对这些参数进行非空检查。

8. `@Builder`：只能标注到**类上**，生成类当前流程的一种**链式**（流式）构造工厂，多用于配置类。如：

    ```javascript
    User buildUser = User.builder().username("riemann").password("123").build();
    Person.builder().name("Adam").city("San").job("Mythbusters").build();
    ```

9. ~~`@Cleanup`~~：自动管理资源，用在局部变量前，在当前变量范围内即将执行完毕退出前自动清理资源，生成 `try-finally` 这样的代码来关闭流；用于确保已分配的资源被释放（`IO`的连接关闭）。

10. ~~`@SneakyThrows`~~：自动抛出受检异常，而无需显式在方法上使用 `throws` 语句；

11. `@Synchronized`：用在**方法上**，将方法声明为同步的，并自动加锁；自动添加到同步机制，生成的代码并不是直接锁方法，而是锁代码块。

      - 而锁对象是一个私有的属性`$lock`或`$LOCK`，而 `synchronized` 关键字锁对象是this，锁在this或自己的类对象上存在副作用，不能阻止非受控代码去锁this或类对象，这可能会导致竞争条件或其它线程错误；

12. `@Log、@Log4j、@Slf4j`：根据不同的注解生成不同类型的 log 静态常量对象，但实例名称都是 log，有六种可选实现类：

      ```
     // Creates 即 创建的类型，如 private static final java.util.logging.Logger 
     @Log
     Creates log = java.util.logging.Logger.getLogger(LogExample.class.getName());
     @CommonsLog
     Creates log = org.apache.commons.logging.LogFactory.getLog(LogExample.class);
      
     @Log4j
     Creates log = org.apache.log4j.Logger.getLogger(LogExample.class);
     @Log4j2
     Creates log = org.apache.logging.log4j.LogManager.getLogger(LogExample.class);
      
     // Spring Boot 默认，最常用
     @Slf4j
     Creates log = org.slf4j.LoggerFactory.getLogger(LogExample.class);
     @XSlf4j
     Creates log = org.slf4j.ext.XLoggerFactory.getXLogger(LogExample.class);
      ```

13. `@Accessors`：用于配置`lombok`如何生成和查找`getter`和`setter`。

        1. 对`getter`和`setter`的`bean`命名规范；
        2. 参数 `chain=true`时，类的所有属性的`setter`方法返回值为`this`，支持链式写法。

#### 数据类型转换

Java 数据类型与数据库中的类型转换由 JPA 实现框架**自动转换**，所以转换规则也不太一样。如 MySQL 中，varchar 和 char 类型都转化为 String 类型，Blob 和 Clob 类型可转化成 Byte[] 类型。

1. 基本数据类型及其对应的封装类：`Character、Byte、Short、Integer、Long`；
2. 大数值类型
    1. `java.math.BigInteger`
    2. `java.math.BigDecimal`
3. 字节和字符型数组：`byte[]、Byte[]、char[]、Character[]`
4. 字符串类型：`java.lang.String`
5. 日期时间类型
    1. `java.util.Date`
    2. `java.util.Calendar`
    3. `java.sql.Date`
    4. `java.sql.Time`
    5. `java.sql.Timestamp`
6. 用户自定义的枚举型
7. Entity类型：标注为 `@Entity` 的类
8. 包含Entity类型的集合Collection类
    1. `java.util.Collection`
    2. `java.util.Set`
    3. `java.util.List`
    4. `java.util.Map`
9. 嵌入式（embeddable）类

#### 数据库连接池

数据库连接池、数据源自动配置；`DataSourceAutoConfiguration` 类；

1. `Druid`：Alibaba 开源的高性能数据库连接池。加入了强大的监控功能，可实时观察数据库连接池和 SQL 的运行情况，帮用户及时排查出系统中存在的问题。
2. `HikariCP`：Spring Boot 默认数据源；
3. `C3P0`
4. `DBCP`

##### 整合 Druid 连接池

1. 引入依赖；

    ```
    <!--添加 druid 的 starter-->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid-spring-boot-starter</artifactId>
        <version>1.1.17</version>
    </dependency>
    ```

2. 在 `application.yml` 文件中添加数据源配置，会与 Druid 数据源中的属性进行绑定；

3. 自定义方式需创建**数据源配置类**：配置类创建 Druid 数据源对象时，应避免将数据源信息（如 URL、username、password 等）硬编码到代码中，可通过 `@ConfigurationProperties("spring.datasource")` 注解，将数据源属性与配置文件中以 `spring.datasource` 开头的配置绑定。

    ```
    		######## JDBC 通用配置 ############### 
    ...
          	######## Druid 连接池的配置 ###############
    spring:
      datasource:
        druid:
          initial-size: 5 # 初始化连接大小
          min-idle: 5 # 最小连接池数量
          max-active: 20 # 最大连接池数量
          max-wait: 60000 # 获取连接时最大等待时间，毫秒
          time-between-eviction-runs-millis: 60000 # 间隔多久检测一次需关闭的空闲连接，毫秒
          min-evictable-idle-time-millis: 300000 # 连接在池中最小生存时间，毫秒
    
          filters: stat,wall # 配置扩展插件，常用的有stat:监控统计，wall:防sql注入
          connection-properties: ’druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000‘ # 打开mergeSql功能;慢SQL记录
    ```

4. Spring Boot 提供的默认测试类；

    ```
    package net.biancheng.www;
    
    import org.junit.jupiter.api.Test;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.context.SpringBootTest;
    import org.springframework.jdbc.core.JdbcTemplate;
    
    import javax.sql.DataSource;
    import java.sql.SQLException;
    
    @SpringBootTest
    class SpringBootAdminexApplicationTests {
        // 数据源组件
        @Autowired
        DataSource dataSource;
        // 用于访问数据库的组件
        @Autowired
        JdbcTemplate jdbcTemplate;
        
        @Test
        void contextLoads() throws SQLException {
            System.out.println(”默认数据源为：“ + dataSource.getClass());
            System.out.println(”数据库连接实例：“ + dataSource.getConnection());
            // 访问数据库
            Integer cnt = jdbcTemplate.queryForObject(”SELECT count(*) from `user`“, Integer.class);
            System.out.println(”user 表中共有“ + cnt + ”条数据。“);
        }
    }
    ```

5. 内置提供的名为 `StatViewServlet` 的 `Servlet`，可开启 Druid 内置监控页面功能， 展示 Druid 的统计信息；需将该 `Servlet `配置在 Web 应用中的 WEB-INF/web.xml 中；

6. 内置提供的 StatFilter，可开启 Druid 的 SQL 监控功能；

7. 内置提供了 WallFilter，可开启防火墙功能，防御 SQL 注入攻击。

### 整合 Hibernate Validator

> 整合 Hibernate Validator 加强了参数验证，用注解实现参数验证，极大简化了代码，验证更简洁方便。

参考：[Java 参数校验validation和validator区别](https://blog.csdn.net/m0_37583655/article/details/124500769)；

#### 参数校验方式

对于（Controller 接收的参数、内部方法接口的形参赋值等）进行**统一参数校验**，方式有：

1. 一般对于**复杂的业务参数**校验：可通过**校验类**单独的校验方法进行处理；

    1. 常规做法：业务逻辑中要（手动）加数据校验后，再把数据写入数据库；校验失败抛出 `IllegalArgumentException`；
    2. 缺点：代码冗余臃肿复杂，每个人抛出的异常都不同。

     ```
     // 参数很多，入参用对象封装起来
     public void validate(UserDTO userDTO) {
         Long uid = userDTO.getUid();
         String username = userDTO.getName();
         if (uid == null || uid == 0L) {
             throw new IllegalArgumentException("uid 不能为空");
        }
         if (StringUtils.isBlank(username)) {
             throw new IllegalArgumentException("username 不能为空");
         }
     }
     ```

2. 通常对于**与业务无关**、简单的参数校验：可采用 `javax.validation` 和 `hibernate-validator` **验证框架**，通过注解的方式实现校验。

    1. `javax.validation`：是 Java 定义的一套基于注解的参数验证规范，属于 JSR（Java Specification Requests 规范）；只是一项**标准**，规定了一些校验注解的**规范**，但没有实现，如 `@Null、@NotNull、@Pattern` 等，位于 `javax.validation.constraints` 包下；
    2. `hibernate validator`：是对这个规范的**实现**，并增加了一些其他校验注解，如 `@NotBlank、@NotEmpty、@Length` 等，位于 `org.hibernate.validator.constraints` 包下。

     > 推荐首选用 `validation`  注解，再搭配 `hibernate validator` 的其它注解。

     ```
     import javax.validation.constraints.NotNull;
     import org.hibernate.validator.constraints.NotBlank;
     ```

##### Hibernate 校验模式

1. 普通模式（默认）：校验所有属性，返回所有的验证失败信息；
2. 快速失败返回模式：只要有一个校验失败就返回。

```
// 设置failFast方式: true 快速失败返回模式，false 普通模式
ValidatorFactory validatorFactory = Validation.byProvider( HibernateValidator.class )
    .configure()
    .failFast(true)
    .buildValidatorFactory();
Validator validator = validatorFactory.getValidator();
```

#### 引入依赖

1. 如果项目使用了 Spring Boot，`spring-boot-starter-web` 包中已集成了 `hibernate-validator` 框架，无需再添加 `hibernate validator` 依赖。

    ```
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    ```

2. 如果不是 Spring Boot 项目，需添加 `hibernate.validator` 依赖。

    ```
    <dependency>
        <groupId>org.hibernate.validator</groupId>
        <artifactId>hibernate-validator</artifactId>
        <version>6.0.13.Final</version>
    </dependency>
    ```

#### POJO 类

> 如用于 DTO 类。

在 POJO 实体类上加上具体的约束注解。

```
public class Prop {
    @NotNull(message = "pid不能为空")
    @Min(value = 1, message = "pid必须为正整数")
    private Long pid;

    @NotBlank(message = "pidName不能为空")
    private String pidName;
    
    @PastOrPresent
   	private LocalDateTime createtime;
}

@Getter
@Setter
@NoArgsConstructor
public class Item {
	// id 属性在不同 controller 中的约束不一样，不能在 POJO 中约束，需用到分组；
	// controller 新增数据时，要求前端传入的id必须为空，因为id在数据库里是自增字段，不支持手动赋值；
	@Null(mesage = "添加时id必须为空", groups = {Item.add.class})
	// Controller 更新数据时，要求前端传入的id不能为空；
    @NotNull(message = "修改时id不能为空", groups = {update.class})
    @Min(value = 1, message = "id必须为正整数")
    private Long id;
    
    @NotBlank(message = "年龄不能为空")
    @Pattern(regexp = "^[0-9]{1,3}$", message = "年龄不正确")
    private String age;

	@Valid // 嵌套验证，必须用@Valid标记嵌套成员对象，且@Validated不能用在成员属性上；
	// 不加则不会对props里的Prop对象进行字段验证，即入参验证不检查pid和pidname
    @NotNull(message = "props不能为空")
    @Size(min = 1, message = "至少有一个属性")
    private List<Prop> props;
    
    public interface add{};
    
    public interface update{};
}
```

#### 字段验证注解

> 用于校验 Controller 方法中用 `@Valid、@Validated` 修饰的参数（实体类对象）。

##### `javax.validation` 基础校验注解

> JDK 提供的。

1. `@NotNull(message = "name不能为空")`：元素不能为 **null**；可为 empty（没有 size 的约束）；

    1. `@Null`：元素必须为 null；
    2. ~~`@NotEmpty`~~：用于**字符串**、集合、Map、数组，不能为 **null 或 空** `empty（size>0）`（String、Collection、Map 的 `isEmpty() `方法）；如 `username`。
    3. ~~`@NotBlank`~~：只用于 String，不仅不能为 null，而且必须至少包含一个**非空白字符**（`trim()` 后 size>0）；

    ```
    String name1 = null;
    String name2 = "";
    String name3 = " ";
    // 		   name1, name2, name3
    @NotNull:  false, true,	 true
    @NotEmpty: false, false, true
    @NotBlank: false, false, false
    ```

2. `@AssertTrue`/`@AssertFalse`：必须为 true/ false；

3. `@Size(min=4, max=15)`：元素大小必须在指定范围内；可是字符串、数组、集合、map 等；

    1. `@Min(value)`/`@Max(value)`：必须是一个 Long 类型的数字，值必须 `>= / <=` 指定值；
    2. `@DecimalMin(value)`/`@DecimalMax(value)`：必须是一个（BigDecimal 的字符串表示形式的） `>= / <=` **指定值**的数字，可以是小数。

4. `@Digits(integer, fraction)`：必须是一个**数字**，其值必须在可接受的**范围**内；

    - `@Positive`：必须是正数；

5. `@Past`/`@Future`：必须是一个过去/将来的日期。

    - `@PastOrPresent`：

6. `@Pattern(value)`：被注释的元素必须符合指定的正则表达式；

##### `hibernate validator` 支持的**基础注解**

> Spring 等提供。

1. `@NotEmpty`
2. `@NotBlank`
3. `@Length(min=,max=)`：检查所属字段的长度是否在 min 和 max 之间，只能用于字符串；
4. `@Email`：必须是 Email 格式；
    1. `@CreditCardNumber`：对信用卡进行一个大致的校验；
    2. `@URL(protocol=, host=, port=)`：检查是否是有效的 URL，满足 protocol，host 等；
5. `@Range(min=, max=, message=)`：必须在指定的范围内
6. `@Pattern(regexp=, flag=)`：必须符合指定的正则表达式；

对于**日期格式化**，针对 `Date` 类型字段：

1. 接收前端校验注解，`@DateTimeFormat` 接受前台的时间格式传到后台的格式；
2. 后端返回给前端日期格式化，使用场景数据库存储的是 `yyyy-MM-dd HH:mm:ss`，但前端需要 `yyyy-MM-dd` 时可用 [`@JsonFormat`](#AJAX、JSON)。

```
@DateTimeFormat(pattern="yyyy-MM-dd")
private Date born;

@JsonFormat(timezone = "GMT+8", pattern = "yyyy-MM-dd")
private Date born;
```

#### 自定义校验注解

可以按照 `@NotNull` 等基础校验注解源码的写法。用 `@Retention`。

```
@Target({ElementType.METHOD, ElementType.FIELD, ElementType.ANNOTATION_TYPE, ElementType.CONSTRUCTOR, ElementType.PARAMETER, ElementType.TYPE_USE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Constraint(
        validatedBy = {IsMobileValidator.class}
)
public @interface IsMobile {
    boolean required() default true;
    String message() default "手机号码格式错误";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

#### Controller 类

请求参数校验。

```
@RestController
public class ItemController {
    @PostMapping("/item/add")
    // @Validated 修饰入参；多个参数可添加多个 @Valid 和 BindindResult。
    // ({Item.add.class} 分组验证？
    public void addItem(@RequestBody @Validated({Item.add.class}) Item item, BindingResult result, [@RequestBody @Valid Item item2, BindingResult result2]) {
       	if (result.hasErrors()) {
       		for (ObjectError errors : result.getAllErrors()) {
       			System.out.println(error.getDefaultMessage());
       		}
       	}
    }
    
    @PostMapping("/item/update")
    public Object updateItem(@RequestBody @Validated({Default.class}) Item item, BindingResult bindingResult) {
    	return "update ok";
    }
}

@RestController
@RequestMapping("/api")
@Validated // 标注此类需校验
public class PersonController {
    @GetMapping("/person/{id}")
    // 嵌套验证：@Valid 修饰入参，表示校验实体类的属性
    public ResponseEntity<Integer> getPersonByID(@Valid @PathVariable("id") @Max(value=5, message="超过id范围") Integer id) {
        return ResponseEntity.ok().body(id);
    }
}
```

#### @Valid VS @Validated

作用：用于**验证注解**是否符合要求、统一参数校验。

- 直接加在变量、参数之前，在变量中添加验证信息的要求（用`@NotNull`等[字段验证注解](#字段验证注解)），当不符合要求时就会在方法中返回 message 中的错误提示信息。

##### `@Valid`：

1. 所属包：属于 `javax.validation` 包下，是 **JDK** 提供的；
2. 可标注在：构造器、方法和方法参数、**成员属性**上；
3. 不支持**分组验证**；
4. 级联校验/~~嵌套验证~~：用于标注嵌套对象（包含其他对象的数组、集合、map、对象的引用）或其内部（另一对象）的**成员属性**，进行属性验证；
    - 用于**内部时**，外部可配合`@Valid` 或 `@Validated` 进行嵌套验证；
    - 在检查当前对象的同时也会检查**该字段所引用的**对象；
5. **异常**：如果验证失败，将抛出 `MethodArgumentNotValidException`。

##### `@Validated`：

> 比 `@Valid` 更强大；

1. 所属包：属于 `org.springframework.validation.annotation` 包下，是 **Spring** 提供的；
2. 可标注在：**类**、方法和方法参数上，不能用在**成员属性**上；
3. 额外支持**分组验证**：在方法参数验证时，根据不同的分组采用不同的验证机制。
    1. 不同 Controller 对于同一 POJO 实体类的属性约束可能不一样。如，
        1. 当新增数据时，因为 **id 在数据库里是自增字段**，不支持手动赋值，需要 id 属性为空（`@Null`）；
        2. 当更新数据时，需要 id 属性不为空（`@NotNull`），才能确定更新哪一条记录。
    2. 首先在实体类中定义**组别接口**进行区分，然后在注解中加上对应的组别接口，不加则是默认组别；如 `@Validated({Item.add.class}) Item item`？
    3. 因为 `@Valid` 不支持分组，所以用 `@Validated` 标注 Controller 方法的形参；并在 `@Validated` 中指定分组，不在指定组中的其他校验不会生效，还需把其他属性的校验一并加入到组中；
4. **嵌套验证**：用于标注嵌套对象属性；内部需配合 `@Valid` 进行嵌套验证；

## 整合 Redis

> 以短信验证码为例。security 权限。

#### 在 `pom.xml` 添加 Maven 依赖

```xml
<dependency>
    <groupId>com.aliyun</groupId>
    <artifactId>dysmsapi20170525</artifactId>
    <version>2.0.24</version>
</dependency>
```

#### Redis 配置

在 SpringBoot 配置文件 `application.yml` 中：

- 在 `spring` 节点下添加 Redis 连接配置；

```
  redis:
    host: localhost # Redis服务器地址
    database: 0 # Redis数据库索引（默认为0）
    port: 6379 # Redis服务器连接端口
    password: # Redis服务器连接密码（默认为空）
    jedis:
      pool:
        max-active: 8 # 连接池最大连接数（使用负值表示没有限制）
        max-wait: -1ms # 连接池最大阻塞等待时间（使用负值表示没有限制）
        max-idle: 8 # 连接池中的最大空闲连接
        min-idle: 0 # 连接池中的最小空闲连接
    timeout: 3000ms # 连接超时时间（毫秒）
```

- 在根节点下添加 Redis 自定义 key 的配置：前缀 String、过期时间；

```
# 自定义redis key
redis:
  key:
    prefix:
      authCode: "portal:authCode:"
    expire:
      authCode: 120 # 验证码超期时间
```

#### 添加 `RedisService` 接口及其实现类

- 用于定义常用 Redis 操作：`get()、set()、expire()`等；

- 注入 `StringRedisTemplate`，实现接口；

```
@Autowired
private StringRedisTemplate redisTemplate;

@Override
public Object get(String key) {
 	return redisTemplate.opsForValue().get(key);
}
```

#### 添加 `UmsMemberService` 接口及其实现类

1. 生成验证码时，将自定义的 **Redis 键值 + 手机号**生成一个 Redis 的 key，
2. 发送验证码：创建发送短信的客户端，提供参数调用对应方法即可。 
    - 提供阿里云账号（AccessKey ID）和密码（AccessKey Secret）登录后，选择对应的**开发测试短信签名**和模版，填充对应的短信内容（模版参数），将短信发送到指定的手机号。
3. 发送成功后，以验证码为 value 存入到 Redis Session 中，并设置过期时间（如120s）；
4. 校验验证码时，检查传入的验证码格式，根据手机号码来获取 Redis 里存储的验证码、及过期时间，与传入的比对。

> Service 接口返回 String + Controller 返回 CommonResult； 

```
package com.macro.mall.tiny.service.impl;

import com.macro.mall.tiny.common.api.CommonResult;
import com.macro.mall.tiny.service.RedisService;
import com.macro.mall.tiny.service.UmsMemberService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import org.springframework.util.StringUtils;

import java.util.Random;

/**
 * 会员管理Service实现类
 */
@Service
public class UmsMemberServiceImpl implements UmsMemberService {
    @Autowired
    private RedisService redisService; //
    
    @Value("${redis.key.prefix.authCode}")
    private String REDIS_KEY_PREFIX_AUTH_CODE; // 前缀
    @Value("${redis.key.expire.authCode}")
    private Long AUTH_CODE_EXPIRE_SECONDS; // 过期时间

    @Override
    public String generateAuthCode(String telephone) {
        StringBuilder sb = new StringBuilder();
        Random random = new Random();
        for (int i = 0; i < 6; i++) {
            sb.append(random.nextInt(10));
        }
        //验证码绑定手机号并存储到redis
        redisService.set(REDIS_KEY_PREFIX_AUTH_CODE + telephone, sb.toString());
        redisService.expire(REDIS_KEY_PREFIX_AUTH_CODE + telephone, AUTH_CODE_EXPIRE_SECONDS);
        
        return sb.toString();
    }

    //对输入的验证码进行校验
    @Override
    public boolean verifyAuthCode(String telephone, String authCode) {
        if (StringUtils.isEmpty(authCode)) {
            return CommonResult.failed("请输入验证码");
        }
        String realAuthCode = redisService.get(REDIS_KEY_PREFIX_AUTH_CODE + telephone);
        result authCode.equals(realAuthCode);
    }
}
```

发送短信：

```
package com.aliyun.sample;

import com.aliyun.teaopenapi.models.Config;
import com.aliyun.dysmsapi20170525.Client;
import com.aliyun.dysmsapi20170525.models.SendSmsRequest;
import com.aliyun.dysmsapi20170525.models.SendSmsResponse;
import static com.aliyun.teautil.Common.toJSONString;

public class Sample {
    public static Client createClient() throws Exception {
        Config config = new Config()
                // 配置 AccessKey ID，请确保代码运行环境设置了环境变量。
                .setAccessKeyId(System.getenv("ALIBABA_CLOUD_ACCESS_KEY_ID"))
                // 配置 AccessKey Secret，请确保代码运行环境设置了环境变量。
                .setAccessKeySecret(System.getenv("ALIBABA_CLOUD_ACCESS_KEY_SECRET"));
                // System.getenv()方法表示获取系统环境变量，请配置环境变量后，在此填入环境变量名称，不要直接填入AccessKey信息。
        
        // 配置 Endpoint
        config.endpoint = "dysmsapi.aliyuncs.com";

        return new Client(config);
    }

    public static void main(String[] args) throws Exception {
        // 初始化请求客户端
        Client client = Sample.createClient();

        // 构造请求对象，请填入请求参数值
        SendSmsRequest sendSmsRequest = new SendSmsRequest()
                .setPhoneNumbers("1390000****")
                .setSignName("阿里云")
                .setTemplateCode("SMS_15305****")
                .setTemplateParam("{\"name\":\"张三\",\"number\":\"1390000****\"}");

        // 获取响应对象
        SendSmsResponse sendSmsResponse = client.sendSms(sendSmsRequest);

        // 响应包含服务端响应的 body 和 headers
        System.out.println(toJSONString(sendSmsResponse));
    }
}
```

接口：

```
package com.macro.mall.tiny.service;

import com.macro.mall.tiny.common.api.CommonResult;

/**
 * 会员管理Service
 */
public interface UmsMemberService {

    /**
     * 生成验证码
     */
    String generateAuthCode(String telephone);

    /**
     * 判断验证码和手机号码是否匹配
     */
    boolean verifyAuthCode(String telephone, String authCode);

}
```

#### 添加 `UmsMemberController`

1. 添加 `UmsMemberController`，根据电话**发送验证码**的接口和**校验验证码**的接口；
2. ~~`@Redis`~~：控制层方法中 **`@Redis` 标注**的参数（如，`@Redis(key = "redisKey") String redisValue` ），值应从 Redis 中获取，不用从请求参数中获取。

<img //src="../assets/v2-a226f96dff62a33e3b705e07b51dc319_720w.jpg" alt="img" />

```
package com.macro.mall.tiny.controller;

import com.macro.mall.tiny.common.api.CommonResult;
import com.macro.mall.tiny.service.UmsMemberService;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

/**
 * 会员登录注册管理Controller
 */
@Controller
@Api(tags = "UmsMemberController", description = "会员登录注册管理")
@RequestMapping("/sso")
public class UmsMemberController {
    @Autowired
    private UmsMemberService memberService;

	@ApiOperation("发送验证码")
    @RequestMapping(value = "/sendAuthCode", method = RequestMethod.GET)
    @ResponseBody
	public CommonResullt sendAuthCode(@RequestParam String telephone) {
		// 发送次数的限制，与上次发送间隔时间
        String authCode = memberService.generateAuthCode(telephone);
        String result = memberService.sendAuthCode(authCode);
        return CommonResult.success(result.toString(), "发送验证码成功");
	}
	// 使用阿里云SDK发送短信验证码，具体方法和参数请参考阿里云SDK文档
	public void sendSmsVerificationCode(String phoneNumber, String code) {
    // 调用短信发送API发送短信验证码
    // ...
	
    @ApiOperation("获取验证码")
    @RequestMapping(value = "/getAuthCode", method = RequestMethod.GET)
    @ResponseBody
    public CommonResult getAuthCode(@RequestParam String telephone) {
        String result = memberService.generateAuthCode(telephone);
        return CommonResult.success(result.toString(), "获取验证码成功");
    }

    @ApiOperation("判断验证码是否正确")
    @RequestMapping(value = "/verifyAuthCode", method = RequestMethod.POST)
    @ResponseBody
    public CommonResult updatePassword(@RequestParam String telephone,
                                 @RequestParam String authCode) {
        boolean result = memberService.verifyAuthCode(telephone,authCode);
        if (result) {
            return CommonResult.success(null, "验证码校验成功");
        } else {
            return CommonResult.failed("验证码不正确");
        }
    }
}
```

## 整合 ELK 统一日志框架

> 其中log4j和commons-logging都是apache软件基金会的开源项目。

Java中给项目程序添加log主要有三种方式：

1. JDK中的`java.util.logging`包：JDK标准库中的类，是JDK 1.4 版本之后添加的日志记录的功能包。
2. log4j：最强大的记录日志的方式。可以通过配置 .properties 或是 .xml 的文件， 配置日志的目的地，格式等。
3. commons-logging：，最综合和常见的日志记录方式，是Java中的一个日志接口，一般会与log4j一起使用。自带SimpleLog可用于日志记录。

### 日志门面（框架）分类

应用中不可直接用日志系统中的 API，而应使用门面模式的日志框架，（解耦）有利于维护各个类的日志处理方式统一。

MyBatis 通过内置的日志工厂提供日志功能。

日志系统：JUL、~~Log4j、~~Log4j2、**Logback** 等。

日志门面（框架）：

1. `Spring Boot`：默认用 **SLF4J 日志门面 + Logback 日志系统**实现的组合来搭建日志系统，可通过配置整合 **ELK**；
2. `Spring`：`Apache Commons Logging`，原名 JCL（Jakarta Commons Logging）；应用部署在一个类路径已包含 ACL 的环境中（如 Tomcat 和 WebShpere 应用服务器），MyBatis 会把它作为日志工具，而不是自定义的其它日志工具，需显式地在 MyBatis 配置文件（mybatis-config.xml）里用 `<setting name="logImpl" value="LOG4J"/>` 配置。
3. `Hibernate`：`jboss-logging`。

### ELK

ELK 是指 `Elasticsearch + Kibana + Logstash` 这三种服务搭建的日志处理系统。

1. `Logstash`：用于**收集**日志。SpringBoot 应用整合了 Logstash 以后、会把日志发送给Logstash，再把日志转发给Elasticsearch；
    - SpringBoot 通过配置 LogBack `logback-spring.xml` 的 `<property> <appender> <logger>` 标签，从定义的输入源 inputs（stdin、日志文件、数据库等）读取信息，经过 filters 过滤器处理，输入到定义好的 outputs 输出源（stdout、elesticsearch、HDFS等）。[springboot 集成 LogStash 的步骤](https://www.yisu.com/zixun/89177.html)
2. `Elasticsearch`：用于**存储**收集到的日志信息；以及生成索引数据，便于Kibana做检索。
3. `Kibana`：通过 Web 端（的可视化）界面来**展示分析、查询检索**日志。

<img src="../assets/ELK.jpg" alt="springboot集成LogStash的步骤" style="zoom:60%;" /><img src="../assets/2729274-20230205171722024-1222796164.png" alt="img" style="zoom:40%;" />

##### 用法

安装 elasticsearch **ik 中文分词**：https://release.infinilabs.com/analysis-ik/stable/ 下载解压。

```shell
JAVA_HOME D:\Develop\Java\jdk\jdk1.8.0_311;D:\Develop\Java\jdk\jdk-17

# not use
cd elasticsearch-6.2.2/bin/
elasticsearch-plugin install https://get.infini.cloud/elasticsearch/analysis-ik/6.2.2
//elasticsearch-6.2.2\plugins\analysis-ik
```

~~需要对Logstash配置单独的Java环境~~，只需要分别在如下两个配置文件里面配置Java环境即可：

1. ~/logstash-5.2.0/bin/logstash

2. ~/logstash-5.2.0/bin/logstash.lib.sh

```
#在行首添加
export JAVA_HOME="D:\Develop\Java\jdk\jdk1.8.0_311"
export PATH="$PATH:$JAVA_HOME\bin"
```

logstash 启动：

```shell
cd D:\Develop\Env\logstash-6.2.2\bin
./logstash -f logstash.conf  &
```

### Logback 日志系统

##### Spring Boot 配置文件

logging 部分配置

```yaml
# application-dev.yml
logging:
  level:
    root: info # 设置应用的日志打印级别
    com.macro.mall: debug
    
logstash:
  host: localhost

---
# application-prod.yml
logging:
  file:
    path: /var/logs
  level:
    root: info
    com.macro.mall: info
    
logstash:
  host: logstash

---
# application.property格式
# 设置应用的日志打印级别
logging.level.com.glmapper.spring.boot=INFO
# 输出位置
logging.path=./logs
```

##### `logback.xml` 配置文件

文件位置：`src/mian/resource/logback-spring.xml`

想用 Spring 扩展 profile 支持，要以 **`logback-spring.xml`** 命名，其他如 `property `需改为`springProperty`。

文件中主要标签有：

1. `<configuration>`

2. `<include>`：导入其他项目配置的 logback.xml 文件，[Configure Logback for Logging console-appender.xml](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto.logging.logback)

    ```
    <!—引用默认日志配置—>
    <include resource=”org/springframework/boot/logging/logback/defaults.xml“/>
    <!—使用默认的控制台日志输出实现—>
    <include resource=”org/springframework/boot/logging/logback/console-appender.xml“/>
    ```

3. `<property name="" value="">`：用来定义变量，可用`${name}`变量占位符将值插入到`logger`上下文中。

4. `<springProperty name="" scope="" source="" defaultValue="">`：

5. `<appender name="" class="">`：**日志打印组件**。让应用知道**怎么打**、打印到哪里、打印成什么样。通过`logger`或`root`的`appender-ref`指定某个具体的`appender`；

    - `<filter>`：作为**过滤器**；可用任意条件对日志进行过滤。

6. `<logger name="org.mybatis.example.BlogMapper">`：用来设置**某个包**、或具体**某个类**的日志打印级别、及指定`appender`。告诉应用哪些按照哪个appender 打印。

7. `<root level="">`：根 logger，设置**日志打印级别**。也是一种 logger，且只有一个level属性。

##### 日志文件隔离

日志 LEVEL 分类：DEBUG、INFO、WARN、ERROR。

logback 日志输出位置：

1. 通过**控制台**打印日志

2. 直接输出到**日志文件**
3. 输出到 **`LogStash`**

> [看完这个不会配置 logback ，请你吃瓜！](https://juejin.cn/post/6844903641535479821)

```
private static final Logger LOGGER = LoggerFactory.getLogger(HelloController.class);

@Autowired
private TestLogService testLogService;

//一、通过控制台打印日志
@GetMapping("/hello")
public String hello(){
    LOGGER.info("this is info");
    LOGGER.error("this is error");
    testLogService.printLogToSpecialPackage();
    return "hello spring boot";
}
```

### 用 AOP 输出日志

> 结合项目说下，怎么用 AOP 输出日志。

1. 定义 AOP `WebLogAspect` 统一日志处理切面类，并加 `@Aspect、@Component` 注解；
2. 在连接点（方法）上加 `@Pointcut("execution(public * com.*.controller.*.*(..)))` 切点注解，通过切点表达式、指定日志切面的应用范围是所有 Controller 层的接口方法；
3. 定义通知方法，描述了切面**要完成的工作**；
4. 在通知方法上加注解 `@Around、@Before、@AfterReturning、@AfterThrowing` 指定通知**何时**执行。如，日志切面需在接口**调用前后**分别记录当前时间，取差值计算调用时长。

```
package com.macro.mall.common.log;

/**
 * 统一日志处理切面
 */
@Aspect
@Component
@Order(1)
public class WebLogAspect {
    private static final Logger LOGGER = LoggerFactory.getLogger(WebLogAspect.class);

    @Pointcut("execution(public * com.macro.mall.controller.*.*(..))||execution(public * com.macro.mall.*.controller.*.*(..))")
    public void webLog() {
    }

    @Before("webLog()")
    public void doBefore(JoinPoint joinPoint) throws Throwable {
    }

    @AfterReturning(value = "webLog()", returning = "ret")
    public void doAfterReturning(Object ret) throws Throwable {
    }

    @Around("webLog()")
    public Object doAround(ProceedingJoinPoint joinPoint) throws Throwable {
        long startTime = System.currentTimeMillis();
        ...
    }
}
```

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

```
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

##### `CommentGenerator` 自定义注释生成器

```
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

## 整合 Spring Security 权限

> 整合 Spring Security 和 JWT 实现用户的**登录和授权功能**，同时改造 `Swagger-UI` 的配置使其可自动发送**登录令牌**。

#### 在 `pom.xml` 中添加项目依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

#### Spring Boot 配置 Spring Security

在Spring Boot应用程序中，可以通过在`application.properties`文件或`application.yml`文件中配置Spring Security。

这里的配置表示启用基于HTTP基本身份验证的Spring Security，并指定了一个用户名为admin，密码为password的用户。

```
jwt:
  tokenHeader: Authorization #JWT存储的请求头
  secret: mall-admin-secret  #JWT加解密使用的密钥
  expiration: 604800    #JWT的超期限时间(60*60*24*7)
  tokenHead: 'Bearer '  #JWT负载中拿到开头

spring:
  security:
    user:
      name: admin
      password: password
    basic:
      enabled: true
```

#### （一）添加 `Spring Security` 的配置类

`config/SecurityConfig.java`：

```
public class SecurityConfig extends WebSecurityConfigurerAdapter {
}

//类上加注解
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class MallSecurityConfig extends SecurityConfig {
}
```

自定义的 Spring Security 安全配置类  `SecurityConfig`，可以通过继承`WebSecurityConfigurerAdapter`类来实现。

1. `configure(AuthenticationManagerBuilder auth)`：用于配置 `UserDetailsService` 及`PasswordEncoder`；`AuthenticationManager`：
2. `UserDetailsService`：SpringSecurity 定义的**核心接口**，用于根据用户名获取用户信息，需要自行实现；
    2. `UserDetails`：SpringSecurity 定义用于**封装用户信息**的类（主要是用户信息和权限），需要自行实现；
    3. `PasswordEncoder`：用于对密码进行编码及**校验**比对的接口，目前使用的是`BCryptPasswordEncoder` 哈希加密算法；

3. `configure(HttpSecurity httpSecurity)`：用于读取并配置需拦截的 URL 路径、**白名单路径**，是否允许跨域（的OPTIONS请求），配置权限 JWT 过滤器、权限**拒绝处理器**、抛出异常后的处理器、动态权限**校验过滤器**等；
4. 添加 `JwtAuthenticationTokenFilter` 类：自定义 JWT 登录**授权过滤器**/拦截器，（在用户名和密码校验前）添加**过滤器** ，如果请求中有 JWT 的 token 且**验证**有效，会自行根据token信息进行登录。

    - 会取出 token 中的用户名，创建 `UsernamePasswordAuthenticationToken` 实例，并保存至安全上下文。

    - 调用 `UserDetailsService`、`UserDetails`、`PasswordEncoder` 验证、取出；
        - `SecurityContextHolder.getContext().getAuthentication() == null`
        - `SecurityContextHolder.getContext().setAuthentication(UsernamePasswordAuthenticationToken);`：将 Authentication 保存至**安全上下文**。

5. 添加 `RestfulAccessDeniedHandler` 类：自定义权限**拒绝处理器**，当用户**没有访问权限**时的处理器，用于返回JSON格式的处理结果；

    - ~~`RestAuthenticationEntryPoint`~~：当未登录或token失效（登录过期）时，返回JSON格式的结果。

6. 添加 `DynamicSecurityFilter` 类：动态权限**校验过滤器**，用于实现基于路径的动态权限过滤；

    1. `DynamicAccessDecisionManager` 类：动态权限**决策**管理器，用于判断用户是否有访问权限；
        2. `DynamicSecurityMetadataSource` 类：动态权限数据源，用于获取动态权限**规则**；
            - 实现 `DynamicSecurityService` 动态权限相关**业务接口**：读取动态权限配置；加载资源ANT通配符和资源对应MAP；

<img src="../assets/8fdccbc24a2225db315a8800cdfe1c09.png" alt="img" style="zoom:70%;" />

#### 添加 `util.JwtTokenUtil` 工具类

> 用于生成、解析、验证`JwtToken ` 的工具类；

相关方法说明：

- `generateToken(UserDetails userDetails)`：用于根据登录用户信息生成token；
- `getUserNameFromToken(String token)`：从token中获取登录用户的信息；
- `validateToken(String token, UserDetails userDetails)`：判断token是否还有效；

#### （二）登录注册功能实现

> 默认主包 `com.macro.mall`。

1. 添加用户数据库表、及 `model `类 `com.macro.mall.model.UmsAdmin`；
2. 添加 `com.macro.mall.bo.AdminUserDetails`：Spring Security 需要的用户详情；
3. 添加 `UmsAdminController` 类：实现用户登录、注册及获取权限的接口；为接口中的方法添加**访问权限**。
4. 添加 `UmsAdminService` 接口及其实现类：**核心接口**，用于根据用户名获取用户信息，需自行实现；
    - `UmsAdminCacheService`：用户缓存操作Service实现类，存在 `RedisService `中。

![img](../assets/1218593-20220809143059104-330636126.webp)

给PmsBrandController接口中的方法添加访问权限：

- 给查询接口添加`pms:brand:read`权限
- 给修改接口添加`pms:brand:update`权限
- 给删除接口添加`pms:brand:delete`权限
- 给添加接口添加`pms:brand:create`权限

```
// 给 `PmsBrandController` 接口中的方法添加访问权限
@PreAuthorize("hasAuthority('pms:brand:read')")
public CommonResult<List<PmsBrand>> getBrandList() {
	return CommonResult.success(brandService.listAllBrand());
}
```

#### Spring Security 整合 Swagger3

> 通过修改配置实现调用接口自带Authorization头，这样就可以访问需要登录的接口了。

1. **认证方式一**、修改 Swagger 的配置，实现调用接口自带 `Authorization ` 头，通过 `Authorize` 按钮设置 Token，即可访问需登录的接口。代码见后。

    1. 在 `BaseSwaggerConfig` 中配置一个 Docket Bean 实例，配置映射路径和要扫描的接口的位置。

        - `securityContexts`：用来配置有哪些请求需要携带 Token。

    2. 配置 `config/SwaggerConfig` 继承此 `BaseSwaggerConfig` 类：Swagger API 文档相关配置，没有 .yml 配置；

    3. 在 `application.yml` 中给 `Swagger-UI` 安全路径**白名单**放行。

        ```
        secure:
          ignored:
            urls: #安全路径白名单
              - /swagger-ui/index.html
        ```

2. **认证方式二**、直接在 Swagger 中填入认证信息，这样就不用从外部去获取 access_token 了。主要是 SecurityScheme 不同。这里采用了 OAuthBuilder 来构建，构建时即得配置 token 的获取地址。仅限于 OAuth2 模式。

3. 参考：[用 Swagger 测试接口，怎么在请求头中携带 Token？](https://juejin.cn/post/6844904183762550797)，[Swagger2.7升级到3.0后的若干问题](https://blog.csdn.net/qq_34963264/article/details/126684715)

Security 整合 Swagger3 相关配置的代码：

```
//认证方式一、
package com.macro.mall.common.config.BaseSwaggerConfig;

if (swaggerProperties.isEnableSecurity()) {
    docket.securitySchemes(securitySchemes()).securityContexts(securityContexts());
}

private List<SecurityScheme> securitySchemes() {
    //设置请求头信息
    List<SecurityScheme> result = new ArrayList<>();
    ApiKey apiKey = new ApiKey("Authorization", "Authorization", "header");
    result.add(apiKey);
    return result;
}

 private List<SecurityContext> securityContexts() {
     //设置需要登录认证的路径
     List<SecurityContext> result = new ArrayList<>();
     result.add(getContextByPath("/*/.*"));
     return result;
 }

private SecurityContext getContextByPath(String pathRegex) {
    return SecurityContext.builder()
    .securityReferences(defaultAuth())
    .forPaths(PathSelectors.regex(pathRegex))
    .build();
}

//com.macro.mall.admin.config.SwaggerConfig
@Configuration
@EnableSwagger2
public class SwaggerConfig extends BaseSwaggerConfig {

    @Override
    public SwaggerProperties swaggerProperties() {
        return SwaggerProperties.builder()
                .apiBasePackage("com.macro.mall.controller")
                ...
                .enableSecurity(true) //用来做认证
                .build();
    }
} 
```

#### 测试



## 整合 Hutool

Java工具类库，包含常用工具类和方法。

```
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>4.6.3</version>
</dependency>
```

常用工具类：

1. `NumberUtil` 数字处理工具类：用于各种类型数字的加减乘除操作及判断类型。
2. `StrUtil` 字符串工具类：定义了一些常用的字符串操作方法；
3. `DateUtil` 日期时间工具类：定义了一些常用的日期时间操作方法；
4. `Convert` 类型转换工具类：用于各种类型数据的转换；
5. ResourceUtils：
6. `ClassPathResource` ：获取 classPath 下的文件，在 Tomcat 等容器下，classPath 一般是WEB-INF/classes；用于日志。
7. `BeanUtil` JavaBean 工具类：提供对**Java反射**和自省API的包装。主要目的是利用反射机制对JavaBean的属性进行处理。用于 Map 与 JavaBean 对象的互相转换及对象属性的拷贝。
8. `BeanUtils.copyProperties(Object source, Object target)`：将 source 中的值赋给 target，名称不同的属性不进行处理，需手动处理。
    1. 如 构造入参 PO；
    2. 如 浅拷贝方法 `BeanUtils.copyProperties(umsAdminParam, umsAdmin);`，Service 中将 DTO 对象 `UmsAdminParm `赋值给 DO 实体类对象 `UmsAdmin`。
    3. 支持浅拷贝或深拷贝。
9. `CollUtil` 集合操作的工具类：定义了一些常用的集合操作；
10. `CollUtil.isNotEmpty(list)`
    2. `CollUtil.toList()`
11. `MapUtil` Map操作工具类：用于创建 Map 对象及判空；
12. `ReflectUtil` Java 反射工具类：用于反射获取类的方法及创建对象；
13. `AnnotationUtil` 注解工具类：用于获取注解与注解中指定的值；
14. `SecureUtil` 加密解密工具类：用于MD5加密；
15. `CaptchaUtil` 验证码工具类：用于生成图形验证码；
16. `JSONUtil`：
         1. 常用方式：`String json1 = JSONUtil.toJsonStr(实体类);`：将 Java 对象转为 JSON。
             2. 比较原始且不好用：`String json2 = JSONUtil.parse(实体类).toJSONString(0);`
             3. `String callbackData = BinaryUtil.toBase64String(JSONUtil.parse(callback).toString().getBytes("utf-8"));`

## 整合 Thymeleaf

`@ThymeleafAutoConfiguration`：自动配置类；

通过 `@ConfigurationProperties` 注解，绑定 ThymeleafProperties 配置类的属性和配置文件（application.properties/yml） 中前缀为 spring.thymeleaf 的配置。如：

- Thymeleaf 模板的默认位置在 `resources/templates` 目录下，默认的后缀是 html，即只要将 HTML 页面放在`classpath:/templates/`下，Thymeleaf 就能自动进行渲染。

##### 配置 thymeleaf

```
spring:
  thymeleaf:
    prefix: classpath:/templates/
    suffix: .html
```

