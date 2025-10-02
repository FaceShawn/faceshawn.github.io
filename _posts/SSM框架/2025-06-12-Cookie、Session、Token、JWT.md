---
title: Cookie、Session、Token、JWT
categories:
  - SSM框架
tags:
  - Spring-Security
  - 认证
  - Cookie
  - Session
  - Token
  - JWT
location:
  - 黄金时代
abbrlink: 'token_jwt'
permalink: 'token_jwt'
date: 2025-06-12 13:42:12
updated: 2025-06-12 13:42:12
---

> 摘要：Spring Security 认证方式，Cookie、Session、Token、JWT。

<!-- more -->

## 目录

[TOC]



### 无状态登录

**有状态服务**：即服务端需要**记录每次会话的客户端信息**，从而识别客户端身份，根据用户身份进行请求的处理。

- 典型的设计如 Tomcat 中通过 Session 来**记录用户认证信息**的方式：
    1. 用户登录后，把用户信息保存在服务端 session 中，给用户一个 cookie 值，记录对应的 session，
    2. 然后下次请求，用户携带 cookie 值来（这一步有浏览器自动完成），服务器就能识别到对应 session，从而找到用户的信息。

**无状态性**，即：

- 服务端不保存任何客户端请求者信息；
- 客户端的每次请求必须具备**自描述信息**，通过这些信息识别客户端身份。

HTTP 是**无状态**的协议（对于事务处理**没有记忆能力**，每次客户端和服务端会话完成时，服务端不会保存任何会话信息）：每个请求都是完全独立的，服务端无法确认当前访问者的身份信息，无法分辨上一次的请求发送者和这一次的发送者是不是同一个人。

- 所以服务器与浏览器为了进行**会话跟踪**（知道是谁在访问我），就必须主动的去维护一个状态，这个状态用于告知服务端前后两个请求是否来自同一浏览器。
- 而这个状态需要通过 cookie 或者 session 去实现。

如何实现无状态？**无状态登录**的流程：

1. 首先客户端发送账户名/密码到服务端进行认证；
2. 认证通过后，服务端将用户信息加密并且编码成一个 **token**，返回给客户端；
3. 以后客户端每次发送请求，都需要携带认证的 token；
4. 服务端对客户端发送来的 token 进行解密，判断是否有效，并且获取用户登录信息。

### Cookie 和 Session 异同

相同：都是用来跟踪浏览器**用户身份**的**会话方式**，但是两者的应用场景不太一样。

区别：

1. 定义：
    - `Cookie`：网站（为了辨别用户身份而）储存在**用户本地客户端**上的数据，用来存储**少量**用户信息（通常经过加密）。
    - `Session`：**服务器**用来存储用户信息、通过服务端记录用户的状态。
2. **安全性**：`Cookie`保存在**客户端**，用户能很容易的获取，**安全性**不高；`Session`保存在服务器，用户不易获取，**安全性更高**。
3. **数据量**、存储大小不同：`Cookie`存储的**数据量小**，（单个 Cookie 保存的数据不能超过 4K）。`Session`储存的数据量远高于 Cookie，但占用**服务器资源**。
4. **存取值的类型不同**：Cookie 只支持存**字符串**数据，想要设置其他类型的数据，需要将其转换成字符串，Session 可以存**任意数据类型**。
5. **有效期不同：** Cookie 可设置为**长时间保持**，比如经常使用的**默认登录**功能。Session 一般失效时间较短，**客户端关闭**（默认情况下）或者 **Session 超时**都会失效。
6. 典型场景、应用案例：
    - `Cookie`应用案例：
        1. 保存**已经登录过的**用户信息，
        2. 保存用户首选项、主题和其他设置信息，
        3. 保存 `SessionId` 或 `Token` ，用于**帮助记录**用户当前的**状态**（因为 HTTP 协议是无状态的），
        4. 用来记录和分析用户行为，比如网上购物时，服务器获取某个页面的**停留状态**、或看了哪些商品，一种常用的实现方式就是将这些信息存放在 `Cookie`。
    - `Session`典型场景：
        1. **购物车**：系统不知道是哪个用户操作的，因为 HTTP 协议是**无状态**的。服务端给特定的用户创建特**定的 `Session`** 之后就可以**标识**这个用户并且**跟踪**这个用户了。
        2. 个性化推送、**免密登录**。如 `HttpSession`。

## Cookie

Cookie：指的就是浏览器里面能**永久存储**的一种数据，仅仅是浏览器实现的一种数据存储功能。

- **由服务器生成**，发送给浏览器，并（以kv形式）保存在浏览器本地（某个文件内）。
    - 在下次向**同一服务器**再发起请求时，被携带并发送到服务器上。
- cookie 是**不可跨域**的： 每个 cookie 都会**绑定单一的域名**，无法在别的域名下获取使用，一级域名和二级域名之间是允许共享使用的（**靠的是 domain）**。

- 由于cookie是存在客户端上的，所以浏览器加入了一些限制确保cookie不会被恶意使用，同时不会占据太多磁盘空间，所以每个域的cookie数量是有限的。

### 保存方式

Cookie的保存方式有两种：

1. 如果没有设置cookie的失效时间，这个cookie就存在于**浏览器进程**；
2. 设置了cookie的失效时间，那么这个cookie就存在于**硬盘**。

### cookie 重要的属性

