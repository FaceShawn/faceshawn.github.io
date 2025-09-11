---
title: Open Fegin 客户端负载均衡
categories:
  - 负载均衡
tags:
  - 微服务
  - 分布式
  - 负载均衡
  - Spring-Cloud
  - 高并发
  - 高性能
  - 高可用
  - OpenFegin
location:
  - 黄金时代
abbrlink: 'openfegin'
permalink: 'openfegin'
date: 2025-07-25 16:41:00
updated: 2025-07-25 16:41:00
---

> 摘要：。

<!-- more -->

---

## 目录

[TOC]

#### Spring Cloud OpenFeign

Spring Cloud OpenFeign 是声明式的**服务调用工具**（RPC、Web Service客户端）。

- Feign 集成了Ribbon和Eureka以提供负载均衡的服务调用、及基于Hystrix的服务容错保护功能。可以不再需要显式地使用这两个组件。

- Feign是声明式的服务调用工具，只需创建一个接口并用注解的方式来配置它，就可以实现对某个服务接口的调用，简化了直接使用RestTemplate来调用服务接口的开发量。
    - 提供了HTTP请求的模板，通过编写简单的接口和注解，就可以定义好HTTP请求的参数、格式、地址等信息。
    - Feign会**完全代理**HTTP请求，开发时只需要像调用方法一样调用它就可以完成服务请求及相关处理。
- **可插拔**的注解支持，包括`Feign`、`JAX-RS`、`SpringMvc`注解。
- 支持可插拔的HTTP编码器和解码器;

- 支持HTTP请求和响应的**压缩**。

可设计一套稳定可靠的弹性客户端调用方案，避免整个系统出现雪崩效应。

将Feign视为Spring RestTemplate使用接口与`endpoints`进行通信。 该接口将在运行时自动实现，不是使用服务URL地址，而是使用服务名称。

##### Feign Client

如果没有Feign，

- 将不得不将EurekaClient的一个实例自动连接到控制器中，
- 通过该实例可以接收一个以service-name命名的服务信息作为Application对象。
- 根据此对象获取该服务的所有实例的列表，选择一个合适的实例，然后使用此实例获取主机名和端口。 
- 这样，可以对任何http客户端发出标准请求。

Feign客户端位于spring-cloud-starter-feign软件包中。 

- 通过使用`@EnableEurekaClient`注解主应用程序类来启用。
- 要使用它，只需使用`@FeignClient("service-name")`注解一个接口，然后将其自动连接到控制器中即可。
    - 创建此类**Feign客户端**的一种好方法是使用`@RequestMapping`注解方法创建接口，并将其放入单独的模块中。 这样，它们可以在服务器和客户端之间共享。
    - 在**服务器端**，可以将它们实现为`@Controller`，而在客户端，可以将其扩展和注解为`@FeignClient`。

<img src="../assets/4721c9b968d62a0c1209a322ef6917d2.png" alt="img" style="zoom: 80%;" />

<img src="../assets/bf94393774cdfa7ed217a57b3410fe66.png" alt="img" style="zoom: 80%;" />

##### 创建 feign-service 客户端模块

> 这里创建一个feign-service模块来演示feign的常用功能。

###### 在pom.xml中添加相关依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

###### 在application.yml中进行配置

```yaml
server:
  port: 8701
spring:
  application:
    name: feign-service
eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://localhost:8001/eureka/

feign:
  hystrix:
    enabled: true #在Feign中开启Hystrix服务降级
  #compression:
    #request:
      #enabled: false #是否对请求进行GZIP压缩
      #mime-types: text/xml,application/xml,application/json #指定压缩的请求数据类型
      #min-request-size: 2048 #超过该大小的请求会被压缩
    #response:
      #enabled: false #是否对响应进行GZIP压缩
logging:
  level: #配置需要开启日志的Feign客户端 : 修改日志级别
    com.macro.cloud.service.UserService: debug
```

