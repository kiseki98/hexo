---
title: 配置中心——Config简介
date: 2022/5/20 18:46:25
tags:
- SpringCloud
- 微服务
- 配置中心
categories:
- [Spring, SpringCloud]
description: 配置中心——Config简介
---

# Spring Cloud Config配置中心

## 配置中心基本概念

### 配置中心概述

对于传统的单体应用而言，常使用配置文件来管理所有配置，比如`SpringBoot`的`application.yml`文件， 但是在微服务架构中全部手动修改的话很麻烦而且不易维护。微服务的配置管理一般有以下需求：

1. 集中配置管理：一个微服务架构中可能有成百上千个微服务，所以集中配置管理是很重要的。
2. 不同环境不同配置：比如数据源配置在不同环境（开发，生产，测试）中是不同的。
3. 运行期间可动态调整：可根据各个微服务的负载情况，动态调整数据源连接池大小等
4. 配置修改后可自动更新：如配置内容发生变化，微服务可以自动更新配置

综上所述对于微服务架构而言，一套**统一的**，**通用的**管理配置机制是不可缺少的总要组成部分。常见的做法就是通过配置服务器进行管理。

### 常见配置中心

1. `Spring Cloud Config`为分布式系统中的外部配置提供服务器和客户端支持。
2. `Apollo`是携程框架部门研发的分布式配置中心，能够集中化管理应用不同环境、不同集群的 配置，配置修改后能够实时推送到应用端，并且具备规范的权限、流程治理等特性，适用于微服务配置 管理场景。
3. `Disconf`专注于各种「分布式系统配置管理」的「通用组件」和「通用平台」, 提供统一的「配置管理服务」包括 百度、滴滴出行、银联、网易、拉勾网、苏宁易购、顺丰科技等知名互联网公司正在使用。`Disconf` 的功能特点描述图：

## Spring Cloud Config简介

`Spring Cloud Config`项目是一个解决分布式系统的配置管理方案。它包含了 `Client` 和 `Server` 两个部分，为分布式系统中的外部配置提供服务器和客户端支持

1. `Server` 提供配置文件的存储、以接口的形式将配置文件的内容提供出去，可以为所有环境中的应用程序管理其外部属性
2. `Client ` 通过接口获取数据、并依据此数据初始化自己的应用
3. 特性：
    1. 它非常适合`Spring`应用，也可以使用在其他语言的应用上。
    2. 随着应用程序通过从开发到测试和生产的部署流程，您可以管理这些环境之间的配置，并确定应用程序具有迁移时需要运行的一切。
    3. 服务器存储后端的默认实现使用`git`，因此它轻松支持标签版本的配置环境，以及可以访问用于管理内容的各种工具。

### Spring Cloud Config服务端特性

1. `HTTP`，为外部配置提供基于资源的`API`（键值对，或者等价的 `YAML` 内容）
2. 属性值的加密和解密（对称加密和非对称加密）
3. 通过使用 `@EnableConfigServer` 在 `SpringBoot` 应用中非常简单的嵌入

### Config客户端的特性（特指Spring应用）

1. 绑定`Config`服务端，并使用远程的属性源初始化`Spring`环境
2. 属性值的加密和解密（对称加密和非对称加密）

![image-20210720223348908](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210720223348908.png)

## Spring Cloud Config入门

### 准备工作

`Config Server `是一个可横向扩展、集中式的配置服务器，它用于集中管理应用程序各个环境下的配置， 默认使用`Git`存储配置文件内容，也可以使用`SVN`存储，或者是本地文件存储

文件命名规则

1. `{applicationname}-{profile}.yml`
2. `{applicationname}-{profile}.properties`
3. `application`为应用名称`profile`指的开发环境（用于区分开发环境，测试环境、生产环境等）

![image-20210720224615605](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210720224615605.png)

### 搭建服务端

#### 引入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config-server</artifactId>
</dependency>
```

#### 配置启动类

```java
@SpringBootApplication
//开启配置中心服务端功能
@EnableConfigServer 
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

#### 配置文件

```yaml
server:
 port: 10000 #服务端口
spring:
 application:
   name: config-server #指定服务名
 cloud:
   config:
     server:
       git:
         uri: https://gitee.com/it-lemon/config-repo.git # 配置git服务地址
         username: # 配置git用户名
         password: # 配置git密码
```

#### 测试获取配置文件

启动此微服务，可以在浏览器上，通过`Server`端访问到`git`服务器上的文件，浏览器响应结果如下（`localhost:8080/application-dev.yml`）

![image-20210720230441933](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210720230441933.png)

### 搭建客户端

#### 引入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

#### 删除application.yml文件

`SpringBoot`的应用配置文件，需要通过`Config Server`获取，这里不再需要。

#### 添加bootstrap.yml

使用加载级别更高的`bootstrap.yml` 文件进行配置。启动应用时会检查此配置文件，在此文件中指定配置中心的服务地址。会自动的拉取所有应用配置并启用

```yaml
spring:
 cloud:
   config:
     name: product # 应用名称，对应git的配置文件名前半部分，参见12.3.1准备工作中的命名规则
     profile: dev # 开发环境，同上，后半部分
     label: master # git 中的分支
     uri: http://localhost:8080 # config server的请求地址
```

### 手动刷新

**存在的问题：**手动刷新虽然会刷新配置，但是每个微服务改了配置文件，都需要刷新对应的微服务很麻烦，所以后面会引入消息总线

我们已经在客户端取到了配置中心的值，但当我们修改`GitHub`上面的值时，服务端`Config Server`能实时获取最新的值，但客户端`Config Client`读的是缓存，无法实时获取最新值。`SpringCloud`已经为我们解决了这个问题，那就是客户端使用`post`去触发`refresh`，获取最新数据，需要依赖`spring-boot-starter-actuator`