| 属性           | 说明                                                         |
| :------------- | :----------------------------------------------------------- |
| **name=value** | 键值对，设置 Cookie 的名称及相对应的值，都必须是**字符串类型** <br />- 如果值为 Unicode 字符，需要为字符编码。<br />- 如果值为二进制数据，则需要使用 BASE64 编码。 |
| **domain**     | 指定 cookie 所属域名，默认是当前域名                         |
| **path**       | 指定 cookie 在哪个**路径（路由）下生效**，默认是 '/'。 如果设置为 `/abc`，则只有 `/abc` 下的路由可以访问到该 cookie，如：`/abc/read`。 |
| **maxAge**     | cookie 失效的时间，单位秒。<br />- 如果为整数，则该 cookie 在 maxAge 秒后失效。<br />- 如果为负数，该 cookie 为临时 cookie ，关闭浏览器即失效，浏览器也不会以任何形式保存该 cookie 。<br />- 如果为 0，表示删除该 cookie 。默认为 -1。 - **比 expires 好用**。 |
| **expires**    | **过期时间**，在设置的某个时间点后该 cookie 就会失效。<br />一般浏览器的 cookie 都是默认储存的，当关闭浏览器结束这个会话的时候，这个 cookie 也就会被删除 |
| **secure**     | 该 cookie 是否仅被使用**安全协议传输**。安全协议有 HTTPS，SSL等，在网络上传输数据之前先将数据加密。<br />默认为false。 当 secure 值为 true 时，cookie 在 HTTP 中是无效，在 HTTPS 中才有效。 |
| **httpOnly**   | 如果给某个 cookie 设置了 httpOnly 属性，则无法**通过 JS 脚本 读取**到该 cookie 的信息，但还是能通过 Application 中手动修改 cookie，所以只是在一定程度上可以**防止 XSS 攻击**，不是绝对的安全。 |

### 在项目中使用

> 以 Spring Boot 项目为例。

1) 创建 `Cookie`，并通过 **`HttpServletResponse`** 返回给客户端；

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

2) 使用 Spring 框架提供的 **`@CookieValue` 注解**获取特定的 cookie 的值；

```java
@GetMapping("/")
public String readCookie(@CookieValue(value = "username", defaultValue = "Atta") String username) {
    return "Hey! My username is " + username;
}
```

3) 通过 **`HttpServletRequest` 的`getCookies()`方法**，读取所有的 `Cookie` 值；

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

## Session（会话）

session 是另一种记录服务器和客户端**会话状态**的机制。

- session 是**基于 cookie 实现**的，session 存储在服务器端，session id 会被存储到客户端的 cookie 中。

### Session-Cookie 方案进行身份验证

客户端和服务器间是通过http协议进行通信，但 http 协议是无状态的，**不同次请求会话**是没有任何关联的，优点是处理速度快。

- 很多时候都是通过 `SessionID` 来识别特定的用户，~~`SessionID` 一般会选择存放在 Redis 中~~。

在一次客户端和服务器之间的**会话**中，客户端（浏览器）向服务器发送请求，

1. 首次访问：
    1. 用户向服务器发送用户名、密码、验证码用于登陆系统。
    2. **服务器生成 Session**：
        1. 服务器验证通过后，会为这个用户创建一个专属的 **Session 对象**（存放该用户的状态数据，如购物车、登录信息等），并存储在 Session 映射区，~~一般会选择存放在 **Redis** 中~~。
        2. 并给这个 Session 对象**分配一个唯一的 `SessionID、JSESSIONID`**，服务器（通过 **HTTP 响应头**中的 `Set-Cookie` 指令），把这个 `SessionID` **返回给浏览器**。
    3. 浏览器：接收到 `SessionID` 后，以 **Cookie 的形式**保存在本地。同时 Cookie 记录此 SessionID 属于哪个域名。
2. 后续访问：
    1. 当用户**保持登录状态**时，后续每次向该服务器发请求，（请求会自动判断此域名下是否存在 Cookie 信息，如果存在），浏览器都会**自动带上**这个存有 `SessionID` 的 Cookie。
    2. 服务器收到请求后，从 Cookie 中拿出 `SessionID`。到**session库**查询是否存在，
        1. 如果存在，那么服务器能找到之前保存的那个 Session 对象，从而知道对应**哪个用户**以及他之前的状态。并进行后续操作。
        2. 如果不存在，就会创建一个JSESSIONID，并在本次请求结束后将JSESSIONID返回给客户端，同时在客户端cookie中进行保存。
    3. 进行后续操作。
3. 当浏览器关闭时，会话就结束了，但会话session还在。默认 session 保留30分钟。

![img](../assets/1548492-20220331163509335-813514932.png)

### 使用 Session 时需要注意

- 客户端 Cookie 支持：依赖 Session 的核心功能要确保用户浏览器开启了 Cookie。
- Session 过期管理：合理设置 Session 的过期时间，平衡安全性和用户体验。
- Session ID 安全：
    - 为包含 `SessionID` 的 Cookie 设置 **`HttpOnly` 标志**可以防止客户端脚本（如 JavaScript）窃取，
    - 设置 **Secure 标志**可以保证 `SessionID` 只在 HTTPS 连接下传输，增加安全性。

### HttpSession 生命周期

创建 HttpSession 对象的时机：

1. 对于JSP：浏览器访问服务端的任何一个JSP或Servlet，服务器不会为JSP创建一个 HttpSession 对象的情况：
    - 若当前的JSP或（Servlet）是客户端访问的当前WEB应用的**第一个资源**，且JSP的page指定属性`session="false"`，当前JSP页面禁用session隐含变量！但可以使用其他的显式的对象；
    - 若当前JSP不是访问的第一个资源，且其他页面已经创建一个（和当前会话关联的）HttpSession对象，直接返回。

2. 对于Servlet而言：若Servlet是客户端访问的第一个WEB应用的资源，则只有调用了`request.getSession([true])` 才会创建HttpSession对象。

Servlet 获取 HttpSession对象：`request.getSession(boolean create)`：

1. create为false，
    1. 若没有和当前JSP页面关联的HttpSession对象，则返回null；
    2. 若有则返回true；
