---
title: Spring Boot
categories:
  - SSM框架
tags:
  - SSM
  - Spring-Boot
  - Thymeleaf
location:
abbrlink: 'spring_boot'
permalink: 'spring_boot'
date: 2025-06-06 13:42:12
updated: 2025-06-06 11:56:00
---

> 摘要：Spring Boot 包括配置、自动装配、数据库等。另外，Spring Boot 整合 Web、Hibernate、MyBatis、Redis、ELK、Swagger-UI、Security、Hutool 等见其它文档。

<!-- more -->

### 目录

[TOC]

### Spring Boot

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

```yaml
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

    ```java
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

```bash
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

```java
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



## 整合 Thymeleaf

`@ThymeleafAutoConfiguration`：自动配置类；

通过 `@ConfigurationProperties` 注解，绑定 ThymeleafProperties 配置类的属性和配置文件（application.properties/yml） 中前缀为 spring.thymeleaf 的配置。如：

- Thymeleaf 模板的默认位置在 `resources/templates` 目录下，默认的后缀是 html，即只要将 HTML 页面放在`classpath:/templates/`下，Thymeleaf 就能自动进行渲染。

##### 配置 thymeleaf

```yaml
spring:
  thymeleaf:
    prefix: classpath:/templates/
    suffix: .html
```

