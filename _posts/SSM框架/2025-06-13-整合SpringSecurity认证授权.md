---
title: Spring Boot 整合 Spring Security 认证和授权
categories:
  - SSM框架
tags:
  - SSM
  - Spring-Boot
  - Spring-Security
  - Swagger-UI
location:
abbrlink: 'spring_boot_security'
permalink: 'spring_boot_security'
date: 2025-06-13 13:42:12
updated: 2025-06-13 11:56:00
---

> 摘要：整合 Spring Security 和 JWT 实现用户的**登录和授权功能**，同时改造 `Swagger-UI` 的配置使其可自动发送**登录令牌**。

<!-- more -->

## 目录

[TOC]

## 整合 Spring Security 权限

#### 添加依赖

在 `pom.xml` 中

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

#### Spring Boot 配置

在Spring Boot应用程序中，可以通过在`application.properties`文件或`application.yml`文件中配置Spring Security。

- 这里的配置表示启用**基于HTTP基本身份验证**的Spring Security，并指定了一个用户名为admin，密码为password的用户。
- 默认情况下，Spring Boot [UserDetailsServiceAutoConfiguration](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/security/servlet/UserDetailsServiceAutoConfiguration.java) 自动化配置类，会创建一个**内存级别**的 [InMemoryUserDetailsManager](https://github.com/spring-projects/spring-security/blob/master/core/src/main/java/org/springframework/security/provisioning/InMemoryUserDetailsManager.java) Bean 对象，提供认证的用户信息。
    - 这里，**添加了** `spring.security.user` 配置项，UserDetailsServiceAutoConfiguration 会基于配置的信息创建一个用户 [User](https://github.com/spring-projects/spring-security/blob/master/core/src/main/java/org/springframework/security/core/userdetails/User.java) 在内存中。
    - 如果，**未添加** `spring.security.user` 配置项，UserDetailsServiceAutoConfiguration 会自动创建一个用户名为 `"user"` ，密码为 **UUID 随机**的用户 [User](https://github.com/spring-projects/spring-security/blob/master/core/src/main/java/org/springframework/security/core/userdetails/User.java) 在内存中。

```yaml
jwt:
  tokenHeader: Authorization #JWT存储的请求头
  secret: mall-admin-secret  #JWT加解密使用的密钥
  expiration: 604800    #JWT的超期限时间(60*60*24*7)
  tokenHead: 'Bearer '  #JWT负载中拿到开头

spring:
  # Spring Security 配置项，对应 SecurityProperties 配置类
  security:
    # 配置默认的 InMemoryUserDetailsManager 的用户账号与密码。
    user:
      name: user # 账号
      password: user # 密码
      roles: ADMIN # 拥有角色
    basic:
      enabled: true
```

#### 添加配置类

```java
// SecurityConfig.java

//实现 Spring Security 在 Web 场景下的自定义配置
public class SecurityConfig extends WebSecurityConfigurerAdapter {
}

//类上加注解
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class MallSecurityConfig extends SecurityConfig {
}
```

自定义的 Spring Security 安全配置类  `SecurityConfig`，可以通过继承`WebSecurityConfigurerAdapter`类、重写方法，实现自定义的 Spring Security 的配置。

1. `configure(AuthenticationManagerBuilder auth)`：用于配置 `UserDetailsService` 及`PasswordEncoder`；`AuthenticationManager`：
2. `UserDetailsService`：SpringSecurity 定义的**核心接口**，用于根据用户名获取用户信息，需要自行实现；
    2. `UserDetails`：定义用于**封装用户信息**的类（主要是用户信息和权限），需要自行实现；
    3. `PasswordEncoder`：用于对密码进行编码及**校验**比对的接口，目前使用的是`BCryptPasswordEncoder` 哈希加密算法；

3. `configure(HttpSecurity httpSecurity)`：用于读取并配置需拦截的 URL 路径、**白名单路径**，是否允许跨域（的OPTIONS请求），配置权限 JWT 过滤器、权限**拒绝处理器**、抛出异常后的处理器、动态权限**校验过滤器**等；
4. 添加 `JwtAuthenticationTokenFilter` 类：自定义 JWT 登录**授权过滤器**/拦截器，（在用户名和密码校验前）添加**过滤器** ，如果请求中有 JWT 的 token 且**验证**有效，会自行根据token信息进行登录。

    - 会取出 token 中的用户名，创建 `UsernamePasswordAuthenticationToken` 实例，并保存至安全上下文。

    - 调用 `UserDetailsService`、`UserDetails`、`PasswordEncoder` 验证、取出；
        - `SecurityContextHolder.getContext().getAuthentication() == null`
        - `SecurityContextHolder.getContext().setAuthentication(UsernamePasswordAuthenticationToken);`：将 Authentication 保存至**安全上下文**。

5. 添加 `RestfulAccessDeniedHandler` 类：自定义权限**拒绝处理器**，当用户**没有访问权限**时的处理器，用于返回JSON格式的处理结果；

    - ~~`RestAuthenticationEntryPoint`~~：当未登录或token失效（登录过期）时，返回JSON格式的结果。

6. 添加 `DynamicSecurityFilter` 类：动态权限**校验过滤器**，用于实现**基于路径**的动态权限过滤；

    1. `DynamicAccessDecisionManager` 类：动态权限**决策**管理器，用于判断用户是否有访问权限；
    2. `DynamicSecurityMetadataSource` 类：动态权限数据源，用于获取动态权限**规则**；
        - 实现 `DynamicSecurityService` 动态权限相关**业务接口**：读取动态权限配置；加载资源ANT通配符和资源对应MAP；

<img src="../assets/8fdccbc24a2225db315a8800cdfe1c09.png" alt="img" style="zoom:80%;" />

##### **示例一**

自定义 Spring Security 的配置，实现**权限控制**。

###### SecurityConfig

在 [`cn.iocoder.springboot.lab01.springsecurity.config`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-01-spring-security/lab-01-springsecurity-demo-role/src/main/java/cn/iocoder/springboot/lab01/springsecurity/config) 包下，创建 [SecurityConfig](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-01-spring-security/lab-01-springsecurity-demo-role/src/main/java/cn/iocoder/springboot/lab01/springsecurity/config/SecurityConfig.java) 配置类，继承 [WebSecurityConfigurerAdapter](https://github.com/spring-projects/spring-security/blob/master/config/src/main/java/org/springframework/security/config/annotation/web/configuration/WebSecurityConfigurerAdapter.java) 抽象类，实现 Spring Security 在 Web 场景下的自定义配置。

- 可以通过重写 WebSecurityConfigurerAdapter 的方法，实现自定义的 Spring Security 的配置。

```java
// SecurityConfig.java

@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    // ...
}
```

###### 重写 `#configure(AuthenticationManagerBuilder auth)` 方法

重写 `#configure(AuthenticationManagerBuilder auth)` 方法，实现 [AuthenticationManager](https://github.com/spring-projects/spring-security/blob/master/core/src/main/java/org/springframework/security/authentication/AuthenticationManager.java) 认证管理器。

```java
// SecurityConfig.java

@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.
            // <X> 使用内存中的 InMemoryUserDetailsManager
            inMemoryAuthentication()
            // <Y> 不使用 PasswordEncoder 密码编码器
            .passwordEncoder(NoOpPasswordEncoder.getInstance())
            // <Z> 配置 admin 用户
            .withUser("admin").password("admin").roles("ADMIN")
            // <Z> 配置 normal 用户
            .and().withUser("normal").password("normal").roles("NORMAL");
}
```

- `<X>` 处，调用 `AuthenticationManagerBuilder#inMemoryAuthentication()` 方法，使用**内存级别**的 [InMemoryUserDetailsManager](https://github.com/spring-projects/spring-security/blob/master/core/src/main/java/org/springframework/security/provisioning/InMemoryUserDetailsManager.java) Bean 对象，提供认证的用户信息。
    - Spring 内置了两种 [UserDetailsManager](https://github.com/spring-projects/spring-security/blob/master/core/src/main/java/org/springframework/security/provisioning/UserDetailsManager.java) 实现：
        - InMemoryUserDetailsManager，和[「2. 快速入门」](https://www.iocoder.cn/Spring-Boot/Spring-Security/#)是一样的。
        - JdbcUserDetailsManager ，基于 **JDBC**的 [JdbcUserDetailsManager](https://github.com/spring-projects/spring-security/blob/master/core/src/main/java/org/springframework/security/provisioning/JdbcUserDetailsManager.java) 。
    - 实际项目中，更多采用调用 `AuthenticationManagerBuilder#userDetailsService(userDetailsService)` 方法，使用自定义实现的 [UserDetailsService](https://github.com/spring-projects/spring-security/blob/master/core/src/main/java/org/springframework/security/core/userdetails/UserDetailsService.java) 实现类，更加**灵活**且**自由**的实现认证的用户信息的读取。
-  `<Y>` 处，调用 `AbstractDaoAuthenticationConfigurer#passwordEncoder(passwordEncoder)` 方法，设置 PasswordEncoder 密码编码器。
    - 在这里，为了方便，我们使用 [NoOpPasswordEncoder](https://github.com/spring-projects/spring-security/blob/master/crypto/src/main/java/org/springframework/security/crypto/password/NoOpPasswordEncoder.java) 。实际上，等于不使用 PasswordEncoder ，不配置的话会报错。
    - 生产环境下，推荐使用 [BCryptPasswordEncoder](https://github.com/spring-projects/spring-security/blob/master/crypto/src/main/java/org/springframework/security/crypto/bcrypt/BCryptPasswordEncoder.java) 。更多关于 PasswordEncoder 的内容，推荐阅读[《该如何设计你的 PasswordEncoder?》](http://www.iocoder.cn/Spring-Security/laoxu/PasswordEncoder/?self)文章。
- `<Z>` 处，配置了「admin/admin」和「normal/normal」两个用户，分别对应 ADMIN 和 NORMAL 角色。相比[「2. 快速入门」](https://www.iocoder.cn/Spring-Boot/Spring-Security/#)来说，可以配置更多的用户。

##### 重写 `#configure(HttpSecurity http)` 方法

然后，重写 `#configure(HttpSecurity http)` 方法，主要配置 URL 的权限控制。

```java
// SecurityConfig.java

@Override
protected void configure(HttpSecurity http) throws Exception {
    http
            // <X> 配置请求地址的权限
            .authorizeRequests()
                .antMatchers("/test/echo").permitAll() // 所有用户可访问
                .antMatchers("/test/admin").hasRole("ADMIN") // 需要 ADMIN 角色
                .antMatchers("/test/normal").access("hasRole('ROLE_NORMAL')") // 需要 NORMAL 角色。
                // 任何请求，访问的用户都需要经过认证
                .anyRequest().authenticated()
            .and()
            // <Y> 设置 Form 表单登录
            .formLogin()
//                    .loginPage("/login") // 登录 URL 地址
                .permitAll() // 所有用户可访问
            .and()
            // 配置退出相关
            .logout()
//                    .logoutUrl("/logout") // 退出 URL 地址
                .permitAll(); // 所有用户可访问
}
```



- `<X>` 处，调用 `HttpSecurity#authorizeRequests()` 方法，开始配置 URL 的**权限控制**。注意看艿艿配置的**四个**权限控制的配置。下面，是配置权限控制会使用到的方法：
    - `#(String... antPatterns)` 方法，配置匹配的 URL 地址，基于 [Ant 风格路径表达式](https://blog.csdn.net/songdexv/article/details/7219686) ，可传入多个。
    - 【常用】`#permitAll()` 方法，所有用户可访问。
    - 【常用】`#denyAll()` 方法，所有用户不可访问。
    - 【常用】`#authenticated()` 方法，登录用户可访问。
    - `#anonymous()` 方法，无需登录，即匿名用户可访问。
    - `#rememberMe()` 方法，通过 [remember me](https://docs.spring.io/spring-security/site/docs/3.0.x/reference/remember-me.html) 登录的用户可访问。
    - `#fullyAuthenticated()` 方法，非 [remember me](https://docs.spring.io/spring-security/site/docs/3.0.x/reference/remember-me.html) 登录的用户可访问。
    - `#hasIpAddress(String ipaddressExpression)` 方法，来自指定 IP 表达式的用户可访问。
    - 【常用】`#hasRole(String role)` 方法， 拥有指定角色的用户可访问。
    - 【常用】`#hasAnyRole(String... roles)` 方法，拥有指定任一角色的用户可访问。
    - 【常用】`#hasAuthority(String authority)` 方法，拥有指定权限(`authority`)的用户可访问。
    - 【常用】`#hasAuthority(String... authorities)` 方法，拥有指定任一权限(`authority`)的用户可访问。
    - 【最牛】`#access(String attribute)` 方法，当 [Spring EL 表达式](https://docs.spring.io/spring/docs/4.3.10.RELEASE/spring-framework-reference/html/expressions.html)的执行结果为 `true` 时，可以访问。
- `<Y>` 处，调用 `HttpSecurity#formLogin()` 方法，设置 Form 表单**登录**。
    - 如果胖友想要自定义登录页面，可以通过 `#loginPage(String loginPage)` 方法，来进行设置。不过这里我们希望像[「2. 快速入门」](https://www.iocoder.cn/Spring-Boot/Spring-Security/#)一样，使用默认的登录界面，所以不进行设置。
- `<Z>` 处，调用 `HttpSecurity#logout()` 方法，配置**退出**相关。
    - 如果胖友想要自定义退出页面，可以通过 `#logoutUrl(String logoutUrl)` 方法，来进行设置。不过这里我们希望像[「2. 快速入门」](https://www.iocoder.cn/Spring-Boot/Spring-Security/#)一样，使用默认的退出界面，所以不进行设置。

##### 示例二

使用 Spring Security 的注解，实现权限控制。

###### 3.3.1 SecurityConfig

修改 [SecurityConfig](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-01-spring-security/lab-01-springsecurity-demo-role/src/main/java/cn/iocoder/springboot/lab01/springsecurity/config/SecurityConfig.java) 配置类，增加 [`@EnableGlobalMethodSecurity`](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/config/annotation/method/configuration/EnableGlobalMethodSecurity.html) 注解，开启对 Spring Security 注解的方法，进行权限验证。

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter
```

###### DemoController

- [`@PermitAll`](https://github.com/jboss/jboss-annotations-api_spec/blob/master/src/main/java/javax/annotation/security/PermitAll.java) 注解，等价于 `#permitAll()` 方法，所有用户可访问。

    > 重要！！！因为在[「3.2.1 SecurityConfig」](https://www.iocoder.cn/Spring-Boot/Spring-Security/#)中，配置了 `.anyRequest().authenticated()` ，任何请求，访问的用户都需要经过认证。所以这里 `@PermitAll` **注解实际是不生效的**。
    >
    > 也就是说，Java Config 配置的权限，和注解配置的权限，两者是**叠加**的。

- [`@PreAuthorize`](https://github.com/spring-projects/spring-security/blob/master/core/src/main/java/org/springframework/security/access/prepost/PreAuthorize.java) 注解，等价于 `#access(String attribute)` 方法，，当 Spring EL 表达式的执行结果为 true 时，可以访问。





#### 添加 `JwtTokenUtil` 工具类

> 用于生成、解析、验证`JwtToken ` 的工具类；

相关方法说明：

- `generateToken(UserDetails userDetails)`：用于根据登录用户信息生成token；
- `validateToken(String token, UserDetails userDetails)`：判断token是否还有效；
    - `getUserNameFromToken(String token)`：从token中获取登录用户的信息；

#### 登录注册功能实现

> 如果没有**自定义**登录界面，所以默认会使用 [DefaultLoginPageGeneratingFilter](https://github.com/spring-projects/spring-security/blob/master/web/src/main/java/org/springframework/security/web/authentication/ui/DefaultLoginPageGeneratingFilter.java) 类，生成上述界面。

实现登录注册：

1. 添加用户数据库表、及 `model `类 `com.macro.mall.model.UmsAdmin`；
2. 添加 `com.macro.mall.bo.AdminUserDetails`：Spring Security 需要的用户详情；
3. 添加 `UmsAdminController` 类：实现用户登录、注册及获取权限的接口；为接口中的方法添加**访问权限**。
4. 添加 `UmsAdminService` 接口及其实现类：**核心接口**，用于根据用户名获取用户信息，需自行实现；
    - `UmsAdminCacheService`：用户**缓存**操作Service实现类，存在 `RedisService `中。

![img](../assets/1218593-20220809143059104-330636126.webp)

给PmsBrandController接口中的方法添加访问权限：

- 给查询接口添加`pms:brand:read`权限
- 给修改接口添加`pms:brand:update`权限
- 给删除接口添加`pms:brand:delete`权限
- 给添加接口添加`pms:brand:create`权限

```java
// 给 `PmsBrandController` 接口中的方法添加访问权限
@PreAuthorize("hasAuthority('pms:brand:read')")
public CommonResult<List<PmsBrand>> getBrandList() {
	return CommonResult.success(brandService.listAllBrand());
}
```

### 整合 Swagger3

> 通过修改配置实现调用接口自带Authorization头，这样就可以访问需要登录的接口了。

参考：

1. [用 Swagger 测试接口，怎么在请求头中携带 Token？](https://juejin.cn/post/6844904183762550797)
2. [Swagger2.7升级到3.0后的若干问题](https://blog.csdn.net/qq_34963264/article/details/126684715)

##### 直接在 Swagger 中填入认证信息

**认证方式二**、直接在 Swagger 中填入认证信息，这样就不用从外部去获取 access_token 了。

- 主要是 SecurityScheme 不同。这里采用了 OAuthBuilder 来构建，构建时即得配置 token 的获取地址。仅限于 OAuth2 模式。

##### Swagger 配置实现自带 `Authorization ` 头

**认证方式一**、修改 Swagger 的配置，实现调用接口自带 `Authorization ` 头，通过 `Authorize` 按钮设置 Token，即可访问需登录的接口。

1. 在 `BaseSwaggerConfig` 中配置一个 Docket Bean 实例，配置映射路径和要扫描的接口的位置。

    - `securityContexts`：用来配置有哪些请求需要携带 Token。

2. 配置 `config/SwaggerConfig` 继承此 `BaseSwaggerConfig` 类：Swagger API 文档相关配置，没有 .yml 配置；

3. 在 `application.yml` 中给 `Swagger-UI` 安全路径**白名单**放行。

    ```yaml
    secure:
      ignored:
        urls: #安全路径白名单
          - /swagger-ui/index.html
    ```

Security 整合 Swagger3 相关配置的代码：

```java
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



## Security 自动配置类

> Spring Security 自动配置类，主要用于相关组件的配置

SecurityAutoConfiguration：

- SecurityProperties
- AuthenticationEntryPoint：认证失败处理类 Bean
- AccessDeniedHandler：权限不够处理器 Bean
- PasswordEncoder：Spring Security 加密器
- TokenAuthenticationFilter：Token 认证过滤器 Bean
- SecurityFrameworkService：

注意，不能和 {@link YudaoWebSecurityConfigurerAdapter} 用一个，原因是会导致初始化报错。

 * 参见 https://stackoverflow.com/questions/53847050/spring-boot-delegatebuilder-cannot-be-null-on-autowiring-authenticationmanager 文档。

### Token 认证过滤器

> TokenAuthenticationFilter，在 SecurityAutoConfiguration 中调用。

- 验证通过后，获得 {@link LoginUser} 信息，并加入到 Spring Security 上下文
    1. 情况一，基于 header[login-user] 获得用户，例如说来自 Gateway 或者其它服务透传
    2. 情况二，基于 Token 获得用户。
        - 注意，这里主要满足直接使用 Nginx 直接转发到 Spring Cloud 服务的场景。

```java
@RequiredArgsConstructor
@Slf4j
public class TokenAuthenticationFilter extends OncePerRequestFilter {

    private final SecurityProperties securityProperties;

    private final GlobalExceptionHandler globalExceptionHandler;

    private final OAuth2TokenCommonApi oauth2TokenApi;

    @Override
    @SuppressWarnings("NullableProblems")
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
            throws ServletException, IOException {
        // 情况一，基于 header[login-user] 获得用户，例如说来自 Gateway 或者其它服务透传
        LoginUser loginUser = buildLoginUserByHeader(request);

        // 情况二，基于 Token 获得用户
        // 注意，这里主要满足直接使用 Nginx 直接转发到 Spring Cloud 服务的场景。
        if (loginUser == null) {
            String token = SecurityFrameworkUtils.obtainAuthorization(request,
                    securityProperties.getTokenHeader(), securityProperties.getTokenParameter());
            if (StrUtil.isNotEmpty(token)) {
                Integer userType = WebFrameworkUtils.getLoginUserType(request);
                try {
                    // 1.1 基于 token 构建登录用户
                    loginUser = buildLoginUserByToken(token, userType);
                    // 1.2 模拟 Login 功能，方便日常开发调试
                    // forlai
                    if (loginUser == null) {
                        loginUser = mockLoginUser(request, token, userType);
                    }
                } catch (Throwable ex) {
                    CommonResult<?> result = globalExceptionHandler.allExceptionHandler(request, ex);
                    ServletUtils.writeJSON(response, result);
                    return;
                }
            }
        }

        // 设置当前用户
        if (loginUser != null) {
            SecurityFrameworkUtils.setLoginUser(loginUser, request);
        }
        // 继续过滤链
        chain.doFilter(request, response);
    }

    private LoginUser buildLoginUserByToken(String token, Integer userType) {
        try {
            // 校验访问令牌, forlai
            OAuth2AccessTokenCheckRespDTO accessToken = oauth2TokenApi.checkAccessToken(token).getCheckedData();
            if (accessToken == null) {
                return null;
            }
            // 用户类型不匹配，无权限
            // 注意：只有 /admin-api/* 和 /app-api/* 有 userType，才需要比对用户类型
            // 类似 WebSocket 的 /ws/* 连接地址，是不需要比对用户类型的
            if (userType != null
                    && ObjectUtil.notEqual(accessToken.getUserType(), userType)) {
                throw new AccessDeniedException("错误的用户类型");
            }
            // 构建登录用户
            return new LoginUser().setId(accessToken.getUserId()).setUserType(accessToken.getUserType())
                    .setInfo(accessToken.getUserInfo()) // 额外的用户信息
                    .setTenantId(accessToken.getTenantId()).setScopes(accessToken.getScopes())
                    .setExpiresTime(accessToken.getExpiresTime());
        } catch (ServiceException serviceException) {
            // 校验 Token 不通过时，考虑到一些接口是无需登录的，所以直接返回 null 即可
            return null;
        }
    }

    /**
     * 模拟登录用户，方便日常开发调试
     *
     * 注意，在线上环境下，一定要关闭该功能！！！
     *
     * @param request 请求
     * @param token 模拟的 token，格式为 {@link SecurityProperties#getMockSecret()} + 用户编号
     * @param userType 用户类型
     * @return 模拟的 LoginUser
     */
    private LoginUser mockLoginUser(HttpServletRequest request, String token, Integer userType) {
        if (!securityProperties.getMockEnable()) {
            return null;
        }
        // 必须以 mockSecret 开头
        if (!token.startsWith(securityProperties.getMockSecret())) {
            return null;
        }
        // 构建模拟用户
        Long userId = Long.valueOf(token.substring(securityProperties.getMockSecret().length()));
        return new LoginUser().setId(userId).setUserType(userType)
                .setTenantId(WebFrameworkUtils.getTenantId(request));
    }

    @SneakyThrows
    private LoginUser buildLoginUserByHeader(HttpServletRequest request) {
        String loginUserStr = request.getHeader(SecurityFrameworkUtils.LOGIN_USER_HEADER);
        if (StrUtil.isEmpty(loginUserStr)) {
            return null;
        }
        try {
            loginUserStr = URLDecoder.decode(loginUserStr, StandardCharsets.UTF_8.name()); // 解码，解决中文乱码问题
            LoginUser loginUser = JsonUtils.parseObject(loginUserStr, LoginUser.class);
            // 用户类型不匹配，无权限
            // 注意：只有 /admin-api/* 和 /app-api/* 有 userType，才需要比对用户类型
            // 类似 WebSocket 的 /ws/* 连接地址，是不需要比对用户类型的
            Integer userType = WebFrameworkUtils.getLoginUserType(request);
            if (userType != null
                    && loginUser != null
                    && ObjectUtil.notEqual(loginUser.getUserType(), userType)) {
                throw new AccessDeniedException("错误的用户类型");
            }
            return loginUser;
        } catch (Exception ex) {
            log.error("[buildLoginUserByHeader][解析 LoginUser({}) 发生异常]", loginUserStr, ex);  ;
            throw ex;
        }
    }

}
```



## 整合 Spring Session

### Spring Session + Redis

使用 Redis 作为 Spring Session 的存储器，这也是生产环境下，主流的选择。

#### 依赖

```xml
<!-- 实现对 Spring Session 使用 Redis 作为数据源的自动化配置 -->
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>

<!-- 实现对 Spring Data Redis 的自动化配置 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <exclusions>
        <!-- 去掉对 Lettuce 的依赖，因为 Spring Boot 优先使用 Lettuce 作为 Redis 客户端 -->
        <exclusion>
            <groupId>io.lettuce</groupId>
            <artifactId>lettuce-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<!-- 引入 Jedis 的依赖，这样 Spring Boot 实现对 Jedis 的自动化配置 -->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
```

#### 配置文件

在 [`resources`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-26/lab-26-distributed-session-01/src/main/resources) 目录下，创建 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-26/lab-26-distributed-session-01/src/main/resources/application.yaml) 配置文件。

```yaml
spring:
  # 对应 RedisProperties 类
  redis:
    host: 127.0.0.1
    port: 6379
    password: # Redis 服务器密码，默认为空。生产中，一定要设置 Redis 密码！
    database: 0 # Redis 数据库号，默认为 0 。
    timeout: 0 # Redis 连接超时时间，单位：毫秒。
    # 对应 RedisProperties.Jedis 内部类
    jedis:
      pool:
        max-active: 8 # 连接池最大连接数，默认为 8 。使用负数表示没有限制。
        max-idle: 8 # 默认连接数最大空闲的连接数，默认为 8 。使用负数表示没有限制。
        min-idle: 0 # 默认连接池最小空闲的连接数，默认为 0 。允许设置 0 和 正数。
        max-wait: -1 # 连接池最大阻塞等待时间，单位：毫秒。默认为 -1 ，表示不限制。
```

#### SessionConfiguration

创建 [SessionConfiguration](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-26/lab-26-distributed-session-01/src/main/java/cn/iocoder/springboot/lab26/distributedsession/config/SessionConfiguration.java) 类，自定义 Spring Session Redis 的配置。

- 在类上，添加 [`@EnableRedisHttpSession`](https://github.com/spring-projects/spring-session/blob/master/spring-session-data-redis/src/main/java/org/springframework/session/data/redis/config/annotation/web/http/EnableRedisHttpSession.java) 注解，开启自动化配置 Spring Session 使用 Redis 作为数据源。该注解有如下属性：
    - `maxInactiveIntervalInSeconds` 属性，Session 不活跃后的过期时间，默认为 1800 秒。
    - `redisNamespace` 属性，在 Redis 的 key 的统一前缀，默认为 `"spring:session"` 。
    - `redisFlushMode` 属性，Redis 会话刷新模式([RedisFlushMode](https://github.com/spring-projects/spring-session/blob/master/spring-session-data-redis/src/main/java/org/springframework/session/data/redis/RedisFlushMode.java))。目前有两种，默认为 `RedisFlushMode.ON_SAVE` 。
        - `RedisFlushMode.ON_SAVE` ，在请求执行完成时，统一写入 Redis 存储。
        - `RedisFlushMode.IMMEDIATE` ，在每次修改 Session 时，立即写入 Redis 存储。
    - `cleanupCron` 属性，清理 Redis Session 会话过期的任务执行 CRON 表达式，默认为 `"0 * * * * *"` 每分钟执行一次。
        - 虽然说，Redis 自带了 key 的过期，但是**惰性删除策略**，实际过期的 Session 还在 Redis 中占用内存。
        - 所以，Spring Session 通过定时任务，删除 Redis 中过期的 Session ，尽快释放 Redis 的内存。
        - 不了解 Redis 的删除过期 key 的策略的胖友，可以看看 [《Redis 中删除过期 Key 的三种策略》](https://blog.csdn.net/a_bang/article/details/52986935/) 文章。
- 在 `#springSessionDefaultRedisSerializer()` 方法，定义了一个 Bean 名字为 `"springSessionDefaultRedisSerializer"` 的 RedisSerializer Bean ，采用 JSON 序列化方式。因为默认情况下，采用 [Java 自带的序列化方式](https://juejin.im/post/5ce3cdc8e51d45777b1a3cdf) ，可读性很差，所以进行替换。

```java
// SessionConfiguration.java

@Configuration
@EnableRedisHttpSession // 自动化配置 Spring Session 使用 Redis 作为数据源
public class SessionConfiguration {

    /**
     * 创建 {@link RedisOperationsSessionRepository} 使用的 RedisSerializer Bean 。
     *
     * 具体可以看看 {@link RedisHttpSessionConfiguration#setDefaultRedisSerializer(RedisSerializer)} 方法，
     * 它会引入名字为 "springSessionDefaultRedisSerializer" 的 Bean 。
     *
     * @return RedisSerializer Bean
     */
    @Bean(name = "springSessionDefaultRedisSerializer")
    public RedisSerializer springSessionDefaultRedisSerializer() {
        return RedisSerializer.json();
    }

}
```

### Spring Session + MongoDB



### 整合 Spring Session + Spring Security 



## 整合 OAuth2



## 整合 JWT



## 项目实战

在开源项目翻了一圈，找到一个相对合适项目 [RuoYi-Vue](https://gitee.com/y_project/RuoYi-Vue) 。主要以下几点原因：

- 基于 Spring Security 实现。
- 基于 RBAC 权限模型，并且支持动态的权限配置。
- 基于 Redis 服务，实现登录用户的信息缓存。
- 前后端分离。同时前端采用 Vue ，相对来说后端会 Vue 的比 React 的多。

### [SysMenu](https://github.com/YunaiV/RuoYi-Vue/blob/master/ruoyi/src/main/java/com/ruoyi/project/system/domain/SysMenu.java) 

菜单权限实体类。

- `menuType` 属性，定义了三种类型。其中，`F` 代表按钮，是为了做页面中的功能级的权限。

- `perms` 属性，对应的权限**标识**字符串。一般格式为 `${大模块}:${小模块}:{操作}` 。示例如下：

    ```
    用户查询：system:user:query
    用户新增：system:user:add
    用户修改：system:user:edit
    用户删除：system:user:remove
    用户导出：system:user:export
    用户导入：system:user:import
    重置密码：system:user:resetPwd
    ```

    - 对于前端来说，每个按钮在展示时，可以判断用户是否有该按钮的权限。如果没有，则进行隐藏。当然，前端在首次进入系统的时候，会请求一次权限列表到本地进行缓存。
    - 对于后端来说，每个接口上会添加 `@PreAuthorize("@ss.hasPermi('system:user:list')")` 注解。在请求接口时，会校验用户是否有该 URL 对应的权限。如果没有，则会抛出权限验证失败的异常。
    - 一个 `perms` 属性，可以对应多个权限**标识**，使用逗号分隔。例如说：`"system:user:query,system:user:add"` 。

### SecurityConfig

在 [SecurityConfig](https://github.com/YunaiV/RuoYi-Vue/blob/master/ruoyi/src/main/java/com/ruoyi/framework/config/SecurityConfig.java) 配置类，继承 WebSecurityConfigurerAdapter 抽象类，实现 Spring Security 在 Web 场景下的自定义配置。

```java
// SecurityConfig.java

@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    // ...

}
```

- 涉及到的配置方法较多。

#### `#configure(AuthenticationManagerBuilder auth)`

重写 `#configure(AuthenticationManagerBuilder auth)` 方法，实现 [AuthenticationManager](https://github.com/spring-projects/spring-security/blob/master/core/src/main/java/org/springframework/security/authentication/AuthenticationManager.java) 认证管理器。

```java
// SecurityConfig.java

/**
 * 自定义用户认证逻辑
 */
@Autowired
private UserDetailsService userDetailsService;

/**
 * 身份认证接口
 */
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.userDetailsService(userDetailsService) // <X>
            .passwordEncoder(bCryptPasswordEncoder()); // <Y>
}

/**
 * 强散列哈希加密实现
 */
@Bean
public BCryptPasswordEncoder bCryptPasswordEncoder() {
    return new BCryptPasswordEncoder();
}
```

- `<X>` 处，调用 `AuthenticationManagerBuilder#userDetailsService(userDetailsService)` 方法，使用自定义实现的 [UserDetailsService](https://github.com/spring-projects/spring-security/blob/master/core/src/main/java/org/springframework/security/core/userdetails/UserDetailsService.java) 实现类，更加**灵活**且**自由**的实现认证的用户信息的读取。在[「7.3.1 加载用户信息」](https://www.iocoder.cn/Spring-Boot/Spring-Security/#)中，我们会看到 RuoYi-Vue 对 UserDetailsService 的自定义实现类。
- `<Y>` 处，调用 `AbstractDaoAuthenticationConfigurer#passwordEncoder(passwordEncoder)` 方法，设置 PasswordEncoder 密码编码器。这里，就使用了 bCryptPasswordEncoder 强散列哈希加密实现。

#### `#configure(HttpSecurity httpSecurity)`

重写 `#configure(HttpSecurity httpSecurity)` 方法，主要配置 URL 的权限控制。

```java
// SecurityConfig.java

/**
 * 认证失败处理类
 */
@Autowired
private AuthenticationEntryPointImpl unauthorizedHandler;

/**
 * 退出处理类
 */
@Autowired
private LogoutSuccessHandlerImpl logoutSuccessHandler;

/**
 * token 认证过滤器
 */
@Autowired
private JwtAuthenticationTokenFilter authenticationTokenFilter;

@Override
protected void configure(HttpSecurity httpSecurity) throws Exception {
    httpSecurity
            // CRSF禁用，因为不使用session
            .csrf().disable()
            // <X> 认证失败处理类
            .exceptionHandling().authenticationEntryPoint(unauthorizedHandler).and()
            // 基于token，所以不需要session
            .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS).and()
            // 过滤请求
            .authorizeRequests()
            // <Y> 对于登录login 验证码captchaImage 允许匿名访问
            .antMatchers("/login", "/captchaImage").anonymous()
            .antMatchers(
                    HttpMethod.GET,
                    "/*.html",
                    "/**/*.html",
                    "/**/*.css",
                    "/**/*.js"
            ).permitAll()
            .antMatchers("/profile/**").anonymous()
            .antMatchers("/common/download**").anonymous()
            .antMatchers("/swagger-ui.html").anonymous()
            .antMatchers("/swagger-resources/**").anonymous()
            .antMatchers("/webjars/**").anonymous()
            .antMatchers("/*/api-docs").anonymous()
            .antMatchers("/druid/**").anonymous()
            // 除上面外的所有请求全部需要鉴权认证
            .anyRequest().authenticated()
            .and()
            .headers().frameOptions().disable();
    httpSecurity.logout().logoutUrl("/logout").logoutSuccessHandler(logoutSuccessHandler); // <Z>
    // <P> 添加 JWT filter
    httpSecurity.addFilterBefore(authenticationTokenFilter, UsernamePasswordAuthenticationFilter.class);
}
```

比较长，选择重点的来看。

- `<X>` 处，设置认证失败时的处理器为 `unauthorizedHandler` 。详细解析，见[「7.6.1 AuthenticationEntryPointImpl」](https://www.iocoder.cn/Spring-Boot/Spring-Security/#)。
- `<Y>` 处，设置用于登录的 `/login` 接口，允许匿名访问。这样，后续我们就可以使用自定义的登录接口。详细解析，见[「7.3 登录 API 接口」](https://www.iocoder.cn/Spring-Boot/Spring-Security/#)。
- `<Z>` 处，设置登出成功的处理器为 `logoutSuccessHandler` 。详细解析，见[「7.6.3 LogoutSuccessHandlerImpl」](https://www.iocoder.cn/Spring-Boot/Spring-Security/#)。
- `<P>` 处，添加 JWT 认证过滤器 `authenticationTokenFilter` ，用于用户使用用户名与密码登录完成后，后续请求基于 JWT 来认证。 详细解析，见[「7.4 JwtAuthenticationTokenFilter」](https://www.iocoder.cn/Spring-Boot/Spring-Security/#)。

#### `#authenticationManagerBean`

重写 `#authenticationManagerBean` 方法，解决无法直接注入 AuthenticationManager 的问题。

- 在方法上，额外添加了 `@Bean` 注解，保证创建出 AuthenticationManager Bean 。

```java
// SecurityConfig.java

@Bean
@Override
public AuthenticationManager authenticationManagerBean() throws Exception {
    return super.authenticationManagerBean();
}
```



### JwtAuthenticationTokenFilter

在 [JwtAuthenticationTokenFilter](https://github.com/YunaiV/RuoYi-Vue/blob/master/ruoyi/src/main/java/com/ruoyi/framework/security/filter/JwtAuthenticationTokenFilter.java) 中，继承 [OncePerRequestFilter](https://github.com/spring-projects/spring-framework/blob/master/spring-web/src/main/java/org/springframework/web/filter/OncePerRequestFilter.java) 过滤器，实现了基于 Token 的认证。

### 权限验证

在[「3. 进阶使用」](https://www.iocoder.cn/Spring-Boot/Spring-Security/#)中，我们看到可以通过 Spring Security 提供的 `@PreAuthorize` 注解，实现基于 Spring EL 表达式的执行结果为 `true` 时，可以访问，从而实现灵活的权限校验。

在 RuoYi-Vue 中，通过 `@PreAuthorize` 注解的特性，使用其 [PermissionService](https://github.com/YunaiV/RuoYi-Vue/blob/master/ruoyi/src/main/java/com/ruoyi/framework/security/service/PermissionService.java) 提供的权限验证的方法。

- 请求 `/system/dict/data/list` 接口，会调用 PermissionService 的 `#hasPermi(String permission)` 方法，校验用户是否有指定的权限。
- 为什么这里会有一个 `@ss` 呢？在 Spring EL 表达式中，调用指定 Bean 名字的方法时，使用 `@` + Bean 的名字。在 RuoYi-Vue 中，声明 PermissionService 的 Bean 名字为 `ss` 。

```java
// SysDictDataController.java

@PreAuthorize("@ss.hasPermi('system:dict:list')")
@GetMapping("/list")
```

#### 判断是否有权限

在 PermissionService 中，定义了 `#hasPermi(String permission)` 方法，判断当前用户是否**有**指定的权限。

#### 判断是否有角色

在 PermissionService 中，定义了 `#hasRole(String role)` 方法，判断当前用户是否**有**指定的角色。

### 各种处理器

在 Ruoyi-Vue 中，提供了各种处理器，处理各种情况，所以我们汇总在[「7.6 各种处理器」](https://www.iocoder.cn/Spring-Boot/Spring-Security/#) 中，一起来瞅瞅。

#### AuthenticationEntryPointImpl

在 [AuthenticationEntryPointImpl](https://github.com/YunaiV/RuoYi-Vue/blob/master/ruoyi/src/main/java/com/ruoyi/framework/security/handle/AuthenticationEntryPointImpl.java) 中，实现 Spring Security [AuthenticationEntryPoint](https://github.com/spring-projects/spring-security/blob/master/web/src/main/java/org/springframework/security/web/AuthenticationEntryPoint.java) 接口，处理认失败的 [AuthenticationException](https://github.com/spring-projects/spring-security/blob/master/core/src/main/java/org/springframework/security/core/AuthenticationException.java) 异常。

#### GlobalExceptionHandler

在 [GlobalExceptionHandler](https://github.com/YunaiV/RuoYi-Vue/blob/master/ruoyi/src/main/java/com/ruoyi/framework/web/exception/GlobalExceptionHandler.java) 中，定义了对 Spring Security 的异常处理。

#### LogoutSuccessHandlerImpl

在 [LogoutSuccessHandlerImpl](https://github.com/YunaiV/RuoYi-Vue/blob/master/ruoyi/src/main/java/com/ruoyi/framework/security/handle/LogoutSuccessHandlerImpl.java) 中，实现 Spring Security [LogoutSuccessHandler](https://github.com/spring-projects/spring-security/blob/master/web/src/main/java/org/springframework/security/web/authentication/logout/LogoutSuccessHandler.java) 接口，自定义退出的处理，主动删除 LoginUser 在 Redis 中的缓存。

### 权限管理

如下的 Controller ，提供了 RuoYi-Vue 的**权限管理**功能，比较简单，胖友自己去瞅瞅即可。

- 用户管理 [SysUserController](https://github.com/YunaiV/RuoYi-Vue/blob/master/ruoyi/src/main/java/com/ruoyi/project/system/controller/SysUserController.java) ：用户是系统操作者，该功能主要完成系统用户配置。
- 角色管理 [SysRoleController](https://github.com/YunaiV/RuoYi-Vue/blob/master/ruoyi/src/main/java/com/ruoyi/project/system/controller/SysRoleController.java) ：角色菜单权限分配、设置角色按机构进行数据范围权限划分。
- 菜单管理 [SysMenuController](https://github.com/YunaiV/RuoYi-Vue/blob/master/ruoyi/src/main/java/com/ruoyi/project/system/controller/SysMenuController.java) ：配置系统菜单，操作权限，按钮权限标识等。

## 权限

### RBAC 权限模型

系统采用 RBAC 权限模型，全称是 `Role-Based Access Control` **基于角色**的访问控制。

- 简单来说，每个用户拥有若干角色，每个角色拥有若干个菜单，菜单中存在菜单权限、按钮权限。
- 这样，就形成了 **“用户<->角色<->菜单”** 的授权模型。 

- 在这种模型中，用户与角色、角色与菜单之间构成了**多对多**的关系。

### Token 认证

[AppAuthController.java](https://github.com/YunaiV/yudao-cloud/tree/master/yudao-module-member/yudao-module-member-server/src/main/java/cn/iocoder/yudao/module/member/controller/app/auth/AppAuthController.java)

Token 存储在数据库中，对应 `system_oauth2_access_token` 访问令牌表的 **`id` 字段**。

- 考虑到访问的性能，缓存在 Redis 的 [`oauth2_access_token:%s`](https://github.com/YunaiV/yudao-cloud/blob/master/yudao-module-system/yudao-module-system-server/src/main/java/cn/iocoder/yudao/module/system/dal/redis/RedisKeyConstants.java#L21)键中。

疑问：为什么不使用 **JWT**（JSON Web Token）？

- JWT 是**无状态的**，无法实现 **Token 的作废**，例如说用户登出系统、修改密码等场景。


默认配置下，Token 有效期为 30 天，可通过 `system_oauth2_client` 表中 `client_id = default` 的记录进行自定义：

![ 表](../assets/04.png)

- 修改 `access_token_validity_seconds` 字段，设置**访问令牌**的过期时间，默认 1800 秒 = 30 分钟
- 修改 `refresh_token_validity_seconds` 字段，设置**刷新令牌**的过期时间，默认 2592000 秒 = 30 天

② 前端调用其它接口，需要在请求头带上 Token 进行访问。

请求头格式如下：

```bash
### Authorization: Bearer 登录时返回的 Token
Authorization: Bearer d2a3cdbc6c53470db67a582bd115103f
```

- 具体的代码实现，可见 [TokenAuthenticationFilter](https://github.com/YunaiV/yudao-cloud/blob/master/yudao-framework/yudao-spring-boot-starter-security/src/main/java/cn/iocoder/yudao/framework/security/core/filter/TokenAuthenticationFilter.java) 过滤器。

#### Token 的模拟机制

考虑到使用 Postman、Swagger 调试接口方便，提供了 **Token 的模拟机制**。

请求头格式如下：

```bash
### Authorization: Bearer test用户编号
Authorization: Bearer test1
```

其中 `"test"` 可自定义，配置项如下：

```yaml
### application-local.yaml

yudao:
  security:
    mock-enable: true # 是否开启 Token 的模拟机制
    mock-secret: test # Token 模拟机制的 Token 前缀
```

### @PreAuthorize 权限注解

[`@PreAuthorize` (opens new window)](https://github.com/spring-projects/spring-security/blob/main/core/src/main/java/org/springframework/security/access/prepost/PreAuthorize.java)是 Spring Security 内置的**前置**权限注解，添加在**接口方法**上，声明需要的权限，实现访问权限的控制。

#### 基于【权限标识】的权限控制

权限标识：对应 `system_menu` 权限相关表的 `permission` 字段，推荐格式为 `${系统}:${模块}:${操作}`，例如说 `system:admin:add` 标识 system 服务的添加管理员。

使用示例如下：

```java
// 符合 system:user:list 权限要求
@PreAuthorize("@ss.hasPermission('system:user:list')")

// 符合 system:user:add 或 system:user:edit 权限要求即可
@PreAuthorize("@ss.hasAnyPermissions('system:user:add', 'system:user:edit')")
```

#### 基于【角色标识】的权限控制

权限标识：对应 `system_role` 表的 `code` 字段， 例如说 `super_admin` 超级管理员、`tenant_admin` 租户管理员。

使用示例如下：

```java
// 属于 user 角色
@PreAuthorize("@ss.hasRole('user')")

// 属于 user 或者 admin 之一
@PreAuthorize("@ss.hasAnyRoles('user', 'admin')")
```

实现原理是什么？

- `@PreAuthorize("@ss.hasPermission('system:user:list')")` 表示调用 **Bean 名字为 `ss`** 的 `#hasPermission(...)` 方法，方法参数为 `"system:user:list"` 字符串。
    - `ss` 对应的 Bean 是 [PermissionServiceImpl](https://github.com/YunaiV/yudao-cloud/blob/master/yudao-module-system/yudao-module-system-server/src/main/java/cn/iocoder/yudao/module/system/service/permission/PermissionServiceImpl.java#L43)类，所以只需要去看该方法的[实现代码](https://github.com/YunaiV/yudao-cloud/blob/master/yudao-module-system/yudao-module-system-server/src/main/java/cn/iocoder/yudao/module/system/service/permission/PermissionServiceImpl.java#L293-L326)。
    - 当 `@PreAuthorize` 注解里的 **Spring EL 表达式**返回 `false` 时，表示没有权限。

### 自定义权限配置

默认配置下，所有接口都需要登录后才能访问，不限于管理后台的 `/admin-api/**` 所有 API 接口、用户 App 的 `/app-api/**` 所有 API 接口。

如下想要自定义权限配置，设置定义 API 接口可以匿名（不登录）进行访问，可以通过下面三种方式：

##### 自定义 AuthorizeRequestsCustomizer 实现

每个 Maven Module 可以实现自定义的 [AuthorizeRequestsCustomizer (opens new window)](https://github.com/YunaiV/yudao-cloud/blob/master/yudao-framework/yudao-spring-boot-starter-security/src/main/java/cn/iocoder/yudao/framework/security/config/YudaoWebSecurityConfigurerAdapter.java)Bean，额外定义每个 Module 的 API 接口的访问规则。例如说 `yudao-module-infra` 模块的 [SecurityConfiguration (opens new window)](https://github.com/YunaiV/yudao-cloud/blob/master/yudao-module-infra/yudao-module-infra-server/src/main/java/cn/iocoder/yudao/module/infra/framework/security/config/SecurityConfiguration.java)类。

```java
@Configuration(proxyBeanMethods = false, value = "infraSecurityConfiguration")
public class SecurityConfiguration {

    @Value("${spring.boot.admin.context-path:''}")
    private String adminSeverContextPath;

    @Bean("infraAuthorizeRequestsCustomizer")
    public AuthorizeRequestsCustomizer authorizeRequestsCustomizer() {
        return new AuthorizeRequestsCustomizer() {

            @Override
            public void customize(AuthorizeHttpRequestsConfigurer<HttpSecurity>.AuthorizationManagerRequestMatcherRegistry registry) {
                // Swagger 接口文档
                registry.requestMatchers("/v3/api-docs/**").permitAll()
                        .requestMatchers("/swagger-ui.html").permitAll()
                        .requestMatchers("/swagger-ui/**").permitAll()
                        .requestMatchers("/swagger-resources/**").permitAll()
                        .requestMatchers("/webjars/**").permitAll()
                        .requestMatchers("/*/api-docs").permitAll();
                // Spring Boot Actuator 的安全配置
                registry.requestMatchers("/actuator").permitAll()
                        .requestMatchers("/actuator/**").permitAll();
                // Druid 监控
                registry.requestMatchers("/druid/**").permitAll();
                // Spring Boot Admin Server 的安全配置
                registry.requestMatchers(adminSeverContextPath).permitAll()
                        .requestMatchers(adminSeverContextPath + "/**").permitAll();
                // 文件读取
                registry.requestMatchers(buildAdminApi("/infra/file/*/get/**")).permitAll();
            }

        };
    }

}
```

友情提示

- `permitAll()` 方法：所有用户可以任意访问，包括带上 Token 访问
- `anonymous()` 方法：匿名用户可以任意访问，带上 Token 访问会报错

如果你对 Spring Security 了解不多，可以阅读艿艿写的 [《芋道 Spring Boot 安全框架 Spring Security 入门 》 (opens new window)](https://www.iocoder.cn/Spring-Boot/Spring-Security/?yudao)文章。

##### `@PermitAll` 注解

在 API 接口上添加 [`@PermitAll` (opens new window)](https://javaee.github.io/javaee-spec/javadocs/javax/annotation/security/PermitAll.html)注解，示例如下：

```java
// FileController.java
@GetMapping("/{configId}/get/{path}")
@PermitAll
public void getFileContent(HttpServletResponse response,
                           @PathVariable("configId") Long configId,
                           @PathVariable("path") String path) throws Exception {
    // ...
}
```

##### 配置文件

在 `application.yaml` 配置文件，设置`yudao.security.permit-all-urls` 配置项。

```yaml
yudao:
  security:
    permit-all-urls:
      - /admin-ui/** # /resources/admin-ui 目录下的静态资源
      - /admin-api/xxx/yyy
```

