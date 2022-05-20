---
title: 链路追踪——Sleuth、Zipkin简介
date: 2022/5/20 17:46:25
tags:
- SpringCloud
- 微服务
- Sleuth
- Zipkin
categories:
- [Spring, SpringCloud]
description: 链路追踪——Sleuth、Zipkin简介
---

# 链路追踪

## 微服务架构下的问题

在大型系统的微服务化构建中，一个系统会被拆分成许多模块。这些模块负责不同的功能，组合成系 统，最终可以提供丰富的功能。在这种架构中，一次请求往往需要涉及到多个服务。互联网应用构建在 不同的软件模块集上，这些软件模块，有可能是由不同的团队开发、可能使用不同的编程语言来实现、 有可能布在了几千台服务器，横跨多个不同的数据中心，也就意味着这种架构形式也会存在一些问题：

1. 如何快速发现问题？
2. 如何判断故障影响范围？
3. 如何梳理服务依赖以及依赖的合理性？
4. 如何分析链路性能问题以及实时容量规划？

分布式链路追踪（`Distributed Tracing`），就是将一次分布式请求还原成调用链路，进行日志记录，性能监控并将一次分布式请求的调用情况集中展示。比如各个服务节点上的耗时、请求具体到达哪台机器 上、每个服务节点的请求状态等等。 目前业界比较流行的链路追踪系统如：`Twitter` 的 `Zipkin`，阿里的鹰眼，美团的`Mtrace`，大众点评的`cat`等，大部分都是基于`google`发表的`Dapper`。`Dapper`阐述了分布式系统，特别是微服务架构中链路追踪的概念、数据表示、埋点、传递、收集、存储与展示等技术细节。

## Sleuth概述

### 简介

`Spring Cloud Sleuth` 主要功能就是在分布式系统中提供追踪解决方案，并且兼容支持了 `Zipkin`，你只需要在`pom`文件中引入相应的依赖即可。

### 相关概念

`Spring Cloud Sleuth` 为`Spring Cloud`提供了分布式根据的解决方案。它大量借用了 `Google Dapper` 的设计。先来了解一下 `Sleuth` 中的术语和相关概念。`Spring Cloud Sleuth `采用的是 `Google` 的开源项目 `Dapper` 的专业术语。

1. `Span`：基本（最小）工作单元（理解为一次微服务调用），例如，在一个新建的`Span`中发送一个`RPC`等同于发送一个回应请求给`RPC`，`Span`通过一个64位`ID`唯一标识，`Trace`以另一个64位`ID`表示，`Span`还有其他数据信息，比如摘要、时间戳事件、关键值注释(`tags`)、`Span`的`ID`、以及进度`ID`(通常是IP地址)  `Span`在不断的启动和停止，同时记录了时间信息，当你创建了一个`Span`，你必须在未来的某个时刻停止它。
2. `Trace`：一系列 `Span` 组成的整个调用链路
3. `Annotation`：用来及时记录一个事件的存在，一些核心 `Annotation` 用来定义一个请求的开始和结束
    1. `cs-Client Sent` ：客户端发起一个请求，这个`Annotation`描述了这个`Span`的开始
    2. `sr-Server Received` ：服务端获得请求并准备开始处理它，如果将其`sr`减去`cs`时间戳便可得到网络延迟
    3. `ss-Server Sent` ：注解表明请求处理的完成(当请求返回客户端)，如果`ss`减去`sr`时间戳便可得 到服务端需要的处理请求时间
    4. `cr-Client Received` ：表明 `Span` 的结束，客户端成功接收到服务端的回复，如果`cr`减去`cs`时间 戳便可得到客户端从服务端获取回复的所有所需时间

![image-20210712205054095](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210712205054095.png)

## 链路追踪Sleuth入门（记录链路信息）

### 配置依赖

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
```

### 修改配置文件

每个微服务都需要添加如下的配置。启动微服务，调用之后，我们可以在控制台观察到`Sleuth`的日志输出。

![image-20210712210804731](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210712210804731.png)

```yaml
logging:
   level:
      root: INFO
      org.springframework.web.servlet.DispatcherServlet: DEBUG
      org.springframework.cloud.sleuth: DEBUG
