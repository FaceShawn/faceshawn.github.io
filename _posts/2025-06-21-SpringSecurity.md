---
title: Spring Security
categories:
  - 微服务
tags:
  - 微服务
  - 分布式
location:
  - 黄金时代
abbrlink: 'spring_security'
permalink: 'spring_security'
date: 2025-06-21 13:42:12
updated: 2025-06-21 13:42:12
---

> 摘要：Redis 分布式缓存。

<!-- more -->

## 目录

[TOC]

## Spring Security

Spring Security：是一种强大的**安全框架**，基于 Spring IOC/DI 和 AOP 功能，为系统提供了**声明式安全访问控制**功能，减少了为系统安全而编写大量重复代码的工作。

可以帮助用户保护Web应用程序和REST API的安全性。它提供了多种身份验证、授权、加密和防止攻击等功能，可以根据用户的需求来选择适合自己应用场景的功能。

使用Spring Security可以提高应用程序的安全性，从而保护用户的敏感信息和数据。

> **核心功能**：认证和授权

###  Spring Security 入门简介

Spring Security 通过使用标准的 Servlet `Filter` 与 Servlet 容器集成。这意味着它适用于任何在 Servlet 容器中运行的应用程序。更具体地说，不需要在基于 Servlet 的应用程序中使用 Spring 来利用 Spring Security。

#### Spring Security 的主要功能

Spring Security 的主要功能包括：

1. **认证（Authenti’cation）**：验证用户的身份（例如用户名/用户 ID 和密码），通过这个凭据，系统得以知道**你是谁**。被称为**身份/用户验证**。
    - Spring Security提供了多种身份验证机制，如**基于表单的**身份验证、HTTP基本身份验证、LDAP身份验证等。
    - **安全上下文 (Security Context)**：存储已认证用户的详细信息，应用程序中可以访问。
2. **授权（Authorization）**：发生在 **认证** 之后，主要掌管**系统权限控制**、**你有权限干什么**。比如有些特定资源只能具有特定权限的人才能访问、删除、添加、更新，比如 admin。Spring Security提供了**RBAC 基于角色的权限访问控制**。
    1. **权限（Permission）**：通常指对**特定资源**的操作能力，如读取、写入或删除。
    2. **角色（Role）**：一组权限的集合。例如，管理员角色可能具有所有权限，而普通用户角色可能只有读取权限。
3. **漏洞防护、安全**：
    1. **加密**：Spring Security提供了多种**加密算法**的支持，如`MD5、SHA、BCrypt`等。可以用加密API来对用户密码等敏感信息进行加密。
    2. **防护机制**：Spring Security提供了多种防止攻击的机制，如 **CSRF保护**、**XSS防护**、**会话管理**、防止会话劫持等。
4. **单点登录**：Spring Security提供了单点登录（SSO）的支持，可以帮助用户在多个应用程序之间实现单点登录。用户只需要进行一次登录，就可以自动登录到其他应用程序中，从而提高用户的使用体验。
5. **集成其他框架**：Spring Security可以与其他框架集成，如`Spring Boot、Spring MVC、Spring Cloud`等。用户可以在不改变现有框架架构的情况下，使用Spring Security来提高应用程序的安全性。

#### 常见的**认证和授权**方式

> 认证和授权一般在系统中结合在一起使用，目的就是为了保护系统的安全性。

1. Session 和 Cookie 方案；
2. JWT；
3. SSO 单点登录；
4. `Spring Security + OAuth2`：强大的可高度定制的**认证和授权框架**，是一套 Web 安全标准。
5. Apache **Shiro** 权限管理框架：强大易用的 Java 安全框架，提供了认证、授权、加密和会话管理功能。

参考：[spring security 超详细使用教程（接入springboot、前后端分离）](https://www.cnblogs.com/xxctx/p/18441536)

### 架构

#### Filter（过滤器）回顾

#### Security Filter

### 认证

#### 用户名、密码

> 通过 PasswodEncoder 接口加密。

#### 持久化



#### 会话管理



#### Cookie 和 Session

##### 简介

相同：都是用来跟踪浏览器**用户身份**的会话方式，但是两者的应用场景不太一样。

`Cookies`：是某些网站为了辨别用户身份而储存在用户本地终端上的数据（通常经过加密）。

区别：

- `Cookie`：是本地**客户端**用来存储少量用户信息的（通常经过加密）；
    1. 保存在客户端，用户能很容易的获取，安全性不高；
    2. 存储的数据量小。
    3. 应用案例：
        1. 保存已经登录过的用户信息，
        2. 保存用户首选项、主题和其他设置信息，
        3. 保存 `SessionId` 或 `Token`  用于记录用户当前的状态（因为 HTTP 协议是无状态的），
        4. 用来记录和分析用户行为，比如网上购物时，服务器获取某个页面的停留状态、或看了哪些商品，一种常用的实现方式就是将这些信息存放在 `Cookie`。
- `Session`：是**服务器**用来存储用户信息、通过服务端记录用户的状态；
    1. 保存在服务器，用户不易获取，**安全性更高**；
    2. 储存的数据量相对大，占用服务器资源；
    3. 典型场景:
        1. **购物车**：系统不知道是哪个用户操作的，因为 HTTP 协议是**无状态**的。服务端给特定的用户创建特定的 `Session` 之后就可以**标识**这个用户并且**跟踪**这个用户了。
        2. 个性化推送、免密登录。如 `HttpSession`。

##### 如何在项目中使用 Cookie？

> 以 Spring Boot 项目为例。

1) 创建 `Cookie`，并通过 `HttpServletResponse` 返回给客户端；

```java
@GetMapping("/change-username")
public String setCookie(HttpServletResponse response) {
    // 创建一个 cookie
    Cookie cookie = new Cookie("username", "Jovan");
    //设置 cookie过期时间
    cookie.setMaxAge(7 * 24 * 60 * 60); // expires in 7 days
    //添加到 response 中
    response.addCookie(cookie);

    return "Username is changed!";
}
```

2) 使用 Spring 框架提供的 `@CookieValue` 注解获取特定的 cookie 的值；

```java
@GetMapping("/")
public String readCookie(@CookieValue(value = "username", defaultValue = "Atta") String username) {
    return "Hey! My username is " + username;
}
```

3) 通过 `HttpServletRequest` 的`getCookies()`方法，读取所有的 `Cookie` 值；