- Feign中的Ribbon配置：在Feign中配置Ribbon可以直接使用Ribbon的配置，具体可以参考[Spring Cloud Ribbon：负载均衡的服务调用](https://mp.weixin.qq.com/s/uKteoRrFjUbbl08NG522YQ)。
- Feign中的Hystrix配置：在Feign中配置Hystrix可以直接使用Hystrix的配置，具体可以参考[Spring Cloud Hystrix：服务容错保护](https://mp.weixin.qq.com/s/lEjojtuH7XOM9emXkd0TkQ)。



###### 通过配置开启更为详细的日志

Feign提供了日志打印功能，可以通过配置来调整日志级别，从而了解Feign中Http请求的细节。

> 通过java配置来使Feign打印最详细的Http请求日志信息。

日志级别：

- `NONE`：默认的，不显示任何日志；
- `BASIC`：仅记录请求方法、URL、响应状态码及执行时间；
- `HEADERS`：除了BASIC中定义的信息之外，还有请求和响应的头信息；
- `FULL`：除了HEADERS中定义的信息之外，还有请求和响应的正文及元数据。

```java
@Configuration
public class FeignConfig {
    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}
```

###### 在启动类上添加@EnableFeignClients注解来启用Feign的客户端功能

```java
@EnableFeignClients
@EnableDiscoveryClient
@SpringBootApplication
public class FeignServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(FeignServiceApplication.class, args);
    }

}
```

###### 添加UserService接口完成对user-service服务的接口绑定

> 通过@FeignClient注解实现了一个Feign客户端，其中的value为user-service表示这是对user-service服务的接口调用客户端。
>
> 可以回想下user-service中的UserController，只需将其改为接口，保留原来的SpringMvc注释即可。

```java
//设置 fallback 服务降级处理类为UserFallbackService.class
@FeignClient(value = "user-service", fallback = UserFallbackService.class)
public interface UserService {

    @PostMapping("/user/create")
    CommonResult create(@RequestBody User user);

    @GetMapping("/user/{id}")
    CommonResult<User> getUser(@PathVariable Long id);

    @GetMapping("/user/getByUsername")
    CommonResult<User> getByUsername(@RequestParam String username);

    @PostMapping("/user/update")
    CommonResult update(@RequestBody User user);

    @PostMapping("/user/delete/{id}")
    CommonResult delete(@PathVariable Long id);
}
```

###### 添加服务降级实现类 UserFallbackService

服务降级使用起来非常方便，只需要为Feign客户端定义的接口添加一个服务降级处理的实现类即可

> 需要注意的是它实现了UserService接口，并且对接口中的每个实现方法进行了服务降级逻辑的实现。

下面为UserService接口添加一个服务降级实现类。

```java
@Component
public class UserFallbackService implements UserService {
    @Override
    public CommonResult create(User user) {
        User defaultUser = new User(-1L, "defaultUser", "123456");
        return new CommonResult<>(defaultUser);
    }

    @Override
    public CommonResult<User> getUser(Long id) {
        User defaultUser = new User(-1L, "defaultUser", "123456");
        return new CommonResult<>(defaultUser);
    }

    @Override
    public CommonResult<User> getByUsername(String username) {
        User defaultUser = new User(-1L, "defaultUser", "123456");
        return new CommonResult<>(defaultUser);
    }

    @Override
    public CommonResult update(User user) {
        return new CommonResult("调用失败，服务被降级",500);
    }

    @Override
    public CommonResult delete(Long id) {
        return new CommonResult("调用失败，服务被降级",500);
    }
}
```

###### 添加UserFeignController调用UserService实现服务调用

```java
@RestController
@RequestMapping("/user")
public class UserFeignController {
    @Autowired
    private UserService userService;

    @GetMapping("/{id}")
    public CommonResult getUser(@PathVariable Long id) {
        return userService.getUser(id);
    }

    @GetMapping("/getByUsername")
    public CommonResult getByUsername(@RequestParam String username) {
        return userService.getByUsername(username);
    }

    @PostMapping("/create")
    public CommonResult create(@RequestBody User user) {
        return userService.create(user);
    }

    @PostMapping("/update")
    public CommonResult update(@RequestBody User user) {
        return userService.update(user);
    }

    @PostMapping("/delete/{id}")
    public CommonResult delete(@PathVariable Long id) {
        return userService.delete(id);
    }
}
```

##### 功能演示

###### 负载均衡功能演示

- 启动eureka-service，两个user-service，feign-service服务，启动后注册中心显示如下：

![](../assets/springcloud_feign_01.png)

- 多次调用[http://localhost:8701/user/1](http://localhost:8701/user/1)进行测试，可以发现运行在8201和8202的user-service服务交替打印如下信息：

```bash
2019-10-04 15:15:34.829  INFO 9236 --- [nio-8201-exec-5] c.macro.cloud.controller.UserController  : 根据id获取用户信息，用户名称为：macro
2019-10-04 15:15:35.492  INFO 9236 --- [io-8201-exec-10] c.macro.cloud.controller.UserController  : 根据id获取用户信息，用户名称为：macro
2019-10-04 15:15:35.825  INFO 9236 --- [nio-8201-exec-9] c.macro.cloud.controller.UserController  : 根据id获取用户信息，用户名称为：macro
```

###### 服务降级功能演示

- 关闭两个user-service服务，重新启动feign-service;

- 调用[http://localhost:8701/user/1](http://localhost:8701/user/1)进行测试，可以发现返回了服务降级信息。

<img src="../assets/springcloud_feign_02.png" style="zoom:67%;" />

###### 日志打印功能

> 调用[http://localhost:8701/user/1](http://localhost:8701/user/1)进行测试，可以看到以下日志。

```bash
2019-10-04 15:44:03.248 DEBUG 5204 --- [-user-service-2] com.macro.cloud.service.UserService      : [UserService#getUser] ---> GET http://user-service/user/1 HTTP/1.1
2019-10-04 15:44:03.248 DEBUG 5204 --- [-user-service-2] com.macro.cloud.service.UserService      : [UserService#getUser] ---> END HTTP (0-byte body)
2019-10-04 15:44:03.257 DEBUG 5204 --- [-user-service-2] com.macro.cloud.service.UserService      : [UserService#getUser] <--- HTTP/1.1 200 (9ms)
2019-10-04 15:44:03.257 DEBUG 5204 --- [-user-service-2] com.macro.cloud.service.UserService      : [UserService#getUser] content-type: application/json;charset=UTF-8
2019-10-04 15:44:03.258 DEBUG 5204 --- [-user-service-2] com.macro.cloud.service.UserService      : [UserService#getUser] date: Fri, 04 Oct 2019 07:44:03 GMT
2019-10-04 15:44:03.258 DEBUG 5204 --- [-user-service-2] com.macro.cloud.service.UserService      : [UserService#getUser] transfer-encoding: chunked
2019-10-04 15:44:03.258 DEBUG 5204 --- [-user-service-2] com.macro.cloud.service.UserService      : [UserService#getUser] 
2019-10-04 15:44:03.258 DEBUG 5204 --- [-user-service-2] com.macro.cloud.service.UserService      : [UserService#getUser] {"data":{"id":1,"username":"macro","password":"123456"},"message":"操作成功","code":200}
2019-10-04 15:44:03.258 DEBUG 5204 --- [-user-service-2] com.macro.cloud.service.UserService      : [UserService#getUser] <--- END HTTP (92-byte body)
```