2. create默认为true，一定返回一个HTTPSession对象：
    1. 若没有和当前JSP页面关联的HttpSession对象，则服务器**创建一个新的**HttpSession对象返回，
    2. 若有则直接返回关联。

销毁 HttpSession 对象的时机：

1. 直接**显式调用**HttpSession的`invalidate()`方法：使HttpSession失效。
2. 服务器卸载了当前Web应用。
3. 超出HttpSession的过期时间。

##### 只要关闭浏览器 ，session 真的就消失了？

不对。

- 对 session 来说，除非程序通知服务器删除一个 session，否则服务器会一直保留，程序一般都是在用户做 log off 的时候发个指令去删除 session。 
- 然而浏览器从来不会主动在关闭之前通知服务器它将要关闭，因此服务器根本不会有机会知道浏览器已经关闭，之所以会有这种错觉，是大部分 session 机制都使用会话 cookie 来保存 session id，而关闭浏览器后这个 session id 就消失了，再次连接服务器时也就无法找到原来的 session。
- 如果服务器设置的 cookie 被保存在硬盘上，或者使用某种手段改写浏览器发出的 HTTP 请求头，把原来的 session id 发送给服务器，则再次打开浏览器仍然能够打开原来的 session。
- 恰恰是**由于关闭浏览器不会导致 session 被删除，迫使服务器为 session 设置了一个失效时间，当距离客户端上一次使用 session 的时间超过这个失效时间时，服务器就认为客户端已经停止了活动，才会把 session 删除以节省存储空间。**

### 分布式会话管理

> 多服务器节点下 Session-Cookie 方案

举个例子：假如部署了两份相同的服务 A，B，

- 用户第一次登陆的时候 ，Nginx 通过**负载均衡机制**将用户请求转发到 A 服务器，此时用户的 Session 信息保存在 A 服务器。
- 结果，用户第二次访问的时候 Nginx 将请求路由到 B 服务器，由于 B 服务器没有保存 用户的 Session 信息，导致用户需要**重新进行登陆**。

> 应该如何避免上面这种情况的出现呢？

有几个方案可供大家参考：

1. 某个用户的所有请求都通过特性的**哈希策略**分配给同一个服务器处理。这样的话，每个服务器都保存了一**部分**用户的 Session 信息。服务器宕机，其保存的所有 Session 信息就完全**丢失**了。
2. 每一个服务器保存的 Session 信息都是**互相同步**的，也就是说每一个服务器都保存了**全量**的 Session 信息。每当一个服务器的 Session 信息发生变化，就将其同步到其他服务器。**成本太大**，节点越多、同步成本也越高。
3. 单独使用一个所有服务器都能访问到的数据节点（比如**缓存**）来存放 Session 信息。为了保证**高可用**，数据节点尽量要避免是单点。
4. **Spring Session** 是一个用于在多个服务器之间**管理会话**的项目。可以与多种后端存储（如 **Redis**、MongoDB 等）集成，从而实现**分布式会话管理**。可以将会话数据存储在共享的外部存储中，以实现**跨服务器**的会话同步和共享。

在考虑高性能之前，一定要做**高可用**。

### Session 的一致性问题

> 分布式架构下 session 共享方案

那么在部署生产环境下的 Tomcat 等 Web 容器时，**一定是需要部署多个节点**。此时，Session 的**一致性**就成为一个问题。

> Session 的一致性，简单来理解，就是相同 sessionid 在多个 Web 容器下，Session 的数据要一致。

以用户使用浏览器，Web 服务器为**两台** TomcatA、TomcatB 举例子。

1. 接上述例子，浏览器已经访问 TomcatA ，获得 sessionid 为 X 。同时，在多台 Tomcat 的情况下，我们需要采用 Nginx 做负载均衡。
2. 浏览器又发起一次请求访问 Web 服务器，Nginx 负载均衡转发请求到 TomcatB 上。TomcatB 会发现**请求的 Cookie** 中**已**存在 sessionid 为 X ，则直接获得 X 对应的 Session 。结果呢，找不到 X 对应的 Session ，只好创建一个 sessionid 为 X 的 Session 。
3. 此时，虽然说浏览器的 sessionid 是 X ，但是对应到两个 Tomcat 中两个 Session 。那么，如果在 TomcatA 上做的 Session 修改，TomcatB 的 Session 还是原样，这样就会出现 **Session 不一致**的问题。

解决 **Session 不一致**的问题，一般来说有三种方案：

#### Session 黏连

> IP 绑定策略

使用 **Nginx 负载均衡器**实现**会话黏连**，将相同 sessionid 的浏览器所发起的请求，**都转发到同一台服务器**。

- 相当于把用户和 A 服务器粘到了一块。
- 这样，就不会存在多个 Web 服务器创建多个 Session 的情况，也就不会发生 Session 不一致的问题。

- **优点：** 简单，不需要对 session 做任何处理。

- **缺点：** 
    - 缺乏容错性，如果当前访问的服务器发生故障，用户被转移到第二个服务器上时，他的 session 信息都将失效。
    - 因为，如果一台服务器**重启**，那么会导致转发到这个服务器上的 Session 全部丢失。
    - 改进：**Session 复制**。

- **适用场景：** 发生故障对客户产生的影响较小；服务器发生故障是低概率事件 。
    - 目前基本不被采用。

- **实现方式：** 以 Nginx 为例，在 upstream 模块配置 **ip_hash 属性**即可实现，将某个 ip的所有请求都定向到同一台服务器上，即将用户与服务器绑定。