```java
@GetMapping("/all-cookies")
public String readAllCookies(HttpServletRequest request) {

    Cookie[] cookies = request.getCookies();
    if (cookies != null) {
        return Arrays.stream(cookies)
                .map(c -> c.getName() + "=" + c.getValue())
            	.collect(Collectors.joining(", "));
    }

    return "No cookies";
}
```

##### Session 原理

在一次客户端和服务器之间的会话中，客户端（浏览器）向服务器发送请求，

1. 首先cookie会自动携带上次请求存储的数据（JSESSIONID），服务器根据**请求参数**中的JSESSIONID到session库查询是否存在，
2. 如果存在，那么服务器就知道此用户是谁，
3. 如果不存在，就会创建一个JSESSIONID，并在本次请求结束后将JSESSIONID返回给客户端，同时在客户端cookie中进行保存；
4. 当浏览器关闭时，会话就结束了，但会话session还在。默认 session 保留30分钟。

客户端和服务器间是通过http协议进行通信，但 http 协议是无状态的，不同次请求会话是没有任何关联的，优点是处理速度快。

<img src="assets/885859-20190925230016412-226837887.png" style="zoom:100%;" />

##### HttpSession 生命周期

创建 HttpSession 对象的时机：

1. 对于JSP：浏览器访问服务端的任何一个JSP或Servlet，服务器不会为JSP创建一个 HttpSession 对象的情况：
    - 若当前的JSP或（Servlet）是客户端访问的当前WEB应用的**第一个资源**，且JSP的page指定属性`session="false"`，当前JSP页面禁用session隐含变量！但可以使用其他的显式的对象；
    - 若当前JSP不是访问的第一个资源，且其他页面已经创建一个（和当前会话关联的）HttpSession对象，直接返回返回。

2. 对于Servlet而言：若Servlet是客户端访问的第一个WEB应用的资源，则只有调用了`request.getSession([true])` 才会创建HttpSession对象。

Servlet 获取 HttpSession对象：`request.getSession(boolean create)`：

1. create为false，若没有和当前JSP页面关联的HttpSession对象，则返回null；若有则返回true；
2. create默认为true一定返回一个HTTPSession对象：若没有和当前JSP页面关联的HttpSession对象，则服务器创建一个新的HttpSession对象返回，若有则直接返回关联。

销毁 HttpSession 对象的时机：

1. 直接调用HttpSession的`invalidate()`方法：使HttpSession失效。
2. 服务器卸载了当前Web应用。
3. 超出HttpSession的过期时间。

##### 如何使用 Session-Cookie 方案进行身份验证？

很多时候我们都是通过 `SessionID` 来实现特定的用户，`SessionID` 一般会选择存放在 Redis 中。

举个例子：

1. 用户向服务器发送用户名、密码、验证码用于登陆系统。
2. 服务器验证通过后，会为这个用户创建一个专属的 **Session 对象**（可以理解为服务器上的一块内存，存放该用户的状态数据，如购物车、登录信息等）存储起来，并给这个 Session **分配一个唯一的 `SessionID`**，一般会选择存放在 **Redis** 中。
3. 服务器通过 **HTTP 响应头**中的 `Set-Cookie` 指令，把这个 `SessionID` 返回给用户的浏览器。
4. 浏览器接收到 `SessionID` 后，会将其以 **Cookie 的形式**保存在本地。
5. 当用户**保持登录状态**时，每次向该服务器发请求，浏览器都会自动带上这个存有 `SessionID` 的 Cookie。
6. 服务器收到请求后，从 Cookie 中拿出 `SessionID`，就能找到之前保存的那个 Session 对象，从而知道这是哪个用户以及他之前的状态了。

##### 使用 Session 时需要注意：

- 客户端 Cookie 支持：依赖 Session 的核心功能要确保用户浏览器开启了 Cookie。
- Session 过期管理：合理设置 Session 的过期时间，平衡安全性和用户体验。
- Session ID 安全：为包含 `SessionID` 的 Cookie 设置 **`HttpOnly` 标志**可以防止客户端脚本（如 JavaScript）窃取，设置 **Secure 标志**可以保证 `SessionID` 只在 HTTPS 连接下传输，增加安全性。

##### 多服务器节点下 Session-Cookie 方案如何做？

举个例子：假如我们部署了两份相同的服务 A，B，

- 用户第一次登陆的时候 ，Nginx 通过**负载均衡机制**将用户请求转发到 A 服务器，此时用户的 Session 信息保存在 A 服务器。
- 结果，用户第二次访问的时候 Nginx 将请求路由到 B 服务器，由于 B 服务器没有保存 用户的 Session 信息，导致用户需要**重新进行登陆**。

> 应该如何避免上面这种情况的出现呢？

有几个方案可供大家参考：

1. 某个用户的所有请求都通过特性的**哈希策略**分配给同一个服务器处理。这样的话，每个服务器都保存了一**部分**用户的 Session 信息。服务器宕机，其保存的所有 Session 信息就完全**丢失**了。
2. 每一个服务器保存的 Session 信息都是**互相同步**的，也就是说每一个服务器都保存了**全量**的 Session 信息。每当一个服务器的 Session 信息发生变化，就将其同步到其他服务器。**成本太大**，节点越多、同步成本也越高。
3. 单独使用一个所有服务器都能访问到的数据节点（比如**缓存**）来存放 Session 信息。为了保证高可用，数据节点尽量要避免是单点。
4. Spring Session 是一个用于在多个服务器之间**管理会话**的项目。可以与多种后端存储（如 Redis、MongoDB 等）集成，从而实现**分布式会话管理**。通过 Spring Session，可以将会话数据存储在共享的外部存储中，以实现**跨服务器**的会话同步和共享。

##### 为什么 Cookie 无法防止 CSRF 攻击，而 Token 可以？