```

其中 `ff8ff8b803a3b558` 是`TraceId`，后面跟着的是`SpanId`，依次调用有一个全局的`TraceId`，将调用链路串起来。仔细分析每个微服务的日志，不难看出请求的具体过程。 查看日志文件并不是一个很好的方法，当微服务越来越多日志文件也会越来越多，通过 `Zipkin` 可以将日志聚合，并进行可视化展示和全文检索。

## Zipkin概述（收集链路信息，友好展示）

`Zipkin` 是 `Twitter` 的一个开源项目，它基于 `Google Dapper` 实现，它致力于收集服务的定时数据，以解决微服务架构中的延迟问题，包括数据的收集、存储、查找和展现。 我们可以使用它来收集各个服务器上请求链路的跟踪数据，并通过它提供的 `REST API` 接口来辅助我们查询跟踪数据以实现对分布式系统的监控程序，从而及时地发现系统中出现的延迟升高问题并找出系统性能瓶颈的根源。除了面向开发的 `API` 接口之外，它也提供了方便的 `UI` 组件来帮助我们直观的搜索跟踪信息和分析请求链路明细，比 如：可以查询某段时间内各用户请求的处理时间等。 `Zipkin` 提供了可插拔数据存储方式：`In-Memory、MySql、Cassandra` 以及 `Elasticsearch`

![image-20210712211432522](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210712211432522.png)

上图展示了 `Zipkin` 的基础架构，它主要由 4 个核心组件构成：

1. `Collector`：收集器组件，它主要用于处理从外部系统发送过来的跟踪信息，将这些信息转换为 `Zipkin` 内部处理的 `Span` 格式，以支持后续的存储、分析、展示等功能
2. `Storage`：存储组件，它主要对处理收集器接收到的跟踪信息，默认会将这些信息存储在内存中， 我们也可以修改此存储策略，通过使用其他存储组件将跟踪信息存储到数据库中
3. `Restful API`：`API` 组件，它主要用来提供外部访问接口。比如给客户端展示跟踪信息，或是外接系统访问以实现监控等
4. `Web UI`：`UI` 组件，基于 `API` 组件实现的上层应用。通过 `UI` 组件用户可以方便而有直观地查询和 分析跟踪信息

`Zipkin` 分为两端

1. 一个是 `Zipkin` 服务端
2. 一个是 `Zipkin` 客户端
    1. 客户端也就是微服务的应用。 客户端会配置服务端的 `URL` 地址
    2. 一旦发生服务间的调用的时候，会被配置在微服务里面的 `Sleuth` 的 监听器监听
    3. 并生成相应的 `Trace` 和 `Span` 信息发送给服务端。
        1. 发送的方式主要有两种，一种是 `HTTP` 报文的方式
        2. 还有一种是消息总线的方式如 `RabbitMQ`

不论哪种方式，我们都需要：

1.  一个 `Eureka` 服务注册中心，这里我们就用之前的 `Eureka` 项目来当注册中心
2. 一个 `Zipkin` 服务端。
3. 多个微服务，这些微服务中配置 `Zipkin` 客户端

## Zipkin的部署和配置

### Zipkin下载

从`Spring Boot 2.0`开始，官方就不再支持使用自建`Zipkin Server`的方式进行服务链路追踪，而是直接提供了编译好的 `jar` 包来给我们使用。可以从官方网站下载先下载`Zipkin`的`Web UI`，我们这里下载的是 `zipkin-server-2.12.9-exec.jar`

### 启动

1. 默认 `Zipkin Server` 的请求端口为 9411
2. `Zipkin Server` 的启动参数可以通过官方提供的`yml`配置文件查找
3. 在浏览器输入` http://127.0.0.1:9411`即可进入到`Zipkin Server`的管理后台

```sh
# 在命令行输入下面命令启动Zipkin Server
java -jar zipkin-server-2.12.9-exec.jar 
```

## 客户端Zipkin+Sleuth整合

### 添加依赖

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

### 修改配置文件

指定了 `Zipkin Server` 的地址，下面制定需采样的百分比，默认为0.1，即10%，这里配置1，是记录全部的`Sleuth`信息，是为了收集到更多的数据（仅供测试用）。在分布式系统中，过于频繁的采样会影响系统性能，所以这里配置需要采用一个合适的值。

```yaml
spring:
   zipkin:
      base-url: http://127.0.0.1:9411/ #zipkin server的请求地址
         sender:
         	type: web #数据传输方式，默认以http的方式向zipkin server发送追踪数据
   sleuth:
      sampler:
         probability: 1.0 #采样的百分比
```

### 测试

以此启动每个微服务，启动`Zipkin Service`。通过浏览器发送一次微服务请求。打开 `Zipkin Service` 控制台，我们可以根据条件追踪每次请求调用过程

