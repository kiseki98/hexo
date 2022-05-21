---
title: RabbitMQ上手和使用
date: 2022/6/13 20:46:25
tags:
- 消息队列
- RabbitMQ
categories:
- [消息队列, RabbitMQ]
description: RabbitMQ上手和使用
---

# 消息队列

## MQ

1. 定义：消息队列，一种**应用程序对应用程序的通信**方法，通过读写出入队的消息实现通信
2. 消息队列是典型的生产者消费者模型
   1. 生产者不断向消息队列生产消息，消费者不断从消息队列中获取消息。
   2. 生产者消费者都是异步，只关心消息的发送和接收，没有业务逻辑的侵入，实现消费者和生产者解耦

## AMQP和JMS

|  平台  |                             约束                             |        消息模型         | 级别 |      |
| :----: | :----------------------------------------------------------: | :---------------------: | :--: | ---- |
| `AMQP` | `AMQP`是通过规定**协议**来统一数据格式，不规定实现方法，是跨语言的 | `AMQP`定义了5种消息模型 | 协议 |      |
| `JMS`  |   `JMS`必须使用`JAVA`语言，定义了统一接口，对消息统一操作    | `JMS`规定了2种消息模型  | 语言 |      |

## 消息确认ACK机制

1. 定义：确认字符，在数据通信中，接收站发给发送站的一种传输类控制字符，表示发来的数据确认接受无误
2. 自动ACK
   1. 开启：认为拿到消息消费掉了，无论是否出现异常都消费掉
   2. 关闭：消费者出现异常会回到`Ready`状态

# RabbitMQ五种工作模型

## Simple简单消息模型

![image-20210602211130378](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210602211130378.png)

1. 组成：生产者、消费者、消息队列
2. `Simple`简单消息模型：生产者将消息发送到队列，消费者从队列中获取消息，队列是消息的缓冲区

## Work工作消息模型

![image-20210602211149033](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210602211149033.png)

1. 组成：一个生产者、多个消费者、消息队列
2. `Work`工作消息模型，用于在多个消费者间分配任务，消息生产者性能很强，消费者较弱可能导致消息堆积，造成内存溢出
3. 主要思想：
   1. 避免执行资源密集型任务时必须等待其完成，相反我们稍后完成任务，我们将任务封装成消息将其发送到队列。
   2. 运行多消费者时，任务将在他们之间共享，消息只能被一个消费者获取，默认还是每个消费者处理相同数目消息，可以按性能分配

## 主要的三种模型

### 共同点(包含以下)

1. 生产者：一个生产者，生产者将消息发送到交换机
2. 交换机：
   1. 作用：交换机不存储消息，将消息发送到队列
   2. 交换机类型：`Fanout、Direcrt、Topic、Headers` 
3. 队列：每个消费者都要绑定消息队列，消息队列可以有多个消费者(5种模型可以嵌套)

### FanOut发布订阅模型

1. 交换机类型`Fanout`
2. 完全同共同点，没有`RoutingKey`

![image-20210602211207667](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210602211207667.png)

### Routine路由模型

1. 交换机类型`Direcrt`
2. 希望不同消息被不同的队列接受这时就要用上`Direcrt`类型的交换机
3. 消费者绑定交换机也需要指定`RoutingKey`，`RoutingKey`相同使消息定向发送
4. 消费者声明队列，绑定队列到交换机

![image-20210602211220103](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210602211220103.png)

### Topic主题模型

1. 交换机类型`Topic`
2. 在路由的基础上实现队列的`RoutingKey`通配符，消费者使用通配符
3. 通配符规则
   1. 命名：一个或多个单词组成`item.goods.insert`
   2. `#`：表示至少匹配一个单词，`item.#`可以匹配`item.goods.insert`
   3. `*`表示只匹配一个单词，`item.*`不能匹配`item.goods.insert`

![image-20210602211236156](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210602211236156.png)

# 持久化

1. 交换机持久化：channel.exchangeDeclare(交换机名，"topic"，durable：true)
2. 队列持久化：channel.queueDeclare(durable：true)
3. 消息持久化：channel.basicPublis(...，MessagePropertie.PRSISTENT_TEXT_PLAN)
4. 不管是消费者还是生产者，在声明队列或者交换机如果存在同名交换机，直接使用重名的交换机，前提是重名的交换机，队列参数一样

# API使用

## 原生API

### 获取连接

```java
public class ConnectionUtil {
    // 设置IP
    public static final String HOST = "101.37.173.191";
    // 设置端口
    public static final Integer PORT = 5673;
    // 设置虚拟主机
    public static final String VIRTUAL_HOST = "/leyou";
    // 设置账号
    public static final String USERNAME = "leyou";
    // 设置密码
    public static final String PASSWORD = "leyou";

    public static Connection getConnection() {
        Connection connection = null;
        try {
            ConnectionFactory factory = new ConnectionFactory();
            factory.setHost(HOST);
            factory.setPort(PORT);
            factory.setVirtualHost(VIRTUAL_HOST);
            factory.setUsername(USERNAME);
            factory.setPassword(PASSWORD);
            connection = factory.newConnection();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (TimeoutException e) {
            e.printStackTrace();
        }
        return connection;
    }
}
```