使用 Session-Cookie 方案进行身份验证时，容易出现[**CSRF **跨站请求伪造](#攻击原理)，如果别人通过 `Cookie` 拿到了 `SessionId` 后就可以**代替**你的身份访问系统了。

但是，使用 `Token` 的话就不会存在这个问题：

1. 在登录成功获得 `Token` 之后，一般会选择存放在 `localStorage` （浏览器本地存储）中。
2. 然后在前端通过某些方式会给每个发到后端的请求加上这个 `Token`，这样就不会出现 CSRF 漏洞的问题。
3. 因为，即使点击了**非法链接**发送了请求到服务端，这个非法请求是不会携带 `Token` 的，所以这个请求将是非法的。

需要注意：不论是 `Cookie` 还是 `Token` 都无法避免 **XSS 跨站脚本攻击** 。

跨站脚本攻击（Cross Site Scripting，XSS）：攻击者会用各种方式将恶意代码注入到其他用户的页面中。就可以通过脚本盗用信息比如 `Cookie` 。

![](assets/20210615161108272.png)

#### 无状态登录

有状态服务：即服务端需要记录每次会话的客户端信息，从而识别客户端身份，根据用户身份进行请求的处理，典型的设计如 Tomcat 中的 Session。

- 例如传统的通过 session 来记录用户认证信息的方式：用户登录后，把用户信息保存在服务端 session 中，给用户一个 cookie 值，记录对应的 session，然后下次请求，用户携带 cookie 值来（这一步有浏览器自动完成），我们就能识别到对应 session，从而找到用户的信息。

无状态性，即：

- 服务端不保存任何客户端请求者信息；
- 客户端的每次请求必须具备自描述信息，通过这些信息识别客户端身份。

如何实现无状态？**无状态登录**的流程：

1. 首先客户端发送账户名/密码到服务端进行认证；
2. 认证通过后，服务端将用户信息加密并且编码成一个 token，返回给客户端；
3. 以后客户端每次发送请求，都需要携带认证的 token；
4. 服务端对客户端发送来的 token 进行解密，判断是否有效，并且获取用户登录信息。

#### 记住我的身份验证

##### 基于哈希简单令牌法

##### 持久化令牌法

##### Remember-Me 的接口和实现

#### JWT

**JWT** (`JSON WEB TOKEN)`：是一种可安全传输的的 JSON 对象。由于使用了数字签名，所以是可信任和安全的。

- 是目前最流行的**跨域认证**解决方案，是一种基于 Token 的认证授权机制。
- 从 JWT 的全称可以看出，JWT 本身也是 Token，一种**规范化**之后的 **JSON 结构的 Token**。

##### JWT 的优势

相比于 Session 认证的方式来说，使用 JWT 进行身份认证主要有下面 4 个优势。

1. 无状态：JWT 自身包含了身份验证所需要的所有信息，因此，我们的服务器不需要存储 JWT **Session** 信息。这显然增加了系统的可用性和伸缩性，大大减轻了服务端的压力。
    - 也导致了最大的缺点：**不可控！**就比如说，我们想要在 JWT 有效期内废弃一个 JWT 或者更改它的权限的话，并不会立即生效，通常需要等到有效期过后才可以。再比如说，当用户 Logout 的话，JWT 也还有效。除非，我们在后端增加额外的处理逻辑比如将失效的 JWT 存储起来，后端先验证 JWT 是否有效再进行处理。具体的解决办法，我们会在后面的内容中详细介绍到，这里只是简单提一下。
2. 有效**避免 CSRF 攻击**：因为 JWT 一般是存在在 **localStorage** 中，使用 JWT 进行身份验证的过程中是不会涉及到 Cookie 的。
    1. CSRF 攻击需要依赖 Cookie ，Session 认证中 Cookie 中的 `SessionID` 是由浏览器发送到服务端的，只要发出请求，Cookie 就会被携带。借助这个特性，即使黑客无法获取你的 `SessionID`，只要让你误点攻击链接，就可以达到攻击效果。
    2. 使用 JWT 进行身份验证不需要依赖 Cookie ，因此可以避免 CSRF 攻击。一般情况下我们使用 JWT 的话，在我们登录成功获得 JWT 之后，一般会选择存放在 localStorage 中。前端的每一个请求后续都会附带上这个 JWT，整个过程压根不会涉及到 Cookie。因此，即使你点击了非法链接发送了请求到服务端，这个非法请求也是不会携带 JWT 的，所以这个请求将是非法的。
3. 适合移动端应用：使用 Session 进行身份认证的话，需要保存一份信息在服务器端，而且这种方式会依赖到 Cookie（需要 Cookie 保存 `SessionId`），所以不适合移动端。但是，使用 JWT 进行身份认证就不会存在这种问题，因为只要 JWT 可以被客户端存储就能够使用，而且 JWT 还可以跨语言使用。
4. 单点登录友好：使用 Session 进行身份认证的话，实现单点登录，需要我们把用户的 Session 信息保存在一台电脑上，并且还会遇到常见的 Cookie 跨域的问题。但是，使用 JWT 进行认证的话， JWT 被保存在客户端，不会存在这些问题。

##### JWT的组成

JWT 本质上就是一组字串，通过（`.`）切分成三个为 Base64Url 编码的部分：`header.payload.signature`。

1. **Header（头部）** : 描述 JWT 的元数据，定义了**生成签名的算法**以及 `Token` 令牌的类型。
2. **Payload（载荷）** : 用来存放实际需要传递的数据，包含声明（Claims），如`sub`（主题）、`jti`（JWT ID）、存放用户名 sub、`token`的生成时间、过期时间。
3. **Signature（签名）**：服务器通过 Payload、Header 和一个密钥（Secret）使用 Header 里面指定的签名算法（默认是 HMAC SHA256）生成。作用是防止 JWT（主要是 payload） 被篡改，一旦`header`和`payload`被篡改，验证将失败。

![JWT 组成](assets/jwt-composition.png)

示例：

```java
{"alg": "HS512"}
{"sub":"admin","created":1489079981393,"exp":1489684781}
//JWT计算公式，secret为加密算法的密钥
String signature = HMACSHA512(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret);

//生成一个JWT实例的字符串
eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJhZG1pbiIsImNyZWF0ZWQiOjE1NTY3NzkxMjUzMDksImV4cCI6MTU1NzM4MzkyNX0.d-iki0193X0bBOETf2UN3r3PotNIEAV7mzIxxeI5IxFyzzkOZxS0PGfF_SK6wxCv2K8S0cZjMkv6b5bCqc0VBw
```

##### JWT 实现认证和授权的原理

1. 用户向服务器发送用户名、密码以及验证码用于登陆接口；
2. 如果用户用户名、密码以及验证码**校验**正确的话，登录成功后，服务端生成已经签名的 Token，即 **JWT**，并返回给客户端；
3. 客户端收到 Token 后自己保存起来（比如浏览器的 `localStorage`，放在 Cookie 中会有 CSRF 风险 ）；
4. 之后客户端发出的所有请求都会在 HTTP 请求头 `Header` 中携带这个 **JWT 令牌**：添加一个 `Authorization` 字段，值为 `JWT` 的 `token`，（`Authorization: Bearer Token`）；
5. 服务端检查 JWT 并从中获取用户相关信息。后台程序通过对 `Authorization` 头中信息解码及**数字签名校验**（再次生成一个 Signature 并对比）来获取其中的用户信息，从而实现认证和授权。

<img src="assets/jwt-1.jpeg" alt="../_images/jwt-1.jpeg" style="zoom: 80%;" />

##### 如何防止 JWT 被篡改？

有了签名之后，即使 JWT 被泄露或者截获，黑客也没办法同时篡改 Signature、Header、Payload。

这是为什么呢？

因为服务端拿到 JWT 之后，会解析出其中包含的 Header、Payload 以及 Signature 。服务端会根据 Header、Payload、密钥**再次生成一个 Signature**。拿新生成的 Signature 和 JWT 中的 Signature 作对比，如果一样就说明 Header 和 Payload 没有被修改。

不过，如果服务端的秘钥也被泄露的话，黑客就可以同时篡改 Signature、Header、Payload 了。黑客直接修改了 Header 和 Payload 之后，再重新生成一个 Signature 就可以了。

JWT 安全的核心在于签名，签名安全的**核心在密钥**。

##### 如何加强 JWT 的安全性？

1. 使用安全系数高的加密算法。
2. 使用成熟的开源库，没必要造轮子。
3. JWT 存放在 localStorage 中而不是 Cookie 中，避免 CSRF 风险。
4. 一定不要将隐私信息存放在 Payload 当中。
5. 密钥一定保管好，一定不要泄露出去。JWT 安全的核心在于签名，签名安全的核心在密钥。
6. Payload 要加入 `exp` （JWT 的过期时间），永久有效的 JWT 不合理。并且，JWT 的过期时间不宜过长。

##### JWT 身份认证常见问题及解决办法

1. 注销登录等场景下 JWT 还有效

2. JWT 的续签问题

3. JWT 体积太大

### 前后端分离的用户认证流程

##### 1. 用户登录

- **前端**：
    - 登录表单：用户在登录界面输入用户名和密码。
    - 前端将这些凭证以 JSON 格式发送到后端的登录 API（例如 `POST /api/login`）。
- **后端**：
    - Spring Security 接收请求，使用 `AuthenticationManager` 进行身份验证。
    - 如果认证成功，后端生成一个 `JWT` 或其他认证令牌，并将其返回给前端。

##### 2. 使用 JWT 进行用户认证

###### 2.1 前端存储 JWT

- 前端收到 JWT 后，可以将其存储在 `localStorage` 或 `sessionStorage` 中，以便后续请求使用。

###### 2.2 发送受保护请求

- 在发送需要认证的请求时，前端将 JWT 添加到请求头中：

```javascript
fetch('/api/protected-endpoint', {
    method: 'GET',
    headers: {
        'Authorization': `Bearer ${token}`
    }
});
```

###### 2.3 后端解析 JWT

- Spring Security 通过**过滤器**来解析和验证 JWT。可以自定义一个 `JwtAuthenticationTokenFilter`类（继承并实现 `OncePerRequestFilter`）以拦截请求，提取 JWT，并验证其有效性。
    - `OncePerRequestFilter`：是实现自定义过滤器的基础，通常用于对请求进行**预处理**或后处理。通过继承该类，可以轻松实现**自定义过滤器**适合用于记录日志、身份验证、权限检查等场景。
    - 是 Spring Security 提供的一个抽象类，确保在每个请求中只执行一次特定的过滤逻辑。

##### 3. 退出登录

由于 JWT 是**无状态**的，后端不需要记录会话状态。用户可以通过前端操作（例如，删除存储的 JWT）来退出登录。可以实现一个**注销接口**，用于前端执行相关逻辑。

##### 4. 保护敏感信息

- 确保 HTTPS：在前后端通信中使用 HTTPS，确保传输中的数据安全。
- 令牌过期：设置 JWT 的有效期，过期后需要用户重新登录。
- 刷新令牌：可以实现刷新令牌的机制，以提高用户体验。

### 授权

#### 访问控制

目前业界主流的**权限模型**有两种：

- 基于**角色**的访问控制（RBAC）
- 基于**属性**的访问控制（ABAC）

##### RBAC 基于角色的权限访问控制

基于角色的权限访问控制（`Role-Based Access Control`，**RBAC**）：指的是通过用户的**角色**授权其相关权限，实现了灵活的访问控制，相比直接授予用户权限，要更加简单、高效、可扩展。

- 这是一种通过角色关联权限，角色同时又关联用户的授权的方式。

- 简单地说：一个用户可以**拥有**若干角色，每一个角色又可以被分配若干**权限、菜单**，这样就构造成“**用户-角色-权限**” 的授权模型。用户与角色、角色与权限之间构成了**多对多**的关系。

<img src="assets/rbac.png" alt="" style="zoom:100%;" />

在 RBAC 权限模型中，权限与角色相关联，用户通过成为包含特定角色的成员而得到这些角色的权限，这就极大地**简化了权限的管理**。

为了实现 RBAC 权限模型，数据库表的常见设计如下（一共 5 张表，2 张建立表之间的联系）：

1. 用户详情表
2. 用户角色联系表
3. 角色详情表
4. 角色权限联系表
5. 权限详情表
6. 角色菜单联系表
7. 菜单详情表

通过这个权限模型，可以创建不同的角色、并为不同的角色分配不同的**权限范围（菜单）**。

<img src="assets/数据库设计-权限.png" alt="" style="zoom:100%;" />



![](assets/books权限管理模块.png)



<img src="Project/assets/rbac02.png" alt="权限模型" style="zoom:50%;" />

##### ABAC 基于属性的权限访问控制

基于属性的访问控制（`Attribute-Based Access Control`，**ABAC**）：原理是通过各种属性来**动态判断**一个操作是否可以被允许。

- 是一种比 `RBAC模型` 更加灵活的授权模型。在云系统中使用的比较多，比如 AWS，阿里云等。

考虑下面这些场景的权限控制：

1. 授权某个人具体某本书的编辑权限
2. 当一个文档的所属部门跟用户的部门相同时，用户可以访问这个文档
3. 当用户是一个文档的拥有者并且文档的状态是草稿，用户可以编辑这个文档
4. 早上九点前禁止 A 部门的人访问 B 系统
5. 在除了上海以外的地方禁止以管理员身份访问 A 系统
6. 用户对 2022-06-07 之前创建的订单有操作权限

可以发现上述的场景通过 `RBAC模型` 很难去实现，因为 `RBAC模型` 仅仅描述了用户可以做什么操作，但是操作的条件，以及操作的数据，`RBAC模型` 本身是没有这些限制的。

但这恰恰是 `ABAC模型` 的长处，`ABAC模型` 的思想是基于用户、访问的数据的属性、以及各种环境因素去动态计算用户是否有权限进行操作。

ABAC 模型的**原理**：

在 `ABAC模型` 中，一个操作是否被允许是基于对象、资源、操作和环境信息共同动态计算决定的。

- **对象**：对象是当前请求访问资源的用户。用户的属性包括 ID，个人资源，角色，部门和组织成员身份等
- **资源**：资源是当前用户要访问的资产或对象，例如文件，数据，服务器，甚至 API
- **操作**：操作是用户试图对资源进行的操作。常见的操作包括“读取”，“写入”，“编辑”，“复制”和“删除”
- **环境**：环境是每个访问请求的上下文。环境属性包含访问的时间和位置，对象的设备，通信协议和加密强度等

在 `ABAC模型` 的决策语句的执行过程中，决策引擎会根据定义好的决策语句，结合对象、资源、操作、环境等因素动态计算出决策结果。每当发生访问请求时，`ABAC模型` 决策系统都会分析属性值是否与已建立的策略匹配。如果有匹配的策略，访问请求就会被通过。

#### 控制请求访问权限的方法

- `permitAll()`：无条件允许任何形式访问，不管登录还是没有登录。
- `au'thenticated()`：只允许已认证的用户访问。
    - `fullyAuthenticated()`：只允许已经登录或者通过 remember-me 登录的用户访问。
- `anonymous()`：允许匿名访问，也就是没有登录才可以访问。
- `denyAll()`：无条件决绝任何形式的访问。
- `hasRole(String)` : 只允许指定的角色访问。
    - `hasAnyRole(String)` : 指定一个或者多个角色，满足其一的用户即可访问。
- `hasAuthority(String)`：只允许具有指定权限的用户访问。
    - `hasAnyAuthority(String)`：指定一个或者多个权限，满足其一的用户即可访问。
- `hasIpAddress(String)` : 只允许指定 ip 的用户访问。

#### 授权服务器

#### 授权 HTTP 请求

### SSO 单点登录

单点登录（SSO，`Single Sign On`）：用户登陆多个**子系统**的其中一个就有权访问与其相关的其他系统。让用户通过**一次性用户身份验证**登录多个应用程序和网站。一旦验证身份，用户就可以访问所有受密码保护的资源，而无需**重复登录**。

好处：

- 用户角度：用户能够做到一次登录多次使用，无需记录多套用户名和密码，省心。
- 系统管理员角度：管理员只需维护好一个统一的账号中心就可以了，方便。
- 新系统开发角度：新系统开发时只需直接对接统一的账号中心即可，简化开发流程，省时。

##### 工作原理

SSO 流程如下：

1. 当用户登录应用程序时，应用程序会**生成 SSO 令牌**并向 SSO 服务发送身份验证请求。 
2. 该服务会检查用户之前是否在系统中进行了身份验证。如果是，它会向应用程序发送一个身份验证确认响应，以授予用户访问权限。 
3. 如果用户没有经过验证的凭证，SSO 服务会将用户**重定向**到中央登录系统并提示用户提交其用户名和密码。
4. 提交后，服务会验证用户凭证并将肯定响应发送到应用程序。 
5. 否则，用户会收到错误消息并且必须重新输入凭证。多次尝试登录失败可能会导致服务阻止用户在固定的时间段内进行更多尝试。 

##### 用户登录/登录校验

##### 跨域登录、登出

##### 类型

SSO 解决方案使用不同的标准和协议来对用户凭证进行验证和身份验证。

1. **SAML**（安全断言标记语言）：是应用程序用来与 SSO 服务**交换身份验证信息**的协议或规则集。使用 XML 来交换用户标识数据。基于 SAML 的 SSO 服务提供更好的安全性和灵活性，因为应用程序不需要在其系统上存储用户凭证。
2. **OAuth**（开放授权）：是一种开放标准，允许应用程序安全地**从其他网站**获取用户信息，而无需提供密码。应用程序不是请求用户密码，而是使用 OAuth 来获得用户访问受密码保护的数据的**权限**。OAuth 通过 API 建立应用程序之间的信任，允许应用程序在已建立的框架中发送和响应身份验证请求。
3. **OIDC、OpenID**：是使用一组用户凭证访问多个站点的方法。允许**服务提供商**承担验证用户凭证的角色。Web 应用程序不是将身份验证令牌传递给第三方身份提供商，而是使用 OIDC 来请求附加信息并验证用户的真实性。
4. **Kerberos**：是一种基于**票证**的身份验证系统，可让两方或多方在网络上相互验证其身份。使用**安全密码学**来防止未经授权访问在服务器、客户端和密钥分发中心之间传输的标识信息。

### SAML2



### OAuth2

**开放授权**（`OAuth`）：是一个行业的标准授权协议，主要用来**授权第三方应用**获取有限的权限。

- 最终目的是：为第三方应用颁发一个**有时效性**的令牌 Token，使得第三方应用能够（通过该令牌）获取相关的资源，如该用户在某一网站上存储的私密资源（如照片）。
- OAuth 允许用户提供一个**令牌（access token）**，而不是**用户名和密码**来访问（存放在特定服务提供者的）数据。
- 而 OAuth 2.0 是对 OAuth 1.0 的完全重新设计，OAuth 2.0 更快，更容易实现，OAuth 1.0 已经被废弃。

OAuth 2.0 常用场景：

- 比较常用的场景就是第三方登录；
- 另外，也常见于支付场景（微信支付、支付宝支付）和开发平台（微信开放平台、阿里开放平台等）。

服务器：

1. `Resource Owner`：资源所有者，"用户"（user）。
2. `User Agent`：用户代理，**客户端**、浏览器。
3. `Authorization server`：**认证服务器**，服务提供商专门用来处理认证的服务器。
4. `Resource server`：**资源服务器**，服务提供商存放用户生成的资源的服务器。与认证服务器，可以是同一台服务器，也可以是不同的。

参考：

- [理解OAuth 2.0 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)
- [https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247488209&idx=2&sn=19b1e44fbb1f4c1210f0fa92a618d871&scene=21#wechat_redirect](做微服务绕不过的 OAuth2，松哥也来和大家扯一扯)，四种认证流程举例

#### 客户端的授权模式

客户端必须得到用户的授权（authorization grant），才能获得令牌（access token）。OAuth 2.0定义了四种授权方式。

1. **授权码模式**（authorization code）：通过客户端的后台服务器，与服务提供商的**认证服务器**进行互动。功能最完整、流程最严密。如常见的第三方平台登录功能。
2. **简化模式**（implicit）：不通过第三方应用程序的服务器（客户端服务器），直接在浏览器中向**认证服务器**申请令牌，跳过了"授权码"这个步骤。
    - 所有步骤在浏览器中完成，令牌对访问者是可见的，且客户端不需要认证。一般用于网站是纯静态页面。
3. **密码模式**（resource owner password credentials）：用户向客户端提供自己的用户名和密码。客户端使用这些信息，向**服务提供商**索要授权、**申请令牌（token）**。
4. **客户端模式**（client credentials）：指客户端以自己的名义，而不是以用户的名义，向"**服务提供商**"进行认证。
5. refresh_token：

<img src="assets/flow.png" alt="img" style="zoom: 67%;" />

#### 授权码模式的认证流程

（A）用户打开客户端以后，客户端要求用户给予授权。

（B）用户同意给予客户端授权。

（C）客户端使用上一步获得的授权，向认证服务器申请令牌。

（D）认证服务器对客户端进行认证以后，确认无误，同意发放令牌。

（E）客户端使用令牌，向资源服务器申请获取资源。

（F）资源服务器确认令牌无误，同意向客户端开放资源。

下图是 Slack OAuth 2.0 第三方登录的示意图：

<img src="assets/20210615151716340.png" style="zoom:80%;" />

### 加密机制

#### PasswordEncoder 接口

Spring Security 提供了多种**加密算法**的实现，开箱即用，非常方便。

这些加密算法实现类的接口是 `PasswordEncoder` ，一共就 3 个必须实现的方法，用于定义密码的加密和验证方法。

0. 配置 `PasswordEncoder`，在 Spring Security 的**配置类**中定义 `PasswordEncoder` bean；
    - 官方推荐基于 `BCryptPasswordEncoder()` 强哈希函数的加密算法实现类。
1. 在注册用户或更新密码时，可以通过 `encode()` 方法来**加密密码**；
2. 在用户登录时，可以使用 `matches()` 方法**验证**输入的密码与存储的加密密码是否匹配。
3. `upgradeEncoding()`

```
public interface PasswordEncoder {
    // 加密也就是对原始密码进行编码
    String encode(CharSequence var1);
    // 比对原始密码和数据库中保存的密码
    boolean matches(CharSequence var1, String var2);
    // 判断加密密码是否需要再次进行加密，默认返回 false
    default boolean upgradeEncoding(String encodedPassword) {
        return false;
    }
}
```

#### 加密算法

加密算法：通常指的是可以将明文转换为密文，并且能够通过某种方式（如密钥）再将密文还原为明文的算法。

- 是一种用数学方法对数据进行变换的技术，目的是保护数据的安全，防止被未经授权的人读取或修改。

日常开发中常见的场景：

1. 密码。
2. 数据库中的敏感数据（比如银行卡号、身份号）需要使用对称加密算法（比如 AES）保存。
3. **网络传输**的敏感数据（比如银行卡号、身份号）需要用 HTTPS + 非对称加密算法（如 RSA）来保证传输数据的安全性。

加密算法可以分为三大类：

1. **哈希算法**：将任意长度的数据（输入信息）转换成一个固定长度的、看似随机的唯一标识（哈希值）。可以用来验证数据的**完整性和一致性**，常见的哈希算法有 `MD、SHA、MAC` 等。
    - 是一种单向过程，这个过程是**不可逆**的，即，不能从哈希值还原出原始信息。
2. **对称加密算法**：是一种加密和解密使用**同一个密钥**的算法，可以用来保护数据的安全性和保密性，常见的对称加密算法有 `DES、3DES、AES` 等。
3. **非对称加密算法**：是一种加密和解密使用**不同的密钥**的算法，可以用来实现数据的安全传输和身份认证，常见的非对称加密算法有 `RSA、DSA、ECC` 等。

#### 哈希算法

哈希算法（散列函数、摘要算法）：作用是对任意长度的数据生成一个固定长度的唯一标识，也叫哈希值、散列值、消息摘要。

![哈希算法效果演示](assets/hash-function-effect-demonstration.png)

哈希算法的是**不可逆**的，无法通过哈希之后的值再得到原值。

哈希值的作用是可以用来验证数据的**完整性和一致性**。

举两个实际的例子：

- 保存密码到数据库时使用哈希算法进行加密，可以通过比较用户输入密码的哈希值和数据库保存的哈希值是否一致，来判断密码是否正确。
- 我们下载一个文件时，可以通过比较文件的哈希值和官方提供的哈希值是否一致，来判断文件是否被篡改或损坏；

哈希算法可以简单分为两类：

1. **加密哈希算法**：安全性相对较高，可以提供一定的数据完整性保护和数据防篡改能力，能够抵御一定的攻击手段，但性能较差，适用于对安全性要求较高的场景。例如 `SHA2、SHA3、SM3、RIPEMD-160、BLAKE2、SipHash` 等等。
2. **非加密哈希算法**：安全性相对较低，易受到暴力破解、冲突攻击等攻击手段的影响，但性能较高，适用于对安全性没有要求的业务场景。例如 `CRC32、MurMurHash3、SipHash` 等。
3. 除了这两种之外，还有一些特殊的哈希算法，例如安全性更高的**慢哈希算法**。

常见的哈希算法有：

1. ~~**MD**~~（Message Digest，消息摘要算法）：MD2、MD4、MD5 等，已经不被推荐使用。
2. **SHA**（Secure Hash Algorithm，安全哈希算法）：SHA-1 系列安全性低，SHA2，SHA3 系列安全性较高。
3. **Bcrypt**（密码哈希算法）：基于 Blowfish 加密算法的密码哈希算法，专门为密码加密而设计，安全性高，属于**慢哈希算法**。
4. 需要**密钥**：哈希算法一般是不需要密钥的，但也存在部分特殊哈希算法需要密钥。例如，MAC 和 SipHash ，在哈希算法的基础上增加了一个密钥，使得只有知道密钥的人才能验证数据的完整性和来源。
    1. **MAC**（Message Authentication Code，消息认证码算法）：HMAC 是一种基于哈希的 MAC，可以与任何安全的哈希算法结合使用，例如 SHA-256。
    2. **SipHash**：加密哈希算法，设计目的是在速度和安全性之间达到一个平衡，用于防御**哈希泛洪 DoS 攻击**。Rust 默认使用 SipHash 作为哈希算法，从 Redis4.0 开始，哈希算法被替换为 SipHash。
5. **国密算法**：例如 SM2、SM3、SM4，其中 SM2 为非对称加密算法，SM3 为哈希算法（安全性及效率和 SHA-256 相当，但更适合国内的应用环境），SM4 为对称加密算法。
6. **CRC**（Cyclic Redundancy Check，循环冗余校验）：CRC32 是一种 CRC 算法，特点是生成 32 位的校验值，通常用于数据完整性校验、文件校验等场景。
7. **MurMurHash**：经典快速的非加密哈希算法，目前最新的版本是 MurMurHash3，可以生成 32 位或者 128 位哈希值。

##### Bcrypt

Java 应用程序的安全框架 Spring Security 支持多种密码编码器，其中 `BCryptPasswordEncoder` 是官方推荐的一种，它使用 BCrypt 算法对用户的密码进行加密存储。

```java
@Bean
public PasswordEncoder passwordEncoder(){
    return new BCryptPasswordEncoder();
}
```

#### 对称加密

对称加密算法：是指加密和解密使用**同一个密钥**的算法，也叫**共享密钥**加密算法。

![对称加密](assets/symmetric-encryption.png)

常见的对称加密算法有 DES、3DES、AES 等。

##### DES 和 3DES

DES（Data Encryption Standard）使用 64 位的密钥（有效秘钥长度为 56 位,8 位奇偶校验位）和 64 位的明文进行加密。

这是一个经典的对称加密算法，但也有明显的缺陷，即 56 位的密钥安全性不足，已被证实可以在短时间内破解。

为了提高 DES 算法的安全性，人们提出了一些变种或者替代方案，例如 3DES（Triple DES）。

3DES（Triple DES）是 DES 向 AES 过渡的加密算法，它使用 2 个或者 3 个 56 位的密钥对数据进行**三次加密**。3DES 相当于是对每个数据块应用三次 DES 的对称加密算法。

##### AES

AES（Advanced Encryption Standard）算法：是一种更先进的对称密钥加密算法，它使用 128 位、192 位或 256 位的密钥对数据进行加密或解密，密钥越长，安全性越高。

#### 非对称加密

非对称加密算法（公开密钥加密算法）：是指加密和解密使用**不同的密钥**的算法。

- 这两个密钥互不相同，一个称为公钥，另一个称为私钥。公钥可以公开给任何人使用，私钥则要保密。
- 如果用公钥加密数据，只能用对应的私钥解密（加密）；
- 如果用私钥加密数据，只能用对应的公钥解密（签名）。这样就可以实现数据的安全传输和身份认证。

![非对称加密](assets/asymmetric-encryption.png)

常见的非对称加密算法有 RSA、DSA、ECC 等。

##### RSA

RSA（Rivest–Shamir–Adleman algorithm）算法：是一种**基于大数分解**的困难性的非对称加密算法，它需要选择两个**大素数**作为私钥的一部分，然后计算出它们的乘积作为公钥的一部分（寻求两个大素数比较简单，而将它们的乘积进行因式分解却极其困难）。

RSA 算法的优点是简单易用，可以用于数据加密和数字签名；缺点是运算速度慢，不适合大量数据的加密。

RSA 算法是是目前应用最广泛的非对称加密算法，像 **SSL/TLS、SSH** 等协议中就用到了 RSA 算法。

##### DSA

DSA（Digital Signature Algorithm）算法：是一种**基于离散对数**的困难性的非对称加密算法。

- 它需要选择一个素数 q 和一个 q 的倍数 p 作为私钥的一部分，然后计算出一个模 p 的原根 g 和一个模 q 的整数 y 作为公钥的一部分。

- DSA 算法的安全性依赖于离散对数的难度，目前已经有 1024 位的 DSA 公钥被成功破解，因此建议使用 2048 位或以上的密钥长度。

- DSA 算法的优点是数字签名速度快，适合生成数字证书；缺点是不能用于数据加密，且签名过程需要随机数。

### 漏洞防护/安全

#### CSRF

##### 概念简介

跨站请求伪造（`Cross-site request forgery`，**CSRF**）：通过（Cookie）**伪造用户请求**去访问**曾经认证过**的网站，让用户**误点击非法链接**、欺骗用户浏览器，让其以用户的名义执行操作。

- 由于浏览器曾经认证过，登录信息**尚未过期**，所以被访问的网站会认为是真正的用户操作而去执行。（如发邮件，发消息，甚至财产操作如转账和购买商品）。

- 也被称为“One Click Attack” 或者Session Riding。

- 跨域：只要网络协议、域名、端口中任何一个不相同就是跨域请求。

XSS 对比 CSRF：

- XSS 利用的是用户对指定网站的信任，**受信任站点**
- CSRF 利用的是网站对用户浏览器的信任。是欺骗用户浏览器，让其以用户的名义执行操作。

##### 攻击原理

过程：

1. 客户端与服务端进行交互时，由于http协议本身是无状态协议，所以引入了cookie进行记录客户端身份。
2. 在cookie中会存放session id用来**识别客户端身份**的。
3. 在**跨域**的情况下，session id可能被**第三方恶意劫持**，通过这个session id向服务端发起请求时，服务端会认为这个请求是合法的。

假如一家银行用以执行转账操作的 URL 地址如下：

```
http://www.examplebank.com/withdraw?account=AccoutName&amount=1000&for=PayeeName
```

那么，恶意攻击者可以在**另一个网站**上放置如下代码：

```
<img src="http://www.examplebank.com/withdraw?account=Alice&amount=1000&for=Badman">
<a src=http://www.mybank.com/Transfer?bankId=11&money=10000>科学理财，年盈利率过万</>
```

##### 防范手段

###### 1. 检查 Referer 首部字段

Referer 首部字段位于 **HTTP 报文**中，用于标识**请求来源**的地址。检查这个首部字段并要求请求来源的地址在**同一个域名**下，可以极大的防止 CSRF 攻击。

局限性：因其完全依赖浏览器发送**正确的 Referer 字段**。虽然 HTTP 协议对此字段的内容有明确的规定，但并无法保证来访的浏览器的具体实现，亦无法保证浏览器没有安全漏洞影响到此字段。并且也存在攻击者攻击某些浏览器，**篡改**其 Referer 字段的可能。

###### 2. 添加校验 Token

在访问敏感数据请求时，要求用户浏览器提供不保存在 Cookie 中、并且攻击者无法伪造的数据作为校验。例如服务器生成**随机数**并附加在表单中，并要求客户端**传回**这个随机数。

###### 3. 输入验证码

因为 CSRF 攻击是在用户无意识的情况下发生的，所以要求用户输入验证码可以让用户知道自己正在做的操作。

#### HTTP 安全响应头



#### HTTP 防火墙



#### 敏感词过滤

系统需要对用户输入的文本进行敏感词过滤如色情、政治、暴力相关的词汇。

敏感词过滤用的使用比较多的 **Trie 树算法** 和 **DFA 算法**。

##### Trie 树

**Trie 树** 也称为字典树、单词查找树，哈希树的一种变种，通常被用于字符串匹配，用来解决在一组字符串集合中快速查找某个字符串的问题。像浏览器搜索的关键词提示就可以基于 Trie 树来做的。

 Trie 树的核心原理其实很简单，就是通过公共前缀来提高字符串匹配效率。

##### AC 自动机

Aho-Corasick（AC）自动机是一种建立在 Trie 树上的一种改进算法，是一种多模式匹配算法，由贝尔实验室的研究人员 Alfred V. Aho 和 Margaret J.Corasick 发明。

AC 自动机算法使用 Trie 树来存放模式串的前缀，通过失败匹配指针（失配指针）来处理匹配失败的跳转。关于 AC 自动机的详细介绍，可以查看这篇文章：[地铁十分钟 | AC 自动机](https://zhuanlan.zhihu.com/p/146369212)。

如果使用上面提到的 DAT 来表示 AC 自动机 ，就可以兼顾两者的优点，得到一种高效的多模式匹配算法。

##### DFA

**DFA**（Deterministic Finite Automata）即确定有穷自动机，与之对应的是 NFA（Non-Deterministic Finite Automata，不确定有穷自动机）。

Hutool  提供了 DFA 算法的实现：

![Hutool 的 DFA 算法](assets/hutool-dfa.png)

```java
WordTree wordTree = new WordTree();
wordTree.addWord("大");
wordTree.addWord("大憨憨");
wordTree.addWord("憨憨");
String text = "那人真是个大憨憨！";
// 获得第一个匹配的关键字
String matchStr = wordTree.match(text);
System.out.println(matchStr);
// 标准匹配，匹配到最短关键词，并跳过已经匹配的关键词
List<String> matchStrList = wordTree.matchAll(text, -1, false, false);
System.out.println(matchStrList);
//匹配到最长关键词，跳过已经匹配的关键词
List<String> matchStrList2 = wordTree.matchAll(text, -1, false, true);
System.out.println(matchStrList2);
```

#### 数据脱敏

数据脱敏：指对某些敏感信息通过**脱敏规则**进行数据的变形，实现**敏感隐私数据**的可靠保护。

- 例如，对于身份证号码，可以使用**掩码算法**（masking）将前几位数字保留，其他位用 “X” 或 "\*" 代替；
- 对于姓名，可以使用**伪造算法**（pseudonymization），将真实姓名替换成随机生成的假名。

##### 常用脱敏规则

常用脱敏规则是为了保护敏感数据的安全性，在处理和存储敏感数据时对其进行变换或修改。

下面是几种常见的脱敏规则：

- 替换(常用)：将敏感数据中的特定字符或字符序列替换为其他字符。例如，将信用卡号中的中间几位数字替换为星号（\*）或其他字符。
- 删除：将敏感数据中的部分内容随机删除。比如，将电话号码的随机 3 位数字进行删除。
- 重排：将原始数据中的某些字符或字段的顺序打乱。例如，将身份证号码的随机位交错互换。
- 加噪：在数据中注入一些误差或者噪音，达到对数据脱敏的效果。例如，在敏感数据中添加一些随机生成的字符。
- 加密（常用）：使用加密算法将敏感数据转换为密文。例如，将银行卡号用 MD5 或 SHA-256 等哈希函数进行散列。常见加密算法总结可以参考这篇文章：<https://javaguide.cn/system-design/security/encryption-algorithms.html> 。

##### 常用脱敏工具

1. Hutool 一个 Java 基础工具类，对文件、流、加密解密、转码、正则、线程、XML 等 JDK 方法进行封装，组成各种 Util 工具类，同时提供以下组件。Hutool 脱敏是通过 \* 来代替敏感信息的，具体实现是在 StrUtil.hide 方法中。
2. Apache ShardingSphere：是一套开源的分布式数据库中间件解决方案组成的生态圈，它由 Sharding-JDBC、Sharding-Proxy 和 Sharding-Sidecar（计划中）这 3 款相互独立的产品组成。 他们均提供标准化的数据分片、分布式事务和数据库治理功能 。
    1. 下面存在一个数据脱敏模块，此模块集成的常用的数据脱敏的功能。其基本原理是对用户输入的 SQL 进行解析拦截，并依靠用户的脱敏配置进行 SQL 的改写，从而实现对原文字段的加密及加密字段的解密。最终实现对用户无感的加解密存储、查询。
    2. 通过 Apache ShardingSphere 可以自动化&透明化数据脱敏过程，用户无需关注脱敏中间实现细节。并且，提供了多种内置、第三方(AKS)的脱敏策略，用户仅需简单配置即可使用。
3. FastJSON：是一个很常用的 Spring Web Restful 接口**序列化**的工具。实现数据脱敏的方式主要有两种：
    - 基于注解 `@JSONField` 实现：需要自定义一个用于脱敏的序列化的类，然后在需要脱敏的字段上通过 `@JSONField` 中的 `serializeUsing` 指定为我们自定义的序列化类型即可。
    - 基于序列化过滤器：需要实现 `ValueFilter` 接口，重写 `process` 方法完成自定义脱敏，然后在 JSON 转换时使用自定义的转换策略。
4. Mybatis-Mate：是为 MyBatis-Plus 提供的企业级模块，旨在更敏捷优雅处理数据。不过，使用之前需要配置授权码（付费）。Mybatis-Mate 支持敏感词脱敏，内置手机号、邮箱、银行卡号等 9 种常用脱敏规则。
5. MyBatis-Flex：类似于 MybatisPlus，MyBatis-Flex 也是一个 MyBatis 增强框架。MyBatis-Flex 同样提供了数据脱敏功能，并且是可以免费使用的。MyBatis-Flex 提供了 `@ColumnMask()` 注解，以及内置的 9 种脱敏规则，开箱即用：