> 具体怎么实现这种方式，可以看看 [《Nginx 第三方模块 —— nginx-sticky-module 的使用（基于cookie的会话保持）》](https://blog.csdn.net/bigtree_3721/article/details/78007853) 文章。

#### Session 复制

Web 服务器之间，进行 Session 复制同步。

- 任何一个服务器上的 session 发生改变（增删改），该节点会把这个 session 的所有内容序列化，然后广播给所有其它节点，不管其他服务器需不需要 session ，以此来保证 session 同步

- **适用场景：** 仅适用于实现 Session 复制的 Web 容器，例如说 Tomcat 、Weblogic 等等。
    - 目前基本也不被采用。

- **优点：** 可容错，各个服务器间 session 能够实时响应。

- **缺点：** 会对网络负荷造成一定压力，如果 session 量大的话可能会造成网络堵塞，拖慢服务器性能。效率低、浪费内存。
    - 改进：**Session 共享**。

<img src="../assets/d47d5cb95483956c76426585574ed133" alt="img" style="zoom: 80%;" />

具体怎么实现这种方式，可以看看 [《Session 共享 —— Tomcat 集群 Session 复制》](session 共享-tomcat集群session复制) 文章。

####  Session 共享 / 持久化

> 外部化存储（常用）

> 不同于上述的两种方案，考虑不再采用 **Web 容器的内存中**存储 Session ），

而是将 Session 存储外部化，**集中存储**、**持久化**到（MySQL、Redis、MongoDB） 等数据库中。所有的机器都来访问这个地方的数据。

- **优点：** 服务器出现问题，session 不会丢失 。
    - Tomcat 就可以**无状态化**，专注提供 Web 服务或者 API 接口，未来拓展扩容也变得更加容易。
    - 实现了 **session 共享**；不用复制了；
    - 服务器**重启** session 不丢失（不过也要注意 session 在 Redis 中的刷新/失效机制）；
    - 不仅可以**跨服务器** session 共享，甚至可以跨平台（例如网页端和 APP 端）
    - 可以水平扩展（增加 Redis 服务器）；
- **缺点：** 如果网站的访问量很大，把 session 存储到数据库中，会对数据库造成很大压力，还需要增加额外的开销维护数据库。
    - 增加了**单点失败**的可能性， 要是那个负责session 的机器挂了， 所有人都得重新登录一遍， 估计得被人骂死。
    - 架构上变得复杂，并且需要多访问一次 Redis。
- 改进：
    - 使用分布式缓存方案，比如 Memcached 、Redis **集群**来缓存 session。增加可靠性。
    - **引入 Token**。

<img src="../assets/4c6008d783611bb047e7524a15576647" alt="img" style="zoom: 50%;" /><img src="../assets/79a566230ee71dc79233e35529074335.jpg" alt="img" style="zoom: 55%;" />

而实现 Session 外部化存储也有两种方式：

- 当然 ① 和 ② 两种方案思路是类似且一致的，只是说**拓展的提供者和位置**不同。
- 相比来说，② 会比 ① 更加通用一些。

##### 使用 Session 管理器

① 基于 Tomcat、Jetty 等 Web 容器**自带的拓展**，使用读取外部存储器的 Session 管理器。例如说：

- [《Redisson Tomcat**会话管理器（Tomcat Session Manager）**》](https://github.com/redisson/redisson/wiki/14.-第三方框架整合#146-spring-session会话管理器) ，实现将 Tomcat 使用 Redis 存储 Session 。
- [《Jetty 集群配置 Session 存储到 MySQL、MongoDB》](https://blog.csdn.net/xiao__gui/article/details/43271509) ，实现 **Jetty** 使用 MySQL、MongoDB 存储 Session 。

##### SessionWrapper 对象

② 基于应用层封装 **[HttpServletRequest](https://github.com/javaee/servlet-spec/blob/master/src/main/java/javax/servlet/http/HttpServletRequest.java) 请求对象**，包装成自己的 RequestWrapper 对象，从而让实现调用 [`HttpServletRequest#getSession()`](https://github.com/javaee/servlet-spec/blob/master/src/main/java/javax/servlet/http/HttpServletRequest.java#L542-L581) 方法时，获得读写外部存储器的 SessionWrapper 对象。例如说，Spring Session。

##### Spring Session

- **Spring Session** 提供了 [SessionRepositoryFilter](https://github.com/spring-projects/spring-session/blob/master/spring-session-core/src/main/java/org/springframework/session/web/http/SessionRepositoryFilter.java) 过滤器，过滤请求时，将请求 HttpServletRequest 对象包装成 [SessionRepositoryRequestWrapper](https://github.com/spring-projects/spring-session/blob/master/spring-session-core/src/main/java/org/springframework/session/web/http/SessionRepositoryFilter.java#L192-L418) 对象。
- 调用 [`SessionRepositoryRequestWrapper#getSession()`](https://github.com/spring-projects/spring-session/blob/master/spring-session-core/src/main/java/org/springframework/session/web/http/SessionRepositoryFilter.java#L325-L328) 方法时，返回的是自己封装的 [HttpSessionWrapper](https://github.com/spring-projects/spring-session/blob/master/spring-session-core/src/main/java/org/springframework/session/web/http/SessionRepositoryFilter.java#L375-L390) 对象。
- 后续，调用 HttpSessionWrapper 的方法，例如说 `HttpSessionWrapper#setAttribute(String name, Object value)` 方法，访问的就是外部数据源，而不是内存中。

```java
// SessionRepositoryFilter.java

protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
    // sessionRepository 是访问外部数据源的操作类，例如说访问 Redis、MySQL 等等
    request.setAttribute(SESSION_REPOSITORY_ATTR, this.sessionRepository);
    

    // 将请求和响应进行包装成 SessionRepositoryRequestWrapper 和 SessionRepositoryResponseWrapper 对象
    SessionRepositoryFilter<S>.SessionRepositoryRequestWrapper wrappedRequest = new SessionRepositoryFilter.SessionRepositoryRequestWrapper(request, response, this.servletContext);
    SessionRepositoryFilter.SessionRepositoryResponseWrapper wrappedResponse = new SessionRepositoryFilter.SessionRepositoryResponseWrapper(wrappedRequest, response);

    // 继续执行下一个过滤器
    try {
        filterChain.doFilter(wrappedRequest, wrappedResponse);
    } finally {
        // 请求结束，提交 Session 到外部数据源
        wrappedRequest.commitSession();
    }
}

// SessionRepositoryFilter#SessionRepositoryRequestWrapper.java

	@Override
	public HttpSessionWrapper getSession() {
		return getSession(true);
	}
```

### 引入 Token

> 为什么要保存这可恶的session呢， 只让每个客户端去保存该多好？

可是如果不保存这些session id , 怎么**验证**客户端发给我的session id 的确是我生成的呢？ 

- 如果不去验证，我们都不知道他们是不是合法登录的用户， 那些不怀好意的家伙们就可以伪造session id , 为所欲为了。

关键点就是**验证** ！

- 比如说， 小F已经登录了系统， 我给他发一个**令牌(token)**， 里边包含了小F的 user id， 下一次小F 再次通过Http 请求访问我的时候， 把这个token 通过Http header 带过来就可以了。
    - 问题：Token是明文的，任何人都可以**伪造** Token。

- 对数据做一个**签名**：比如说用**加密算法** + （只有我才知道的）密钥， 对数据做一个签名， 把这个签名和数据一起作为token 。 由于密钥别人不知道， 就无法伪造token了。
- （这个token 我不保存）， 当小F把这个token 给我发过来的时候，我再用**同样的**加密算法和密钥，对数据再计算一次签名， 和token 中的签名做个比较，
    1.  如果相同， 我就知道小F已经登录过了，并且可以直接取到小F的user id , 
    2.  如果不相同， 数据部分肯定被人篡改过， 我就告诉发送者：对不起，没有认证。

<img src="../assets/e0444d8858508032c9c8b11b51850a0f" alt="img" style="zoom: 80%;" /><img src="../assets/a38d75882e1575ff9ddf336f934d8dc6" alt="img" style="zoom: 80%;" />

- Token 中的数据是明文保存的（虽然我会用Base64做下编码， 但那不是加密）， 还是可以被别人看到的， 所以不能在其中保存像**密码**这样的敏感信息。

- **CSRF **跨站请求伪造？：当然， 如果一个人的token 被别人偷走了， 那也没办法， 我也会认为小偷就是合法用户， 这其实和一个人的session id 被别人偷走是一样的。

- 这样一来， 我就不保存session id 了， 只是生成token , 然后验证token ， 我用**CPU计算时间**换取了我的 session 存储空间 ！

解除了session id这个负担， 可以说是无事一身轻， 机器集群现在可以轻松地做**水平扩展**， 用户访问量增大， 直接加机器就行。这种**无状态**的感觉实在是太好了！

## Token（令牌）

### Acesss Token

Acesss Token：访问资源接口（API）时所需要的**资源凭证**。

- **简单 token 的组成：** uid（用户唯一的身份标识）、time（当前时间的时间戳）、sign（签名，token 的前几位以哈希算法压缩成的一定长度的十六进制字符串）。
- 特点：
    1. 服务端**无状态化**、**可扩展性**好
    2. 支持**移动端**设备
    3. 安全
    4. 支持跨程序调用
- 基于 token 的用户认证是一种服务端**无状态**的认证方式，服务端不用存放 token 数据。
    - 用解析 token 的计算时间**换取** session 的存储空间，从而减轻服务器的压力，减少频繁的查询数据库。
- token 完全由应用管理，所以它可以避开**同源策略**。

#### 基于 token 的身份验证流程

1. 客户端使用用户名跟密码**请求登录**。
2. 服务端收到请求，去**验证**用户名与密码。
3. 验证成功后，服务端会**签发一个 token** 并把这个 token 发送给客户端。
4. 客户端收到 token 以后，会把它**存储起来**，比如放在 cookie 里或者 localStorage 里。
5. 客户端每次向服务端请求资源时，都需要**带着**（服务端签发的）token（放到 HTTP 的 Header 里）。
6. 服务端收到请求，然后去**验证**客户端请求里面带着的 token ，如果验证成功，就向客户端返回请求的数据。

<img src="../assets/3be751e707405a06201a0ef60c698f39.jpg" alt="img" style="zoom:50%;" />

### Refresh Token

> 另外一种 token——refresh token

refresh token：是专用于**刷新 access token** 的 token。

- 如果没有 refresh token，也可以刷新 access token，但每次刷新都要用户输入登录用户名与密码，会很麻烦。
- 有了 refresh token，可以减少这个麻烦，客户端**直接用 refresh token 去更新** access token，无需用户进行额外的操作。
- **Access Token 的有效期**比较短，当过期而失效时，使用 Refresh Token 就可以获取到新的 Token，如果 Refresh Token 也失效了，用户就只能重新登录了。
- Refresh Token 及过期时间是存储在服务器的数据库中，只有在申请新的 Acesss Token 时才会验证，不会对业务接口响应时间造成影响，也不需要向 Session 一样一直保持在内存中以应对大量的请求。

![img](../assets/2b9dec4537eff5e7d345345f3cda224b10a7ad.jpg)

### Token 和 Session 的区别

- Session 是一种**记录**服务器和客户端**会话状态**的机制，使服务端有状态化，可以记录会话信息。而 Token 是**令牌**，访问资源接口（API）时所需要的**资源凭证**。Token 使服务端**无状态化**，不会存储会话信息。
- Session 和 Token 并不矛盾，作为身份认证 **Token 安全性**比 Session 好，因为每一个请求都有签名还能防止监听以及重放攻击，而 Session 就必须依赖链路层来保障通讯安全了。如果需要实现**有状态的会话**，仍然可以增加 Session 来在服务器端保存一些状态。
- 所谓 Session 认证只是简单的把 User 信息存储到 Session 里，因为 SessionID 的不可预测性，暂且认为是安全的。而 Token ，如果指的是 OAuth Token 或类似的机制的话，提供的是 **认证 和 授权** ，认证是针对用户，授权是针对 App 。
    - 其目的是让某 App 有权利访问某用户的信息。这里的 Token 是唯一的。不可以转移到其它 App上，也不可以转到其它用户上。
    - Session 只提供一种简单的认证，即只要有此 SessionID ，即认为有此 User 的全部权利。是需要严格保密的，这个数据应该只保存在站方，不应该共享给其它网站或者第三方 App。
- 所以简单来说：如果你的用户数据可能需要和第三方共享，或者允许第三方调用 API 接口，用 Token 。如果永远只是自己的网站，自己的 App，用什么就无所谓了。

### CSRF 攻击

> 为什么 Cookie 无法防止 CSRF 攻击，而 Token 可以？

使用 Session-Cookie 方案进行身份验证时，容易出现 **CSRF **跨站请求伪造：如果别人通过 `Cookie` 拿到了 `SessionId` 后就可以**代替**你的身份访问系统了。

但是，使用 `Token` 的话就不会存在这个问题：

1. 在登录成功获得 `Token` 之后，一般会选择存放在 `localStorage` （浏览器本地存储）中。
2. 然后在前端（通过某些方式）会给每个（发到后端的）请求加上这个 `Token`，这样就不会出现 CSRF 漏洞的问题。
3. 因为，即使点击了**非法链接**发送了请求到服务端，这个非法请求是不会携带 `Token` 的，所以这个请求将是非法的。

需要注意：不论是 `Cookie` 还是 `Token` 都无法避免 **XSS 跨站脚本攻击** 。

##### XSS

跨站脚本攻击（Cross Site Scripting，XSS）：攻击者会用各种方式将**恶意代码**注入到其他用户的页面中。就可以通过脚本盗用信息比如 `Cookie` 。

<img src="../assets/20210615161108272.png" style="zoom: 67%;" />

## JWT

**JWT** (`JSON WEB TOKEN)`：是一种可安全传输的的 JSON 对象。由于使用了**数字签名**，所以是可信任和安全的。

- 是目前最流行的**跨域认证**解决方案，是一种**基于 Token** 的**认证授权机制**。
- 从全称可以看出，JWT 本身也是 Token，一种规范化之后的 **JSON 结构的 Token**。
    - JWT 是为了在网络应用环境间**传递声明**而执行的一种基于 JSON 的开放标准（[RFC 7519](https://tools.ietf.org/html/rfc7519)）。
    - JWT 的声明一般被用来，在身份提供者和服务提供者间**传递**被认证的用户身份信息，以便于从资源服务器获取资源。比如用在用户登录上。
- 可以使用 HMAC 算法或者是 RSA 的公/私秘钥对 JWT 进行签名。因为数字签名的存在，这些传递的信息是可信的。
- 用于解决**使用 Session 进行身份认证**遇到的问题。

#### Token 和 JWT 的区别

相同：

1. 都是访问资源的令牌
2. 都可以记录用户的信息
3. 都是使服务端无状态化
4. 都是只有验证成功后，客户端才能访问服务端上受保护的资源

区别：

- Token：服务端验证客户端发送过来的 Token 时，还需要**查询数据库**获取用户信息，然后验证 Token 是否有效。
- JWT： 将 Token 和 Payload 加密后存储于客户端，服务端只需要**使用密钥解密**进行校验（校验也是 JWT 自己实现的）即可，不需要查询或者减少查询数据库，因为 JWT 自包含了用户信息和加密的数据。

#### 优势

相比于 Session 认证的方式来说，使用 JWT 进行身份认证主要有下面 4 个优势。

1. **无状态**：JWT 自身包含了身份验证所需要的所有信息，因此，服务器不需要存储 JWT **Session** 信息。增加了系统的可用性和伸缩性，大大减轻了服务端的压力。
    - 因为用户的状态不再存储在服务端的内存中，所以这是一种无状态的认证机制。
    - 也导致了最大的缺点：**不可控！**就比如说，想要在 JWT 有效期内废弃一个 JWT 或者更改它的权限的话，并不会立即生效，通常需要等到有效期过后才可以。
    - 再比如说，当用户 Logout 的话，JWT 也还有效。除非，在后端增加额外的处理逻辑，比如将失效的 JWT 存储起来，后端先验证 JWT 是否有效再进行处理。具体的解决办法，在后面详细介绍。
2. 有效**避免 CSRF 攻击**：因为 JWT 一般是存在在 **localStorage** 中，使用 JWT 进行身份验证的过程中是**不会涉及到 Cookie** 的。
    1. CSRF 攻击需要**依赖 Cookie** ，Session 认证中 Cookie 中的 `SessionID` 是由浏览器发送到服务端的，只要发出请求，Cookie 就会被携带。借助这个特性，即使黑客无法获取你的 `SessionID`，只要让你误点攻击链接，就可以达到攻击效果。
    2. 使用 JWT 进行身份验证**不需要依赖 Cookie** ，因此可以避免 CSRF 攻击。
        1. 一般情况下使用 JWT 的话，在登录成功获得 JWT 之后，一般会选择存放在 localStorage 中。前端的每一个请求后续都会附带上这个 JWT，整个过程压根不会涉及到 Cookie。
        2. 因此，即使点击了**非法链接**发送了请求到服务端，这个非法请求也不会携带 JWT，所以这个请求将是非法的。
3. 适合**移动端**应用：使用 Session 进行身份认证的话，需要保存一份信息在服务器端，而且这种方式会依赖到 Cookie（需要 Cookie 保存 `SessionId`），所以不适合移动端。但是，使用 JWT 进行身份认证就不会存在这种问题，因为只要 JWT 可以被客户端存储就能够使用，而且 JWT 还可以跨语言使用。
4. **单点登录**友好：使用 Session 进行身份认证的话，实现单点登录，需要把用户的 Session 信息保存在一台电脑上，并且还会遇到常见的 **Cookie 跨域**的问题。但是，使用 JWT 进行认证的话， JWT 被保存在客户端，不会存在这些问题。
    - 因为 JWT 并**不使用 Cookie** 的，所以你可以使用任何域名提供你的 API 服务而不需要担心**跨域资源共享问题**（CORS）
5. 因为 JWT 是自包含的（内部包含了一些会话信息），因此减少了需要查询数据库的需要

缺点：

疑问：为什么不使用 **JWT**（JSON Web Token）？

- JWT 是**无状态的**，无法实现 **Token 的作废**，例如说用户登出系统、修改密码等场景。

- 推荐阅读 [《还分不清 Cookie、Session、Token、JWT？》 (opens new window)](https://www.iocoder.cn/Fight/Confused-about-cookies-sessions,-Tokens-JWT/?yudao)文章。

#### 组成

JWT 本质上就是一组字串，通过（`.`）切分成三个为 Base64Url 编码的部分：`header.payload.signature`。

1. **Header（头部）** : 描述 JWT 的元数据，定义了**生成签名的算法**以及 `Token` 令牌的类型。
2. **Payload（载荷）** : 用来存放实际需要传递的数据，包含声明（Claims），如`sub`（主题、存放用户名）、`jti`（JWT ID）、`token`的生成时间、过期时间。
3. **Signature（签名）**：服务器通过 Payload、Header 和一个密钥（Secret）使用 Header 里面指定的签名算法（默认是 HMAC SHA256）生成。作用是防止 JWT（主要是 payload） **被篡改**，一旦`header`和`payload`被篡改，验证将失败。

![JWT 组成](../assets/jwt-composition.png)

示例：

```java
Header: {"alg": "HS512"}
Payload: {"sub":"admin","created":1489079981393,"exp":1489684781}
//JWT计算公式，secret为加密算法的密钥
String signature = HMACSHA512(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret);

//生成一个JWT实例的字符串
eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJhZG1pbiIsImNyZWF0ZWQiOjE1NTY3NzkxMjUzMDksImV4cCI6MTU1NzM4MzkyNX0.d-iki0193X0bBOETf2UN3r3PotNIEAV7mzIxxeI5IxFyzzkOZxS0PGfF_SK6wxCv2K8S0cZjMkv6b5bCqc0VBw
```

### 实现认证和授权的原理

1. 用户向服务器发送用户名、密码以及验证码（用于登陆接口）；
2. 如果**校验**正确，登录成功后，服务端生成已经签名的 Token，即 **JWT**，并返回给客户端；
3. 客户端收到 Token 后自己保存起来（比如浏览器的 `localStorage`，放在 Cookie 中会有 CSRF 风险 ）；
4. 之后当用户希望访问一个受保护的路由或者资源时，客户端（发出的所有请求都会）在 HTTP 请求头 `Header` 中携带这个 **JWT 令牌**：添加一个 `Authorization` 字段，使用 Bearer 模式值为 `JWT`，（`Authorization: Bearer <Token>`）；
5. 服务端（的保护路由将会）检查请求头 Authorization 中的 JWT 信息，
    - 通过对 `Authorization` 头中信息**解码**及**数字签名校验**（再次生成一个 Signature 并对比），
    - 并从中获取其中的用户信息，从而实现认证和授权。
    - 如果合法，则允许用户的行为。

<img src="../assets/jwt-1.jpeg" alt="../_images/jwt-1.jpeg" style="zoom: 80%;" />

### 使用方式

- 客户端收到服务器返回的 JWT，可以储存在 Cookie 里面，也可以储存在 localStorage。

#### 方式一

- 当用户希望访问一个受保护的路由或者资源时

    - 可以把它放在 Cookie 里面自动发送，但是这样**不能跨域**，
    - 所以更好的做法是，放在 **HTTP 请求头**信息的 Authorization 字段里，使用 Bearer 模式添加 JWT。
    
    ```http
    GET /calendar/v1/events
Host: api.example.com
    Authorization: Bearer <token>
    ```
    
    - 用户的状态不会存储在服务端的内存中，这是一种 **无状态的认证机制**
    - 服务端的保护路由将会检查请求头 Authorization 中的 JWT 信息，如果合法，则允许用户的行为。
    - 由于 JWT 是自包含的，因此减少了需要查询数据库的需要
    - JWT 的这些特性使得我们可以完全依赖其无状态的特性提供数据 API 服务，甚至是创建一个下载流服务。
    - 因为 JWT 并不使用 Cookie ，所以你可以使用任何域名提供你的 API 服务而**不需要担心跨域资源共享问题**（CORS）

#### 方式二

- 跨域时，可以把 JWT 放在 POST 请求的**数据体**里。

#### 方式三

- 通过 URL 传输

```http
http://www.example.com/user?token=xxx
```

### 如何防止被篡改？

有了签名之后，即使 JWT 被泄露或者截获，黑客也没办法同时篡改 Signature、Header、Payload。

- 因为服务端拿到 JWT 之后，会解析出其中包含的 Header、Payload 以及 Signature 。并根据 Header、Payload、密钥**再次生成一个 Signature**。
- 拿新生成的 Signature 和 JWT 中的 Signature 作对比，如果一样就说明 Header 和 Payload 没有被修改。

不过，如果服务端的秘钥也被泄露的话，黑客就可以同时篡改 Signature、Header、Payload 了。黑客直接修改了 Header 和 Payload 之后，再重新生成一个 Signature 就可以了。

JWT 安全的核心在于**签名**，签名安全的**核心在密钥**。

### 如何加强安全性？

1. 使用安全系数高的**加密算法**。
2. 使用成熟的开源库，没必要造轮子。
3. JWT 存放在 **localStorage** 中而不是 Cookie 中，避免 CSRF 风险。
4. 一定不要将**隐私信息**存放在 Payload 当中。
5. **密钥**一定保管好，不要泄露出去。JWT 安全的核心在于**签名**，签名安全的核心在**密钥**。
6. Payload 要加入 `exp` （JWT 的过期时间），永久有效的 JWT 不合理。并且，JWT 的过期时间不宜过长。

### 身份认证常见问题及解决办法

1. 注销登录等场景下 JWT 还有效
2. JWT 的续签问题
3. JWT 体积太大

## Cookie、Session、Token、JWT 常见问题

> 需要考虑的问题

#### 使用 cookie 时

- 因为存储在客户端，容易被客户端篡改，使用前需要验证合法性
- 不要存储敏感数据，比如用户密码，账户余额
- 使用 httpOnly 在一定程度上提高安全性
- 尽量减少 cookie 的体积，能存储的数据量不能超过 4kb
- 设置正确的 domain 和 path，减少数据传输
- **cookie 无法跨域**
- 一个浏览器针对一个网站最多存 20 个Cookie，浏览器一般只允许存放 300 个Cookie
- **移动端对 cookie 的支持不是很好，而 session 需要基于 cookie 实现，所以移动端常用的是 token**

#### 使用 session 时

- 将 session 存储在服务器里面，当用户同时在线量比较多时，这些 session 会占据较多的内存，需要在服务端定期的去清理过期的 session
- 当网站采用**集群部署**的时候，会遇到多台 web 服务器之间如何做 session 共享的问题。因为 session 是由单个服务器创建的，但是处理用户请求的服务器不一定是那个创建 session 的服务器，那么该服务器就无法拿到之前已经放入到 session 中的登录凭证之类的信息了。
- 当多个应用要共享 session 时，除了以上问题，还会遇到跨域问题，因为不同的应用可能部署的主机不一样，需要在各个应用做好 cookie 跨域的处理。
- **sessionId 是存储在 cookie 中的，假如浏览器禁止 cookie 或不支持 cookie 怎么办？** 一般会把 sessionId 跟在 url 参数后面即重写 url，所以 session 不一定非得需要靠 cookie 实现
- **移动端对 cookie 的支持不是很好，而 session 需要基于 cookie 实现，所以移动端常用的是 token**

#### 使用 token 时

- 如果你认为用数据库来存储 token 会导致查询时间太长，可以选择放在内存当中。比如 redis 很适合你对 token 查询的需求。
- **token 完全由应用管理，所以它可以避开同源策略**
- **token 可以避免 CSRF 攻击(因为不需要 cookie 了)**
- **移动端对 cookie 的支持不是很好，而 session 需要基于 cookie 实现，所以移动端常用的是 token**

#### 使用 JWT 时

- 因为 JWT 并不依赖 Cookie 的，所以你可以使用任何域名提供你的 API 服务而不需要担心跨域资源共享问题（CORS）
- JWT 默认是不加密，但也是可以加密的。生成原始 Token 以后，可以用密钥再加密一次。
- JWT 不加密的情况下，不能将秘密数据写入 JWT。
- JWT 不仅可以用于认证，也可以用于交换信息。有效使用 JWT，可以降低服务器查询数据库的次数。
- JWT 最大的优势是服务器不再需要存储 Session，使得服务器认证鉴权业务可以方便扩展。但这也是 JWT 最大的缺点：由于服务器不需要存储 Session 状态，因此使用过程中无法废弃某个 Token 或者更改 Token 的权限。也就是说一旦 JWT 签发了，到期之前就会始终有效，除非服务器部署额外的逻辑。
- JWT 本身包含了认证信息，一旦泄露，任何人都可以获得该令牌的所有权限。为了减少盗用，JWT的有效期应该设置得比较短。对于一些比较重要的权限，使用时应该再次对用户进行认证。
- JWT 适合一次性的命令认证，颁发一个有效期极短的 JWT，即使暴露了危险也很小，由于每次操作都会生成新的 JWT，因此也没必要保存 JWT，真正实现无状态。
- 为了减少盗用，JWT 不应该使用 HTTP 协议明码传输，要使用 HTTPS 协议传输。

#### 使用加密算法时

- 绝不要以**明文存储**密码
- 永远使用 哈希算法 来处理密码，绝不要使用 Base64 或其他编码方式来存储密码，这和以明文存储密码是一样的，使用哈希，而不要使用编码。编码以及加密，都是双向的过程，而密码是保密的，应该只被它的所有者知道， 这个过程必须是单向的。哈希正是用于做这个的，从来没有解哈希这种说法， 但是编码就存在解码，加密就存在解密。
- 绝不要使用弱哈希或已被破解的哈希算法，像 MD5 或 SHA1 ，只使用强密码哈希算法。
- 绝不要以明文形式显示或发送密码，即使是对密码的所有者也应该这样。如果你需要 “忘记密码” 的功能，可以随机生成一个新的 **一次性的**（这点很重要）密码，然后把这个密码发送给用户。