![image-20210712213235703](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210712213235703.png)

单击该`Trace`可以看到请求的细节

![image-20210712213254021](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210712213254021.png)

### 需要优化的问题

1. 链路数据如何保存（不做额外配置，保存在内存中，数据量大内存溢出，服务器宕机，内存数据丢失）
2. 如何优化数据采集过程（数据采集调用 `HTTP   ` 请求会占用一定资源，调用过程持续等待，同步和阻塞，拖慢核心业务，使用消息队列异步发送链路信息）

## 问题一解决：链路数据持久化（存储到数据库中）

### 准备MySQL数据库和表（Zipkin官方提供）

```sql
/*
SQLyog Ultimate v11.33 (64 bit)
MySQL - 5.5.58 : Database - zipkin
*********************************************************************
*/


/*!40101 SET NAMES utf8 */;

/*!40101 SET SQL_MODE=''*/;

/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;
CREATE DATABASE /*!32312 IF NOT EXISTS*/`zipkin` /*!40100 DEFAULT CHARACTER SET utf8 */;

USE `zipkin`;

/*Table structure for table `zipkin_annotations` */

DROP TABLE IF EXISTS `zipkin_annotations`;

CREATE TABLE `zipkin_annotations` (
  `trace_id_high` bigint(20) NOT NULL DEFAULT '0' COMMENT 'If non zero, this means the trace uses 128 bit traceIds instead of 64 bit',
  `trace_id` bigint(20) NOT NULL COMMENT 'coincides with zipkin_spans.trace_id',
  `span_id` bigint(20) NOT NULL COMMENT 'coincides with zipkin_spans.id',
  `a_key` varchar(255) NOT NULL COMMENT 'BinaryAnnotation.key or Annotation.value if type == -1',
  `a_value` blob COMMENT 'BinaryAnnotation.value(), which must be smaller than 64KB',
  `a_type` int(11) NOT NULL COMMENT 'BinaryAnnotation.type() or -1 if Annotation',
  `a_timestamp` bigint(20) DEFAULT NULL COMMENT 'Used to implement TTL; Annotation.timestamp or zipkin_spans.timestamp',
  `endpoint_ipv4` int(11) DEFAULT NULL COMMENT 'Null when Binary/Annotation.endpoint is null',
  `endpoint_ipv6` binary(16) DEFAULT NULL COMMENT 'Null when Binary/Annotation.endpoint is null, or no IPv6 address',
  `endpoint_port` smallint(6) DEFAULT NULL COMMENT 'Null when Binary/Annotation.endpoint is null',
  `endpoint_service_name` varchar(255) DEFAULT NULL COMMENT 'Null when Binary/Annotation.endpoint is null',
  UNIQUE KEY `trace_id_high` (`trace_id_high`,`trace_id`,`span_id`,`a_key`,`a_timestamp`) COMMENT 'Ignore insert on duplicate',
  KEY `trace_id_high_2` (`trace_id_high`,`trace_id`,`span_id`) COMMENT 'for joining with zipkin_spans',
  KEY `trace_id_high_3` (`trace_id_high`,`trace_id`) COMMENT 'for getTraces/ByIds',
  KEY `endpoint_service_name` (`endpoint_service_name`) COMMENT 'for getTraces and getServiceNames',
  KEY `a_type` (`a_type`) COMMENT 'for getTraces',
  KEY `a_key` (`a_key`) COMMENT 'for getTraces',
  KEY `trace_id` (`trace_id`,`span_id`,`a_key`) COMMENT 'for dependencies job'
) ENGINE=InnoDB DEFAULT CHARSET=utf8 ROW_FORMAT=COMPRESSED;

/*Data for the table `zipkin_annotations` */

/*Table structure for table `zipkin_dependencies` */

DROP TABLE IF EXISTS `zipkin_dependencies`;

CREATE TABLE `zipkin_dependencies` (
  `day` date NOT NULL,
  `parent` varchar(255) NOT NULL,
  `child` varchar(255) NOT NULL,
  `call_count` bigint(20) DEFAULT NULL,
  UNIQUE KEY `day` (`day`,`parent`,`child`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 ROW_FORMAT=COMPRESSED;

/*Data for the table `zipkin_dependencies` */

/*Table structure for table `zipkin_spans` */

DROP TABLE IF EXISTS `zipkin_spans`;

CREATE TABLE `zipkin_spans` (
  `trace_id_high` bigint(20) NOT NULL DEFAULT '0' COMMENT 'If non zero, this means the trace uses 128 bit traceIds instead of 64 bit',
  `trace_id` bigint(20) NOT NULL,
  `id` bigint(20) NOT NULL,
  `name` varchar(255) NOT NULL,
  `parent_id` bigint(20) DEFAULT NULL,
  `debug` bit(1) DEFAULT NULL,
  `start_ts` bigint(20) DEFAULT NULL COMMENT 'Span.timestamp(): epoch micros used for endTs query and to implement TTL',
  `duration` bigint(20) DEFAULT NULL COMMENT 'Span.duration(): micros used for minDuration and maxDuration query',
  UNIQUE KEY `trace_id_high` (`trace_id_high`,`trace_id`,`id`) COMMENT 'ignore insert on duplicate',
  KEY `trace_id_high_2` (`trace_id_high`,`trace_id`,`id`) COMMENT 'for joining with zipkin_annotations',
  KEY `trace_id_high_3` (`trace_id_high`,`trace_id`) COMMENT 'for getTracesByIds',
  KEY `name` (`name`) COMMENT 'for getTraces and getSpanNames',
  KEY `start_ts` (`start_ts`) COMMENT 'for getTraces ordering and range'
) ENGINE=InnoDB DEFAULT CHARSET=utf8 ROW_FORMAT=COMPRESSED;

/*Data for the table `zipkin_spans` */

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;
```