#### 引入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>`spring-boot-starter-actuator</artifactId>
</dependency>
```

#### 对应Controller加上@RefreshScope

在哪里需要使用动态修改的属性就在哪个 `Controller` 添加注解 `@RefreshScope`，加上这个注解后使用post请求触发属性，被使用配置文件的属性会被刷新

```java
// 开启动态刷新，手动刷新后value的值会被刷新
@RefreshScope
@RestController
public class TestController{
    // 下面使用的会被动态修改的属性
    @Value("${productValue}")
    private String productValue;
    /**
     * 访问首页
     */
    @GetMapping("/index")
    public String index(){
        return "hello springboot！productValue：" + productValue;
   }
}
```

#### 配置文件开放端点

在 `PostMan` 中访问 `http://localhost:9002/actuator/bus-refresh `使用 `post` 提交,查看数据已经发生了变化

```yaml
spring:
 cloud:
   config:
     name: product # 应用名称，对应git的配置文件名前半部分，参见12.3.1准备工作中的命名规则
     profile: dev # 开发环境，同上，后半部分
     label: master # git 中的分支
     uri: http://localhost:8080 # config server的请求地址
management:
 endpoints:
   web:
     exposure:
       include: /bus-refresh # 开启动态刷新的请求地点
```

## 配置中心的高可用

### 服务端改造

#### 引入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

#### 配置文件

这样`Server`端的改造就完成了。先启动`Eureka`注册中心，在启动`Server`端，在浏览器中访问： `http://localhost:8761/` 就会看到`Server`端已经注册了到注册中心了

```yaml
server:
  port: 10000 #服务端口
spring:
  application:
    name: config-server #指定服务名
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/it-lemon/config-repo.git
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: guest
    password: guest
management:
  endpoints:
    web:
      exposure:
        include: bus-refresh # 开启动态刷新的请求地点，配合消息总线，实现请求服务端即可更新全部客户端
eureka:
  client:
    serviceUrl:
      defaultZone: http://127.0.0.1:8761/eureka/
  instance:
    preferIpAddress: true
    instance-id: ${spring.cloud.client.ip-address}:${server.port} #spring.cloud.client.ip-address:获取ip地址
```

### 客户端改造

#### 引入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

#### 配置文件

```yaml
spring:
 cloud:
   config:
     name: product # 应用名称，对应git的配置文件名前半部分，参见12.3.1准备工作中的命名规则
     profile: dev # 开发环境，同上，后半部分
     label: master # git 中的分支
     # uri: http://localhost:8080 config server的请求地址，由于高可用，不再使用uri了
     discovery:
     	enable: true
     	service-id: config-server # config server在eureka中的服务名
eureka:
  client:
    serviceUrl:
      defaultZone: http://127.0.0.1:8761/eureka/
  instance:
    preferIpAddress: true
    instance-id: ${spring.cloud.client.ip-address}:${server.port} #spring.cloud.client.ip-address:获取ip地址
```

## 消息总线bus（实现配置信息动态更新）

在微服务架构中，通常会使用轻量级的消息代理来构建一个共用的消息主题来连接各个微服务实例，它广播的消息会被所有在注册中心的微服务实例监听和消费，也称消息总线。 `SpringCloud`中也有对应的解决方案，`Spring Cloud Bus` 将分布式的节点用轻量的消息代理连接起来， 可以很容易搭建消息总线，配合`Spring Cloud Config` 实现微服务应用配置信息的动态更新

![image-20210720235031428](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210720235031428.png)

根据此图我们可以看出利用`Spring Cloud Bus`做配置更新的步骤

1. 提交代码触发`post`请求给`bus/refresh`
2. `Server`端接收到请求并发送给`Spring Cloud Bus`
3. `Spring Cloud Bus`接到消息并通知给其它客户端
4. 其它客户端接收到通知，请求`Server`端获取最新配置
5. 全部客户端均获取到最新的配置

## 消息总线整合配置中心

### 引入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-bus</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
</dependency>
```

### 服务端配置

```yaml
server:
 port: 10000 #服务端口
spring:
 application:
   name: config-server #指定服务名
 cloud:
   config:
     server:
       git:
         uri: https://gitee.com/it-lemon/config-repo.git
 rabbitmq:
   host: 127.0.0.1
   port: 5672
   username: guest
   password: guest
management: # 原来是config client配置这个属性，git配置文件修改，都要手动更新对应的客户端，现在引入消息总线只用手动更新服务端即可
 endpoints:
   web:
     exposure:
       include: bus-refresh
eureka:
 client:
   serviceUrl:
     defaultZone: http://127.0.0.1:8761/eureka/
 instance:
   preferIpAddress: true
   instance-id: ${spring.cloud.client.ip-address}:${server.port}
#spring.cloud.client.ip-address:获取ip地址
```

### 客户端配置（bootstrap.yml）

需要在`git`对应的配置文件中添加 `RabbitMQ` 的配置

![image-20210720235609001](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210720235609001.png)

```yaml
# bootstrap.yml配置文件
server:
 port: 9002
eureka:
 client:
   serviceUrl:
     defaultZone: http://127.0.0.1:8761/eureka/
spring:
 cloud:
   config:
     name: product
     profile: dev
     label: master
     discovery:
       enabled: true
       service-id: config-server # config server的服务名
```

重新启动对应的`eureka-server` ， `config-server` ， `product-service`，配置信息刷新后，只需要向配置中心发送对应的请求，即可刷新每个客户端的配置