### 生产者

```java
public class CustomProducer {

    private static final String SIMPLE_QUEUE_NAME = "MQ_SIMPLE_QUEUE";
    private static final String SIMPLE_QUEUE_MESSAGE = "Hello World!";
    private static final String PUBLISH_SUBSCRIBE_EXCHANGE_NAME = "publish_subscribe_exchange_fanout";
    private static final String PUBLISH_SUBSCRIBE_EXCHANGE_TYPE = "fanout";

    public static void main(String[] args) {
        // 实现AutoCloseable接口可以使用try-with-resource
        try (
                //获取MQ连接
                Connection connection = ConnectionUtil.getConnection();
                //从连接中获取Channel通道对象
                Channel channel = connection.createChannel();
        ) {
            // 1.Simple、Work使用如下代码，声明Queue队列
            channel.queueDeclare(SIMPLE_QUEUE_NAME, false, false, false, null);
            // 2.FanOut、Routing、Topic使用如下代码，声明交换机
            channel.exchangeDeclare(PUBLISH_SUBSCRIBE_EXCHANGE_NAME, PUBLISH_SUBSCRIBE_EXCHANGE_TYPE);
            //发送消息到队列MQ_SIMPLE_QUEUE
            channel.basicPublish("", SIMPLE_QUEUE_NAME, null, SIMPLE_QUEUE_MESSAGE.getBytes(StandardCharsets.UTF_8));
        } catch (IOException | TimeoutException e) {
            e.printStackTrace();
        }
    }
}
```

### 消费者

**设置按性能分配**：`channel.basicQos(1);`需要关闭`ACK`

```java
public class CustomConsumer {
    private static final String SIMPLE_QUEUE_NAME = "MQ_SIMPLE_QUEUE";
    private static final String PUBLIC_SUBSCRIBE_QUEUE_NAME = "public_subscribe_queue_name01";
    private static final String PUBLISH_SUBSCRIBE_EXCHANGE_NAME = "publish_subscribe_exchange_fanout";

    public static void main(String[] args) {
        try (
                //获取MQ连接对象
                Connection connection = ConnectionUtil.getConnection();
                //创建消息通道对象
                Channel channel = connection.createChannel()
        ) {
            //	声明queue队列
            channel.queueDeclare(SIMPLE_QUEUE_NAME, false, false, false, null);
            // FanOut、Routing、Topic才需要下面代码，将队列绑定交换机 
            channel.queueBind(PUBLIC_SUBSCRIBE_QUEUE_NAME, PUBLISH_SUBSCRIBE_EXCHANGE_NAME, "");
            //创建消费者对象
            DefaultConsumer consumer = new DefaultConsumer(channel) {
                @Override
                public void handleDelivery(
                    String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body
                ) throws IOException {
                    //消息消费者获取消息
                    String message = new String(body, StandardCharsets.UTF_8);
                }
            };
            //监听消息队列
            channel.basicConsume(SIMPLE_QUEUE_NAME, true, consumer);
        } catch (IOException | TimeoutException e) {
            e.printStackTrace();
        }
    }
}
```

## Spring使用RabbitMQ

### 生产者

```java
@Controller
public class Producer {
    private static final String SIMPLE_ROUTING_KEY = "MQ_SIMPLE_QUEUE";
    private static final String SIMPLE_QUEUE_MESSAGE = "Hello World!";
    private static final String PUBLISH_SUBSCRIBE_EXCHANGE_NAME = "publish_subscribe_exchange_fanout";

    @Autowired
    private AmqpTemplate amqpTemplate;

    @PostMapping("sendMsg")
    public void send() {
        Message message = new Message(SIMPLE_QUEUE_MESSAGE.getBytes(StandardCharsets.UTF_8));
        amqpTemplate.send(PUBLISH_SUBSCRIBE_EXCHANGE_NAME, SIMPLE_ROUTING_KEY, message);
    }
}
```

### 消费者

```java
@Controller
public class Consumer {
    @Autowired
    private SearchService searchService;

    @RabbitListener(
            bindings = @QueueBinding(
                    value = @Queue(value = "LEYOU.SEARCH.SAVE.QUEUE", declare = "true"),
                    exchange = @Exchange(
                        value = "LEYOU.ITEM.EXCHANGE", 
                        ignoreDeclarationExceptions = "true", 
                        type = ExchangeTypes.TOPIC, 
                        declare = "true"),
                    key = {"item.insert", "item.update"}
            ))
    public void save(Long id) throws IOException {
        //监听保存的方法,方法的形参是消息的载体，方法形参会自动接收消息
        if (id == null) return;
        searchService.save(id);
    }
}
```

