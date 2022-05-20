---
title: 消息中间件——Stream简介
date: 2022/5/20 17:46:25
tags:
- SpringCloud
- 微服务
- 消息队列
categories:
- [Spring, SpringCloud]
description: 消息中间件——Stream简介
---


# Spring Cloud Stream

存在的问题及解决：`SpringC loud Stream` **系统解耦**

1. 消息中间件主要解决应用解耦，异步消息，流量削锋等问题，实现高性能，高可用，可伸缩和最终一致性架构
2. 不同的中间件其实现方式，内部结构是不一样的。如常见的`RabbitMQ`和`Kafka`，
3. 由于这两个消息中间件的架构上的不同，像 `RabbitMQ` 有 `exchange`，`kafka` 有 `Topic`，`partitions` 分区
4. 如果用了两个消息队列的其中一种，后面的业务需求，想往另外一种消息队列进行迁移，这时候无疑就是一个灾难性的，一大堆东西都要重新推倒重新做，因为它跟我们的系统**耦合了**，这时候 `Spring Cloud Stream` 给我们提供了一种解耦合的方式

## 概述

1. `Spring Cloud Stream`由一个中间件中立的核组成。
2. 应用通过`Spring Cloud Stream`插入的`input`(相当于消费者`consumer`，它是从队列中接收消息的)和`output`(相当于生产者`producer`，它是从队列中发送消息的。)通道与外界交流。
3. 通道通过指定中间件的`Binder`实现与外部代理连接。业务开发者不再关注具体消息中间件，只需关注`Binder`对应用程序提供的抽象概念来使用消息中间件实现业务即可。

![image-20210719213600011](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210719213600011.png)

## 核心概念（绑定器Binder）

### 绑定器（不同消息中间件不同绑定器）

1. `Binder`绑定器是`Spring Cloud Stream`中一个非常重要的概念。
    1. 在没有绑定器这个概念的情况下，我们的`SpringBoot`应用要直接与消息中间件进行信息交互
    2. 由于各消息中间件构建的初衷不同，它们的实现细节上会有较大的差异性，这使得我们实现的消息交互逻辑就会非常笨重
    3. 因为对具体的中间件实现细节有太重的依赖，当中间件有较大的变动升级、或是更换中间件的时候，我们就需要付出非常大的代价来实施
2. 通过定义绑定器作为中间层，实现了应用程序与消息中间件`Middleware`细节之间的隔离。
    1. 通过向应用程序暴露统一的`Channel`通道，使得应用程序不需要再考虑各种不同的消息中间件的实现。
    2. 当需要升级消息中间件，或者是更换其他消息中间件产品时，我们需要做的就是更换对应的`Binder`绑定器而不需要修改任何应用逻辑
3. 通过配置把应用和`Spring Cloud Stream` 的 `Binder`绑定在一起，之后只需要修改 `Binder`的配置来达到动态修改`topic`、`exchange`、`type`等一系列信息

### 发布订阅模型

1. 在`Spring Cloud Stream`中的消息通信方式遵循了发布-订阅模式
2. 当一条消息被投递到消息中间件之 后，它会通过共享的`Topic`主题进行广播，消息消费者在订阅的主题中收到它并触发自身的业务逻辑处理。
3. 这里所提到的 `Topic` 主题是`Spring Cloud Stream`中的一个抽象概念，用来代表发布共享消息给消费者的地方。
4. 在不同的消息中间件中， `Topic` 可能对应着不同的概念，比如：在`RabbitMQ`中的它对应 了`Exchange`、而在`Kakfa`中则对应了 `Kafka` 中的 `Topic`。

![image-20210719214352099](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210719214352099.png)

## 入门案例

消费者通道和生产者通道是不一样的，如上如发布订阅，配置通道的`destination`相同，可以实现不同通道的消息的生产和消费

### 引入依赖

```xml
<dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-stream</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
</dependency>
```

### 消息生产者

#### 定义通道binding（内置通道output）

