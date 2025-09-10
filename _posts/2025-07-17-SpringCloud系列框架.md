---
title: Spring Cloud 系列框架
categories:
  - 微服务
tags:
  - 微服务
  - 分布式
location:
  - 黄金时代
abbrlink: 'spring_cloud'
permalink: 'spring_cloud'
date: 2025-07-17 16:41:00
updated: 2025-07-17 16:41:00
---

> 摘要：基于Spring Boot之上的用来**快速构建微服务系统**的工具集，拥有功能完善的轻量级微服务组件。。

<!-- more -->

---

### 目录

[TOC]

## Spring Project

根据官网，Spring Project 包括的所有项目有：

- [Spring Boot](https://spring.io/projects/spring-boot)：旨在简化创建产品级的 Spring 应用和服务，简化了配置文件，使用嵌入式web服务器，含有诸多开箱即用微服务功能，可以和spring cloud联合部署。
- [Spring Framework](https://spring.io/projects/spring-framework)即通常所说的spring 框架，是一个开源的Java/Java EE**全功能栈**应用程序框架，其它spring项目如spring boot也依赖于此框架。
- [Spring Data](https://spring.io/projects/spring-data)：是一个数据访问及操作的工具包，封装了很多种数据及数据库的访问相关技术，包括：jdbc、Redis、MongoDB、Neo4j等。
- [Spring Cloud](https://spring.io/projects/spring-cloud)：微服务**工具包**，为开发者提供了在分布式系统的**配置管理、服务发现、断路器、智能路由、微代理、控制总线**等开发工具包。
- [Spring Cloud Data Flow](https://spring.io/projects/spring-cloud-dataflow)
- [Spring Security](https://spring.io/projects/spring-security)：是一个能够为基于Spring的企业应用系统提供**声明式**的**安全访问控制**解决方案的安全框架。安全工具包，如 OAuth2。
- [Spring Authorization Server](https://spring.io/projects/spring-authorization-server)
- [Spring for GraphQL](https://spring.io/projects/spring-graphql)
- [Spring Session](https://spring.io/projects/spring-session)：session管理的开发工具包，让你可以把session保存到redis等，进行集群化session管理。
- [Spring Integration](https://spring.io/projects/spring-integration)
- [Spring HATEOAS](https://spring.io/projects/spring-hateoas)
- [Spring Modulith](https://spring.io/projects/spring-modulith)
- [Spring REST Docs](https://spring.io/projects/spring-restdocs)
- [Spring AI](https://spring.io/projects/spring-ai)
- Spring XD：是一种运行时环境（服务器软件，非开发框架），组合spring技术，如spring batch、spring boot、spring data，采集大数据并处理。
- [Spring Batch](https://spring.io/projects/spring-batch)：批处理框架，或说是批量任务执行管理器，功能包括任务调度、日志记录/跟踪等。
- [Spring AMQP](https://spring.io/projects/spring-amqp)：**消息队列**操作的工具包，主要是封装了**RabbitMQ**的操作。
- [Spring CredHub](https://spring.io/projects/spring-credhub)
- [Spring for Apache Kafka](https://spring.io/projects/spring-kafka)
- [Spring LDAP](https://spring.io/projects/spring-ldap)
- [Spring for Apache Pulsar](https://spring.io/projects/spring-pulsar)
- [Spring Shell](https://spring.io/projects/spring-shell)
- [Spring Statemachine](https://spring.io/projects/spring-statemachine)
- [Spring Vault](https://spring.io/projects/spring-vault)
- [Spring Web Flow](https://spring.io/projects/spring-webflow)：目标是成为管理Web应用页面流程的最佳方案，将页面跳转流程单独管理，并可配置。
- [Spring Web Services](https://spring.io/projects/spring-ws)：是基于Spring的Web服务框架，提供SOAP服务开发，允许通过多种方式创建Web服务。
- Spring Mobile：是Spring MVC的扩展，用来简化手机上的Web应用开发。
- Spring for Android：是Spring框架的一个扩展，其主要目的在乎简化Android本地应用的开发，提供RestTemplate来访问Rest服务。

## Spring Cloud

> 见 SSM 框架文档。

[Spring Cloud](http://projects.spring.io/spring-cloud/)：是基于Spring Boot之上的用来**快速构建微服务系统**的工具集，拥有功能完善的轻量级微服务组件。

- 微服务**工具包**，为开发者提供了在分布式系统的**配置管理、服务发现、断路器、智能路由、微代理、控制总线**等开发工具包。
- 利用Spring Boot的开发便利性巧妙地简化了分布式系统基础设施的开发，都可以用Spring Boot的开发风格做到一键启动和部署。
    - Spring并没有重复制造轮子，只是将目前各家公司开发的比较成熟、经得起实际考验的服务框架组合起来，通过Spring Boot风格进行再封装屏蔽掉了复杂的配置和实现原理，最终给开发者留出了一套简单易懂、易部署和易维护的分布式系统开发工具包。

Spring Cloud 本身并不是一个开箱即用的框架，它是一套微服务规范，共有两代实现。

- Spring Cloud Netflix 是 Spring Cloud 的第一代实现，主要由 Eureka、Ribbon、Feign、Hystrix 等组件组成。
- Spring Cloud Alibaba 是 Spring Cloud 的第二代实现，主要由 Nacos、Sentinel、Seata 等组件组成。

<img src="assets/spring-cloud-img.png" alt="spring-cloud" style="zoom: 33%;" />

### 版本

当SpringCloud的发布内容积累到临界点或者一个重大BUG被解决后，会发布一个"service releases"版本，简称SRX版本，比如Greenwich.SR2就是SpringCloud发布的Greenwich版本的第2个SRX版本。

- 目前主流的版本为SpringBoot 3.5.x RELEAE 和 SpringCloud Release Train（2025.0.x aka **Northfields**）。

```xml
<spring-boot.version>3.2.2</spring-boot.version>
<spring-cloud.version>2023.0.1</spring-cloud.version>
<spring-cloud-alibaba.version>2023.0.1.0</spring-cloud-alibaba.version>
```

SpringCloud和SpringBoot版本对应关系：参见[官网](https://spring.io/projects/spring-cloud#overview)

| SpringCloud Release Train                                    | spring-cloud-alibaba | Spring Boot Generation                |
| ------------------------------------------------------------ | -------------------- | ------------------------------------- |
| [2025.0.x](https://github.com/spring-cloud/spring-cloud-release/wiki/Spring-Cloud-2025.0-Release-Notes) aka Northfields |                      | 3.5.x                                 |
| [2024.0.x](https://github.com/spring-cloud/spring-cloud-release/wiki/Spring-Cloud-2024.0-Release-Notes) aka Moorgate |                      | 3.4.x                                 |
| [2023.0.x](https://github.com/spring-cloud/spring-cloud-release/wiki/Spring-Cloud-2023.0-Release-Notes) aka **Leyton** |                      | 3.3.x, **3.2.x**                      |
| [2022.0.x](https://github.com/spring-cloud/spring-cloud-release/wiki/Spring-Cloud-2022.0-Release-Notes) aka Kilburn |                      | 3.0.x, 3.1.x (Starting with 2022.0.3) |
| [2021.0.x](https://github.com/spring-cloud/spring-cloud-release/wiki/Spring-Cloud-2021.0-Release-Notes) aka Jubilee |                      | 2.6.x, 2.7.x (Starting with 2021.0.3) |
| [2020.0.x](https://github.com/spring-cloud/spring-cloud-release/wiki/Spring-Cloud-2020.0-Release-Notes) aka Ilford |                      | 2.4.x, 2.5.x (Starting with 2020.0.3) |
| [Hoxton](https://github.com/spring-cloud/spring-cloud-release/wiki/Spring-Cloud-Hoxton-Release-Notes) |                      | 2.2.x, 2.3.x (Starting with SR5)      |
| [Greenwich](https://github.com/spring-projects/spring-cloud/wiki/Spring-Cloud-Greenwich-Release-Notes) |                      | 2.1.x                                 |
| [Finchley](https://github.com/spring-projects/spring-cloud/wiki/Spring-Cloud-Finchley-Release-Notes) |                      | 2.0.x                                 |
| [Edgware](https://github.com/spring-projects/spring-cloud/wiki/Spring-Cloud-Edgware-Release-Notes) |                      | 1.5.x                                 |
| [Dalston](https://github.com/spring-projects/spring-cloud/wiki/Spring-Cloud-Dalston-Release-Notes) |                      | 1.5.x                                 |

![img](assets/qyg3a6hgnidgi_c6991cb4efa3484594cbaec03aa0f88b.png)

排列如下表（最新版本用*标记）：

Spring Cloud Alibaba Version	Spring Cloud Version	Spring Boot Version
2.2.10-RC1*	Spring Cloud Hoxton.SR12	2.3.12.RELEASE
2.2.9.RELEASE	Spring Cloud Hoxton.SR12	2.3.12.RELEASE
2.2.8.RELEASE	Spring Cloud Hoxton.SR12	2.3.12.RELEASE
2.2.7.RELEASE	Spring Cloud Hoxton.SR12	2.3.12.RELEASE
2.2.6.RELEASE	Spring Cloud Hoxton.SR9	2.3.2.RELEASE
2.2.1.RELEASE	Spring Cloud Hoxton.SR3	2.2.5.RELEASE
2.2.0.RELEASE	Spring Cloud Hoxton.RELEASE	2.2.X.RELEASE
2.1.4.RELEASE	Spring Cloud Greenwich.SR6	2.1.13.RELEASE
2.1.2.RELEASE	Spring Cloud Greenwich	2.1.X.RELEASE
2.0.4.RELEASE(停止维护，建议升级)	Spring Cloud Finchley	2.0.X.RELEASE
1.5.1.RELEASE(停止维护，建议升级)	Spring Cloud Edgware	1.5.X.RELEASE



### Spring Cloud 子项目

<img src="assets/SpringCloud_1.png" alt="img" style="zoom: 60%;" />

![img](assets/v2-724c2b466ea3a1ab4d5675cad4abab69_r.jpg)

Spring Cloud包含了多个子项目（针对分布式系统中涉及的多个不同开源产品），包括：

- Spring Cloud **Config**：配置管理工具包，让你可以把配置放到远程服务器，集中化管理集群配置，目前支持本地存储、Git以及Subversion。集中配置管理工具，分布式系统中统一的外部配置管理，默认使用Git来存储配置，可以支持客户端配置的刷新及加密、解密操作。
- [Spring Cloud Gateway](https://spring.io/projects/spring-cloud-gateway)：API网关组件，对请求提供路由及过滤功能。可以认为是 Zuul 的下一代，无论从易用性和性能方便都有所提高。
- Spring Cloud **Netflix**：针对多种Netflix组件提供的开发工具包，Netflix OSS 开源组件集成，包括Eureka、Ribbon、Hystrix、Feign、Zuul等核心组件。
    - **Zuul**：服务**网关**，对请求提供路由及过滤功能。边缘服务工具，相当于是服务后端所有请求的**前门**。在云平台上提供**动态路由**、监控、弹性、安全等边缘服务。主要实现了路由转发和过滤器功能，对于处理一些数据聚合、鉴权、监控、统计类的功能非常好用。
    - **Eureka**：**服务发现**，基于 REST，用于定位服务，以实现**云端的负载均衡**和中间层服务器的故障转移。服务治理组件，包括服务端的注册中心和客户端的服务发现机制。
    - **Ribbon**：提供**客户端负负载均衡**功能，负载均衡的服务调用组件，具有多种负载均衡调用策略。
        - 例如一个服务提供者部署了 3 个实例，那么使用 Ribbon 可以指定**负载均衡算法**请求其中一个实例。如果配合 Eureka ，使用起来非常简单。
    - **Hystrix**：断路器。服务容错管理工具，旨在通过控制服务和第三方库的节点，从而对延迟和故障提供更强大的容错能力。服务容错组件，实现了断路器模式，为依赖服务的出错和延迟提供了容错能力。
        - 假设有 3 个服务提供实例，其中有一个实例由于某种原因挂掉了，那么当再有请求进来的时候，如果还是向这个实例上发请求，那将会导致**请求积压**阻塞。熔断器可以将这个有问题的实例**下线**，这样一来，再有新的请求进来，就不会再发到这个有问题的实例上了。
        - Turbine：是聚合服务器发送事件流数据的一个工具，用来监控集群下hystrix的metrics情况。
    - **Feign**：基于Ribbon和Hystrix的声明式服务调用组件；
    - ~~Archaius~~：配置管理API，包含一系列配置管理API，提供动态类型化属性、线程安全配置操作、轮询框架、回调机制等功能。
- [24. Spring Cloud OpenFeign](https://spring-cloud-wiki.readthedocs.io/zh-cn/latest/pages/feign.html)：**客户端负载均衡**。声明式REST调用。是一种声明式、模板化的HTTP客户端。基于Ribbon和Hystrix的声明式服务调用组件，可以动态创建基于Spring MVC注解的接口实现用于服务调用，在SpringCloud 2.0中已经取代Feign成为了一等公民。
- Spring Cloud **Consul**：封装了Consul操作，consul是一个服务发现与配置工具，与Docker容器可以无缝集成。基于Hashicorp Consul的服务治理组件。
- Spring Cloud **Zookeeper**：基于Apache Zookeeper的服务治理组件。操作Zookeeper的工具包，用于使用zookeeper方式的服务注册和发现。
- Spring Cloud **Sleuth**：分布式请求链路跟踪，日志收集工具包。封装了Dapper和log-based追踪以及Zipkin和HTrace操作，为SpringCloud应用实现了一种分布式追踪解决方案。支持使用Zipkin、HTrace和基于日志（例如ELK）的跟踪。
- Spring Cloud **Security**：基于spring security的安全工具包，为应用程序添加安全控制。安全工具包，对Zuul代理中的负载均衡OAuth2客户端及登录认证进行支持。
- Spring Cloud Stream：数据流操作开发包，封装了与**Redis,Rabbit、Kafka**等发送接收消息。轻量级事件驱动微服务框架，可以使用简单的声明式模型来发送及接收消息，主要实现为Apache Kafka及RabbitMQ。
- [Spring Cloud Alibaba](https://spring.io/projects/spring-cloud-alibaba)
- Spring Cloud **Bus**：事件、消息总线，用于在集群（例如，配置变化事件）中传播状态变化，可与Spring Cloud Config联合实现热部署。用于传播集群状态变化的消息总线，使用轻量级消息代理链接分布式系统中的节点，可以用来动态刷新集群中的服务配置。
- [Spring Cloud Circuit Breaker](https://spring.io/projects/spring-cloud-circuitbreaker)：**断路器**。主要是为了解决当某个方法调用失败的时候，调用**后备方法**来替代失败的方法，以达到容错／阻止级联错误的功能。
    - 使用`@EnableCircuitBreaker`来启用断路器支持，用 `@HystrixCommand(fallbackMethod="fallbackOper")` 来指定后备方法。
    - 还提供了一个控制台来监控断路器的运行情况，通过`@EnableHystrixDashboard`注解开启。
- Spring Batch 作业管理
- [Spring Cloud Task](https://spring.io/projects/spring-cloud-task)：提供云端计划任务管理、任务调度。用于快速构建短暂、有限数据处理任务的微服务框架，用于向应用中添加功能性和非功能性的特性。
- Spring Cloud for Cloud Foundry：通过Oauth2协议绑定服务到**CloudFoundry**，CloudFoundry是VMware推出的开源PaaS云平台。
- Spring Cloud CLI：基于 Spring Boot CLI，可以让你以命令行方式快速建立云组件。
- Spring Cloud Commons
- Spring Cloud Data Flow：大数据操作工具，通过命令行方式操作数据流。作为Spring XD的替代产品，它是一个混合计算模型，结合了流数据与批量数据的处理方式。
- Spring Cloud Connectors：便于云端应用程序在各种PaaS平台连接到后端，如：数据库和消息代理服务。
- Spring Cloud Cluster：提供Leadership选举，如：Zookeeper, Redis, Hazelcast, Consul等常见状态模式的抽象和实现。
- Spring Cloud Starters：Spring Boot式的启动项目，为Spring Cloud提供开箱即用的依赖管理。

常用依赖包如下：

```
spring-cloud-starter-parent 具备spring-boot-starter-parent同样功能并附加Spring Cloud的依赖  
spring-cloud-starter-config 默认的配置服务依赖，快速自动引入服务的方式，端口8888  
spring-cloud-config-server／client 用户自定义配置服务的服务端／客户端依赖  
spring-cloud-starter-eureka-server 服务发现的Eureka Server依赖  
spring-cloud-starter-eureka 服务发现的Eureka客户端依赖  
spring-cloud-starter-hystrix／zuul／feign／ribbon 断路器（Hystrix），智能路由（Zuul），客户端负载均衡（Ribbon）的依赖  
angular-ui-router 页面分发路由依赖  
```

### ELK 技术栈



### Spring Cloud Stream

数据流操作开发包，封装了与**Redis,Rabbit、Kafka**等发送接收消息。

### Spring Cloud Alibaba 架构

Spring Cloud Alibaba 致力于提供微服务开发的一站式解决方案。此项目包含开发分布式应用微服务的必需组件，方便开发者通过 Spring Cloud 编程模型轻松使用这些组件来开发分布式应用服务。

![img](assets/b35b7a9eefdef534bb3b4869ec0f7118.png)

核心开源组件：

- **Sentinel：**把流量作为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。
- **Nacos：**一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。
- **RocketMQ：**一款开源的[分布式消息系统](https://cloud.tencent.com/product/ckafka?from_column=20065&from=20065)，基于高可用分布式集群技术，提供低延时的、高可靠的消息发布与订阅服务。
- **Dubbo：**是一款高性能 Java RPC 框架。
- **Seata：**阿里巴巴开源产品，一个易于使用的高性能微服务分布式**事务**解决方案。

核心商业化组件：

- **Alibaba Cloud OSS：**阿里云对象存储服务（Object Storage Service，简称 OSS），是阿里云提供的海量、安全、低成本、高可靠的云存储服务。可以在任何应用、任何时间、任何地点存储和访问任意类型的数据。
- **Alibaba Cloud SchedulerX：**阿里中间件团队开发的一款分布式任务调度产品，提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。
- **Alibaba Cloud SMS：**覆盖全球的短信服务，友好、高效、智能的互联化通讯能力，帮助企业迅速搭建客户触达通道。

Spring Cloud Alibaba 官方文档：https://github.com/alibaba/spring-cloud-alibaba/wiki

<img src="assets/spring-cloud-alibaba-img.png" alt="spring-cloud" style="zoom: 67%;" />