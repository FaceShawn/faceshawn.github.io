---
title: Spring Cloud Stream RocketMQ 组件样例
categories:
  - 消息队列
tags:
  - 微服务
  - 分布式
  - 高并发
  - 高性能
  - 高可用
  - Spring-Cloud-Alibaba
  - 消息队列
  - RocketMQ
  - Spring-Cloud-Stream
  - Spring-Cloud-Stream-RocketMQ
  - 样例
location:
  - 黄金时代
abbrlink: 'rocket_mq_stream_example'
permalink: 'rocket_mq_stream_example'
date: 2025-08-23 16:41:00
updated: 2025-12-23 16:41:00
---

> 摘要：三种消息发送方式，两种消息消费方式，三种高级消息类型。

<!-- more -->

---

## 目录

[TOC]

## SpringBoot、Spring Cloud Stream 整合 RocketMQ

#### 区别

Spring Boot 和 Spring Cloud Stream 整合RocketMQ是两种不同的方法，它们在使用RocketMQ消息队列时有一些**关键区别**：

- **Spring Boot** 整合RocketMQ：这种方式对RocketMQ的使用**更为直接**，适用于需要更多**自定义控制**的场景，例如特定的**消息处理逻辑、异常处理**等。

    1. 使用rocketmq-client依赖，使用rocketmq官方**原生方式**操作mq。
        - 通常是直接集成**RocketMQ客户端SDK**，用于在Spring Boot应用中使用RocketMQ。
    2. RocketMQ的维护和配置需要手动完成。需要**配置**RocketMQ的**生产者和消费者**，设置主题（Topic）和标签（Tag），并编写代码来发送和接收消息。
    
- **Spring Cloud Stream** 整合RocketMQ：Spring Cloud Stream是Spring Cloud项目的一部分，提供了一种更抽象的消息处理方式（更高级的抽象），以简化消息系统的使用。使开发者可以**更关注业务逻辑**，而不必处理底层消息队列的细节。

    2. 使用 **RocketMQTemplate**，只需要配置即可使用，比较简单。

    3. 整合RocketMQ**抽象了**消息生产者和消费者的**细节**，通过**Binder**来管理消息队列的连接和配置。

    4. 可以通过声明式的方式定义输入（input）和输出（output），并且不需要关心底层消息通信的细节，例如主题和标签。

    4. 配置和维护方面，Spring Cloud Stream通过Binder层来处理。

         


#### 如何选择

综上所述，总之，选择哪种方式更合适取决于项目需求、团队的技术栈和经验。

- Spring Boot整合RocketMQ需要**显式配置**RocketMQ的细节，更适合需要更多**自定义控制**的场景，或者已经有丰富的RocketMQ经验。
- 而Spring Cloud Stream整合RocketMQ提供了**更高级的抽象**，简化了消息队列的使用，适用于关注业务逻辑而不想处理底层（消息队列）细节的场景。希望更快速地集成和更高级的抽象。

选择哪种方式取决于项目的需求、复杂度、开发团队的偏好、技术栈、经验。

- 选择Spring Boot整合RocketMQ的场景：
    1. 需要更多**自定义控制**：如果需要完全掌控RocketMQ的配置和细节，或者应用需要与RocketMQ进行更复杂的交互，（例如自定义消息处理逻辑、**事务性消息**等）。
    2. **现有项目集成**：如果已经有一个使用Spring Boot的项目，并且想要将RocketMQ集成到现有项目中，Spring Boot整合更直接，因为它允许**轻松地添加**RocketMQ依赖和配置。
    3. 更大的**团队专业知识**：如果团队在RocketMQ的使用和管理方面拥有深厚的专业知识，并且愿意自己配置和维护RocketMQ连接和配置，那么Spring Boot整合可以提供更多的**自定义选项**。

- 选择Spring Cloud Stream整合RocketMQ的场景：
    1. **快速集成**：如果需要快速集成消息队列、并且不希望处理繁琐的配置细节，Spring Cloud Stream提供了更简化的**声明式配置**，使集成变得更容易。
    2. **简化的抽象**：如果更关注业务逻辑、而不愿意处理消息队列的底层细节，Spring Cloud Stream提供了**更高级的抽象**，使只需关注消息通道和处理逻辑。
    3. **微服务架构**：如果应用是基于**微服务架构**构建的，Spring Cloud Stream的特性使得在不同微服务之间传递消息变得更加容易。
    4. **事件驱动架构**：如果应用采用事件驱动架构，Spring Cloud Stream提供了更便捷的方式来定义和处理事件消息。



实例**场景**：假设正在构建一个电子商务平台，需要处理订单的**消息通知**。

#### Spring Boot 整合步骤

1. **配置**：在Spring Boot应用中，需要配置RocketMQ的连接信息、生产者和消费者，并定义主题和标签，如下：
2. **生产者**：编写订单服务，使用RocketMQ的生产者将订单通知**发送**到"order-topic"主题。
3. **消费者**：编写订单处理服务，使用RocketMQ的消费者**监听**"order-topic"主题，并在收到消息时，执行相应的处理逻辑。
4. **维护**：配置RocketMQ的连接、主题和标签，以及处理消息消费的错误和异常。

```yaml
# application.properties
rocketmq.producer.group=my-producer-group
rocketmq.consumer.group=my-consumer-group
rocketmq.namesrv-addr=localhost:9876

# RocketMQ Producer配置
rocketmq.producer.topic=order-topic
rocketmq.producer.tag=order-tag

# RocketMQ Consumer配置
rocketmq.consumer.topic=order-topic
rocketmq.consumer.tag=order-tag
```

#### Spring Cloud Stream 整合步骤

1. **依赖**：添加Spring Cloud Stream和RocketMQ的依赖。

    ```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-stream-rocketmq</artifactId>
    </dependency>
    ```

2. **声明式配置**：在应用的配置文件中，定义输入（input）和输出（output）通道：

    ```yaml
    spring:
      cloud:
        stream:
          bindings:
            order-out:
              destination: order-topic
            order-in:
              destination: order-topic
    ```

3. **生产者**：在订单服务中，只需**注入`MessageChannel`**，然后将订单通知发送到`order-out`通道。

    ```java
    @Autowired
    @Output("order-out")
    private MessageChannel output;
    
    public void sendOrderNotification(Order order) {
        output.send(MessageBuilder.withPayload(order).build());
    }
    ```

4. **消费者**：在订单处理服务中，只需注入`MessageChannel`，然后**监听**`order-in`通道，当有新订单消息到达时，执行相应的处理逻辑。

    ```java
    @Autowired
    @Input("order-in")
    private SubscribableChannel input;
    
    @StreamListener("order-in")
    public void handleOrderMessage(Order order) {
        // 处理订单通知逻辑
    }
    ```

5. 维护：Spring Cloud Stream通过**Binder层**来管理RocketMQ的连接和配置，简化了配置和维护工作。

## Spring Cloud Stream