### 修改Server启动命令

```sh
# STORAGE_TYPE : 存储类型
# STORAGE_TYPE : 存储类型
# MYSQL_TCP_PORT：mysql端口
# MYSQL_TCP_PORT：mysql端口
# MYSQL_TCP_PORT：mysql端口
# MYSQL_TCP_PORT：mysql端口
java -jar zipkin-server-2.12.9-exec.jar --STORAGE_TYPE=mysql --MYSQL_HOST=127.0.0.1 --MYSQL_TCP_PORT=3306 --MYSQL_DB=zipkin --MYSQL_USER=root --MYSQL_PASS=11111
```

![image-20210712220835901](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210712220835901.png)

## 问题二解决：基于消息中间件异步收集数据（消息中间件同样需要持久化）

在默认情况下，`Zipkin` 客户端和 `Server` 之间是使用 `HTTP` 请求的方式进行通信（即同步的请求方式），在 网络波动，`Server` 端异常等情况下可能存在信息收集不及时的问题。`Zipkin` 支持与 `RabbitMQ` 整合完成异步消息传输。

![image-20210712220951606](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210712220951606.png)

### RabbitMQ的安装与启动（略）

### 服务端启动

```sh
# RABBIT_ADDRESSES ： 指定RabbitMQ地址
# RABBIT_ADDRESSES ： 指定RabbitMQ地址
# RABBIT_PASSWORD ： 密码（默认guest）
java -jar zipkin-server-2.12.9-exec.jar --RABBIT_ADDRESSES=127.0.0.1:5672
```

![image-20210712221120265](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210712221120265.png)

### 客户端配置

#### 配置依赖

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.cloud</groupId>
  	<artifactId>spring-cloud-sleuth-zipkin</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.amqp</groupId>
	<artifactId>spring-rabbit</artifactId>
</dependency>
```

#### 配置Rabbit MQ

```yaml
spring:
   zipkin: # baser-url用于链路追踪存储在内存中
      #base-url: http://127.0.0.1:9411/ #zipkin server的请求地址，存储在内存中
      sender:
         type: rabbit # 消息投递方式，rabbit
         #type: web #请求方式,默认以http的方式向zipkin server发送追踪数据
   sleuth:
      sampler:
         probability: 1.0 #采样的百分比
   rabbitmq:
      host: localhost
      port: 5672
      username: guest
      password: guest
      listener: # 这里配置了重试策略
         direct:
            retry:
               enabled: true
         simple:
            retry:
               enabled: true
```

### 测试

关闭`Zipkin Server`，并随意请求连接。打开`RabbitMQ`管理后台可以看到，消息已经推送到`RabbitMQ`。 当`Zipkin Server`启动时，会自动的从`RabbitMQ`获取消息并消费，展示追踪数据

可以看到如下效果：

1. 请求的耗时时间不会出现突然耗时特长的情况
2. 当`ZipkinServer`不可用时（比如关闭、网络不通等），追踪信息不会丢失，因为这些信息会保存在`Rabbitmq`服务器上，直到`Zipkin`服务器可用时，再从`RabbitMq`中取出这段时间的信息
