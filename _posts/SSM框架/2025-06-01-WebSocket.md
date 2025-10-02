---
title: WebSocket 框架
categories:
  - SSM框架
tags:
  - Spring-MVC
  - Spring
  - HTTP
  - Tomcat
  - WebSocket
location:
abbrlink: 'websocket'
permalink: 'websocket'
date: 2025-06-01 13:42:12
updated: 2025-06-01 11:56:00
---

> 摘要：最大特点就是：服务器可以**主动**向客户端发送数据（推送信息），这样就可以完成**实时性**较高的需求。提供 **Token 认证**、WebSocket **集群广播**、Message 监听。

<!-- more -->

## 目录

[TOC]

> WebSocket 框架，支持多节点的广播。
>
> 提供 **Token 认证**、WebSocket **集群广播**、Message 监听。

WebSocket 协议：

- 最大特点就是：服务器可以**主动**向客户端发送数据（推送信息），这样就可以完成**实时性**较高的需求。是真正的双向平等对话，属于[服务器推送技术](https://en.wikipedia.org/wiki/Push_technology)的一种。

- 例如说，聊天 IM 即使通讯功能、**消息订阅服务**、网页游戏等等。

同时，因为 WebSocket 使用 **TCP 通信**，可以避免**重复创建连接**，提升通信质量和效率。

- 例如说，美团的长连接服务，具体可以看看 [《美团点评移动网络优化实践》](https://tech.meituan.com/2017/03/17/shark-sdk.html) 。

> 参考[芋道 Spring Boot WebSocket 入门](https://www.iocoder.cn/Spring-Boot/WebSocket/)

## 简介

### 为什么需要 WebSocket？

> 已经有了 HTTP 协议，为什么还需要另一个协议？它能带来什么好处？

- 因为 HTTP 协议有一个缺陷：通信只能由客户端发起。HTTP 协议做不到服务器主动向客户端推送信息。
- 这种**单向请求**的特点，注定了如果服务器有**连续的状态变化**，客户端要获知就非常麻烦。
    - 只能使用["轮询"](https://www.pubnub.com/blog/2014-12-01-http-long-polling/)：每隔一段时候，就发出一个询问，了解服务器有没有新的信息。最典型的场景就是聊天室。
    - 轮询的**效率低**，非常浪费资源（因为必须不停连接，或者 HTTP 连接始终打开）。WebSocket 就是这样发明的。

### 其他特点

其他特点包括：

（1）建立在 **TCP 协议**之上，服务器端的实现比较容易。

（2）与 HTTP 协议有着良好的**兼容性**。默认端口也是**80和443**，并且**握手阶段**采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。

（3）数据格式比较轻量，**性能开销小**，通信高效。

（4）可以发送文本，也可以发送二进制数据。

（5）没有同源限制，客户端可以与任意服务器通信。

（6）协议**标识符**是`ws`（如果加密，则为`wss`），服务器网址就是 URL。

> ```markup
> ws://example.com:80/some/path
> ```

<img src="../assets/bg2017051503.jpg" alt="img" style="zoom:80%;" />

### 与 HTTP 的关系

基本上但凡提到WebSocket和HTTP的关系都会有以下两条：

1. WebSocket 和 HTTP 都是基于**TCP协议**的两个不同的协议。
2. WebSocket **依赖于** HTTP 连接。

<img src="../assets/bg2017051502.png" alt="img" style="zoom: 80%;" />

#### 如何从连接的HTTP协议转化为WebSocket协议？

每个WebSocket连接都始于一个HTTP请求。 具体来说，WebSocket协议在**第一次握手连接**时，通过HTTP协议在传送WebSocket支持的版本号，协议的字版本号，原始地址，主机地址等等一些列字段给服务器端：

```http
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key:dGhlIHNhbXBsZSBub25jZQ==
Origin: http://example.com
Sec-WebSocket-Version: 13
```

注意，关键的地方是，这里面有个Upgrade首部，用来把当前的HTTP请求升级到WebSocket协议，这是HTTP协议本身的内容，是为了扩展支持其他的通讯协议。 

- 如果服务器支持新的协议，则必须返回101：

```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept:s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

至此，HTTP请求物尽其用，如果成功则触发onopen事件，否则触发onerror事件，后面的传输则不再依赖HTTP协议。 

#### WebSocket为什么要依赖于HTTP协议的连接？

1. 第一，WebSocket设计上就是天生为HTTP**增强通信**（全双工通信等），所以在HTTP协议连接的基础上是很自然的一件事，并因此而能获得HTTP的诸多便利。 
2. 第二，这诸多便利中有一条很重要，基于HTTP连接将获得最大的一个兼容支持，比如即使服务器不支持WebSocket也能**建立HTTP通信**，只不过返回的是onerror而已，这显然比服务器无响应要好的多。

## WebSocket 解决方案

在实现提供 WebSocket 服务的项目中，一般有如下几种解决方案：

- **方案一** [Spring WebSocket](https://docs.spring.io/spring-framework/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/html/websocket.html)
- 方案二 [Tomcat WebSocket](https://www.cnblogs.com/xdp-gacl/p/5193279.html)
- 方案三 [Netty WebSocket](https://netty.io/news/2012/11/15/websocket-enhancement.html)

有个涉及到 IM 即使通讯的项目，采用的是方案三。

- 主要原因是，我们对 Netty 框架的实战、原理与源码，都相对熟悉一些。所以就考虑了它。
- 并且，除了需要支持 WebSocket 协议，我们还想提供原生的 Socket 协议。

如果仅仅是仅仅提供 WebSocket 协议的支持，可以考虑采用方案一或者方案二。

> 在使用上，两个方案是比较接近的。二者的实现代码，没啥差别。

1. 相比来说，**方案一** Spring WebSocket 内置了对 **[STOMP](https://docs.spring.io/spring-framework/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/html/websocket.html#websocket-stomp-overview) 协议**的支持。
2. 不过，还是采用方案二 Tomcat WebSocket 来作为入门示例。

[JSR-356](https://www.oracle.com/technical-resources/articles/java/jsr356.html) 规范：定义了 Java 针对 WebSocket 的 API ，即 [Javax WebSocket](https://mvnrepository.com/artifact/javax.websocket) 。

- 目前，主流的 Web 容器都已经提供了 JSR-356 的实现，例如说 Tomcat、Jetty、Undertow 等等。

## Tomcat WebSocket

使用 Tomcat WebSocket 搭建一个 WebSocket 的示例。提供如下消息的功能支持：

- 身份认证请求
- 私聊消息
- 群聊消息

考虑到让示例更加易懂，先做成全局有且仅有一个大的聊天室，即建立上 WebSocket 的连接，都自动动进入该聊天室。

### 依赖

```xml
<!-- 实现对 WebSocket 相关依赖的引入，方便~ -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

### WebsocketServerEndpoint

创建 [WebsocketServerEndpoint](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-25/lab-websocket-25-01/src/main/java/cn/iocoder/springboot/lab25/springwebsocket/websocket/WebsocketServerEndpoint.java) 类，定义 Websocket 服务的**端点（EndPoint）**。代码如下：

- 在类上，添加 `@Controller` 注解，保证创建一个 WebsocketServerEndpoint Bean 。
- 在类上，添加 JSR-356 定义的 [`@ServerEndpoint`](https://github.com/eclipse-ee4j/websocket-api/blob/master/api/server/src/main/java/javax/websocket/server/ServerEndpoint.java) 注解，标记这是一个 WebSocket EndPoint ，路径为 `/` 。
- WebSocket 一共有四个事件，分别对应使用 JSR-356 定义的 [`@OnOpen`](https://github.com/eclipse-ee4j/websocket-api/blob/master/api/client/src/main/java/javax/websocket/OnOpen.java)、[`@OnMessage`](https://github.com/eclipse-ee4j/websocket-api/blob/master/api/client/src/main/java/javax/websocket/OnMessage.java)、[`@OnClose`](https://github.com/eclipse-ee4j/websocket-api/blob/master/api/client/src/main/java/javax/websocket/OnClose.java)、[`@OnError`](https://github.com/eclipse-ee4j/websocket-api/blob/master/api/client/src/main/java/javax/websocket/OnError.java) 注解。

```java
// WebsocketServerEndpoint.java

@Controller
@ServerEndpoint("/")
public class WebsocketServerEndpoint {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @OnOpen
    public void onOpen(Session session, EndpointConfig config) {
        logger.info("[onOpen][session({}) 接入]", session);
    }

    @OnMessage
    public void onMessage(Session session, String message) {
        logger.info("[onOpen][session({}) 接收到一条消息({})]", session, message); // 生产环境下，请设置成 debug 级别
    }

    @OnClose
    public void onClose(Session session, CloseReason closeReason) {
        logger.info("[onClose][session({}) 连接关闭。关闭原因是({})}]", session, closeReason);
    }

    @OnError
    public void onError(Session session, Throwable throwable) {
        logger.info("[onClose][session({}) 发生异常]", session, throwable);
    }

}
```

### WebSocketConfiguration

创建 [WebsocketServerEndpoint](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-25/lab-websocket-25-01/src/main/java/cn/iocoder/springboot/lab25/springwebsocket/config/WebSocketConfiguration.java) 配置类。

- 在 `#serverEndpointExporter()` 方法中，创建 ServerEndpointExporter Bean 。作用是扫描添加有 `@ServerEndpoint` 注解的 Bean 。
- **@EnableWebSocket**：使用 Spring WebSocket 时添加该注解

```java
// WebSocketConfiguration.java

@Configuration
// @EnableWebSocket // 无需添加该注解，因为我们并不是使用 Spring WebSocket
public class WebSocketConfiguration {

    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }

}
```

### 测试

使用 [WEBSOCKET 在线测试工具](http://www.easyswoole.com/wstool.html)  测试 WebSocket 连接

### 消息

- 在 HTTP 协议中，是基于 **Request/Response 请求响应**的**同步**模型，进行交互。
- 在 Websocket 协议中，是基于 **Message 消息**的**异步**模型，进行交互。

> 这一点是很大的不同的，具体的消息类，感受会更明显。

所以在这个示例中，采用的 Message 采用 JSON 格式编码，主要考虑便捷性。

- Message 也可以考虑 [Protobuf](https://github.com/protocolbuffers/protobuf) 等更加高效且节省流量的编码格式。

Message 格式如下：

1. `type` 字段，消息类型。指定使用哪个 MessageHandler 消息处理器。
    - 因为 WebSocket 协议，不像 HTTP 协议有 **URI** 可以区分不同的 API 请求操作，所以需要在 Message 里，增加`type` 字段用于**标识消息类型** 。
2. `body` 字段，消息体。不同的消息类型，会有不同的消息体。

```json
{
    type: "", // 消息类型
    body: {} // 消息体
}
```

#### Message

创建 [Message](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-25/lab-websocket-25-01/src/main/java/cn/iocoder/springboot/lab25/springwebsocket/message/Message.java) 接口，基础消息体，所有消息体都要实现该接口。

- 目前作为一个标记接口，未定义任何操作。

```java
// Message.java

public interface Message {
}
```

#### 认证相关 Message

##### AuthRequest

创建 [AuthRequest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-25/lab-websocket-25-01/src/main/java/cn/iocoder/springboot/lab25/springwebsocket/message/AuthRequest.java) 类，用户认证请求。

- `TYPE` 静态属性，消息类型为 `AUTH_REQUEST` 。
- `accessToken` 属性，**认证 Token** 。在 WebSocket 协议中，也需要认证当前连接，用户身份是什么。
    - 一般情况下，采用用户调用 **HTTP 登录接口**，登录成功后返回的访问令牌 `accessToken` 。
    - 可以看看 [《基于 Token 认证的 WebSocket 连接》](https://yq.aliyun.com/articles/229057) 文章。

```java
// AuthRequest.java

public class AuthRequest implements Message {

    public static final String TYPE = "AUTH_REQUEST";

    /**
     * 认证 Token
     */
    private String accessToken;
    
    // ... 省略 set/get 方法
    
}
```

##### AuthResponse

虽然说，WebSocket 协议是基于 **Message 模型**进行交互。但是，这并不意味着它的操作，不需要响应结果。例如说，**用户认证请求**，是需要用户认证响应的。所以，创建 [AuthResponse](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-25/lab-websocket-25-01/src/main/java/cn/iocoder/springboot/lab25/springwebsocket/message/AuthResponse.java) 类，作为用户认证响应。

- `TYPE` 静态属性，消息类型为 `AUTH_REQUEST` 。实际上，在每个 Message 实现类上，都增加了 **`TYPE` 静态属性**，作为消息类型。下面就不重复赘述了。
- `code` 属性，响应状态码。
- `message` 属性，响应提示。

```java
// AuthResponse.java

public class AuthResponse implements Message {

    public static final String TYPE = "AUTH_RESPONSE";

    /**
     * 响应状态码
     */
    private Integer code;
    /**
     * 响应提示
     */
    private String message;
    
    // ... 省略 set/get 方法
    
}
```

##### UserJoinNoticeRequest

在本示例中，用户成功认证之后，会**广播**用户加入群聊的**通知 Message** ，使用 [UserJoinNoticeRequest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-25/lab-websocket-25-01/src/main/java/cn/iocoder/springboot/lab25/springwebsocket/message/UserJoinNoticeRequest.java) 。

```java
// UserJoinNoticeRequest.java

public class UserJoinNoticeRequest implements Message {

    public static final String TYPE = "USER_JOIN_NOTICE_REQUEST";
    private String nickname;
    
    // ... 省略 set/get 方法
}
```

##### 优化小想法

实际上，可以在需要使用到 Request/Response 模型的地方，将 Message 进行拓展：

- Request 抽象类，增加 **`requestId` 字段**，表示请求编号。
- Response 抽象类，增加 `requestId` 字段，和每一个 Request 请求映射上。同时，里面统一定义 `code` 和 `message` 属性，表示**响应状态码和响应提示**。

这样，

- 在使用到**同步模型**的业务场景下，Message 实现类使用 **Request/Reponse 作为后缀**。例如说，用户认证请求、删除一个好友请求等等。

- 而在使用到**异步模型**的业务场景下，Message 实现类还是继续 **Message 作为后缀**。例如说，发送一条消息，用户操作完后，无需阻塞等待结果

#### 发送消息相关 Message

##### SendToOneRequest

创建 [SendToOneRequest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-25/lab-websocket-25-01/src/main/java/cn/iocoder/springboot/lab25/springwebsocket/message/SendToOneRequest.java) 类，发送给指定人的私聊消息的 Message。

```java
// SendToOneRequest.java

public class SendToOneRequest implements Message {

    public static final String TYPE = "SEND_TO_ONE_REQUEST";

    /**
     * 发送给的用户
     */
    private String toUser;
    /**
     * 消息编号
     */
    private String msgId;
    /**
     * 内容
     */
    private String content;
    
    // ... 省略 set/get 方法
    
}
```

##### SendToAllRequest

创建 [SendToAllRequest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-25/lab-websocket-25-01/src/main/java/cn/iocoder/springboot/lab25/springwebsocket/message/SendToAllRequest.java) 类，发送给所有人的群聊消息的 Message。

```java
// SendToAllRequest.java

public class SendToAllRequest implements Message {

    public static final String TYPE = "SEND_TO_ALL_REQUEST";

    /**
     * 消息编号
     */
    private String msgId;
    /**
     * 内容
     */
    private String content;
    
    // ... 省略 set/get 方法
     
}
```

##### SendResponse

在服务端接收到发送消息的请求，需要**异步**响应发送是否成功。所以，创建 [SendResponse](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-25/lab-websocket-25-01/src/main/java/cn/iocoder/springboot/lab25/springwebsocket/message/SendResponse.java) 类，发送消息响应结果的 Message 。

```java
// SendResponse.java

public class SendResponse implements Message {

    public static final String TYPE = "SEND_RESPONSE";

    /**
     * 消息编号
     */
    private String msgId;
    /**
     * 响应状态码
     */
    private Integer code;
    /**
     * 响应提示
     */
    private String message;
    
    // ... 省略 set/get 方法
    
}
```

- 重点看 `msgId` 字段，消息编号。客户端在发送消息，通过使用 [UUID](https://zh.wikipedia.org/zh-hans/通用唯一识别码) 算法，生成全局唯一消息编号。这样，服务端通过 SendResponse 消息响应，通过 `msgId` 做映射。

##### SendToUserRequest

在**服务端**接收到发送消息的请求，需要转发消息给**对应的人**。所以，创建 [SendToUserRequest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-25/lab-websocket-25-01/src/main/java/cn/iocoder/springboot/lab25/springwebsocket/message/SendToUserRequest.java) 类，发送消息给一个用户的 Message 。

```java
// SendResponse.java

public class SendToUserRequest implements Message {

    public static final String TYPE = "SEND_TO_USER_REQUEST";

    /**
     * 消息编号
     */
    private String msgId;
    /**
     * 内容
     */
    private String content;
    
    // ... 省略 set/get 方法
     
}
```

- 相比 SendToOneRequest 来说，少一个 **`toUser` 字段**。因为，可以通过 WebSocket 连接，已经知道发送给谁了。

### 消息处理器

每个**客户端**发起的 Message 消息类型，会声明对应的 **MessageHandler 消息处理器**。

- 这个就类似在 SpringMVC 中，每个 **API 接口**对应一个 Controller 的 **Method 方法**。

所有的 MessageHandler 们，都放在 [`cn..springboot.lab25.springwebsocket.handler`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-25/lab-websocket-25-01/src/main/java/cn/iocoder/springboot/lab25/springwebsocket/handler) 包路径下。

#### MessageHandler

创建 [MessageHandler](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-25/lab-websocket-25-01/src/main/java/cn/iocoder/springboot/lab25/springwebsocket/handler/MessageHandler.java) 接口，消息处理器接口。

- 定义了泛型 `<T>` ，需要是 Message 的实现类。
- 定义的两个接口方法。

```java
// MessageHandler.java

public interface MessageHandler<T extends Message> {

    /**
     * 执行处理消息
     *
     * @param session 会话
     * @param message 消息
     */
    void execute(Session session, T message);

    /**
     * @return 消息类型，即每个 Message 实现类上的 TYPE 静态字段
     */
    String getType();

}
```

#### AuthMessageHandler

创建 [AuthMessageHandler](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-25/lab-websocket-25-01/src/main/java/cn/iocoder/springboot/lab25/springwebsocket/handler/AuthMessageHandler.java) 类，处理 AuthRequest 消息。

```java
// AuthMessageHandler.java

@Component
public class AuthMessageHandler implements MessageHandler<AuthRequest> {

    @Override
    public void execute(Session session, AuthRequest message) {
        // 如果未传递 accessToken 
        if (StringUtils.isEmpty(message.getAccessToken())) {
            WebSocketUtil.send(session, AuthResponse.TYPE,
                    new AuthResponse().setCode(1).setMessage("认证 accessToken 未传入"));
            return;
        }

        // 添加到 WebSocketUtil 中
        WebSocketUtil.addSession(session, message.getAccessToken()); // 考虑到代码简化，先直接使用 accessToken 作为 User

        // 判断是否认证成功。这里，假装直接成功
        WebSocketUtil.send(session, AuthResponse.TYPE, new AuthResponse().setCode(0));

        // 通知所有人，某个人加入了。这个是可选逻辑，仅仅是为了演示
        WebSocketUtil.broadcast(UserJoinNoticeRequest.TYPE,
                new UserJoinNoticeRequest().setNickname(message.getAccessToken()));
        // 考虑到代码简化，先直接使用 accessToken 作为 User
    }

    @Override
    public String getType() {
        return AuthRequest.TYPE;
    }
}
```

#### SendToOneRequest

创建 [SendToOneHandler](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-25/lab-websocket-25-01/src/main/java/cn/iocoder/springboot/lab25/springwebsocket/handler/SendToOneHandler.java) 类，处理 SendToOneRequest 消息。

```java
// SendToOneRequest.java

@Component
public class SendToOneHandler implements MessageHandler<SendToOneRequest> {

    @Override
    public void execute(Session session, SendToOneRequest message) {
        // 这里，假装直接成功
        SendResponse sendResponse = new SendResponse().setMsgId(message.getMsgId()).setCode(0);
        WebSocketUtil.send(session, SendResponse.TYPE, sendResponse);

        // 创建转发的消息
        SendToUserRequest sendToUserRequest = new SendToUserRequest().setMsgId(message.getMsgId())
                .setContent(message.getContent());
        // 广播发送
        WebSocketUtil.send(message.getToUser(), SendToUserRequest.TYPE, sendToUserRequest);
    }

    @Override
    public String getType() {
        return SendToOneRequest.TYPE;
    }

}
```

#### SendToAllHandler

创建 [SendToAllHandler](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-25/lab-websocket-25-01/src/main/java/cn/iocoder/springboot/lab25/springwebsocket/handler/SendToAllHandler.java) 类，处理 SendToAllRequest 消息。

```java
// SendToAllRequest.java

@Component
public class SendToAllHandler implements MessageHandler<SendToAllRequest> {

    @Override
    public void execute(Session session, SendToAllRequest message) {
        // 这里，假装直接成功
        SendResponse sendResponse = new SendResponse().setMsgId(message.getMsgId()).setCode(0);
        WebSocketUtil.send(session, SendResponse.TYPE, sendResponse);

        // 创建转发的消息
        SendToUserRequest sendToUserRequest = new SendToUserRequest().setMsgId(message.getMsgId())
                .setContent(message.getContent());
        // 广播发送
        WebSocketUtil.broadcast(SendToUserRequest.TYPE, sendToUserRequest);
    }

    @Override
    public String getType() {
        return SendToAllRequest.TYPE;
    }

}
```

### WebSocketUtil

创建 [WebSocketUtil](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-25/lab-websocket-25-01/src/main/java/cn/iocoder/springboot/lab25/springwebsocket/util/WebSocketUtil) 工具类，主要提供两方面的功能：

- **Session 会话的管理**
- 多种发送消息的方式

```java
// WebSocketUtil.java

@Slf4j
public class WebSocketUtil {

    // ========== 会话相关 ==========

    /**
     * Session 与用户的映射
     */
    private static final Map<Session, String> SESSION_USER_MAP = new ConcurrentHashMap<>();
    /**
     * 用户与 Session 的映射
     */
    private static final Map<String, Session> USER_SESSION_MAP = new ConcurrentHashMap<>();

    /**
     * 添加 Session 。在这个方法中，会添加用户和 Session 之间的映射
     *
     * @param session Session
     * @param user 用户
     */
    public static void addSession(Session session, String user) {
        // 更新 USER_SESSION_MAP
        USER_SESSION_MAP.put(user, session);
        // 更新 SESSION_USER_MAP
        SESSION_USER_MAP.put(session, user);
    }

    /**
     * 移除 Session
     *
     * @param session Session
     */
    public static void removeSession(Session session) {
        // 从 SESSION_USER_MAP 中移除
        String user = SESSION_USER_MAP.remove(session);
        // 从 USER_SESSION_MAP 中移除
        if (user != null && user.length() > 0) {
            USER_SESSION_MAP.remove(user);
        }
    }

    // ========== 消息相关 ==========

    /**
     * 广播发送消息给所有在线用户
     *
     * @param type 消息类型
     * @param message 消息体
     * @param <T> 消息类型
     */
    public static <T extends Message> void broadcast(String type, T message) {
        // 创建消息
        String messageText = buildTextMessage(type, message);
        // 遍历 SESSION_USER_MAP ，进行逐个发送
        for (Session session : SESSION_USER_MAP.keySet()) {
            sendTextMessage(session, messageText);
        }
    }

    /**
     * 发送消息给单个用户的 Session
     *
     * @param session Session
     * @param type 消息类型
     * @param message 消息体
     * @param <T> 消息类型
     */
    public static <T extends Message> void send(Session session, String type, T message) {
        // 创建消息
        String messageText = buildTextMessage(type, message);
        // 遍历给单个 Session ，进行逐个发送
        sendTextMessage(session, messageText);
    }

    /**
     * 发送消息给指定用户
     *
     * @param user 指定用户
     * @param type 消息类型
     * @param message 消息体
     * @param <T> 消息类型
     * @return 发送是否成功
     */
    public static <T extends Message> boolean send(String user, String type, T message) {
        // 获得用户对应的 Session
        Session session = USER_SESSION_MAP.get(user);
        if (session == null) {
            LOGGER.error("[send][user({}) 不存在对应的 session]", user);
            return false;
        }
        // 发送消息
        send(session, type, message);
        return true;
    }

    /**
     * 构建完整的消息
     *
     * @param type 消息类型
     * @param message 消息体
     * @param <T> 消息类型
     * @return 消息
     */
    private static <T extends Message> String buildTextMessage(String type, T message) {
        JSONObject messageObject = new JSONObject();
        messageObject.put("type", type);
        messageObject.put("body", message);
        return messageObject.toString();
    }

    /**
     * 真正发送消息
     *
     * @param session Session
     * @param messageText 消息
     */
    private static void sendTextMessage(Session session, String messageText) {
        if (session == null) {
            LOGGER.error("[sendTextMessage][session 为 null]");
            return;
        }
        RemoteEndpoint.Basic basic = session.getBasicRemote();
        if (basic == null) {
            LOGGER.error("[sendTextMessage][session 的  为 null]");
            return;
        }
        try {
            basic.sendText(messageText);
        } catch (IOException e) {
            LOGGER.error("[sendTextMessage][session({}) 发送消息{}) 发生异常",
                    session, messageText, e);
        }
    }

}
```

### 完善 WebsocketServerEndpoint

在本小节，会修改 [WebsocketServerEndpoint](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-25/lab-websocket-25-01/src/main/java/cn/iocoder/springboot/lab25/springwebsocket/config/WebSocketConfiguration.java) 的代码，完善其功能。

#### 初始化 MessageHandler 集合

实现 [InitializingBean](https://github.com/spring-projects/spring-framework/blob/master/spring-beans/src/main/java/org/springframework/beans/factory/InitializingBean.java) 接口，在 `#afterPropertiesSet()` 方法中，扫描所有 MessageHandler Bean ，添加到 MessageHandler 集合中。

- 通过这样的方式，可以**避免手动配置** MessageHandler 与消息类型的映射。

```java
// WebsocketServerEndpoint.java

    /**
     * 消息类型与 MessageHandler 的映射
     *
     * 注意，这里设置成静态变量。虽然说 WebsocketServerEndpoint 是单例，但是 Spring Boot 还是会为每个 WebSocket 创建一个 WebsocketServerEndpoint Bean 。
     */
    private static final Map<String, MessageHandler> HANDLERS = new HashMap<>();

    @Autowired
    private ApplicationContext applicationContext;

    @Override
    public void afterPropertiesSet() throws Exception {
        // 通过 ApplicationContext 获得所有 MessageHandler Bean
        applicationContext.getBeansOfType(MessageHandler.class).values() // 获得所有 MessageHandler Bean
                .forEach(messageHandler -> HANDLERS.put(messageHandler.getType(), messageHandler)); // 添加到 handlers 中
        logger.info("[afterPropertiesSet][消息处理器数量：{}]", HANDLERS.size());
    }
```

#### onOpen

重新实现 `#onOpen(Session session, EndpointConfig config)` 方法，实现连接时，使用 `accessToken` 参数进行用户认证。

- `<1>` 处，解析 `ws://` 地址上的 `accessToken` 的请求参。例如说：`ws://127.0.0.1:8080?accessToken=你好` 。
- `<2>` 处，创建 AuthRequest 消息类型，并设置 `accessToken` 属性。
- `<3>` 处，获得 AuthRequest 消息类型对应的 MessageHandler 消息处理器，然后调用 `MessageHandler#execute(session, message)` 方法，执行处理用户认证请求。

```java
// WebsocketServerEndpoint.java

@OnOpen
public void onOpen(Session session, EndpointConfig config) {
    logger.info("[onOpen][session({}) 接入]", session);
    // <1> 解析 accessToken
    List<String> accessTokenValues = session.getRequestParameterMap().get("accessToken");
    String accessToken = !CollectionUtils.isEmpty(accessTokenValues) ? accessTokenValues.get(0) : null;
    // <2> 创建 AuthRequest 消息类型
    AuthRequest authRequest = new AuthRequest().setAccessToken(accessToken);
    // <3> 获得消息处理器
    MessageHandler<AuthRequest> messageHandler = HANDLERS.get(AuthRequest.TYPE);
    if (messageHandler == null) {
        logger.error("[onOpen][认证消息类型，不存在消息处理器]");
        return;
    }
    messageHandler.execute(session, authRequest);
}
```

打开三个浏览器创建，分别设置服务地址如下：

- `ws://127.0.0.1:8080/?accessToken=芋艿`
- `ws://127.0.0.1:8080/?accessToken=番茄`
- `ws://127.0.0.1:8080/?accessToken=土豆`

然后，逐个点击「开启连接」按钮，进行 WebSocket 连接。最终效果如下图：

![建立连接](../assets/buildconnect02.png)

- 在红圈中，可以看到 AuthResponse 的消息。
- 在黄圈中，可以看到 UserJoinNoticeRequest 的消息。

#### onMessage

重新实现 `#onMessage(Session session, String message)` 方法，实现不同的消息，转发给不同的 MessageHandler 消息处理器。

```java
// WebsocketServerEndpoint.java

@OnMessage
public void onMessage(Session session, String message) {
    logger.info("[onOpen][session({}) 接收到一条消息({})]", session, message); // 生产环境下，请设置成 debug 级别
    try {
        // <1> 获得消息类型
        JSONObject jsonMessage = JSON.parseObject(message);
        String messageType = jsonMessage.getString("type");
        // <2> 获得消息处理器
        MessageHandler messageHandler = HANDLERS.get(messageType);
        if (messageHandler == null) {
            logger.error("[onMessage][消息类型({}) 不存在消息处理器]", messageType);
            return;
        }
        // <3> 解析消息
        Class<? extends Message> messageClass = this.getMessageClass(messageHandler);
        // <4> 处理消息
        Message messageObj = JSON.parseObject(jsonMessage.getString("body"), messageClass);
        messageHandler.execute(session, messageObj);
    } catch (Throwable throwable) {
        logger.info("[onMessage][session({}) message({}) 发生异常]", session, throwable);
    }
}
```

- `<1>` 处，获得消息类型，从 `"type"` 字段中。

- `<2>` 处，获得消息类型对应的 MessageHandler 消息处理器。

- `<3>` 处，调用 `#getMessageClass(MessageHandler handler)` 方法，通过 MessageHandler 中，通过解析其类上的泛型，获得消息类型对应的 Class 类。代码如下：

    ```java
    // WebsocketServerEndpoint.java
    
    private Class<? extends Message> getMessageClass(MessageHandler handler) {
        // 获得 Bean 对应的 Class 类名。因为有可能被 AOP 代理过。
        Class<?> targetClass = AopProxyUtils.ultimateTargetClass(handler);
        // 获得接口的 Type 数组
        Type[] interfaces = targetClass.getGenericInterfaces();
        Class<?> superclass = targetClass.getSuperclass();
        while ((Objects.isNull(interfaces) || 0 == interfaces.length) && Objects.nonNull(superclass)) { // 此处，是以父类的接口为准
            interfaces = superclass.getGenericInterfaces();
            superclass = targetClass.getSuperclass();
        }
        if (Objects.nonNull(interfaces)) {
            // 遍历 interfaces 数组
            for (Type type : interfaces) {
                // 要求 type 是泛型参数
                if (type instanceof ParameterizedType) {
                    ParameterizedType parameterizedType = (ParameterizedType) type;
                    // 要求是 MessageHandler 接口
                    if (Objects.equals(parameterizedType.getRawType(), MessageHandler.class)) {
                        Type[] actualTypeArguments = parameterizedType.getActualTypeArguments();
                        // 取首个元素
                        if (Objects.nonNull(actualTypeArguments) && actualTypeArguments.length > 0) {
                            return (Class<Message>) actualTypeArguments[0];
                        } else {
                            throw new IllegalStateException(String.format("类型(%s) 获得不到消息类型", handler));
                        }
                    }
                }
            }
        }
        throw new IllegalStateException(String.format("类型(%s) 获得不到消息类型", handler));
    }
    ```

    - 这是参考 [`rocketmq-spring`](https://github.com/apache/rocketmq-spring/) 项目的 [`DefaultRocketMQListenerContainer#getMessageType()`](https://github.com/apache/rocketmq-spring/blob/8b2426ea89d704d61c00497a1320b9b9ccd5a61e/rocketmq-spring-boot/src/main/java/org/apache/rocketmq/spring/support/DefaultRocketMQListenerContainer.java#L408-L435) 方法，进行略微修改。
- 如果对 Java 的泛型机制没有做过一点了解，可能略微有点硬核。可以先暂时跳过，知道意图即可。
  
- `<4>` 处，调用 `MessageHandler#execute(session, message)` 方法，执行处理请求。

- 另外，这里增加了 `try-catch` 代码，避免整个执行的过程中，发生异常。如果在 onMessage 事件的处理中，发生异常，该消息对应的 Session 会话会被**自动**关闭。显然，这个不符合我们的要求。例如说，在 MessageHandler 处理消息的过程中，发生一些异常是无法避免的。

继续基于上述创建的三个浏览器，先点击「清空消息」按钮，清空下消息，打扫下上次测试展示出来的接收得到的 Message 。当然，WebSocket 的连接，不需要去断开。

在第一个浏览器中，分别发送两种聊天消息：

- 一条 SendToOneRequest 私聊消息

    ```java
    {
        type: "SEND_TO_ONE_REQUEST",
        body: {
            toUser: "番茄",
            msgId: "eaef4a3c-35dd-46ee-b548-f9c4eb6396fe",
            content: "我是一条单聊消息"
        }
    }
    ```

- 一条 SendToAllHandler 群聊消息：

    ```java
    {
        type: "SEND_TO_ALL_REQUEST",
        body: {
            msgId: "838e97e1-6ae9-40f9-99c3-f7127ed64747",
            content: "我是一条群聊消息"
        }
    }
    ```


最终结果如下图：

![发送消息](../assets/buildconnect03.png)

- 在红圈中，可以看到一条 SendToUserRequest 的消息，仅有第二个浏览器（番茄）收到。
- 在黄圈中，可以看到三条 SendToUserRequest 的消息，所有浏览器都收到。

#### onClose

重新实现 `#onClose(Session session, CloseReason closeReason)` 方法，实现**移除**关闭的 Session 。

```java
// WebsocketServerEndpoint.java

@OnClose
public void onClose(Session session, CloseReason closeReason) {
    logger.info("[onClose][session({}) 连接关闭。关闭原因是({})}]", session, closeReason);
    WebSocketUtil.removeSession(session);
}
```

#### onError

`#onError(Session session, Throwable throwable)` 方法，保持不变。

```java
// WebsocketServerEndpoint.java

@OnError
public void onError(Session session, Throwable throwable) {
    logger.info("[onClose][session({}) 发生异常]", session, throwable);
}
```



## Spring WebSocket

### WebSocketUtil

因为 Tomcat WebSocket 使用的是 **[Session](https://github.com/eclipse-ee4j/websocket-api/blob/master/api/client/src/main/java/javax/websocket/Session.java) 作为会话**，而 Spring WebSocket 使用的是 **[WebSocketSession](https://github.com/spring-projects/spring-framework/blob/master/spring-websocket/src/main/java/org/springframework/web/socket/WebSocketSession.java) 作为会话**，导致需要**略微**修改下 WebSocketUtil 工具类。

改动非常略微，点击 [WebSocketUtil](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-25/lab-websocket-25-02/src/main/java/cn/iocoder/springboot/lab25/springwebsocket/util/WebSocketUtil.java) 查看下，秒懂的噢。 主要有两点：

1. 将所有使用 Session 类的地方，调整成 WebSocketSession 类。
2. 将发送消息，从 Session 修改成 WebSocketSession 。

### 消息处理器

将 [`cn.iocoder.springboot.lab25.springwebsocket.handler`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-25/lab-websocket-25-02/src/main/java/cn/iocoder/springboot/lab25/springwebsocket/handler) 包路径下的消息处理器们，使用到 Session 类的地方，调整成 WebSocketSession 类。

### WebSocketSession

```java
@Slf4j
@RequiredArgsConstructor
public abstract class AbstractWebSocketMessageSender implements WebSocketMessageSender {

    private final WebSocketSessionManager sessionManager;

	/**
     * 发送消息
     *
     * @param sessionId Session 编号
     * @param userType 用户类型
     * @param userId 用户编号
     * @param messageType 消息类型
     * @param messageContent 消息内容
     */
    public void send(String sessionId, Integer userType, Long userId, String messageType, String messageContent) {
        // 1. 获得 Session 列表
        ...
        // 2. 执行发送
        doSend(sessions, messageType, messageContent);
    }

    /**
     * 发送消息的具体实现
     *
     * @param sessions Session 列表
     * @param messageType 消息类型
     * @param messageContent 消息内容
     */
    public void doSend(Collection<WebSocketSession> sessions, String messageType, String messageContent) {
        JsonWebSocketMessage message = new JsonWebSocketMessage().setType(messageType).setContent(messageContent);
        String payload = JsonUtils.toJsonString(message); // 关键，使用 JSON 序列化
        sessions.forEach(session -> {
            // 1. 各种校验，保证 Session 可以被发送
            if (session == null) {
                log.error("[doSend][session 为空, message({})]", message);
                return;
            }
            if (!session.isOpen()) {
                log.error("[doSend][session({}) 已关闭, message({})]", session.getId(), message);
                return;
            }
            // 2. 执行发送
            try {
                session.sendMessage(new TextMessage(payload));
                log.info("[doSend][session({}) 发送消息成功，message({})]", session.getId(), message);
            } catch (IOException ex) {
                log.error("[doSend][session({}) 发送消息失败，message({})]", session.getId(), message, ex);
            }
        });
    }
}
```

#### WebSocketSessionManager



### ~~WebSocketMessage~~

### WebSocketMessageListener

- DemoReceiveMessage 示例：server -> client 同步消息
- DemoSendMessage 示例：client -> server 发送消息

```java

/**
 * WebSocket 示例：单发消息
 */
@Component
public class DemoWebSocketMessageListener implements WebSocketMessageListener<DemoSendMessage> {

    @SuppressWarnings("SpringJavaAutowiredFieldsWarningInspection")
    @Autowired(required = false) // 由于 yudao.websocket.enable 配置项，可以关闭 WebSocket 的功能，所以这里只能不强制注入
    private WebSocketMessageSender webSocketMessageSender;

    @Override
    public void onMessage(WebSocketSession session, DemoSendMessage message) {
        Long fromUserId = WebSocketFrameworkUtils.getLoginUserId(session);
        // 情况一：单发
        if (message.getToUserId() != null) {
            DemoReceiveMessage toMessage = new DemoReceiveMessage().setFromUserId(fromUserId)
                    .setText(message.getText()).setSingle(true);
            webSocketMessageSender.sendObject(UserTypeEnum.ADMIN.getValue(), message.getToUserId(), // 给指定用户
                    "demo-message-receive", toMessage);
            return;
        }
        // 情况二：群发
        DemoReceiveMessage toMessage = new DemoReceiveMessage().setFromUserId(fromUserId)
                .setText(message.getText()).setSingle(false);
        webSocketMessageSender.sendObject(UserTypeEnum.ADMIN.getValue(), // 给所有用户
                "demo-message-receive", toMessage);
    }

    @Override
    public String getType() {
        return "demo-message-send";
    }

}
```



### WebSocketMessageSender

RPC 服务调用的入口。

#### AbstractWebSocketMessageSender

```java
@Slf4j
@RequiredArgsConstructor
public abstract class AbstractWebSocketMessageSender implements WebSocketMessageSender {

    private final WebSocketSessionManager sessionManager;

```

#### RocketMQWebSocketMessageSender

RocketMQ 实现

```java
/**
 * 基于 RocketMQ 的 {@link WebSocketMessageSender} 实现类
 */
@Slf4j
public class RocketMQWebSocketMessageSender extends AbstractWebSocketMessageSender {

    private final RocketMQTemplate rocketMQTemplate;

    private final String topic;
    ....
}
```

### WebSocketShakeInterceptor

在 [`cn.iocoder.springboot.lab25.springwebsocket.websocket`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-25/lab-websocket-25-02/src/main/java/cn/iocoder/springboot/lab25/springwebsocket/websocket/) 包路径下，创建 [DemoWebSocketShakeInterceptor](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-25/lab-websocket-25-02/src/main/java/cn/iocoder/springboot/lab25/springwebsocket/websocket/DemoWebSocketShakeInterceptor.java) 拦截器。

因为 WebSocketSession 无法获得 ws 地址上的请求参数，所以只好通过该拦截器，获得 `accessToken` 请求参数，设置到 `attributes` 中。

### DemoWebSocketHandler

在 [`cn.iocoder.springboot.lab25.springwebsocket.websocket`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-25/lab-websocket-25-02/src/main/java/cn/iocoder/springboot/lab25/springwebsocket/websocket/) 包路径下，创建 [DemoWebSocketHandler](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-25/lab-websocket-25-02/src/main/java/cn/iocoder/springboot/lab25/springwebsocket/websocket/DemoWebSocketHandler.java) 处理器。

- 在类上，添加 **`@EnableWebSocket` 注解**，开启 Spring WebSocket 功能。
- 实现 **WebSocketConfigurer 接口**，自定义 WebSocket 的配置。
    - 具体的，可以看看 `#registerWebSocketHandlers(registry)` 方法，配置 **WebSocket 处理器、拦截器**，以及允许跨域。

### WebSocketConfiguration



## IM 消息送达保证机制

在上述的提供的 Tomcat WebSocket 和 Spring WebSocket 示例中，相当于在 WebSocket 实现了自定义的子协议，就是基于 `type` + `body` 的消息结构。

> [《IM 消息送达保证机制实现》](http://www.52im.net/thread-294-1-1.html) 文章。

实际场景下，在使用 WebSocket 还是原生 Socket 也好，都需要考虑，**如何保证消息一定送达给用户**？

- 如果用户不处于在线的时候，消息持久化到 MySQL、MongoDB 等数据库中。这个是正确，且是必须要做的。

我们在一起考虑下边界场景，客户端网络环境较差，特别是在移动端场景下，出现网络**闪断**，可能会出现连接实际已经断开，而服务端以为客户端处于在线的情况。此时，服务端会将消息发给客户端，那么消息实际就发送到“空气”中，产生丢失的情况。

要解决这种情况下的问题，需要引入客户端的 **ACK 消息机制**。目前，主流的有两种做法。

#### ~~基于**每条消息编号 ACK**~~

第一种，基于**每一条消息编号 ACK** 。整体流程如下：

1. 无论客户端是否在线，服务端都先把接收到的消息持久化到数据库中。如果客户端此时在线，服务端将**完整消息**推送给客户端。
2. 客户端在接收到消息之后，发送 ACK 消息编号给服务端，告知已经收到该消息。服务端在收到 ACK 消息编号的时候，标记该消息已经发送成功。
3. 服务端定时轮询，在线的客户端，是否有超过 N 秒未 ACK 的消息。如果有，则重新发送消息给对应的客户端。

这种方案，因为客户端逐条 ACK 消息编号，所以会导致客户端和服务端交互次数过多。当然，客户端可以异步批量 ACK 多条消息，从而减少次数。

不过因为服务端仍然需要**定时轮询**，也会导致服务端压力较大。所以，这种方案基本已经不采用了。

#### 基于**滑动窗口 ACK**

第二种，基于**滑动窗口 ACK** 。整体流程如下：

1. 无论客户端是否在线，服务端都先把接收到的消息持久化到数据库中。如果客户端此时在线，服务端将**消息编号**推送给客户端。
2. 客户端在接收到**消息编号**之后，和本地的消息编号进行比对。如果比本地的小，说明该消息已经收到，忽略不处理；如果比本地的大，使用**本地的**消息编号，向服务端拉取**大于**本地的消息编号的消息列表，即增量消息列表。拉取完成后，更新消息列表中最大的消息编号为**新的本地的**消息编号。
3. 服务端在收到客户端拉取增量的消息列表时，将请求的编号记录到数据库中，用于知道客户端此时本地的最新消息编号。
4. 考虑到服务端将**消息编号**推送给客户端，也会存在丢失的情况，所以客户端会每 N 秒定时向服务端拉取**大于**本地的消息编号的消息列表。

这种方式，在业务被称为**推拉结合**的方案，在**分布式消息队列、配置中心、注册中心**实现实时的数据同步，经常被采用。

并且，采用这种方案的情况下，客户端和服务端不一定需要使用**长连接**，也可以使用**长轮询**所替代。客户端发送带有消息版本号的 HTTP 请求到服务端。

- 如果服务端**已有**比客户端新的消息编号，则直接返回增量的消息列表。
- 如果服务端**没有**比客户端新的消息编号，则 HOLD 住请求，直到有新的消息列表可以返回，或者 HTTP 请求超时。
- 客户端在收到 HTTP 请求超时时，立即又重新发起带有消息版本号的 HTTP 请求到服务端。如此反复循环，通过消息编号作为**增量标识**，达到实时获取消息的目的。