[Spring Cloud Stream](https://github.com/spring-cloud/spring-cloud-stream) 是一个用于构建基于**消息**的微服务应用框架，使用 [Spring Integration](https://www.oschina.net/p/spring+integration) 与 **Broker** 进行连接。

- 微服务中会经常使用**消息中间件**，用于在服务与服务之间传递消息，例如RabbitMQ、Kafka和RocketMQ。
- 提供了消息中间件的**统一抽象**，推出了 `publish-subscribe、consumer groups、partition` 这些统一的概念。
- 因为对消息中间件的进一步封装，可以做到（开发人员）代码层面对消息中间件的**无感知**，甚至于**动态的切换**中间件(rabbitmq切换为rocketmq或者kafka)，使得微服务开发的**高度解耦**，服务可以关注更多自己的业务流程。

Spring Cloud Stream 内部有两个概念：**Binder** 和 **Binding**。

#### Binder

① **[Binder](https://github.com/spring-cloud/spring-cloud-stream/blob/master/spring-cloud-stream/src/main/java/org/springframework/cloud/stream/binder/Binder.java)**：跟**消息中间件**集成的组件，用来**创建对应的 Binding**。各消息中间件都有自己的 Binder 具体实现。

1. Kafka 实现了 [KafkaMessageChannelBinder](https://github.com/spring-cloud/spring-cloud-stream-binder-kafka/blob/master/spring-cloud-stream-binder-kafka/src/main/java/org/springframework/cloud/stream/binder/kafka/KafkaMessageChannelBinder.java)；
2. RabbitMQ 实现了 [RabbitMessageChannelBinder](https://github.com/spring-cloud/spring-cloud-stream-binder-rabbit/blob/master/spring-cloud-stream-binder-rabbit/src/main/java/org/springframework/cloud/stream/binder/rabbit/RabbitMessageChannelBinder.java)；
3. RocketMQ 实现了 [RocketMQMessageChannelBinder](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-stream-binder-rocketmq/src/main/java/com/alibaba/cloud/stream/binder/rocketmq/RocketMQMessageChannelBinder.java)；

```java
public interface Binder<T, 
    C extends ConsumerProperties, // 消费者配置
    P extends ProducerProperties> { // 生产者配置
    
    // 创建消费者的 Binding
    Binding<T> bindConsumer(String name, String group, T inboundBindTarget, C consumerProperties);

    // 创建生产者的 Binding
    Binding<T> bindProducer(String name, T outboundBindTarget, P producerProperties);
}
```

#### Binding

② **[Binding](https://github.com/spring-cloud/spring-cloud-stream/blob/master/spring-cloud-stream/src/main/java/org/springframework/cloud/stream/binder/Binding.java)**：在**消息中间件**与**应用程序**提供的 Provider 和 Consumer 之间提供了一个桥梁。

- 实现了开发者只需使用应用程序的 Provider 或 Consumer 生产或消费数据即可，屏蔽了开发者与底层消息中间件的接触。

- 包括 Input Binding 和 Output Binding。

#### input

input：应用程序通过input（相当于**消费者consumer**）与Spring Cloud Stream 中Binder交互，而Binder负责与消息中间件交互，因此，我们只需关注如何与Binder交互即可，而无需关注与具体消息中间件的交互。

#### output

output：（相当于生产者producer）与Spring Cloud Stream中Binder交互。

| **组成**        | **说明**                                                     |
| --------------- | ------------------------------------------------------------ |
| Binder          | Binder是应用与消息中间件之间的封装，目前实现了Kafka和RabbitMQ的Binder，通过Binder可以很方便的连接中间件，可以动态的改变消息类型(对应于Kafka的topic，RabbitMQ的exchange)，这些都可以通过配置文件来实现 |
| @Input          | 该注解标识输入通道，通过该输入通道接收消息进入应用程序       |
| @Output         | 该注解标识输出通道，发布的消息将通过该通道离开应用程序       |
| @StreamListener | 监听队列，用于消费者的队列的消息接收                         |
| @EnableBinding  | 将信道channel和exchange、topic绑定在一起                     |

最终整体交互如下图所示：

<img src="../assets/RocketMQ_stream_01.png" alt="Spring Cloud Stream Application" style="zoom: 45%;" />![img](../assets/2443180-20220507181727153-308979953.png)

### Spring Cloud Stream RocketMQ

> [Spring Cloud Alibaba](https://spring.io/projects/spring-cloud-alibaba) 提供的 [Spring Cloud Stream RocketMQ](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-stream-binder-rocketmq/) 组件，基于 [Spring Cloud Stream](https://github.com/spring-cloud/spring-cloud-stream) 的编程模型，接入 RocketMQ 作为**消息中间件**，实现**消息驱动**的微服务。

Spring Cloud Alibaba RocketMQ Binder 使用步骤如下：

#### RocketMQ Binder 依赖

如果要在项目中引入 RocketMQ Binder，需要引入如下 maven 依赖：

```xml
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-stream-binder-rocketmq</artifactId>
</dependency>
```

或者可以使用 Spring Cloud Stream RocketMQ Starter：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rocketmq</artifactId>
</dependency>
```

####  RocketMQ Binder 实现

RocketMQ Binder 的实现依赖于 **[RocketMQ-Spring](https://github.com/apache/rocketmq-spring) 框架**。

RocketMQ-Spring 框架是 RocketMQ **与 Spring Boot 的整合**，主要提供了 3 个特性：

1. 使用 `RocketMQTemplate` 用来**统一发送消息**，包括**同步、异步发送消息和事务消息**。
2. `@RocketMQTransactionListener` 注解用来处理**事务消息的监听和回查**。
3. `@RocketMQMessageListener` 注解用来**消费消息**。

RocketMQ Binder 的**核心类 `RocketMQMessageChannelBinder `**实现了 **Spring Cloud Stream 规范**，内部构建会 `RocketMQInboundChannelAdapter` 和 `RocketMQMessageHandler`。

- `RocketMQMessageHandler` 会基于 Binding 配置构造 `RocketMQTemplate`，`RocketMQTemplate` 内部会把 `spring-messaging` 模块内 `org.springframework.messaging.Message` 消息类转换成 RocketMQ 的消息类 `org.apache.rocketmq.common.message.Message`，然后发送出去。

- `RocketMQInboundChannelAdapter` 也会基于 Binding 配置构造 `RocketMQListenerBindingContainer`，`RocketMQListenerBindingContainer` 内部会启动 RocketMQ `Consumer` 接收消息。

注：  在使用 RocketMQ Binder 的同时也可以配置 rocketmq.**，用于触发 RocketMQ Spring 相关的 AutoConfiguration。

目前 Binder 支持在 `Header` 中设置相关的 key 来进行 RocketMQ Message 消息的特性设置。

- 比如 `TAGS`、`DELAY`、`TRANSACTIONAL_ARG`、`KEYS`、`WAIT_STORE_MSG_OK`、`FLAG` 表示 RocketMQ 消息对应的标签。

```java
MessageBuilder builder = MessageBuilder.withPayload(msg)
    .setHeader(RocketMQHeaders.TAGS, "binder")
    .setHeader(RocketMQHeaders.KEYS, "my-key")
    .setHeader("DELAY", "1");
Message message = builder.build();
output().send(message);
```

### 常用API

#### MessageBuilder

> org.springframework.integration.support.MessageBuilder

在Spring Integration中，一般情况下，不会直接操作这些类来创建Message。通常使用org.springframework.integration.support.MessageBuilder来生成一些message的对象。

我们可以直接使用要传输的数据创建Message对象如下，使用一个字符串“Hello，world！”来创建一个Message：

![img](https://upload-images.jianshu.io/upload_images/5891170-2d2fff3bbeb465ba.png?imageMogr2/auto-orient/strip|imageView2/2/w/588/format/webp)

MessageBuilder提供了一些方法来设置header如：

[![img](https://upload-images.jianshu.io/upload_images/5891170-6bd0c6d147f73ad6.png?imageMogr2/auto-orient/strip|imageView2/2/w/604/format/webp)](https://upload-images.jianshu.io/upload_images/5891170-6bd0c6d147f73ad6.png?imageMogr2/auto-orient/strip|imageView2/2/w/604/format/webp)

另外，MessageBuilder也可以直接通过一个已存在的Message对象来创建新的Message对象，设置Header，如：

[![img](https://upload-images.jianshu.io/upload_images/5891170-a9e64a4a9e26ce4c.png?imageMogr2/auto-orient/strip|imageView2/2/w/656/format/webp)](https://upload-images.jianshu.io/upload_images/5891170-a9e64a4a9e26ce4c.png?imageMogr2/auto-orient/strip|imageView2/2/w/656/format/webp)

### 常用注解

#### **@StreamListener**

- condition 参数作为条件设定，其支持 SpEL 表达式，通过条件约束即可满足我们的需求。例如：

```java
/*********************************实现多个监听者应用******************************/
@StreamListener(value = MySink.USER_CHANNEL, condition = "headers['flag']=='aa'")
public void userReceiveByHeader1(@Payload User user) {
    LOGGER.info("Received from {} channel : {} with head (flag:aa)", MySink.USER_CHANNEL, user.getUsername());
}
 
@StreamListener(value = MySink.USER_CHANNEL, condition = "headers['flag']=='bb'")
public void userReceiveByHeader2(@Payload User user) {
    LOGGER.info("Received from {} channel : {} with head (flag:bb)", MySink.USER_CHANNEL, user.getUsername());
}
```

#### @RocketMQMessageListener

> 使用了 `@RocketMQMessageListener` 注解，设置每个 RocketMQ 消费者 Consumer 的消息监听器的配置。

创建 [Demo01Consumer](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-31/lab-31-rocketmq-demo/src/main/java/cn/iocoder/springboot/lab31/rocketmqdemo/consumer/Demo01Consumer.java) 类，实现 Rocket-Spring 定义的 [RocketMQListener](https://github.com/apache/rocketmq-spring/blob/3f89080df8f797cad0d1f9eb8badb5050b09a553/rocketmq-spring-boot/src/main/java/org/apache/rocketmq/spring/core/RocketMQListener.java) 接口，消费消息。

- 在类上，添加了`@RocketMQMessageListener`注解，声明消费的 Topic 是"DEMO_01"，消费者分组是"demo01-consumer-group-DEMO_01"。一般情况下，建议一个消费者分组，仅消费一个 Topic 。这样做会有两个好处：
    - 每个消费者分组职责单一，只消费一个 Topic 。
    - 每个消费者分组是独占一个线程池，这样能够保证多个 Topic 隔离在不同线程池，保证隔离性，从而避免一个 Topic 消费很慢，影响到另外的 Topic 的消费。
- 实现 RocketMQListener 接口，在 `T` 泛型里，设置消费的消息对应的类。此处，我们就设置了 Demo01Message 类。

```java
// Demo01Consumer.java

@Component
@RocketMQMessageListener(
        topic = Demo01Message.TOPIC,
        consumerGroup = "demo01-consumer-group-" + Demo01Message.TOPIC
)
public class Demo01Consumer implements RocketMQListener<Demo01Message> {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Override
    public void onMessage(Demo01Message message) {
        logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
    }

}
```



#### **@Payload**

内容为消息体

#### **@Headers**

获取所有的Header头信息，是一个Map

#### **@Header**

获取指定name的头信息。

#### **@SendTo**

```java
@Component
public class ReceiveListener {

    @StreamListener("MQRece")
    @SendTo("back-push")
    public byte[] receive(byte[] bytes){
        log.info("接受消息："+new String(bytes));
        return "ok".getBytes();
    }

}
```

配置：

```java
spring.cloud.stream.bindings.back-push.destination=back-topic
spring.cloud.stream.bindings.back-push.group=back-group
```

### Stream & RocketMQ常用配置

Spring Cloud Stream 和RocketMQ都提供了很多配置项，接下来我们就来总结下。

> 特别注意：
>
> 下面所描述的配置项或属性仅仅是基于spring-cloud-starter-stream-rocketmq 2.2.7.RELEASE 进行总结，跟实际的开源的rocketmq版本里面的属性还是有出入的，而且不同的版本，可能也会不一样，使用的时候请注意。

```yaml
spring:
  cloud:
    # Spring Cloud Stream 配置项，对应 BindingServiceProperties 类
    stream:
      # Binding 配置项，对应 BindingProperties Map
      bindings:
        output1: #provider 生产者
          destination: test-topic # 目的地。这里使用 RocketMQ Topic
          content-type: application/json # 内容格式。这里使用 JSON
          group: test-group # 生产者分组
          # producer生产者配置, 对应 ProducerProperties 类
          producer:
            partitionCount: 2 #消息生产需要广播的消费者数量。即消息分区的数量
            partitionKeyExpression: payload.id #分区 key 表达式。该表达式基于 Spring EL，从消息中获得分区 key
        input1: #consumer 消费者
          destination: test-topic # 目的地。这里使用 RocketMQ Topic
          content-type: application/json # 内容格式。这里使用 JSON
          group: test-group # 消费者分组
          # consumer消费者配置, 对应 ConsumerProperties 类
          consumer:
            partitioned: true #开启消费者分区功能

      # Spring Cloud Stream RocketMQ 配置项
      rocketmq:
        # RocketMQ Binder 配置项，对应 RocketMQBinderConfigurationProperties 类
        binder:
          name-server: 101.133.227.13:9876 # RocketMQ Namesrv 地址
          group: rocketmq-group # 分组，必须的，全局唯一，具体用户暂不明
        # RocketMQ 自定义 Binding 配置项，对应 RocketMQExtendedBindingProperties Map
        bindings:
          output1:
            # RocketMQ Producer 配置项，对应 RocketMQProducerProperties 类
            producer:
              sendMsgTimeout: 3000 #发送消息超时时间
              producerType: Trans #消息类型，分为普通消息和事务消息，由 RocketMQProducerProperties.ProducerType 定义
              sendType: Sync #消息发送的类型，默认是同步的，分为同步、异步、单向，由 RocketMQProducerProperties.SendType 定义
          input1:
            # RocketMQ Consumer 配置项，对应 RocketMQConsumerProperties 类
            consumer:
              enabled: true # 是否开启消费，默认为 true
              subscription: myTag||look # 基于 Tag 订阅，多个 Tag 使用 || 分隔，默认为空
              messageModel: CLUSTERING # 消息消费模式，分为集群消费和广播消费，默认是集群消费，由 MessageModel 定义
              push:
                orderly: true # 是否顺序消费，默认为 false 并发消费。
      instance-count: 2 #当前消费者的总实例个数，即应用程序部署的实例数量。
      instance-index: 0 #0、1、2......当前实例的索引号，从 0 开始，最大为 -1 。用于消息生产的时候锁定该实例，不同的实例（应用）索引号不同
```

## 配置项信息

> RocketMQ 

Spring Cloud Alibaba Stream RocketMQ 提供的配置项挺多的。

#### **Binder Properties**

以 `spring.cloud.stream.rocketmq.binder` 为前缀。

| 配置项                   | 说明                                             | 默认值                |
| :----------------------- | :----------------------------------------------- | :-------------------- |
| `name-server`            | RocketMQ NameServer 地址                         | `127.0.0.1:9876`      |
| `access-key`             | 阿里云账号 AccessKey                             |                       |
| `secret-key`             | 阿里云账号 SecretKey                             |                       |
| `enable-msg-trace`       | 是否为 Producer 和 Consumer 开启**消息轨迹**功能 | `true`                |
| `customized-trace-topic` | 消息轨迹开启后存储的 Topic 名称                  | `RMQ_SYS_TRACE_TOPIC` |

#### **Consumer Properties**

以 `spring.cloud.stream.rocketmq.bindings.<channelName>.consumer.` 为前缀。

| 配置项                          | 说明                                                         | 默认值  |
| :------------------------------ | :----------------------------------------------------------- | :------ |
| `enable`                        | 是否启用 Consumer                                            | `true`  |
| `tags`                          | Consumer 基于 **TAGS 订阅**，多个 tag 以 `||` 分割           |         |
| `sql`                           | Consumer 基于 SQL 订阅                                       |         |
| `broadcasting`                  | 是Consumer 是否是**广播消费**模式。如果想让所有的订阅者都能接收到消息，可以使用广播模式 | `false` |
| `orderly`                       | Consumer 是否**同步消费**消息模式                            | `false` |
| `delayLevelWhenNextConsume`     | **异步消费**消息模式下**消费失败重试策略**：-1, 不重复，直接放入死信队列；0, Broker 控制重试策略；>0, Client 控制重试策略 | 0       |
| `suspendCurrentQueueTimeMillis` | 同步消费消息模式下消费失败后**再次消费**的时间间隔           | 1000    |

#### **Provider Properties**

| 配置项                          | 说明                                               | 默认值  |
| :------------------------------ | :------------------------------------------------- | :------ |
| `enable`                        | 是否启用 Producer                                  | `true`  |
| `group`                         | Producer **分组**                                  |         |
| `maxMessageSize`                | 消息发送的最大字节数                               | 8249344 |
| `transactional`                 | 是否发送**事务消息**                               | `false` |
| `sync`                          | 是否使用**同步方式发送**消息                       | `false` |
| `vipChannelEnabled`             | 是否在 Vip Channel 上发送消息                      | `true`  |
| `sendMessageTimeout`            | 发送消息的超时时间(毫秒)                           | 3000    |
| `compressMessageBodyThreshold`  | 消息体压缩阀值(当消息体超过 4k 的时候会被压缩)     | 4096    |
| `retryTimesWhenSendFailed`      | 在**同步发送**消息的模式下，消息发送失败的重试次数 | 2       |
| `retryTimesWhenSendAsyncFailed` | 在**异步发送**消息的模式下，消息发送失败的重试次数 | 2       |
| `retryNextServer`               | 消息发送失败的情况下是否重试其它的 Broker          | `false` |

## 三种消息发送方式

消息队列RocketMQ Producer 提供三种方式来发送**普通消息**：同步（Sync）发送、异步（Async）发送和单向（Oneway）发送。

1. 发送同步消息：`sync`
    1. `retryTimesWhenSendFailed` ：在**同步发送**消息的模式下，消息发送失败的重试次数  2  
    2. `retryTimesWhenSendAsyncFailed` ：在**异步发送**消息的模式下，消息发送失败的重试次数  2
2. 发送异步消息
3. oneway 单向发送消息

### 同步发送

同步发送是指消息发送方发出一条消息后，会在收到服务端同步响应之后才发下一条消息的通讯方式。

- 一般业务场景下，使用同步发送消息较多。
- 此种方式应用场景非常广泛，例如重要通知邮件、**报名短信通知**、营销短信系统等。

<img src="../assets/2443180-20220507181726547-499176850.png" alt="img" style="zoom: 80%;" />

在配置文件添加sendType配置即可：

```yaml
spring:
  cloud:
    stream:
      #RocketMQ 通用配置
      rocketmq:
        binder:
          #客户端接入点，必填  rocketMQ的连接地址，binder高度抽象
          name-server: localhost:9876
          # 不加, 会报错：Property 'group' is required - producerGroup
          group: rocketmq-group
        bindings:
          output1:
            producer:
              sendType: Sync # 同步发送消息，默认是异步的。消息发送类型由RocketMQProducerProperties.SendType定义
```

### 异步发送

异步发送：指发送方发出一条消息后，不等服务端返回响应，接着发送下一条消息的通讯方式。

- 消息队列RocketMQ版的异步发送，需要您实现异步发送回调接口（SendCallback）。
- 消息发送方在发送了一条消息后，不需要等待服务端响应即可发送第二条消息。
- 发送方通过回调接口接收服务端响应，并处理响应结果。
- 异步发送一般用于链路耗时较长，对**响应时间较为敏感**的业务场景，例如，视频上传后通知启动转码服务，转码完成后通知推送转码结果等。

<img src="../assets/p71667.png" alt="img" style="zoom:80%;" />

```yaml
spring:
  cloud:
    stream:
      #RocketMQ 通用配置
      rocketmq:
        binder:
          #客户端接入点，必填  rocketMQ的连接地址，binder高度抽象
          name-server: localhost:9876
          # 不加, 会报错：Property 'group' is required - producerGroup
          group: rocketmq-group
        bindings:
          output1:
            producer:
              sendType: Async # 异步发送消息。消息发送类型由RocketMQProducerProperties.SendType定义
```

### OneWay发送

发送方只负责发送消息，不等待服务端返回响应且没有回调函数触发，即只发送请求不等待应答。此方式发送消息的过程耗时非常短，一般在微秒级别。

- 适用于某些耗时非常短，但对**可靠性要求并不高**的场景，例如**日志收集**。

<img src="../assets/p71668.png" alt="img" style="zoom:80%;" />

```yaml
spring:
  cloud:
    stream:
      #RocketMQ 通用配置
      rocketmq:
        binder:
          #客户端接入点，必填  rocketMQ的连接地址，binder高度抽象
          name-server: localhost:9876
          # 不加, 会报错：Property 'group' is required - producerGroup
          group: rocketmq-group
        bindings:
          output1:
            producer:
              sendType: OneWay # 单向发送消息。消息发送类型由RocketMQProducerProperties.SendType定义
```

### 基本样例、发送同步消息

> 快速入门、发送同步消息

#### 代码结构

message

producer:

- controller

consumer

- listener

#### Message

创建 [Demo01Message](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-producer-demo/src/main/java/cn/iocoder/springcloudalibaba/labx6/rocketmqdemo/producerdemo/message/Demo01Message.java) 类，示例 Message 消息。

```java
public class Demo01Message {

    /**
     * 编号
     */
    private Integer id;

    // ... 省略 setter/getter/toString 方法
}
```

#### 搭建生产者

##### 引入依赖

```xml
<!-- 引入 Spring Cloud Alibaba Stream RocketMQ 相关依赖，将 RocketMQ 作为消息队列，并实现对其的自动配置 -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rocketmq</artifactId>
</dependency>
```

##### 配置文件

创建 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-producer-demo/src/main/resources/application.yml) 配置文件，添加 Spring Cloud Alibaba RocketMQ 相关配置。

```yaml
server:
  port: 18080
spring:
  application:
    name: demo-producer-application
  cloud:
    # Spring Cloud Stream 配置项，对应 BindingServiceProperties 类
    stream:
      # 通用的 Stream Binding 配置项，对应 BindingProperties Map
      bindings:
      	# 其中，*key* demo01-output 为 Binding 的名字，作为 Output Binding，用于生产者发送消息。
      	# 要注意，虽然说 Binding 分成 Input 和 Output 两种类型，但是在配置项中并不会体现出来，而是要在稍后搭配 `@Input` 还是 `@Output` 注解，才会有具体的区分。
        demo01-output:
          # **主题（Topic）**：表示一类消息的集合，每个主题包含若干条消息，每条消息只能属于一个主题，是 RocketMQ 进行消息订阅的基本单位。
          destination: DEMO-TOPIC-01 # 目的地。在 RocketMQ 中，使用 Topic 作为目的地。
          content-type: application/json # 内容格式。这里使用 JSON。因为稍后将发送消息的类型为 POJO，使用 JSON 进行序列化。
          producer:
            partition-key-expression: payload['id'] # 顺序消息。分区 key 表达式。该表达式基于 Spring EL，从消息中获得分区 key。
          
      # Spring Cloud Stream RocketMQ 配置项
      rocketmq:
        # RocketMQ Binder 配置项，对应 RocketMQBinderConfigurationProperties 类
        binder:
          # RocketMQ Namesrv 地址。
          # 名称服务：充当路由消息的提供者。生产者或消费者能够通过名字服务查找各主题相应的 Broker IP 列表。多个 Namesrv 实例组成集群，但相互独立，没有信息交换。
          name-server: 127.0.0.1:9876 # RocketMQ Namesrv 地址
        # RocketMQ 自定义 Binding 配置项，对应 RocketMQBindingProperties Map，其中 *key* 为 Binding 的名字，需要对应上。
        # 用于对通用的 `spring.cloud.stream.bindings` 配置项的增强，实现 RocketMQ Binding 独特的配置。
        bindings:
          # 对名字为 `demo01-output` 的 Binding 进行增强，进行 Producer 的配置
          demo01-output:
            # RocketMQ Producer 配置项，对应 RocketMQProducerProperties 类
            producer:
              group: test # 生产者分组.
              sync: true # 是否同步发送消息，默认为 false 异步。一般业务场景下，使用同步发送消息较多，所以这里设置为 `true` 同步消息。
```

- `group`：生产者分组。同一类 Producer 的集合，这类 Producer 发送同一类消息且发送逻辑一致。

    - 如果发送的是事务消息且原始生产者在发送之后崩溃，则 Broker 服务器会联系同一生产者组的其他生产者实例以提交或回溯消费。

- `sync`：是否同步发送消息，默认为 `false` 异步。

    > 使用 RocketMQ 发送三种类型的消息：同步消息（sync）、异步消息（async）和单向消息（oneway）。其中前两种消息是可靠的，因为会有发送是否成功的应答。

##### MySource 接口

创建 [MySource](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-producer-demo/src/main/java/cn/iocoder/springcloudalibaba/labx6/rocketmqdemo/producerdemo/message/MySource.java) 接口，声明名字为 **Output Binding**。

```java
public interface MySource {

    // 通过 `@Output` 注解，声明了一个名字为 `demo01-output` 的 Output Binding。
    // 注意，这个名字要和配置文件中的 `spring.cloud.stream.bindings` 配置项对应上。
    @Output("demo01-output")
    // `@Output` 注解的方法的返回结果为 MessageChannel 类型，可以使用它发送消息。
    MessageChannel demo01Output();
}
```

同时，MessageChannel 提供的发送消息的方法如下：

```java
@FunctionalInterface
public interface MessageChannel {

	long INDEFINITE_TIMEOUT = -1;
	
	default boolean send(Message<?> message) {
		return send(message, INDEFINITE_TIMEOUT);
	}
    
	boolean send(Message<?> message, long timeout);
}
```

那么，是否要实现 MySource 接口呢？

- 答案是**不需要**，全部交给 Spring Cloud Stream 的 [BindableProxyFactory](https://github.com/spring-cloud/spring-cloud-stream/blob/master/spring-cloud-stream/src/main/java/org/springframework/cloud/stream/binding/BindableProxyFactory.java) 来解决。
- BindableProxyFactory 会通过**动态代理**，**自动**实现 MySource 接口。 并扫描带有 `@Output` 注解的方法，**自动**创建方法的返回值。

例如说，`#demo01Output()` 方法被**自动**创建**返回结果**为 [DirectWithAttributesChannel](https://github.com/spring-cloud/spring-cloud-stream/blob/master/spring-cloud-stream/src/main/java/org/springframework/cloud/stream/messaging/DirectWithAttributesChannel.java)，它是 **MessageChannel 的子类**。

> 友情提示：可以在 BindableProxyFactory 的 `#afterPropertiesSet()` 和 `#invoke(MethodInvocation invocation)` 方法上，都打上一个断点，然后进行愉快的调试。

##### Controller

创建 [Demo01Controller](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-producer-demo/src/main/java/cn/iocoder/springcloudalibaba/labx6/rocketmqdemo/producerdemo/controller/Demo01Controller.java) 类，提供**发送消息的 HTTP 接口** `send()`。

```java
@RestController
@RequestMapping("/demo01")
public class Demo01Controller {

    // 使用 `@Autowired` 注解，注入 MySource Bean
    @Autowired
    private MySource mySource;

    @GetMapping("/send")
    public boolean send() {
        // <1> 创建 Message 对象
        Demo01Message message = new Demo01Message()
                .setId(new Random().nextInt());
        // <2> 使用 MessageBuilder 创建 Spring Message 对象，并设置消息内容为 Message 对象
        Message<Demo01Message> springMessage = MessageBuilder.withPayload(message).build();
        // <3> 通过 MySource 获得 MessageChannel 对象，然后发送消息
        return mySource.demo01Output().send(springMessage);
    }
}
```

##### ProducerApplication

创建 [ProducerApplication](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-producer-demo/src/main/java/cn/iocoder/springcloudalibaba/labx6/rocketmqdemo/producerdemo/ProducerApplication.java) 类，启动应用。

```java
@SpringBootApplication
// 声明指定接口开启 Binding 功能，扫描其 @Input 和 @Output 注解。这里设置为 MySource 接口
@EnableBinding(MySource.class)
public class ProducerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ProducerApplication.class, args);
    }
}
```

#### 搭建消费者

创建 [`labx-06-sca-stream-rocketmq-consumer-demo`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-demo/) 项目，作为消费者。

##### 引入依赖

创建 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-demo/pom.xml) 文件中，引入 Spring Cloud Alibaba RocketMQ 相关依赖。

##### 配置文件

创建 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-demo/src/main/resources/application.yml) 配置文件，添加 Spring Cloud Alibaba RocketMQ 相关配置。

- 总体来说，和生产者「 配置文件」是比较接近的，所以只说差异点。

```yaml
server:
  port: ${random.int[10000,19999]} # 随机端口，方便启动多个消费者

spring:
  application:
    name: demo-consumer-application
  cloud:
    stream:
      bindings:
        # 配置了一个名字为 `demo01-input` 的 Input Binding，用于消费者消费消息。
        demo01-input:
          destination: DEMO-TOPIC-01 
          content-type: application/json
          group: demo01-consumer-group-DEMO-TOPIC-01 # 消费者分组
          consumer: 
            max-attempts: 1 #消费重试
      rocketmq:
        binder:
          name-server: 127.0.0.1:9876 
        bindings:
          demo01-input:
            # RocketMQ Consumer 配置项，对应 RocketMQConsumerProperties 类
            consumer:
              enabled: true # 是否开启消费，默认为 false 使用集群消费
              broadcasting: false # 是否使用广播消费，默认为 false 使用集群消费
              
              ##
              delay-level-when-next-consume: 0 # 异步消费消息模式下消费失败重试策略，默认为 0
              orderly: true # 是否顺序消费，默认为 false 并发消费。
              tags: yunai || yutou # 基于 Tag 订阅，多个 Tag 使用 || 分隔，默认为空
              sql: # 基于 SQL 订阅，默认为空
              
```

- `group`：消费者分组。
    - **消费者组（Consumer Group）**：同一类 Consumer 的集合，这类 Consumer 通常消费同一类消息且消费逻辑一致。消费者组使得在消息消费方面，实现负载均衡和容错的目标变得非常容易。
    - 要注意的是，消费者组的消费者实例必须订阅完全相同的 Topic。RocketMQ 支持两种消息模式：集群消费（Clustering）和广播消费（Broadcasting）。

- `broadcasting`： 是否使用广播消费，默认为 `false` 使用集群消费。

    > - **集群消费（Clustering）**：集群消费模式下，相同 Consumer Group 的每个 Consumer 实例平均分摊消息。
    > - **广播消费（Broadcasting）**：广播消费模式下，相同 Consumer Group 的每个 Consumer 实例都接收全量的消息。

这里一点要注意！！一定要理解集群消费和广播消费的差异。举个例子，以有两个消费者分组 A 和 B 的场景举例子：

假设每个消费者分组各启动**一个**实例，此时发送一条消息，

- 广播消费模式下，？该消息会被两个**消费者分组** `"consumer_group_01"` 和 `"consumer_group_02"` 都各自消费一次。
- 集群消费模式下，该消息会被分组 A 的**某个**实例消费一次，被分组 B 的**某个**实例也消费一次。

通过**集群消费**的机制，可以实现针对相同 Topic ，不同消费者分组实现各自的业务逻辑。例如说：用户注册成功时，发送一条 Topic 为 `"USER_REGISTER"` 的消息。然后，不同模块使用不同的消费者分组，订阅该 Topic ，实现各自的拓展逻辑：

- 积分模块：判断如果是手机注册，给用户增加 20 积分。
- 优惠劵模块：因为是新用户，所以发放新用户专享优惠劵。
- 站内信模块：因为是新用户，所以发送新用户的欢迎语的站内信。
- ... 等等

优点：

1. 这样，就可以将**注册成功后的**业务拓展逻辑，实现业务上的**解耦**，未来也更加容易拓展。
2. 同时，也提高了注册接口的性能，避免用户需要等待业务拓展逻辑执行完成后，才响应注册成功。
3. 同时，相同消费者分组的多个实例，可以实现**高可用**，保证在一个实例意外挂掉的情况下，其它实例能够顶上。
4. 并且，多个实例都进行消费，能够提升**消费速度**。

##### MySink

创建 [MySink](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-demo/src/main/java/cn/iocoder/springcloudalibaba/labx6/rocketmqdemo/consumerdemo/listener/MySink.java) 接口，声明名字为 Input Binding。

- 通过 [`@Input`](https://github.com/spring-cloud/spring-cloud-stream/blob/master/spring-cloud-stream/src/main/java/org/springframework/cloud/stream/annotation/Input.java) 注解，声明了一个名字为 `demo01-input` 的 Input Binding。
- 注意，这个名字要和配置文件中的 `spring.cloud.stream.bindings` 配置项对应上。

```java
public interface MySink {

    String DEMO01_INPUT = "demo01-input";

    @Input(DEMO01_INPUT)
    SubscribableChannel demo01Input();
}
```

同时，`@Input` 注解的方法的返回结果为 [SubscribableChannel](https://github.com/spring-projects/spring-framework/blob/master/spring-messaging/src/main/java/org/springframework/messaging/SubscribableChannel.java) 类型，可以使用它订阅消息来消费。

MessageChannel 提供的订阅消息的方法如下：

```java
public interface SubscribableChannel extends MessageChannel {

	boolean subscribe(MessageHandler handler); // 订阅

	boolean unsubscribe(MessageHandler handler); // 取消订阅
}
```

##### Consumer

创建 [Demo01Consumer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-demo/src/main/java/cn/iocoder/springcloudalibaba/labx6/rocketmqdemo/consumerdemo/listener/Demo01Consumer.java) 类，消费消息。

- 在方法上，添加 [`@StreamListener`](https://github.com/spring-cloud/spring-cloud-stream/blob/master/spring-cloud-stream/src/main/java/org/springframework/cloud/stream/annotation/StreamListener.java) 注解，声明对应的 **Input** Binding。这里使用 `MySink.DEMO01_INPUT`。
- 又因为消费的消息是 POJO 类型，所以需要添加 [`@Payload`](https://github.com/spring-projects/spring-framework/blob/master/spring-messaging/src/main/java/org/springframework/messaging/handler/annotation/Payload.java) 注解，声明需要进行反序列化成 POJO 对象。

```java
@Component
public class Demo01Consumer {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @StreamListener(MySink.DEMO01_INPUT)
    public void onMessage(@Payload Demo01Message message) {
        logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
    }
}
```

##### ConsumerApplication

创建 [ConsumerApplication](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-demo/src/main/java/cn/iocoder/springcloudalibaba/labx6/rocketmqdemo/consumerdemo/ConsumerApplication.java) 类，启动应用。

- 使用 [`@EnableBinding`](https://github.com/spring-cloud/spring-cloud-stream/blob/master/spring-cloud-stream/src/main/java/org/springframework/cloud/stream/annotation/EnableBinding.java) 注解，声明指定接口开启 Binding 功能，扫描其 `@Input` 和 `@Output` 注解。
- 这里，设置为 MySink 接口。

```java
@SpringBootApplication
@EnableBinding(MySink.class)
public class ConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }
}
```

#### 测试场景

##### 单集群多实例

在**一个**消费者集群、启动**两个**实例，测试在集群消费的情况下的表现。

① 执行 **Consumer**Application 两次，启动两个**消费者**的实例，从而实现在消费者分组 `demo01-consumer-group-DEMO-TOPIC-01` 下有两个消费者实例。

- 此时在 IDEA 控制台看到 RocketMQ 相关的日志如下：

```bash
INFO 50472 --- [main] s.b.r.c.RocketMQListenerBindingContainer : running container: RocketMQListenerBindingContainer{consumerGroup='demo01-consumer-group-DEMO-TOPIC-01', nameServer='[127.0.0.1:9876]', topic='DEMO-TOPIC-01', consumeMode=CONCURRENTLY, selectorType=TAG, selectorExpression='null', messageModel=CLUSTERING}
INFO 50472 --- [main] .c.s.b.r.i.RocketMQInboundChannelAdapter : started com.alibaba.cloud.stream.binder.rocketmq.integration.RocketMQInboundChannelAdapter@1cd3b138
```

> 友情提示：因为 IDEA 默认同一个程序只允许启动 1 次，所以我们需要配置 DemoProviderApplication 为 `Allow parallel run`。如下图所示：![Allow parallel run](https://static.iocoder.cn/images/Spring-Cloud-Alibaba/2020-06-01/13.png)

② 执行 **Producer**Application，启动**生产者**的实例。

- 之后，请求 http://127.0.0.1:18080/demo01/send 接口三次，发送三条消息。
- 此时在 IDEA 控制台看到消费者打印日志如下：

```bash
// ConsumerApplication 控制台 01
INFO 50472 --- [MessageThread_1] c.i.s.l.r.c.listener.Demo01Consumer: [onMessage][线程编号:78 消息内容：Demo01Message{id=-1682643477}]
INFO 50472 --- [MessageThread_1] c.i.s.l.r.c.listener.Demo01Consumer: [onMessage][线程编号:78 消息内容：Demo01Message{id=1890257867}]

// ConsumerApplication 控制台 02
INFO 50534 --- [MessageThread_1] c.i.s.l.r.c.listener.Demo01Consumer: [onMessage][线程编号:80 消息内容：Demo01Message{id=1401668556}]
```

**符合预期**。从日志可以看出，每条消息仅被消费一次。

##### 多集群多实例

在**二个**消费者集群**各**启动**两个**实例，测试在集群消费的情况下的表现。

① 执行 **Consumer**Application 两次，启动两个**消费者**的实例，从而实现在消费者分组 `demo01-consumer-group-DEMO-TOPIC-01` 下有两个消费者实例。

② 修改 `labx-06-sca-stream-rocketmq-consumer-demo` 项目的配置文件，修改 `spring.cloud.stream.bindings.demo01-input.group` 配置项，将消费者分组改成 `X-demo01-consumer-group-DEMO-TOPIC-01`。

- 然后，执行 **Consumer**Application 两次，再启动两个**消费者**的实例，从而实现在消费者分组 `X-demo01-consumer-group-DEMO-TOPIC-01` 下有两个消费者实例。

③ 执行 **Producer**Application，启动**生产者**的实例。

- 之后，请求 http://127.0.0.1:18080/demo01/send 接口三次，发送三条消息。
- 此时在 IDEA 控制台看到消费者打印日志如下：

```bash
// 消费者分组 `demo01-consumer-group-DEMO-TOPIC-01` 的ConsumerApplication 控制台 01
INFO 50472 --- [MessageThread_1] c.i.s.l.r.c.listener.Demo01Consumer: [onMessage][线程编号:78 消息内容：Demo01Message{id=-276398167}]
INFO 50472 --- [MessageThread_1] c.i.s.l.r.c.listener.Demo01Consumer: [onMessage][线程编号:78 消息内容：Demo01Message{id=-250975158}]

// 消费者分组 `demo01-consumer-group-DEMO-TOPIC-01` 的ConsumerApplication 控制台 02
INFO 50534 --- [MessageThread_1] c.i.s.l.r.c.listener.Demo01Consumer: [onMessage][线程编号:80 消息内容：Demo01Message{id=412281482}]

// 消费者分组 `X-demo01-consumer-group-DEMO-TOPIC-01` 的ConsumerApplication 控制台 01
INFO 51092 --- [MessageThread_1] c.i.s.l.r.c.listener.Demo01Consumer: [onMessage][线程编号:51 消息内容：Demo01Message{id=-276398167}]
INFO 51092 --- [MessageThread_1] c.i.s.l.r.c.listener.Demo01Consumer: [onMessage][线程编号:51 消息内容：Demo01Message{id=-250975158}]

// 消费者分组 `X-demo01-consumer-group-DEMO-TOPIC-01` 的ConsumerApplication 控制台 02
INFO 51096 --- [MessageThread_1] c.i.s.l.r.c.listener.Demo01Consumer: [onMessage][线程编号:77 消息内容：Demo01Message{id=412281482}]
```

**符合预期**。从日志可以看出，每条消息被**每个**消费者集群都进行了消费，且仅被消费一次。

### 批量发送消息



## 两种消息消费方式

1. 集群消费：
2. 广播消费：`broadcasting`，如果想让所有的订阅者都能接收到消息，可以使用广播模式

同步、异步消费：

1. **同步消费**消息模式： `orderly`
    1. 同步消费失败后**再次消费**的时间间隔：`suspendCurrentQueueTimeMillis`
2. **异步消费**消息模式：
    1. **异步消费失败重试策略**：`delayLevelWhenNextConsume` ，-1, 不重复，直接放入死信队列；0, Broker 控制重试策略；>0, Client 控制重试策略  0  

### 集群消费

集群消费模式下，相同Consumer Group的每个Consumer实例**平均分摊**同一个Topic的消息。

- 即每条消息只会被发送到Consumer Group中的**某个**Consumer。
- 是最常用的消费模式。

<img src="../assets/1540879-20220106154013268-1007868027.png" alt="img" style="zoom: 80%;" />

### 广播消费

广播消费模式下，相同Consumer Group的每个Consumer实例都接收同一个Topic的**全量消息**。

- 即每条消息都会被发送到Consumer Group中的**每个**Consumer。
- 例如说，在应用中，缓存了**数据字典**等配置表在内存中，可以通过 RocketMQ 广播消费，实现每个应用节点都消费消息，刷新本地内存的缓存。
- 又例如说，基于 WebSocket 实现了 **IM 聊天**，在给用户主动发送消息时，因为不知道用户连接的是哪个提供 WebSocket 的应用，所以可以通过 RocketMQ 广播消费，每个应用判断当前用户是否是和自己提供的 WebSocket 服务连接，如果是，则推送消息给用户。

<img src="https://img2020.cnblogs.com/blog/1540879/202201/1540879-20220106154013500-120269960.png" alt="img" style="zoom:80%;" />

**广播消费**：当使用广播消费模式时，每条消息推送给集群内所有的消费者，保证消息至少被每个消费者消费一次。

![img](https://img2024.cnblogs.com/blog/2487169/202403/2487169-20240328135421183-2127747904.png)

广播消费主要用于两种场景：**消息推送**和**缓存同步**。

#### 消息推送

下图是专车的司机端推送机制，用户下单之后，订单系统生成专车订单，派单系统会根据相关算法将订单派给某司机，司机端就会收到派单推送消息。

![img](https://img2024.cnblogs.com/blog/2487169/202403/2487169-20240328135419672-1923452486.png)

推送服务是一个 TCP 服务（自定义协议），同时也是一个消费者服务，消息模式是广播消费。

司机打开司机端 APP 后，APP 会通过负载均衡和推送服务创建长连接，推送服务会保存 TCP 连接引用 （比如司机编号和 TCP channel 的引用）。

派单服务是生产者，将派单数据发送到 MetaQ , 每个推送服务都会消费到该消息，推送服务判断本地内存中是否存在该司机的 TCP channel ， 若存在，则通过 TCP 连接将数据推送给司机端。

#### 缓存同步

高并发场景下，很多应用使用本地缓存，提升系统性能 。

本地缓存可以是 HashMap 、ConcurrentHashMap ，也可以是缓存框架 Guava Cache 或者 Caffeine cache 。

![img](https://img2024.cnblogs.com/blog/2487169/202403/2487169-20240328135424470-991863943.png)

如上图，应用A启动后，作为一个 RocketMQ 消费者，消息模式设置为广播消费。为了提升接口性能，每个应用节点都会将字典表加载到本地缓存里。

当字典表数据变更时，可以通过业务系统发送一条消息到 RocketMQ ，每个应用节点都会消费消息，刷新本地缓存。

#### 示例

搭建一个 Spring Cloud Stream 消费异常处理机制的示例。考虑方便，直接复用[「2. 快速入门」](https://www.iocoder.cn/Spring-Cloud-Alibaba/RocketMQ/#)小节的项目，

- 使用 [`labx-06-sca-stream-rocketmq-producer-demo`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-producer-demo/) 发送消息，
- 从 [`labx-06-sca-stream-rocketmq-consumer-demo`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-demo/) 复制出 [`labx-06-sca-stream-rocketmq-consumer-broadcasting`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-broadcasting/) 来**演示广播消费**。

##### 配置文件

修改 [`application.yml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-broadcasting/src/main/resources/application.yml) 配置文件，设置 `broadcasting` 配置项为 `true`，开启**广播消费**的模式。

```yaml
spring:
  cloud:
    stream:
      rocketmq:
        bindings:
          demo01-input:
            consumer:
              broadcasting: true # 是否使用广播消费，默认为 false 使用集群消费
spring:
  cloud:
    stream:
      #RocketMQ 通用配置
      rocketmq:
        binder:
          #客户端接入点，必填  rocketMQ的连接地址，binder高度抽象
          name-server: localhost:9876
          # 不加, 会报错：Property 'group' is required - producerGroup
          group: rocketmq-group
        bindings:
          input2: # 与 spring.cloud.stream.bindings.input2对应上
            consumer:
              subscription: myTag||look # 基于 Tag 订阅，多个 Tag 使用 || 分隔，默认为空
          input1:
            consumer:
              messageModel: BROADCASTING # 消费模式，由MessageModel定义
```

##### 简单测试

① 执行 **Consumer**Application 两次，启动两个**消费者**的实例，从而实现在消费者分组 `demo01-consumer-group-DEMO-TOPIC-01` 下有两个消费者实例。

② 执行 **Producer**Application，启动**生产者**的实例。

之后，请求 http://127.0.0.1:18080/demo01/send 接口三次，发送三条消息。此时在 IDEA 控制台看到消费者打印日志如下：

- **符合预期**。从日志可以看出，每条消息仅被每个消费者消费了一次。

```bash
// ConsumerApplication 控制台 01
INFO 68510 --- [MessageThread_1] c.i.s.l.r.c.listener.Demo01Consumer      : [onMessage][线程编号:78 消息内容：Demo01Message{id=-335590634}]
INFO 68510 --- [MessageThread_1] c.i.s.l.r.c.listener.Demo01Consumer      : [onMessage][线程编号:78 消息内容：Demo01Message{id=283364059}]
INFO 68510 --- [MessageThread_1] c.i.s.l.r.c.listener.Demo01Consumer      : [onMessage][线程编号:78 消息内容：Demo01Message{id=-1253930234}]

// ConsumerApplication 控制台 02
INFO 68519 --- [MessageThread_1] c.i.s.l.r.c.listener.Demo01Consumer      : [onMessage][线程编号:75 消息内容：Demo01Message{id=-335590634}]
INFO 68519 --- [MessageThread_1] c.i.s.l.r.c.listener.Demo01Consumer      : [onMessage][线程编号:75 消息内容：Demo01Message{id=283364059}]
INFO 68519 --- [MessageThread_1] c.i.s.l.r.c.listener.Demo01Consumer      : [onMessage][线程编号:75 消息内容：Demo01Message{id=-1253930234}]
```

### 消费重试

RocketMQ 提供**消费重试**的机制。在消息**消费失败**时，RocketMQ 会通过**消费重试**机制，重新投递该消息给 Consumer ，让 Consumer 有机会重新消费消息，实现消费成功。

当然，RocketMQ 并不会无限重新投递消息给 Consumer 重新消费，而是在默认情况下，达到 16 次重试次数时，Consumer 还是消费失败时，该消息就会进入到**死信队列**。

> 死信队列用于处理无法被正常消费的消息。当一条消息初次消费失败，消息队列会自动进行消息重试；达到最大重试次数后，若消费依然失败，则表明消费者在正常情况下无法正确地消费该消息，此时，消息队列不会立刻将消息丢弃，而是将其发送到该消费者对应的特殊队列中。
>
> RocketMQ 将这种正常情况下无法被消费的消息称为死信消息（Dead-Letter Message），将存储死信消息的特殊队列称为死信队列（Dead-Letter Queue）。
>
> 在 RocketMQ 中，可以通过使用 console 控制台对死信队列中的消息进行重发来使得消费者实例再次进行消费。

每条消息的失败重试，是有一定的间隔时间。

- 实际上，消费重试是基于[「5. 定时消息」](https://www.iocoder.cn/Spring-Cloud-Alibaba/RocketMQ/#) 来实现，第一次重试消费按照延迟级别为 **3** 开始。
- 所以，默认为 16 次重试消费，也非常好理解，毕竟延迟级别最高为 18 呀。

不过要注意，只有**集群消费**模式下，才有消息重试。

下面，来搭建一个 RocketMQ 消息重试的使用示例。考虑方便，直接复用[「2. 快速入门」](https://www.iocoder.cn/Spring-Cloud-Alibaba/RocketMQ/#)的项目，

- 使用 [`labx-06-sca-stream-rocketmq-producer-demo`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-producer-demo/) 发送消息，
- 从 [`labx-06-sca-stream-rocketmq-consumer-demo`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-demo/) 复制出 [`labx-06-sca-stream-rocketmq-consumer-retry`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-retry/) 来**模拟消费失败后的重试**。

#### 配置文件

修改 [`application.yml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-retry/src/main/resources/application.yml) 配置文件，增加**消费重试**相关的两个配置项 `delay-level-when-next-consume` 和 `max-attempts`。

① 对于 `delay-level-when-next-consume` 配置项，一共有三种选择：

- -1：不重复，直接放入死信队列
- 0：RocketMQ Broker（or Consumer）控制重试策略

每天消息**首次**消费失败时，Consumer 会**发回**给 Broker，并告诉 Broker 按照什么延迟级别**开始**，不断重新投递给 Consumer 直到消费成功或者到达最大延迟级别。

- 举个例子，如果这里设置了 `delay-level-when-next-consume` 配置项为 18，则 2 小时后 Broker 会投递该消息给 Consumer 进行重新消费。

一般情况下，设置 `delay-level-when-next-consume` 配置项为 0 即可，使用 Broker 控制重试策略即可。

- 默认配置下，Broker 会使用延迟级别从 3 开始，10 秒后 Broker 会投递该消息给 Consumer 进行重新消费。

② 对于 `max-attempts` 配置项，每次拉取到消息到本地时，如果消费重试，本地重试的最大总次数（包括第一次）。

- 这个是 Spring Cloud Stream 提供的**通用**消费重试功能，是 **Consumer** 级别的，而 RocketMQ 提供的**独有**消费重试功能，是 **Broker** 级别的。

因为 Spring Cloud Stream 提供的重试**间隔**，是通过 sleep 实现，会占掉当前线程，影响 Consumer 的消费速度，所以这里并不推荐使用，因此设置 `max-attempts` 配置项为 1，**禁用 Spring Cloud Stream 提供的重试功能，使用 RocketMQ 提供的重试功能**。

> 友情提示：如果无法保证消费重试不会带来副作用，也就是说无法保证消费的幂等性，建议关闭消费重试功能，即设置 `delay-level-when-next-consume` 配置项为 -1，`max-attempts` 配置项为 1。

```yaml
spring:
  application:
    name: demo-consumer-application
  cloud:
    stream:
      bindings:
        demo01-input:
          group: demo01-consumer-group-DEMO-TOPIC-01 # 消费者分组
          consumer:
            max-attempts: 1
      rocketmq:
        bindings:
          demo01-input:
            consumer:
              enabled: true # 是否开启消费，默认为 true
              broadcasting: false # 是否使用广播消费，默认为 false 使用集群消费
              delay-level-when-next-consume: 0 # 异步消费消息模式下消费失败重试策略，默认为 0

server:
  port: ${random.int[10000,19999]} # 随机端口，方便启动多个消费者
```

#### Consumer

修改 [Demo01Consumer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-retry/src/main/java/cn/iocoder/springcloudalibaba/labx6/rocketmqdemo/consumerdemo/listener/Demo01Consumer.java) 类，在消费消息时抛出异常，从而模拟消费错误。

```java
@Component
public class Demo01Consumer {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @StreamListener(MySink.DEMO01_INPUT)
    public void onMessage(@Payload Demo01Message message) {
        logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
        // <X> 注意，此处抛出一个 RuntimeException 异常，模拟消费失败
        throw new RuntimeException("我就是故意抛出一个异常");
    }
}
```

#### 简单测试

① 执行 **Consumer**Application，启动**消费者**的实例。

② 执行 **Producer**Application，启动**生产者**的实例。

之后，请求 http://127.0.0.1:18080/demo01/send 接口，发送一条消息。IDEA 控制台输出日志如下：

```bash
// Demo01Consumer 第一次消费失败，抛出 RuntimeException 异常
INFO 61116 --- [MessageThread_1] c.i.s.l.r.c.listener.Demo01Consumer      : [onMessage][线程编号:69 消息内容：Demo01Message{id=-604160799}]
ERROR 61116 --- [MessageThread_1] o.s.integration.handler.LoggingHandler   : org.springframework.messaging.MessagingException: // ... 省略

// Demo01Consumer 第一次重试消费失败，抛出 RuntimeException 异常。间隔了 10 秒，对应延迟级别 3 。
INFO 61116 --- [MessageThread_1] c.i.s.l.r.c.listener.Demo01Consumer      : [onMessage][线程编号:69 消息内容：Demo01Message{id=-604160799}]
ERROR 61116 --- [MessageThread_1] o.s.integration.handler.LoggingHandler   : org.springframework.messaging.MessagingException: // ... 省略

// Demo01Consumer 第二次重试消费失败，抛出 RuntimeException 异常。间隔了 30 秒，对应延迟级别 4 。
INFO 61116 --- [MessageThread_1] c.i.s.l.r.c.listener.Demo01Consumer      : [onMessage][线程编号:69 消息内容：Demo01Message{id=-604160799}]
ERROR 61116 --- [MessageThread_1] o.s.integration.handler.LoggingHandler   : org.springframework.messaging.MessagingException: // ... 省略

// ... 省略，后续还有重试
```

**符合预期**。从日志中，我们可以看到，消息因为消费失败后，又重试消费了多次。

### 消费异常处理机制

在 Spring Cloud Stream 中，提供了**通用**的消费异常处理机制，可以拦截到消费者**消费消息**时发生的异常，进行自定义的处理逻辑。

下面，来搭建一个 Spring Cloud Stream 消费异常处理机制的示例。

- 考虑方便，直接复用[「5. 消费重试」](https://www.iocoder.cn/Spring-Cloud-Alibaba/RocketMQ/#)小节的项目，使用 [`labx-06-sca-stream-rocketmq-producer-demo`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-producer-demo/) 发送消息，
- 从 [`labx-06-sca-stream-rocketmq-consumer-retry`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-retry/) 复制出 [`labx-06-sca-stream-rocketmq-consumer-error-handler`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-error-handler/) 来**演示消费异常处理机制**。

#### Consumer

修改 [Demo01Consumer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-error-handler/src/main/java/cn/iocoder/springcloudalibaba/labx6/rocketmqdemo/consumerdemo/listener/Demo01Consumer.java) 类，增加消费异常处理方法。

在 Spring Integration 的设定中，若 `#onMessage()` 方法消费消息发生异常时，会发送错误消息（[ErrorMessage](https://github.com/spring-projects/spring-framework/blob/master/spring-messaging/src/main/java/org/springframework/messaging/support/ErrorMessage.java)）到**对应的错误 Channel**（`<destination>.<group>.errors`）中。

- 同时，所有错误 Channel 都桥接到了 Spring Integration 定义的**全局错误 Channel**(`errorChannel`)。

因此，有**两种**方式来实现异常处理：

1. **局部**的异常处理：通过订阅指定**错误 Channel**。在 `#handleError(ErrorMessage errorMessage)` 方法上，声明了 **[`@ServiceActivator`](https://github.com/spring-projects/spring-integration/blob/master/spring-integration-core/src/main/java/org/springframework/integration/annotation/ServiceActivator.java) 注解**，订阅**指定错误 Channel**的错误消息，实现 `#onMessage()` 方法的局部异常处理。如下图所示：
2. **全局**的异常处理：通过订阅**全局错误 Channel**。在 `#globalHandleError(ErrorMessage errorMessage)` 方法上，声明了 **`@StreamListener` 注解**，订阅**全局错误 Channel**的错误消息，实现全局异常处理。

在**全局**和**局部**异常处理都定义的情况下，错误消息仅会被**符合条件**的**局部**错误异常处理。

- 如果没有符合条件的，错误消息才会被**全局**异常处理。

![对应关系](../assets/rocketmq_ex_21.jpg)

```java
@Component //
public class Demo01Consumer {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @StreamListener(MySink.DEMO01_INPUT) // 订阅**指定错误 Channel**的错误消息，实现 `#onMessage()` 方法的**局部**异常处理。
    // 对应 DEMO-TOPIC-01.demo01-consumer-group-DEMO-TOPIC-01,对应的错误 Channel（`<destination>.<group>.errors`），
    // 对应配置文件cloud.stream.banding..demo1_input
    public void onMessage(@Payload Demo01Message message) {
        logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
        // <X> 注意，此处抛出一个 RuntimeException 异常，模拟消费失败
        throw new RuntimeException("我就是故意抛出一个异常");
    }

    @ServiceActivator(inputChannel = "DEMO-TOPIC-01.demo01-consumer-group-DEMO-TOPIC-01.errors")
    public void handleError(ErrorMessage errorMessage) {
        logger.error("[handleError][payload：{}]", ExceptionUtils.getRootCauseMessage(errorMessage.getPayload()));
        logger.error("[handleError][original Message：{}]", errorMessage.getOriginalMessage());
        logger.error("[handleError][headers：{}]", errorMessage.getHeaders());
    }

    @StreamListener(IntegrationContextUtils.ERROR_CHANNEL_BEAN_NAME) // errorChannel,订阅**全局错误 Channel**的错误消息，实现**全局**异常处理。
    public void globalHandleError(ErrorMessage errorMessage) {
        logger.error("[globalHandleError][payload：{}]", ExceptionUtils.getRootCauseMessage(errorMessage.getPayload()));
        logger.error("[globalHandleError][originalMessage：{}]", errorMessage.getOriginalMessage());
        logger.error("[globalHandleError][headers：{}]", errorMessage.getHeaders());
    }
}
```

#### 简单测试

① 执行 **Consumer**Application，启动**消费者**的实例。

② 执行 **Producer**Application，启动**生产者**的实例。

之后，请求 http://127.0.0.1:18080/demo01/send 接口，发送一条消息。IDEA 控制台输出日志如下：

- 不过要注意，如果异常处理方法成功，没有重新抛出异常，会认定为该消息被**消费成功**，所以就不会进行消费重试。

```bash
// onMessage 方法
INFO 67767 --- [MessageThread_1] c.i.s.l.r.c.listener.Demo01Consumer      : [onMessage][线程编号:60 消息内容：Demo01Message{id=-317670393}]
ERROR 67767 --- [MessageThread_1] c.i.s.l.r.c.listener.Demo01Consumer      : [handleError][payload：RuntimeException: 我就是故意抛出一个异常]

// handleError 方法
ERROR 67767 --- [MessageThread_1] c.i.s.l.r.c.listener.Demo01Consumer      : [handleError][original Message：GenericMessage [payload=byte[17], headers={rocketmq_QUEUE_ID=3, rocketmq_TOPIC=DEMO-TOPIC-01, rocketmq_FLAG=0, rocketmq_RECONSUME_TIMES=0, rocketmq_MESSAGE_ID=0A258102FE8918B4AAC2620411310017, rocketmq_SYS_FLAG=0, id=dc6dafb1-b303-7931-5977-45f319b935d9, CLUSTER=DefaultCluster, rocketmq_BORN_HOST=10.37.129.2, contentType=application/json, rocketmq_BORN_TIMESTAMP=1582130833713, timestamp=1582130854444}]]
ERROR 67767 --- [MessageThread_1] c.i.s.l.r.c.listener.Demo01Consumer      : [handleError][headers：{id=cdf37b5d-878c-3d85-1f40-7711a3642a16, timestamp=1582130854489}]
```

## 高级消息类型

### **顺序消息**

RocketMQ 提供了两种顺序级别：

- **普通**顺序消息（**分区**顺序）：Producer 将相关联的消息发送到相同的消息队列。
- 完全**严格**顺序（**全局**顺序）：在【普通顺序消息】的基础上，Consumer 严格顺序消费。

[官方文档](https://github.com/apache/rocketmq/blob/master/docs/cn/features.md#2-消息顺序)是这么描述的：

> 消息有序，指的是一类消息消费时，能按照发送的顺序来消费。
>
> - 例如：一个订单产生了三条消息分别是订单创建、订单付款、订单完成。消费时要按照这个顺序消费才能有意义，但是同时订单之间是可以并行消费的。
> - RocketMQ 可以严格的保证消息有序。
>
> 顺序消息分为全局顺序消息与分区顺序消息，全局顺序是指某个 Topic 下的所有消息都要保证顺序；部分顺序消息只要保证每一组消息被顺序消费即可。
>
> - 全局顺序：对于指定的一个 Topic，所有消息按照严格的先入先出（FIFO）的顺序进行发布和消费。
>     - 适用场景：性能要求不高，所有的消息严格按照 FIFO 原则进行消息发布和消费的场景
> - 分区顺序：对于指定的一个 Topic，所有消息根据 Sharding key 进行区块分区。 同一个分区内的消息按照严格的 FIFO 顺序进行发布和消费。Sharding key 是顺序消息中用来区分不同分区的关键字段，和普通消息的 Key 是完全不同的概念。
>     - 适用场景：性能要求高，以 Sharding key 作为分区字段，在同一个区块中严格的按照 FIFO 原则进行消息发布和消费的场景。

下面，来搭建一个 Spring Cloud Stream 消费异常处理机制的示例。考虑方便，直接复用[「2. 快速入门」](https://www.iocoder.cn/Spring-Cloud-Alibaba/RocketMQ/#)小节的项目：

- 从 [`labx-06-sca-stream-rocketmq-producer-demo`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-producer-demo/) 复制出 [`labx-06-sca-stream-rocketmq-producer-orderly`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-producer-orderly/) 来演示**发送顺序消息**。
- 从 [`labx-06-sca-stream-rocketmq-consumer-demo`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-demo/) 复制出 [`labx-06-sca-stream-rocketmq-consumer-broadcasting`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-broadcasting/) 来演示**顺序消费消息**。

#### 搭建生产者

##### 配置文件

修改 [`application.yml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-producer-orderly/src/main/resources/application.yml) 配置文件，添加 `partition-key-expression` 配置项，设置 Producer 发送顺序消息的 Sharding key。

① `partition-key-expression` 配置项，该表达式基于 [Spring EL](https://docs.spring.io/spring/docs/4.3.10.RELEASE/spring-framework-reference/html/expressions.html)，从消息中获得 Sharding key。

- 这里，设置该配置项为 `payload['id']`，表示从 Spring [Message](https://github.com/spring-projects/spring-framework/blob/master/spring-messaging/src/main/java/org/springframework/messaging/Message.java) 的 payload 的 `id`。稍后发送的消息的 payload 为 Demo01Message，那么 `id` 就是 `Demo01Message.id`。
- 如果想从消息的 headers 中获得 Sharding key，可以设置为 `headers['partitionKey']`。

```yaml
spring:
  cloud:
    stream:
      bindings:
        demo01-output:
          producer:
            partition-key-expression: payload['id'] # 分区 key 表达式。该表达式基于 Spring EL，从消息中获得分区 key。
      rocketmq:
        bindings:
          demo01-output:
            producer:
              group: test # 生产者分组
              sync: true # 是否同步发送消息，默认为 false 异步。
server:
  port: 18080
```

② Spring Cloud Stream 使用 [PartitionHandler](https://github.com/spring-cloud/spring-cloud-stream/blob/master/spring-cloud-stream/src/main/java/org/springframework/cloud/stream/binder/PartitionHandler.java) 进行 Sharding key 的获得与计算，最终 Sharding key 的结果为 `key.hashCode() % partitionCount`。

在获取到 Sharding key 之后，Spring Cloud Alibaba Stream RocketMQ 提供的 [PartitionMessageQueueSelector](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-stream-binder-rocketmq/src/main/java/com/alibaba/cloud/stream/binder/rocketmq/provisioning/selector/PartitionMessageQueueSelector.java) 选择消息发送的队列。

以发送一条 `id` 为 1 的 Demo01Message 消息为示例，最终会发送到对应 RocketMQ Topic 的队列为 1。计算过程如下：

```bash
// 第一步，PartitionHandler 使用 `partition-key-expression` 表达式，从 Message 中获得 Sharding key
key => 1

// 第二步，PartitionHandler 计算最终的 Sharding key
// 默认情况下，每个 RocketMQ Topic 的队列总数是 4。
key => key.hashCode() % partitionCount = 1.hashCode() % 4 = 1 % 4 = 1

// 第三步，PartitionMessageQueueSelector 获得对应 RocketMQ Topic 的队列
队列 => queues.get(key) = queues.get(1)
```

这样，就能保证**相同 Sharding Key** 的消息，发送到**相同的对应 RocketMQ Topic 的队列**中。

- 当然，前提是该 Topic 的队列总数不能变，不然计算的 Sharding Key 会发生变化。

##### Controller

修改 [Demo01Controller](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-producer-orderly/src/main/java/cn/iocoder/springcloudalibaba/labx6/rocketmqdemo/producerdemo/controller/Demo01Controller.java) 类，增加发送 3 条**顺序**消息的 HTTP 接口。

- 每次发送的 3 条消息使用相同的 `id`，配合上使用它作为 Sharding key，就可以发送对应 Topic 的**相同队列**中。
- 另外，整列发送的虽然是**顺序**消息，但是和发送**普通**消息的代码是**一模一样**的。

```java
@GetMapping("/send_orderly")
public boolean sendOrderly() {
    // 发送 3 条相同 id 的消息
    int id = new Random().nextInt();
    for (int i = 0; i < 3; i++) {
        // 创建 Message
        Demo01Message message = new Demo01Message().setId(id);
        // 创建 Spring Message 对象
        Message<Demo01Message> springMessage = MessageBuilder.withPayload(message)
                .build();
        // 发送消息
        mySource.demo01Output().send(springMessage);
    }
    return true;
}
```

#### 搭建消费者

从 [`labx-06-sca-stream-rocketmq-consumer-demo`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-demo/) 复制出 [`labx-06-sca-stream-rocketmq-consumer-broadcasting`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-broadcasting/) 来**演示顺序消费消息**

##### 配置文件

修改 [`application.yml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-orderly/src/main/resources/application.yml) 配置文件，添加 `orderly` 配置项，设置 Consumer 顺序消费消息。

```yaml
spring:
  cloud:
    stream:
      rocketmq:
        bindings:
          demo01-input:
            consumer:
              enabled: true
              broadcasting: false # 是否使用广播消费，默认为 false 使用集群消费
              orderly: true # 是否顺序消费，默认为 false 并发消费。
```

##### Consumer

修改 [Demo01Consumer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-orderly/src/main/java/cn/iocoder/springcloudalibaba/labx6/rocketmqdemo/consumerdemo/listener/Demo01Consumer.java) 类，在消费消息时，打印出消息所在**队列编号**和**线程编号**，

- 这样通过队列编号可以判断消息是否**顺序发送**，通过线程编号可以判断消息是否**顺序消费**。

```java
@Component
public class Demo01Consumer {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @StreamListener(MySink.DEMO01_INPUT)
    public void onMessage(Message<?> message) {
        logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
    }
}
```

#### 简单测试

① 执行 **Consumer**Application，启动**消费者**的实例。

② 执行 **Producer**Application，启动**生产者**的实例。

之后，请求 http://127.0.0.1:18080/demo01/send_orderly 接口，发送顺序消息。IDEA 控制台输出日志如下：

- `id` 为 58569988 的消息被发送到 RocketMQ 消息队列编号为 0，并且在线程编号为 76 的线程中消费。
- 可以在多调用几次接口，继续尝试。

```bash
INFO 74637 --- [MessageThread_1] c.i.s.l.r.c.listener.Demo01Consumer      : [onMessage][线程编号:76 消息内容：GenericMessage [payload={"id":58569988}, headers={rocketmq_QUEUE_ID=0, rocketmq_RECONSUME_TIMES=0, scst_partition=0, rocketmq_BORN_TIMESTAMP=1582205212037, rocketmq_TOPIC=DEMO-TOPIC-01, rocketmq_FLAG=0, spring_json_header_types={"scst_partition":"java.lang.Integer"}, rocketmq_MESSAGE_ID=0A25810236DE18B4AAC26672FD850006, rocketmq_SYS_FLAG=0, id=945725a1-abfb-218a-d480-b220adff9549, CLUSTER=DefaultCluster, rocketmq_BORN_HOST=10.37.129.2, contentType=application/json, timestamp=1582205212044}]]

INFO 74637 --- [MessageThread_1] c.i.s.l.r.c.listener.Demo01Consumer      : [onMessage][线程编号:76 消息内容：GenericMessage [payload={"id":58569988}, headers={rocketmq_QUEUE_ID=0, rocketmq_RECONSUME_TIMES=0, scst_partition=0, rocketmq_BORN_TIMESTAMP=1582205212039, rocketmq_TOPIC=DEMO-TOPIC-01, rocketmq_FLAG=0, spring_json_header_types={"scst_partition":"java.lang.Integer"}, rocketmq_MESSAGE_ID=0A25810236DE18B4AAC26672FD870007, rocketmq_SYS_FLAG=0, id=86a0e912-3cba-8b5b-3928-a7ef0ad80036, CLUSTER=DefaultCluster, rocketmq_BORN_HOST=10.37.129.2, contentType=application/json, timestamp=1582205212046}]]

INFO 74637 --- [MessageThread_1] c.i.s.l.r.c.listener.Demo01Consumer      : [onMessage][线程编号:76 消息内容：GenericMessage [payload={"id":58569988}, headers={rocketmq_QUEUE_ID=0, rocketmq_RECONSUME_TIMES=0, scst_partition=0, rocketmq_BORN_TIMESTAMP=1582205212041, rocketmq_TOPIC=DEMO-TOPIC-01, rocketmq_FLAG=0, spring_json_header_types={"scst_partition":"java.lang.Integer"}, rocketmq_MESSAGE_ID=0A25810236DE18B4AAC26672FD890008, rocketmq_SYS_FLAG=0, id=b04416a3-60c2-bf42-a5a4-fe3c5079cc55, CLUSTER=DefaultCluster, rocketmq_BORN_HOST=10.37.129.2, contentType=application/json, timestamp=1582205212046}]]
```

### 延时、定时消息样例

**定时消息**，是指消息发到 Broker 后，不能立刻被 Consumer 消费，要到特定的时间点或者等待特定的时间后才能被消费。

#### 使用场景

比如电商里，提交了一个订单就可以发送一个延时消息，1h后去检查这个订单的状态，如果还是未付款就**取消订单**释放库存。

#### 延迟级别和时间精度

现在RocketMq并不支持**任意时间**的延时，需要设置几个固定的延时等级，从1s到2h分别对应着等级1到18
消息消费失败会进入延时消息队列。

- 消息发送时间与设置的延时等级和重试次数有关，详见代码`SendMessageProcessor.java`
- 如果想要**任一时刻**的定时消息，可以考虑借助 MySQL + Job 来实现。又或者考虑使用 [DDMQ](https://github.com/didi/DDMQ/)（滴滴打车基于 RocketMQ 和 Kafka 改造的开源消息队列）。

```java
// org/apache/rocketmq/store/config/MessageStoreConfig.java

private String messageDelayLevel = "1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h";
```



| 延迟级别 | 时间 | 延迟级别 | 时间 | 延迟级别 | 时间 |
| :------- | :--- | :------- | :--- | :------- | :--- |
| 1        | 1s   | 7        | 3m   | 13       | 9m   |
| 2        | 5s   | 8        | 4m   | 14       | 10m  |
| 3        | 10s  | 9        | 5m   | 15       | 20m  |
| 4        | 30s  | 10       | 6m   | 16       | 30m  |
| 5        | 1m   | 11       | 7m   | 17       | 1h   |
| 6        | 2m   | 12       | 8m   | 18       | 2h   |

#### Controller

修改 [Demo01Controller](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-producer-demo/src/main/java/cn/iocoder/springcloudalibaba/labx6/rocketmqdemo/producerdemo/controller/Demo01Controller.java) 类，增发送**定时**消息的 HTTP 接口。

```java
    private Logger logger = LoggerFactory.getLogger(getClass());

    @GetMapping("/send_delay")
    public boolean sendDelay() {
        // 创建 Message
        Demo01Message message = new Demo01Message().setId(new Random().nextInt());
        // 创建 Spring Message 对象
        Message<Demo01Message> springMessage = MessageBuilder.withPayload(message)
                .setHeader(MessageConst.PROPERTY_DELAY_TIME_LEVEL, "3") // <X> 设置消息的延迟级别为 3，10 秒后消费，从而发送定时消息
                .build();
        // 发送消息
        boolean sendResult = mySource.demo01Output().send(springMessage);
        logger.info("[sendDelay][发送消息完成, 结果 = {}]", sendResult);
        return sendResult;
    }
```

#### 简单测试

① 执行 **Consumer**Application，启动**消费者**的实例，等待传入订阅消息。

② 执行 **Producer**Application，启动**生产者**的实例，发送延时消息。

之后，请求 http://127.0.0.1:18080/demo01/send_delay 接口，发送延迟 10 秒的定时消息。

IDEA 控制台输出日志如下：

```bash
// Producer 的控制台
INFO 57143 --- [io-18080-exec-5] c.i.s.l.r.p.controller.Demo01Controller  : [sendDelay][发送消息完成, 结果 = true]

// Consumer 的控制台
INFO 57133 --- [MessageThread_1] c.i.s.l.r.c.listener.Demo01Consumer      : [onMessage][线程编号:61 消息内容：Demo01Message{id=618574636}]
```

**符合预期**。在 Producer 发送的消息之后，Consumer 确实 10 秒后才消费消息。

> 将会看到消息的消费比存储时间晚10秒。

### **事务消息**

在分布式消息队列中，目前唯一提供**完整的**事务消息的，只有 RocketMQ 。

- RocketMQ提供类似**XA或Open XA**的**分布式事务**功能，通过消息队列RocketMQ版事务消息，能达到分布式事务的**最终一致**。
- 虽然 RabbitMQ 和 Kafka 也有事务消息，也支持发送事务消息的发送，以及后续的事务消息的 commit提交或 rollbackc 回滚。
- 但是要考虑一个极端的情况，在**本地数据库事务已经提交**时，如果因为网络原因，又或者崩溃等意外，导致事务消息**没有被 commit** ，最终导致这条**事务消息丢失**，分布式事务出现问题。

- 相比来说，RocketMQ 提供**事务回查机制**：如果应用超过一定时长未 commit 或 rollback 这条事务消息，RocketMQ 会主动回查应用，询问这条事务消息是 commit 还是 rollback ，从而实现事务消息的状态最终能够被 commit 或是 rollback ，达到最终**事务的一致性**。

事务消息交互流程如下图所示：

<img src="../assets/p69402.png" alt="img" style="zoom:80%;" />

分布式事务，推荐阅读如下两篇文章：

- [《阿里云消息队列 MQ —— 事务消息》](https://www.alibabacloud.com/help/zh/doc-detail/43348.htm)
- [《芋道 RocketMQ 源码解析 —— 事务消息》](http://www.iocoder.cn/RocketMQ/message-transaction/?self)【todo】

> 虽然说 RabbitMQ、Kafka 并未提供完整的事务消息，但是社区里，已经基于它们之上拓展，提供了事务回查的功能。例如说：[Myth](https://dromara.org/zh-cn/docs/myth/index.html) ，采用消息队列解决**分布式事务**的开源框架，基于 Java 语言来开发（JDK1.8），支持 Dubbo，Spring Cloud，Motan 等 RPC 框架进行分布式事务。

下面，来搭建一个 RocketMQ 定时消息的使用示例。

- 直接复用[「2. 快速入门」](https://www.iocoder.cn/Spring-Cloud-Alibaba/RocketMQ/#)小节的项目，修改 [`labx-06-sca-stream-rocketmq-producer-transaction`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-producer-transaction/) 发送**事务消息**，
- 继续使用 [`labx-06-sca-stream-rocketmq-consumer-demo`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-demo/) 消费消息。

#### 配置文件

修改 [`application.yml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-producer-transaction/src/main/resources/application.yml) 配置文件，添加 `transactional` 配置项为 `true`，设置 Producer 发送**事务**消息。

```yaml
spring:
  cloud:
    stream:
      rocketmq:
        bindings:
          demo01-output:
            producer:
              group: test # 生产者分组
              sync: true
              transactional: true # 是否发送事务消息，默认为 false。
```

#### Controller

修改 [Demo01Controller](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-producer-transaction/src/main/java/cn/iocoder/springcloudalibaba/labx6/rocketmqdemo/producerdemo/controller/Demo01Controller.java) 类，增加发送**事务**消息的 HTTP 接口。

- 因为 Spring Cloud Stream 在设计时，并没有考虑事务消息，所以只好在 `<X>` 处，通过 Header 传递参数。
- 又因为 Header 后续会被转换成 String 类型，导致无法获得正确的真实的**原始**参数，所以这里先使用 JSON 将 `args` 参数序列化成字符串，这样后续可以使用 JSON 反序列化回来。

```java
@GetMapping("/send_transaction")
public boolean sendTransaction() {
    // 创建 Message
    Demo01Message message = new Demo01Message()
            .setId(new Random().nextInt());
    // 创建 Spring Message 对象
    Args args = new Args().setArgs1(1).setArgs2("2");
    Message<Demo01Message> springMessage = MessageBuilder.withPayload(message)
            .setHeader("args", JSON.toJSONString(args)) // <X>
            .build();
    // 发送消息
    return mySource.demo01Output().send(springMessage);
}

public static class Args { // 这里作为示例，所以直接这么写了

    private Integer args1;
    private String args2;
    // ... 省略 setter、getter、toString 方法
}
```

#### TransactionListenerImpl

创建 [TransactionListenerImpl](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-producer-transaction/src/main/java/cn/iocoder/springcloudalibaba/labx6/rocketmqdemo/producerdemo/listener/TransactionListenerImpl.java) 类，实现 [RocketMQLocalTransactionListener](https://github.com/apache/rocketmq/blob/master/client/src/main/java/org/apache/rocketmq/client/producer/TransactionListener.java) 接口，实现 **MQ 事务的监听**。

① 在类上，添加 [`@RocketMQTransactionListener`](https://github.com/apache/rocketmq-spring/blob/master/rocketmq-spring-boot/src/main/java/org/apache/rocketmq/spring/annotation/RocketMQTransactionListener.java) 注解，声明监听器的是生产者分组是 `"test"` 的 Producer 发送的事务消息。因为 RocketMQ 是**回查**（请求）指定指定生产分组下的 Producer，从而获得事务消息的状态，所以一定要正确设置。

③ 实现 `#executeLocalTransaction(...)` 方法，实现**执行**本地事务。

- 注意，这是一个**模板方法**。在调用这个方法之前，Spring Cloud Alibaba Stream RocketMQ 已经使用 Producer 发送了一条**事务**消息。然后根据该方法执行的返回的 [RocketMQLocalTransactionState](https://github.com/apache/rocketmq-spring/blob/3f89080df8f797cad0d1f9eb8badb5050b09a553/rocketmq-spring-boot/src/main/java/org/apache/rocketmq/spring/core/RocketMQLocalTransactionState.java) 结果，提交还是回滚该事务消息。

④ 实现 `#checkLocalTransaction(...)` 方法，**检查**本地事务。

- 在事务消息长事件未被提交或回滚时，RocketMQ 会回查事务消息对应的生产者分组下的 Producer ，获得事务消息的状态。此时，该方法就会被调用。

一般来说，有两种方式实现本地事务回查时，返回事务消息的状态。

**第一种**，通过 `msg` 消息，获得某个业务上的标识或者编号，然后去数据库中查询业务记录，从而判断该事务消息的状态是提交还是回滚。

**第二种**，记录 `msg` 的事务编号，与事务状态到数据库中。

1. 第一步，在 `#executeLocalTransaction(...)` 方法中，先存储一条 `id` 为 `msg` 的事务编号，状态为 `RocketMQLocalTransactionState.UNKNOWN` 的记录。
2. 第二步，调用带有**事务的**业务 Service 的方法。在该 Service 方法中，在逻辑都执行成功的情况下，更新 `id` 为 `msg` 的事务编号，状态变更为 `RocketMQLocalTransactionState.COMMIT` 。这样，我们就可以伴随这个事务的提交，更新 `id` 为 `msg` 的事务编号的记录的状为 `RocketMQLocalTransactionState.COMMIT` ，美滋滋。。
3. 第三步，要以 `try-catch` 的方式，调用业务 Service 的方法。如此，如果发生异常，回滚事务的时候，可以在 `catch` 中，更新 `id` 为 `msg` 的事务编号的记录的状态为 `RocketMQLocalTransactionState.ROLLBACK` 。😭 极端情况下，可能更新失败，则打印 error 日志，告警知道，人工介入。
4. 如此三步之后，在 `#executeLocalTransaction(...)` 方法中，就可以通过查找数据库，`id` 为 `msg` 的事务编号的记录的状态，然后返回。

**相比来说**，倾向第二种，实现更加简单通用，对于业务开发者，更加友好。

```java
@RocketMQTransactionListener(txProducerGroup = "test")
public class TransactionListenerImpl implements RocketMQLocalTransactionListener {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Override
    public RocketMQLocalTransactionState executeLocalTransaction(Message msg, Object arg) {
        // 从消息 Header 中解析到 args 参数，并使用 JSON 反序列化
        Demo01Controller.Args args = JSON.parseObject(msg.getHeaders().get("args", String.class),
                Demo01Controller.Args.class);
        // ... local transaction process, return rollback, commit or unknown
        logger.info("[executeLocalTransaction][执行本地事务，消息：{} args：{}]", msg, args);
        // 为了模拟 RocketMQ 回查 Producer 来获得事务消息的状态，所以返回了未知状态。
        return RocketMQLocalTransactionState.UNKNOWN;
    }

    @Override
    public RocketMQLocalTransactionState checkLocalTransaction(Message msg) {
        // ... check transaction status and return rollback, commit or unknown
        logger.info("[checkLocalTransaction][回查消息：{}]", msg);
        // 这里直接返回提交状态。
        return RocketMQLocalTransactionState.COMMIT;
    }
}
```

#### 简单测试

① 执行 **Consumer**Application，启动**消费者**的实例。

② 执行 **Producer**Application，启动**生产者**的实例。

之后，请求 http://127.0.0.1:18080/demo01/send_transaction 接口，发送**事务**消息。IDEA 控制台输出日志如下：

```bash
// ProduerApplication 控制台
// ### TransactionListenerImpl 执行 executeLocalTransaction 方法，先执行本地事务的逻辑
INFO 83052 --- [io-18080-exec-1] c.i.s.l.r.p.l.TransactionListenerImpl    : [executeLocalTransaction][执行本地事务，消息：GenericMessage [payload=byte[17], headers={args={"args1":1,"args2":"2"}, rocketmq_TOPIC=DEMO-TOPIC-01, rocketmq_FLAG=0, rocketmq_TRANSACTION_ID=0A258102446C18B4AAC2670C237B0000, id=d8604733-9083-5d19-15b4-bda0c549e9d1, contentType=application/json, timestamp=1582215248772}] args：Args{args1=1, args2='2'}]
// ### Producer 发送事务消息成功，但是因为 executeLocalTransaction 方法返回的是 UNKOWN 状态，所以事务消息并未提交或者回滚
// ### RocketMQ Broker 在发送事务消息 30 秒后，发现事务消息还未提交或是回滚，所以回查 Producer 。此时，checkLocalTransaction 方法返回 COMMIT ，所以该事务消息被提交

INFO 83052 --- [pool-1-thread-1] c.i.s.l.r.p.l.TransactionListenerImpl    : [checkLocalTransaction][回查消息：GenericMessage [payload=byte[17], headers={rocketmq_QUEUE_ID=0, TRANSACTION_CHECK_TIMES=1, rocketmq_BORN_TIMESTAMP=1582215248763, args={"args1":1,"args2":"2"}, rocketmq_TOPIC=DEMO-TOPIC-01, rocketmq_FLAG=0, rocketmq_MESSAGE_ID=0A25810200002A9F000000000002868F, rocketmq_TRANSACTION_ID=0A258102446C18B4AAC2670C237B0000, rocketmq_SYS_FLAG=0, id=62383992-5015-f957-41e7-75ec5ace4496, CLUSTER=DefaultCluster, rocketmq_BORN_HOST=10.37.129.2, contentType=application/json, timestamp=1582215288685}]]

// ConsumerApplication 控制台
// ### 事务消息被提交，所以该消息被 Consumer 消费
INFO 83058 --- [MessageThread_1] c.i.s.l.r.c.listener.Demo01Consumer      : [onMessage][线程编号:79 消息内容：Demo01Message{id=1950986029}]
```

## **消息过滤**

RocketMQ 提供了两种方式给 Consumer 进行消息的过滤：

- 基于 Tag 过滤

    > **标签（Tag）**：为消息设置的标志，用于**同一主题下**区分不同类型的消息。
    >
    > - 来自同一业务单元的消息，可以根据不同业务目的在同一主题下设置不同标签。
    > - 标签能够有效地保持代码的清晰度和连贯性，并优化 RocketMQ 提供的查询系统。
    > - 消费者可以根据 Tag 实现对不同子主题的不同消费逻辑，实现更好的扩展性。

- 基于 [SQL92](https://rocketmq.apache.org/rocketmq/filter-messages-by-sql92-in-rocketmq/) 过滤

消息过滤目前是在 **Broker** 端实现的，

- 优点是减少了 Broker 和 Consumer 之间的**无用消息**的网络传输，
- 缺点是增加了 Broker 的负担、而且实现相对复杂。

一般情况下，使用 Tag 过滤较多，来搭建一个 RocketMQ 使用 Tag 进行消息过滤的示例。考虑方便，直接复用[「2. 快速入门」](https://www.iocoder.cn/Spring-Cloud-Alibaba/RocketMQ/#)小节的项目：

- 修改 [`labx-06-sca-stream-rocketmq-producer-demo`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-producer-demo/) 发送**带有 Tag 的消息**。
- 从 [`labx-06-sca-stream-rocketmq-consumer-demo`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-demo/) 复制出 [`labx-06-sca-stream-rocketmq-consumer-filter`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-filter/) 来**使用 Tag 过滤消息**来消费。

### 搭建生产者

#### Controller

修改 [Demo01Controller](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-producer-demo/src/main/java/cn/iocoder/springcloudalibaba/labx6/rocketmqdemo/producerdemo/controller/Demo01Controller.java) 类，增加发送 3 条**带 Tag 的**消息的 HTTP 接口。

```java
@GetMapping("/send_tag")
public boolean sendTag() {
    for (String tag : new String[]{"yunai", "yutou", "tudou"}) {
        // 创建 Message
        Demo01Message message = new Demo01Message()
                .setId(new Random().nextInt());
        // 创建 Spring Message 对象
        Message<Demo01Message> springMessage = MessageBuilder.withPayload(message)
                .setHeader(MessageConst.PROPERTY_TAGS, tag) // <X> 设置 Tag，通过添加头 `MessageConst.PROPERTY_TAGS`，设置发送消息的 Tag。
                .build();
        // 发送消息
        mySource.demo01Output().send(springMessage);
    }
    return true;
}
```

### 搭建消费者

> RocketMQ **独有**的 **Broker** 级别的消息过滤机制

分为两种：

1. RocketMQ **独有**的 **Broker** 级别的消息过滤机制，
2. Spring Cloud Stream 提供了**通用**的 **Consumer** 级别的效率过滤器机制。

####  **Broker** 级别的消息过滤

修改 [`application.yml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-filter/src/main/resources/application.yml) 配置文件，设置 `tags` 配置项为 `yunai || yutou`，只消费带有 Tag 为 `yunai` 或 `yutou` 的消息。

- 如果想要基于 SQL92 过滤消息，可以通过设置 `sql` 配置项。

```yaml
spring:
  cloud:
    stream:
      rocketmq:
        # RocketMQ 自定义 Binding 配置项，对应 RocketMQBindingProperties Map
        bindings:
          demo01-input:
            # RocketMQ Consumer 配置项，对应 RocketMQConsumerProperties 类
            consumer:
              enabled: true # 是否开启消费，默认为 true
              broadcasting: false # 是否使用广播消费，默认为 false 使用集群消费
              tags: yunai || yutou # 基于 Tag 订阅，多个 Tag 使用 || 分隔，默认为空
              sql: # 基于 SQL 订阅，默认为空
```

#### Consumer 级别的消息过滤

- 只需要使用 `@StreamListener` 注解的 `condition` 属性，设置消息**满足指定 Spring EL 表达式**的情况下，才进行消费。

> ```
> > /**
> >  * A condition that must be met by all items that are dispatched to this method.
> >  * @return a SpEL expression that must evaluate to a {@code boolean} value.
> >  */
> > String condition() default "";
> >
> ```

修改 [Demo01Consumer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-demo/src/main/java/cn/iocoder/springcloudalibaba/labx6/rocketmqdemo/consumerdemo/listener/Demo01Consumer.java) 类，使用 `@StreamListener` 注解的 `condition` 属性来过滤消息。

```java
@Component
public class Demo01Consumer {

    private Logger logger = LoggerFactory.getLogger(getClass());

//    @StreamListener(MySink.DEMO01_INPUT)
//    public void onMessage(@Payload Demo01Message message) {
//        logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
//    }

    // 这里设置消息的 Header 带有的 `rocketmq_TAGS` 值为 `yunai` 时，才进行消费。
    @StreamListener(value = MySink.DEMO01_INPUT, condition = "headers['rocketmq_TAGS'] == 'yunai'")
    public void onMessage(@Payload Demo01Message message) {
        logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
    }
}
```

### 简单测试

① 执行 **Consumer**Application，启动**消费者**的实例。

② 执行 **Producer**Application，启动**生产者**的实例。

之后，请求 http://127.0.0.1:18080/demo01/send_tag 接口，发送带有 Tag 的消息。IDEA 控制台输出日志如下：

#### ~~Broker 级别~~

- 只消费了**两条**消息，目测 Tag 为 `tudou` 的消息已经被过滤了。

```bash
INFO 81013 --- [MessageThread_1] c.i.s.l.r.c.listener.Demo01Consumer      : [onMessage][线程编号:76 消息内容：Demo01Message{id=687868446}]
INFO 81013 --- [MessageThread_1] c.i.s.l.r.c.listener.Demo01Consumer      : [onMessage][线程编号:76 消息内容：Demo01Message{id=1088622557}]
```

#### Consumer 级别

- 只消费了**一条**消息，目测 Tag 为 `tudou` 的消息**被 Broker 过滤**，Tag 为 `yutou` 的消息**被 Consumer 过滤**。
- 要**注意**，被过滤掉的消息，后续是无法被消费掉了，效果和消费成功是一样的。

```bash
// Tag 为 `yunai` 的消息被消费
INFO 81438 --- [MessageThread_1] c.i.s.l.r.c.listener.Demo01Consumer      : [onMessage][线程编号:64 消息内容：Demo01Message{id=124549390}]

// Tag 为 `yutou` 的消息被过滤
WARN 81438 --- [MessageThread_1] .DispatchingStreamListenerMessageHandler : Cannot find a @StreamListener matching for message with id: 5edff575-b9a7-e011-154a-532077994685
```

## 死信队列



## 监控端点

Spring Cloud Stream 的 [`endpoint`](https://github.com/spring-cloud/spring-cloud-stream/blob/master/spring-cloud-stream/src/main/java/org/springframework/cloud/stream/endpoint/BindingsEndpoint.java) 模块，基于 Spring Boot Actuator，提供了自定义监控端点 [`bindings`](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-sentinel/src/main/java/com/alibaba/cloud/sentinel/endpoint/SentinelEndpoint.java) 和 [`channels`](https://github.com/spring-cloud/spring-cloud-stream/blob/master/spring-cloud-stream/src/main/java/org/springframework/cloud/stream/endpoint/ChannelsEndpoint.java)，用于获取 Spring Cloud Stream 的 Binding 和 Channel 信息。

同时，Spring Cloud Alibaba Stream RocketMQ 拓展了 Spring Boot Actuator 内置的 `health` 端点，通过自定义的 [RocketMQBinderHealthIndicator](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-stream-binder-rocketmq/src/main/java/com/alibaba/cloud/stream/binder/rocketmq/actuator/RocketMQBinderHealthIndicator.java)，获取 RocketMQ 客户端的健康状态。

> 可以后续阅读[《芋道 Spring Boot 监控端点 Actuator 入门》](http://www.iocoder.cn/Spring-Boot/Actuator/?self)文章。

来搭建一个 Stream RocketMQ 监控端点的使用示例。

- 从 [`labx-06-sca-stream-rocketmq-consumer-demo`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-demo/) 复制出 [`labx-06-sca-stream-rocketmq-consumer-actuator`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-actuator/)，查看**生产者**的监控端点结果。
- 从 [`labx-06-sca-stream-rocketmq-consumer-demo`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-demo/) 复制出 [`labx-06-sca-stream-rocketmq-consumer-filter`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-actuator/)，查看**消费者**的监控端点结果。

### 引入依赖

在 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-actuator/pom.xml) 文件中，额外引入 Spring Boot Actuator 相关依赖。

```xml
<!-- 实现对 Actuator 的自动化配置 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

#### 配置文件

修改 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-actuator/src/main/resources/application.yml) 配置文件，**额外**增加 Spring Boot Actuator 配置项。

```yaml
management:
  endpoints:
    web:
      exposure:
        include: '*' # 需要开放的端点。默认值只打开 health 和 info 两个端点。通过设置 * ，可以开放所有端点。
  endpoint:
    # Health 端点配置项，对应 HealthProperties 配置类
    health:
      enabled: true # 是否开启。默认为 true 开启。
      show-details: ALWAYS # 何时显示完整的健康信息。默认为 NEVER 都不展示。可选 WHEN_AUTHORIZED 当经过授权的用户；可选 ALWAYS 总是展示。
```

### 搭建生产者

#### 简单测试

① 使用 ProducerApplication 启动生产者。

② 访问应用的 `bindings` 监控端点 http://127.0.0.1:18080/actuator/bindings，返回结果如下图：<img src="https://static.iocoder.cn/images/Spring-Cloud-Alibaba/2020-06-01/31.png" alt=" 监控端点" style="zoom: 33%;" />

③ 访问应用的 `channels` 监控端点 http://127.0.0.1:18080/actuator/channels，返回结果如下图：<img src="https://static.iocoder.cn/images/Spring-Cloud-Alibaba/2020-06-01/32.png" alt=" 监控端点" style="zoom:50%;" />

④ 访问应用的 `health` 监控端点 http://127.0.0.1:18080/actuator/health，返回结果如下图：<img src="https://static.iocoder.cn/images/Spring-Cloud-Alibaba/2020-06-01/33.png" alt=" 监控端点" style="zoom:50%;" />

### 搭建消费者

#### 简单测试

① 使用 ConsumerApplication 启动消费者，随机端口为 19541。

② 访问应用的 `bindings` 监控端点 http://127.0.0.1:19541/actuator/bindings，返回结果如下图：<img src="https://static.iocoder.cn/images/Spring-Cloud-Alibaba/2020-06-01/34.png" alt=" 监控端点" style="zoom:33%;" />

③ 访问应用的 `channels` 监控端点 http://127.0.0.1:19541/actuator/channels，返回结果如下图：<img src="https://static.iocoder.cn/images/Spring-Cloud-Alibaba/2020-06-01/35.png" alt=" 监控端点" style="zoom: 50%;" />

④ 访问应用的 `health` 监控端点 http://127.0.0.1:19541/actuator/health，返回结果如下图：<img src="https://static.iocoder.cn/images/Spring-Cloud-Alibaba/2020-06-01/36.png" alt=" 监控端点" style="zoom:50%;" />

## 阿里云 RocketMQ 服务

> 接入阿里云的消息队列 RocketMQ

使用 Spring Cloud Alibaba Stream RocketMQ 实现阿里云 RocketMQ 的消息的发送与消费。

- 目前开源的 Java SDK 可以接入阿里云 RocketMQ 服务。

> 如果您已使用开源 Java SDK 进行生产，只需参考方法，重新配置参数，即可实现无缝上云。
>
> **前提条件**
>
> - 已在阿里云 MQ 控制台创建资源，包括 Topic、Group ID（GID）、接入点（Endpoint），以及 AccessKeyId 和 AccessKeySecret。
> - 已下载开源 RocketMQ 4.5.1 或以上版本，以支持连接阿里云 MQ。

使用阿里云 MQ 服务需要配置 AccessKey、SecretKey 以及云上的 NameServer 地址。

- Note：0.1.2 & 0.2.2 & 0.9.0 及以上才支持该功能
- Note：topic 和 group 请以 实例id% 为前缀进行配置。比如 topic 为 "test"，需要配置成 "实例id%test"
- Figure 2. NameServer 的获取(配置中请去掉 http:// 前缀)

```yaml
spring.cloud.stream.rocketmq.binder.access-key=YourAccessKey
spring.cloud.stream.rocketmq.binder.secret-key=YourSecretKey
spring.cloud.stream.rocketmq.binder.name-server=NameServerInMQ
```

![TB1vtK3cxD1gK0jSZFyXXciOVXa 1414 300](https://camo.githubusercontent.com/770bcbbdb3749073e0333b6eb5ab44f674ace9686ec49e9d581095182840124a/68747470733a2f2f696d672e616c6963646e2e636f6d2f7466732f54423176744b3363784431674b306a535a4679585863694f5658612d313431342d3330302e706e67)

搭建一个 Stream RocketMQ 监控端点的使用示例。

- 从 [`labx-06-sca-stream-rocketmq-consumer-demo`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-demo/) 复制出 [`labx-06-sca-stream-rocketmq-consumer-aliyun`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-aliyun/)，接入阿里云 RocketMQ 作为**生产者**。
- 从 [`labx-06-sca-stream-rocketmq-consumer-demo`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-demo/) 复制出 [`labx-06-sca-stream-rocketmq-consumer-aliyun`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-aliyun/)，接入阿里云 RocketMQ 作为**消费者**。

### 搭建生产者

修改 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-producer-aliyun/src/main/resources/application.yml) 配置文件，添加 `access-key`、`secret-key` 配置项，设置访问阿里云 RocketMQ 的账号。

- 注意，`<ALIYUN>` 处的三个配置项，也要修改成阿里云 RocketMQ 的 Namesrv、Topic、Producer Group。

```yaml
spring:
  application:
    name: demo-producer-application
  cloud:
    # Spring Cloud Stream 配置项，对应 BindingServiceProperties 类
    stream:
      # Binding 配置项，对应 BindingProperties Map
      bindings:
        demo01-output:
          destination: TOPIC_YUNAI_TEST # 目的地。这里使用 RocketMQ Topic <ALIYUN>
          content-type: application/json # 内容格式。这里使用 JSON
      # Spring Cloud Stream RocketMQ 配置项
      rocketmq:
        # RocketMQ Binder 配置项，对应 RocketMQBinderConfigurationProperties 类
        binder:
          name-server: onsaddr.mq-internet-access.mq-internet.aliyuncs.com:80 # RocketMQ Namesrv 地址 <ALIYUN>
          access-key: ${ALIYUN_ACCESS_KEY} # 阿里云账号 AccessKey
          secret-key: ${ALIYUN_SECRET_KEY} # 阿里云账号 SecretKey
        # RocketMQ 自定义 Binding 配置项，对应 RocketMQBindingProperties Map
        bindings:
          demo01-output:
            # RocketMQ Producer 配置项，对应 RocketMQProducerProperties 类
            producer:
              group: GID_PRODUCER_GROUP_YUNAI_TEST # 生产者分组 <ALIYUN>
              sync: true # 是否同步发送消息，默认为 false 异步。

server:
  port: 18080
```

### 搭建消费者

修改 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-06-spring-cloud-stream-rocketmq/labx-06-sca-stream-rocketmq-consumer-aliyun/src/main/resources/application.yml) 配置文件，添加 `access-key`、`secret-key` 配置项，设置访问阿里云 RocketMQ 的账号。

- 注意，`<ALIYUN>` 处的三个配置项，也要修改成阿里云 RocketMQ 的 Namesrv、Topic、Consumer Group。

```yaml
spring:
  application:
    name: demo-consumer-application
  cloud:
    # Spring Cloud Stream 配置项，对应 BindingServiceProperties 类
    stream:
      # Binding 配置项，对应 BindingProperties Map
      bindings:
        demo01-input:
          destination: TOPIC_YUNAI_TEST # 目的地。这里使用 RocketMQ Topic <ALIYUN>
          content-type: application/json # 内容格式。这里使用 JSON
          group: GID_PRODUCER_GROUP_YUNAI_TEST # 消费者分组 <ALIYUN>
      # Spring Cloud Stream RocketMQ 配置项
      rocketmq:
        # RocketMQ Binder 配置项，对应 RocketMQBinderConfigurationProperties 类
        binder:
          name-server: onsaddr.mq-internet-access.mq-internet.aliyuncs.com:80 # RocketMQ Namesrv 地址 <ALIYUN>
          access-key: ${ALIYUN_ACCESS_KEY} # 阿里云账号 AccessKey
          secret-key: ${ALIYUN_SECRET_KEY} # 阿里云账号 SecretKey
        # RocketMQ 自定义 Binding 配置项，对应 RocketMQBindingProperties Map
        bindings:
          demo01-input:
            # RocketMQ Consumer 配置项，对应 RocketMQConsumerProperties 类
            consumer:
              enabled: true # 是否开启消费，默认为 true
              broadcasting: false # 是否使用广播消费，默认为 false 使用集群消费

server:
  port: ${random.int[10000,19999]} # 随机端口，方便启动多个消费者
```

### 13.3 简单测试

① 执行 **Consumer**Application，启动**消费者**的实例。

② 执行 **Producer**Application，启动**生产者**的实例。

之后，请求 http://127.0.0.1:18080/demo01/send 接口，发送消息。IDEA 控制台输出日志如下：

```bash
// ConsumerApplication 控制台
INFO 85901 --- [MessageThread_1] c.i.s.l.r.c.listener.Demo01Consumer      : [onMessage][线程编号:89 消息内容：Demo01Message{id=-724066118}]
```

说明接入阿里云 RocketMQ 成功。



## [ Spring Cloud Alibaba 事件总线 Bus RocketMQ 入门](https://www.iocoder.cn/Spring-Cloud-Alibaba/Bus-RocketMQ/)

 [Spring Cloud Bus RocketMQ](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-starters/spring-cloud-starter-bus-rocketmq/) 组件，基于 [Spring Cloud Bus](https://github.com/spring-cloud/spring-cloud-bus) 的编程模型，接入 RabbitMQ 消息队列，实现**事件总线**的功能。

> Spring Cloud Bus 是**事件、消息总线**，用于在集群（例如，配置变化事件）中传播状态变化，可与 [Spring Cloud Config](http://www.iocoder.cn/Spring-Cloud/Spring-Cloud-Config/?self) 联合实现热部署。

在[《芋道 Spring Boot 事件机制 Event 入门》](http://www.iocoder.cn/Spring-Boot/Event/?self)文章，我们已经了解到，Spring 内置了事件机制，可以实现 **JVM 进程内**的事件发布与监听。但是如果想要**跨 JVM 进程**的事件发布与监听，此时它就无法满足我们的诉求了。

因此，Spring Cloud Bus 在 Spring 事件机制的基础之上进行**拓展**，结合 RabbitMQ、Kafka、RocketMQ 等等消息队列作为事件的**“传输器”**，通过发送事件（消息）到消息队列上，从而广播到订阅该事件（消息）的所有节点上。最终如下图所示：

![整体模型](../assets/springcloudbus01.png)



[1. 概述](http://www.iocoder.cn/Spring-Cloud-Alibaba/Bus-RocketMQ/)

[2. 快速入门](http://www.iocoder.cn/Spring-Cloud-Alibaba/Bus-RocketMQ/)

[3. 监控端点](http://www.iocoder.cn/Spring-Cloud-Alibaba/Bus-RocketMQ/)

[4. 集成到 Spring Cloud Config](http://www.iocoder.cn/Spring-Cloud-Alibaba/Bus-RocketMQ/)



## 性能测试 —— RocketMQ 基准测试

### 性能指标

以 TPS 作为我们测试的重点。当然，也和阿里云相同的计算方式，按照消息大小基数为 **1KB** 。

阿里云按照 **TPS 的峰值**进行定价，分别是（单位：月）：

- 5 千条/秒 ：29480 元
- 1 万条/秒 ：34176 元
- 5 万条/秒 ：52960 元
- 10 万条/秒 ：76440 元
- 100 万条/秒 ：1056505 元

> 并且，Producer 发送一条消息，Consumer 消费一条消息，单独计算一条，也就是 2 条！

### 测试工具

目前可用于 RocketMQ 测试的工具，暂时没有。所幸，RocketMQ 的 [benchmark](https://github.com/apache/rocketmq/tree/master/example/src/main/java/org/apache/rocketmq/example/benchmark) 包下，提供了性能基准测试的工具。

- [Producer.java](https://github.com/apache/rocketmq/blob/master/example/src/main/java/org/apache/rocketmq/example/benchmark/Producer.java) ：测试普通 MQ 生产者的性能。
- [Consumer.java](https://github.com/apache/rocketmq/blob/master/example/src/main/java/org/apache/rocketmq/example/benchmark/Consumer.java) ：测试 MQ 消费者的性能。
- [TransactionProducer.java](https://github.com/apache/rocketmq/blob/master/example/src/main/java/org/apache/rocketmq/example/benchmark/TransactionProducer.java) ：测试事务 MQ 生产者的性能。

事务消息暂时不测试，所以需要使用的就是 Producer 和 Consumer 类。

### 测试环境

我们买了两台阿里云 ECS ，用于搭建 RocketMQ 的一个**主从**集群。

会测试三轮，每一轮的目的分别是：

- 1、Producer 在**不同并发下的发送 TPS** 。此时，会使用**异步复制、异步刷盘**的 RocketMQ 集群。
- 2、Producer 在**不同集群模式下的** RocketMQ 的发送 TPS 。此时，会使用相同的并发数。
- 3、Consumer 的**消费 TPS** 。

> 注意，用于 RocketMQ 使用同一的 CommitLog 存储，所以 Topic 数量或是 Topic 的队列数，不影响 Producer 的发送 TPS 。
>
> 当然，更多的 Topic 的队列数，可以更多的 Consumer 消费，考虑到测试只使用单个 Consumer ，所以还是默认队列大小为 4 ，不进行调整。