```java
// 发送消息时需要定义一个接口，不同的是接口方法的返回对象是 MessageChannel，下面是 Spring Cloud Stream 内置的接口
public interface Source {
    // 这就接口声明了一个 通道 binding 名为 “output”。这个binding 声明了一个消息输出流，也就是消息的生产者
    String OUTPUT = "output";
    @Output("output")
    MessageChannel output();
}
```

#### 配置application.yml

```yaml
server:
  port: 7001 #服务端口
spring:
  application:
    name: rabbitmq-producer #指定服务名
  rabbitmq:
    addresses: 127.0.0.1
    username: itcast
    password: itcast
    virtual-host: myhost
  cloud:
    stream:
      bindings:
        output: # 通道的名称，通道不同于RabbitMQ的队列，如上面的binding接口Source声明名称为output
          destination: itcast-default  # 指定了消息发送的目的地，对应RabbitMQ，会发送到exchange是itcast-default的所有消息队列中。
          contentType: text/plain # 用于指定消息的类型。具体可以参考 spring cloud stream docs
        outputOrder: # 通道名称
          destination: testChannel
          producer:
            partition-key-expression: payload
            partition-count: 2
      binders:
        defaultRabbit:
          type: rabbit
```

#### 发送消息（通过MessageChannel）

```java
@SpringBootApplication
// 启用消息通道，只要在哪个类使用MessageChannel就需要下面的注解
@EnableBinding(Source.class)
public class Application implements CommandLineRunner {
    // 通过下面的MessageChannel发送消息会发送到配置的spring.cloud.stream.bindings下的output（通道名为output的配置）
    @Autowired
    @Qualifier("output")
    MessageChannel output;
    
    @Override
    public void run(String... strings) throws Exception {
        //发送MQ消息，实现了CommandLineRunner接口，服务启动就会执行run方法，通过内置工具类MessageBuilder创建消息
        output.send(MessageBuilder.withPayload("hello world").build());
   }
    
    public static void main(String[] args) {
        SpringApplication.run(Application.class);
   }
}
```

### 消息消费者

#### 定义通道bingding（内置通道input）

```java
// 同发送消息一致，在Spring Cloud Stream中接受消息，需要定义一个接口，如下是内置的一个接口
public interface Sink {
    String INPUT = "input";
    
    // 注释@Input对应的方法，需要返回SubscribableChannel ，并且参入一个参数值,这就接口声明了一个通道 binding 命名为 “input” 
    @Input("input")
    SubscribableChannel input();
}
```

#### 配置application.yml

```yaml
server:
  port: 7003 #服务端口
spring:
  application:
    name: rabbitmq-consumer #指定服务名
  rabbitmq:
    addresses: 127.0.0.1
    username: itcast
    password: itcast
    virtual-host: myhost
  cloud:
    stream:
      instanceCount: 2
      instanceIndex: 1
      bindings:
        input: # 通道的名称，通道不同于RabbitMQ的队列，如上面的binding接口Sink声明名称为input
          destination: itcast-default # 指定了消息获取的目的地，对应于MQ就是 exchange，这里的exchange就是itcast-default
        inputOrder: # 通道名称
          destination: testChannel
          group: group-2
          consumer:
            partitioned: true
      binders:
        defaultRabbit:
          type: rabbit
```

#### 测试消息消费

1. 定义一个`class` ，并且添加注解`@EnableBinding(Sink.class)` ，其中`Sink`就是上述的接口。同时定义一个方法（此处是 `input`）标明注解为 `@StreamListener(Processor.INPUT)`，方法参数为 `Message`
2. 所有发送 `exchange` 为`itcast-default`的`MQ`消息都会被投递到这个临时队列，并且触发上述的方法

```java
@SpringBootApplication
// 启用消息通道，只要在哪个类使用@StreamListener就需要下面的注解
@EnableBinding(Sink.class)
public class Application {
    // 监听通道 binding 为 Sink.INPUT 的消息，监听方法使用@StreamListener修饰
    @StreamListener(Sink.INPUT)
    public void input(Message<String> message) {
        System.out.println("监听收到：" + message.getPayload());
   }
    public static void main(String[] args) {
        SpringApplication.run(Application.class);
   }
}
```

### 自定义消息通道binding

```java
// 消息通道的配置同之前的
public interface OrderProcessor {
    String INPUT_ORDER = "inputOrder";
    String OUTPUT_ORDER = "outputOrder";
    
    // @Input指定输入通道名称
    @Input(INPUT_ORDER)
    SubscribableChannel inputOrder();
    
    // @Output指定输出通道名称
    @Output(OUTPUT_ORDER)
    MessageChannel outputOrder();
}
```

### 消息分组（解决同一个消息被多个消费者消费）

1. 通常在生产环境，我们的每个服务都不会以单节点的方式运行在生产环境
2. 当同一个服务启动多个实例的时候，这些实例都会绑定到同一个消息通道的目标主题`Topic`上（相当于通道配置中的`destination`）
3. 默认情况下，当生产者发出一条消息到绑定通道上，这条消息会产生多个副本被每个消费者实例接收和处理
4. 但是有些业务场景之下，我们希望生产者产生的消息只被其中一个实例消费，这个时候我们需要为这些消费者设置消费组来实现这样的功能
5. 解决方案：设置分组，只需要在服务消费者端设置 `spring.cloud.stream.bindings.input.group` 属性即可
    1. 在`bindings`通道配置下配置`group`
    2. 在同一个`group`中的多个消费者只有一个可以获取到消息并消费

```yaml
server:
 port: 7003 #服务端口
spring:
 application:
   name: rabbitmq-consumer #指定服务名
 rabbitmq:
   addresses: 127.0.0.1
   username: itcast
   password: itcast
   virtual-host: myhost
 cloud:
   stream:
     bindings:
       input:
         destination: itcast-default
       inputOrder:
         destination: testChannel
         group: group-2 # 设置分组
     binders:
       defaultRabbit:
         type: rabbit
```



![image-20210720001316713](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210720001316713.png)

### 消息分区

1. 有一些场景需要满足，同一个特征的数据被同一个实例消费
2. 比如同一个`id`的传感器监测数据必须被同一 个实例统计计算分析, 否则可能无法获取全部的数据
3. 又比如部分异步任务，首次请求启动`task`，第二次请求取消`task`，此场景就必须保证两次请求至同一实例

![image-20210720002906721](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210720002906721.png)

#### 消费者配置

从下面的配置中，我们可以看到增加了这三个参数

1. `spring.cloud.stream.bindings.input.consumer.partitioned` ：通过该参数开启消费者分区功能
2.  `spring.cloud.stream.instanceCount` ：该参数指定了当前消费者的总实例数量
3. `spring.cloud.stream.instanceIndex` ：该参数设置当前实例的索引号，从0开始

```yaml
cloud:
   stream:
     instance-count: 2 # 消费者实例数
     instance-index: 0 # 当前索引
     bindings:
       input:
         destination: itcast-default
       inputOrder:
         destination: testChannel
         group: group-2
         consumer:
           partitioned: true # 开启分区支持
     binders:
       defaultRabbit:
         type: rabbit
```

#### 生产者配置

从下面的配置中，我们可以看到增加了这两个参数：

1. `spring.cloud.stream.bindings.output.producer.partitionKeyExpression` ：通过该参数指定了分区键的表达式规则，我们可以根据实际的输出消息规则来配置`SpEL`来生成合适的分区键
2. `spring.cloud.stream.bindings.output.producer.partitionCount` ：该参数指定了消息分 区的数量

```yaml
spring:
 application:
   name: rabbitmq-producer #指定服务名
 rabbitmq:
   addresses: 127.0.0.1
   username: itcast
   password: itcast
   virtual-host: myhost
 cloud:
   stream:
     bindings:
       input:
         destination: itcast-default
         producer:
           partition-key-expression: payload # 分区关键字
           partition-count: 2
     binders:
       defaultRabbit:
         type: rabbit
```

#### 完成配置

1. 到这里消息分区配置就完成了，我们可以再次启动这两个应用，同时消费者启动多个
2. 但需要注意的是要为消费者指定不同的实例索引号，这样当同一个消息被发给消费组时，我们可以发现只有一个消费实例在接收和处理这些相同的消息
