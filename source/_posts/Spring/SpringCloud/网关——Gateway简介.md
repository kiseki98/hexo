---
title: 网关——Gateway简介
date: 2022/5/20 16:46:25
tags:
- SpringCloud
- 微服务
- Gateway
categories:
- [Spring, SpringCloud]
description: 网关——Gateway简介
---

# Spring GateWay

`Zuul 1.x` 是一个基于阻塞 `IO` 的 `API Gateway` 以及  `Servlet`；直到 2018 年 5 月，`Zuul 2.x`（基于 `Netty`，也是非阻塞的，支持长连接）才发布，但`Spring Cloud  `暂时还没有整合计划。`Spring Cloud Gateway` 比 `Zuul 1.x` 系列的性能和功能整体要好。

## GateWay介绍

### 简介

是 `Spring` 官方基于 `Spring 5.0`，`Spring Boot 2.0` 和 `Project Reactor` 等技术开发的网关，旨在为微服务架构提供一种简单而有效的统一的 `API` 路由管理方式，统一访问接口。`Spring Cloud Gateway` 作为 `Spring Cloud` 生态系中的网关，目标是替代 `Netflix Zuul`，其不仅提供统一的路由方式，并且基于`Filter` 链的方式提供了网关基本的功能，例如：安全，监控/埋点，和限流等。它是基于 `Nttey` 的响应式开发模式

### 核心概念

![image-20210706212228357](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210706212228357.png)

1. 路由 `Route`：
    1. 路由是网关最基础的部分，路由信息由一个`ID`、一个目的`URL`、一组断言工厂和一组 `Filter` 组成
    2. 如果断言为真，则说明请求`URL`和配置的路由匹配
2. 断言 `Predicates`：
    1. `Java8`中的断言函数，`Spring Cloud Gateway`中的断言函数输入类型是 `Spring5.0` 框架中的`ServerWebExchange`
    2. `Spring Cloud Gateway` 中的断言函数允许开发者去定义匹配来自 `HttpRequest` 中的任何信息，比如请求头和参数等
3. 过滤器 `Filter`：是一个标准的`Spring WebFilter`，`Spring Cloud Gateway `中的 `Filter` 分为两种类型，可以对请求和响应进行处理
    1. `Gateway Filter`：针对某个微服务的过滤器
    2. `Global Filter`：针对整个系统的过滤器

## 搭建环境

### 引入依赖

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

### Gateway依赖问题

注意 `Spring Cloud Gateway` 使用的 `web` 框架为 `Webflux`，和 `SpringMVC` 依赖不兼容。引入的限流组件是 `Hystrix`，`Redis` 底层不再使用 `jedis`，而是`lettuce`。

### 配置启动类

```java
// 不需要配置多余注解
@SpringBootApplication
public class GatewayServerApplication {
	public static void main(String[] args) {
		SpringApplication.run(GatewayServerApplication.class, args);
	}
}
```

### 编写配置文件

```yaml
server:
   port: 8080 #服务端口
spring:
   application:
      name: api-gateway #指定服务名
         cloud:
            gateway:
               routes: # - 表示一个元素，接受的是数组
                  filters: xxx # 过滤规则，暂时没用。
			   - id: product-service # 我们自定义的路由 ID，保持唯一，
				 uri: http://127.0.0.1:9002 # 目标服务地址
				# 路由条件，Predicate接受一个输入参数，返回一个布尔结果。该接口包含多种默认方法来将Predicate组合成其他复杂的逻辑（与，或，非）。
			     predicates: # 断言：一组路由条件，路径匹配条件Path是其中一个
			        - Path=/product/** # 此处和Zuul不同的是，GateWay会把完整请求路径加上，/product并不会被省略
```

### 路由内置断言类型

![image-20210706221046833](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210706221046833.png)

```yaml
#路由断言之后匹配
spring:
   cloud:
      gateway:
         routes:
         - id: after_route
           uri: http://127.0.0.1:9002
           #路由断言之前匹配
           predicates:
            - Path= /product/**
            # 时间校验
            - After=xxxxx # 路由断言之后：某时间之后，如下午三点后
            - Before=xxxxxxx # 路由断言之前：某时间之前，如下午三点前
            - Between=xxxx,xxxx # 路由断言之间：某时间段，如下午三点到五点
            - Cookie=chocolate, ch.p # 路由断言Cookie匹配：此predicate匹配给定名称chocolate和值匹配的正则表达式ch.p
            - Header=X-Request-Id, \d+ # 路由断言Header匹配：header名称匹配X-Request-Id,且值匹配正则表达式\d+
            - Host=**.somehost.org,**.anotherhost.org # 路由断言匹配Host匹配：匹配下面Host主机地址,**代表可变参数
            - Method=GET # 路由断言Method匹配，匹配的是请求的HTTP方法
            - Path=/foo/{segment},/bar/{segment} # 路由断言匹配，{segment}为可变参数
            - Query=baz 或 Query=foo,ba. # 路由断言Query匹配：将请求的参数是否包含baz进行匹配，也可以进行正则表达式匹配 (参数包含foo,并且foo的值匹配ba.)
            - RemoteAddr=192.168.1.1/24 # 路由断言RemoteAddr匹配：将匹配192.168.1.1~192.168.1.254之间的ip地址，其中24为子网掩码位数即255.255.255.0                       
```

## 动态路由（面向服务路由）

搭建环境中已经展示了基本的路由配置方式，和`Zuul`一样配置`IP`和`Port`十分麻烦，下面展示简化配置方法（引入注册中心）

### 引入依赖

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

### 配置动态路由、重写转发路径、微服务名称路由转发

1. 动态路由配置：以`lb://`开头，表示从注册中心拉取请求地址
2. 重写转发路径：
    1. 配置文件中路由配置的属性`spring.cloud.gateway.routes.filters：- RewritePath=/product/(?<segment>.*), /$\{segment}`
    2. 这样还是不够方便，每个微服务都要自己配置，但是这样写可以指定局部过滤器如 `filters` 的 `RewritePath` 属性
3. 微服务名称路由转发：配置下面两项
    1. 开启：`spring.cloud.discovery.locator.enable=true`
    2. 微服务名称小写呈现：`spring.cloud.discovery.locator.lower-case-server-id=true`

```yaml
server:
   port: 8080 #服务端口
spring:
   application:
      name: api-gateway #指定服务名
   cloud:
      gateway:
         routes:
           - id: product-service
            uri: lb://shop-service-product # lb://表示从注册中心拉取微服务请求路径
            predicates:
             - Path=/product/** # 请求转发依旧会携带/product
            filters: # 路径重写过滤器，通过RewritePath配置重写转发的url，将/product-service/(?.*)，重写为{segment}
             - RewritePath=/product/(?<segment>.*), /$\{segment} # 将前面重写成后面的，省略掉/product了，yml中$要写成$\
      discovery: # 开启根据微服务名称路由转发，即使没有在上面配置路由
         locator:
            enable: true # 开始根据微服务名称进行路由转发
            lower-case-server-id: true # 因为微服务名称全是大写的，true表示微服务名称以小写形式呈现
eureka:
   client:
      serviceUrl:
         defaultZone: http://127.0.0.1:8761/eureka/
         registry-fetch-interval-seconds: 5 # 获取服务列表的周期：5s
   instance:
      preferIpAddress: true
      ip-address: 127.0.0.1
```

## 过滤器

### 过滤器生命周期

`Spring Cloud Gateway` 的 `Filter` 的生命周期不像 `Zuul` 的那么丰富，它只有两个：`pre`和 `post`

1. `PRE`：在请求被路由之前调用。我们可利用这种过滤器实现身份验证、在集群中选择请求的微服务、记录调试信息等
2. `POST`：在路由到微服务以后执行。这种过滤器可用来为响应添加标准的 `HTTP Header`、收集统计信息和指标、将响应从微服务发送给客户端等

![image-20210706230247414](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210706230247414.png)

### 过滤器类型

1. `GatewayFilter`：针对单个路由或者一个分组的路由，通常使用配置文件实现
2. `GlobalFilter`：针对所有的路由，使用较多，通常需要自定义

### 局部过滤器（通常使用内置过滤器配置）

局部过滤器`GatewayFilter`，是针对单个路由的过滤器。可以对访问的 `URL` 过滤，进行切面处理。在 `Spring Cloud Gateway `中通过 `GatewayFilter` 的形式内置了很多不同类型的局部过滤器。这里简单将  `Spring Cloud Gateway `内置的所有过滤器工厂整理成了一张表格，虽然不是很详细，但能作为速览使用。

如下：

通常使用局部过滤器都是使用内置过滤器，如下面配置文件的 `RewritePath`，表格过滤器工厂列对应 `filters` 下面的属性，过滤器工厂的类则要在加上`GatewayFilterFactory    ` 再查看

```yaml
spring:
   application:
      name: api-gateway #指定服务名
   cloud:
      gateway:
         routes:
           - id: product-service
            uri: lb://shop-service-product # lb://表示从注册中心拉取微服务请求路径
            predicates:
             - Path=/product/** # 请求转发依旧会携带/product
            filters: # 路径重写过滤器，通过RewritePath配置重写转发的url，将/product-service/(?.*)，重写为{segment}
             - RewritePath=/product/(?<segment>.*), /$\{segment} # 将前面重写成后面的，省略掉/product了，yml中$要写成$\
```

![image-20210706231155995](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210706231155995.png)

![image-20210706231209440](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210706231209440.png)

每个过滤器工厂都对应一个实现类，并且这些类的名称必须以 `GatewayFilterFactory` 结尾，这是 `Spring Cloud Gateway` 的一个约定，例如 `AddRequestHeader` 对应的实现类为 `AddRequestHeaderGatewayFilterFactory` 。对于这些过滤器的使用方式可以参考官方文档

### 全局过滤器（使用较多）

全局过滤器 `GlobalFilter` 作用于所有路由，`Spring Cloud Gateway` 定义了 `GlobalFilter` 接口，用户可以自定义实现自己的 `GlobalFilter`。通过全局过滤器可以实现对权限的统一校验，安全性验证等功能，并且全局过滤器也是程序员使用比较多的过滤器。 `Spring Cloud Gateway `内部也是通过一系列的内置全局过滤器对整个路由转发进行处理如下：

![image-20210706231817643](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210706231817643.png)

### 自定以全局过滤器

```java
// 需要实现GlobalFilter（过滤方法）和Ordered（优先级）接口，并且注入到Spring容器
@Component
public class DemoFilter implements GlobalFilter, Ordered {
    // 执行过滤器业务方法
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // 下面返回值表示继续向下执行其他过滤器
        return chain.filter(exchange);
    }

    // 指定执行顺序
    @Override
    public int getOrder() {
        return 0;
    }
}
```

## 统一鉴权

内置的过滤器已经可以完成大部分的功能，但是对于企业开发的一些业务功能处理，还是需要我们自己编写过滤器来实现的，那么我们一起通过代码的形式自定义一个过滤器，去完成统一的权限校验。

开发中的鉴权逻辑：

1. 当客户端第一次请求服务时，服务端对用户进行信息认证（登录）
2.  认证通过，将用户信息进行加密形成`token`，返回给客户端，作为登录凭证
3. 以后每次请求，客户端都携带认证的`token`服务端对`token`进行解密，判断是否有效

如下图，对于验证用户是否已经登录鉴权的过程可以在网关层统一检验。检验的标准就是请求中是否携带`token`凭证以及`token`的正确性。

![image-20210706232740195](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210706232740195.png)

**代码实现**

```java
/*
* 自定义全局过滤器需要实现GlobalFilter和Ordered接口。
* 在filter方法中完成过滤器的逻辑判断处理
* 在getOrder方法指定此过滤器的优先级，返回值越大级别越低
* ServerWebExchange就相当于当前请求和响应的上下文相当于Zuul的RequestContext，存放着重要的请求-响应属性、请求实例和响应实例等等。
* 一个请求中的request，response都可以通过ServerWebExchange获取
* 调用chain.filter继续向下游执行
*/

@Component
public class AuthorizeFilter implements GlobalFilter, Ordered {

    @Override
	public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChainchain) {
		//String url = exchange.getRequest().getURI().getPath();
		//忽略以下url请求,进行登录不需要验证
		//if(url.indexOf("/login") >= 0){
		// return chain.filter(exchange);
		// }
        String token = exchange.getRequest().getQueryParams().getFirst("token");
        if (StringUtils.isBlank(token)) {
            log.info( "token is empty ..." );
            exchange.getResponse().setStatusCode( HttpStatus.UNAUTHORIZED );
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }
    @Override
    public int getOrder() {
        return 0;
    }
}
```

## 网关限流

某个微服务压力过大，需要网关来限流，其实`Sentinel`也可以做限流，但是需要对每个资源进行限流（不能做整个服务限流），网关可以对微服务集群限流

### 常见限流算法

#### 计数器算法

1. **核心：单位时间、最大请求数（阈值）**
2. 计数器限流算法是最简单的一种限流实现方式。
3. 其本质是通过维护一个单位时间内的计数器，每次请求计数器加1
4. 当单位时间内计数器累加到大于设定的阈值，则之后的请求都被拒绝，直到单位时间已经过去，再将计数器重置为零

![image-20210707230933981](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210707230933981.png)

#### 漏桶算法（削峰填谷）

1. **核心：队列大小（漏桶大小），输出速率大小（底部开口大小）**
2. **用途：和令牌桶一致**
3. **存在缺陷：网关的漏桶算法保护别的微服务，队列大小设置不当导致队列缓存过大导致网关崩溃**
4. 漏桶算法可以很好地限制容量池的大小，从而防止流量暴增
5. 漏桶可以看作是一个带有常量服务时间的单服务器队列，如果漏桶（包缓存）溢出，那么数据包会被丢弃
6. 在网络中，漏桶算法可以控制端口的流量输出速率，平滑网络上的突发流量，实现流量整形，从而为网络提供一个稳定的流量

![image-20210707231037957](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210707231037957.png)

#### 令牌桶算法

1. **核心：令牌桶（存放令牌）、放令牌、请求获取令牌（主要保护网关自己）**
2. **用途：可以针对指定请求路径，请求头，请求参数等限流，具体请看网关限流的基于Filter限流**
3. 漏桶算法对比令牌桶算法
    1. 令牌桶算法是对漏桶算法的一种改进，漏桶算法能够限制请求调用的速率（输出速率固定），不能应对突发调用
    2. 而令牌桶算法能够在限制调用的平均速率的同时还允许一定程度的突发调用（输出速率取决于桶中令牌数量）
4. 在令牌桶算法中，存在一个桶，用来存放固定数量的令牌。
5. 算法中存在一种机制，以一定的速率往桶中放令牌。每次请求调用需要先获取令牌，才有机会继续执行，否则选择等待可用的令牌或者直接拒绝。
6. 放令牌这个动作是持续不断的进行，如果桶中令牌数达到上限，就丢弃令牌，所以就存在这种情况，桶中一直有大量的可用令牌，这时进来的请求就可以直接拿到令牌执行，比如设置`qps`为100，那么限流器初始化完成一秒后，桶中就已经有100个令牌了，这时服务还没完全启动好，等启动完成对外提供服务时，该限流器可以抵挡瞬时的100个请求。所以
7. 只有桶中没有令牌时，请求才会进行等待，最后相当于以一定的速率执行。

![image-20210707231130042](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210707231130042.png)

### 基于Filter的限流（需要Redis）

`Spring Cloud Gateway` 官方就提供了基于令牌桶的限流支持。基于其内置的过滤器工厂 `RequestRateLimiterGatewayFilterFactory` 实现。在过滤器工厂中是通过 `Redis` 和 `lua` 脚本结合的方式进行流量控制。所以需要引入 `Redis` 依赖

#### 环境搭建

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifatId>spring-boot-starter-data-redis-reactive</artifactId>
</dependency>
```

#### 配置文件

```yaml
spring:
   application:
      name: api-gateway #指定服务名
   cloud:
      gateway:
         routes:
         - id: order-service在 application.yml 中添加了redis的信息，并配置了RequestRateLimiter的限流过滤器：
           uri: lb://shop-service-order
           predicates: # 断言，判断是否转发
              - Path=/product/** # 请求转发依旧会携带/product
           filters: # 过滤器配置一系列过滤器
            - RewritePath=/order-service/(?<segment>.*), /$\{segment} # 路由重写，转发后去掉前缀
            - name: RequestRateLimiter # 局部过滤器，作为限流使用
              args:  # 使用SpEL从容器中获取对象
                 key-resolver: '#{@pathKeyResolver}'  #{@beanName}从 Spring 容器中获取 Bean 对象。用于限流的Redis的key解析器的 Bean 名称
                 redis-rate-limiter.replenishRate: 1 # 令牌桶每秒填充平均速率
                 redis-rate-limiter.burstCapacity: 3 # 令牌桶的总容量
   redis:
      host: localhost
      port: 6379
```

#### 配置KeyResolver（Redis的key的解析器）

基于什么形式，其实就是 `Redis` 中的 `Key` 存放什么，基于路径就存放路径看，基于`IP`就存放`IP`

```java
// @Bean不指定bean的名称默认是方法名
@Configuration
public class KeyResolverConfiguration {
    // 基于请求路径的限流
    @Bean
    public KeyResolver pathKeyResolver() {
        return exchange -> Mono.just(
                exchange.getRequest().getPath().toString()
        );
    }
    // 基于请求ip地址的限流
    @Bean
    public KeyResolver ipKeyResolver() {
        return exchange -> Mono.just(
                exchange.getRequest().getHeaders().getFirst("X-Forwarded-For")
        );
    }
    // 基于用户的限流
    @Bean
    public KeyResolver userKeyResolver() {
        return exchange -> Mono.just(
                exchange.getRequest().getQueryParams().getFirst("user")
        );
    }
}
```

#### 使用Redis监控key

比如针对请求路径，会把请求路径作为`Key`存放在`Redis`中

大括号中就是我们的限流`Key`,这边是`IP`，本地的就是`localhost`

1. `timestamp`：存储的是当前时间的秒数，也就是`System.currentTimeMillis() / 1000`或者 `Instant.now().getEpochSecond()`
2. `tokens`：存储的是当前这秒钟的对应的可用的令牌数量

![image-20210708221132077](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210708221132077.png)

### 基于Sentinel的限流（通过GatewayRuleManager添加限流规则）

`Sentinel` 支持对 `Spring Cloud Gateway、Zuul` 等主流的 `API Gateway` 进行限流

提供两种维度的限流

1. `Route` 维度：即在 `Spring` 配置文件中配置的路由条目，资源名为对应的路由`ID`
2. 自定义 `API` 维度：用户可以利用 `Sentinel` 提供的 `API` 来自定义一些 `API` 分组

`Sentinel 1.6.0` 引入了 `Sentinel API Gateway Adapter Common` 模块，此模块中包含网关限流的规则 和自定义 `API` 的实体和管理逻辑：

1. `GatewayFlowRule` ：网关限流规则，针对 `API Gateway` 的场景定制的限流规则，可以针对不同 `Route` 或自定义的 `API` 分组进行限流，支持针对请求中的参数、`Header`、来源 `IP` 等进行定制化的 限流。
2. `ApiDefinition` ：用户自定义的 `API` 定义分组，可以看做是一些 `URL` 匹配的组合。比如我们可以 定义一个 `API` 分组叫 `my_api` ，请求 `path` 模式为 `/foo/**` 和 `/baz/**` 的都归到 `my_api` 这个 `API` 分组下面。限流的时候可以针对这个自定义的 `API` 分组维度进行限流。

![image-20210708223203876](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210708223203876.png)

#### 环境搭建

```xml
<!-- 导入Sentinel 的响应依赖 -->
<dependency>
	<groupId>com.alibaba.csp</groupId>
	<artifactId>sentinel-spring-cloud-gateway-adapter</artifactId>
	<version>x.y.z</version>
</dependency>
```

#### 编写配置类

```java
/**
* GatewayRuleManager 添加限流规则
* GatewayCallbackManager 配置友好的限流提示信息、
* GatewayApiDefinitionManager
*/
@Configuration
public class GatewayConfiguration {

	private final List<ViewResolver> viewResolvers;

	private final ServerCodecConfigurer serverCodecConfigurer;

	public GatewayConfiguration(ObjectProvider<List<ViewResolver>> viewResolversProvider,
	                            ServerCodecConfigurer serverCodecConfigurer) {
		this.viewResolvers = viewResolversProvider.getIfAvailable(Collections::emptyList);
		this.serverCodecConfigurer = serverCodecConfigurer;
	}

	// 配置限流的异常处理器:SentinelGatewayBlockExceptionHandler
	@Bean
	@Order(Ordered.HIGHEST_PRECEDENCE)
	public SentinelGatewayBlockExceptionHandler sentinelGatewayBlockExceptionHandler() {
		return new SentinelGatewayBlockExceptionHandler(viewResolvers, serverCodecConfigurer);
	}

	// 配置限流过滤器
	@Bean
	@Order(Ordered.HIGHEST_PRECEDENCE)
	public GlobalFilter sentinelGatewayFilter() {
		return new SentinelGatewayFilter();
	}

    /**
    * 配置初始化的限流参数,整个对象创建完成执行的初始化方法，下面分别对应两种限流规则
    * 指定资源的限流规则:
    * 			1.配置资源名称（路由id）
    *			2.配置统计时间
    *			3.配置限流阈值
    */
	@PostConstruct
	public void initGatewayRules() {
		Set<GatewayFlowRule> rules = new HashSet<>();
        // GatewayFlowRule 类型：针对路由配置的规则
		rules.add(
			new GatewayFlowRule("order-service") //资源名称，路由id或者自定义 API 限流份组名称
					.setCount(1) // 限流阈值
					.setIntervalSec(1) // 统计时间窗口，单位是秒，默认是 1 秒
		);
		rules.add(new GatewayFlowRule("order-service")
				.setCount(1)
				.setIntervalSec(1)
				.setParamItem(new GatewayParamFlowItem()
						.setParseStrategy(SentinelGatewayConstants.PARAM_PARSE_STRATEGY_URL_PARAM).setFieldName("id")
				)
		);
        // ApiDefinition 类型：用户自定义限流组，下面的product_api和product_api是组名，initCustomizedApis中定义路径
		rules.add(new GatewayFlowRule("product_api")
				.setCount(1)
				.setIntervalSec(1)
		);
		rules.add(new GatewayFlowRule("order_api")
				.setCount(1)
				.setIntervalSec(1)
		);
		GatewayRuleManager.loadRules(rules);
        
        // 使用参数限流，除了路由和API分组还可以使用参数限流（IP、请求头等）
        rules.add(new GatewayFlowRule("order-service")
				.setCount(1)
				.setIntervalSec(1)
				.setParamItem(new GatewayParamFlowItem()
                  // 通过指定PARAM_PARSE_STRATEGY_URL_PARAM表示从url中获取参数，setFieldName指定参数名称
				.setParseStrategy(SentinelGatewayConstants.PARAM_PARSE_STRATEGY_URL_PARAM)
                  .setFieldName("id") // 参数名称
        );
	}
                  
    // 自定义 API 限流分组
	@PostConstruct
	private void initCustomizedApis() {
		Set<ApiDefinition> definitions = new HashSet<>();
		ApiDefinition api1 = new ApiDefinition("product_api") // 定义分组
				.setPredicateItems(new HashSet<ApiPredicateItem>() {{ // 配置限流规则
					add(new ApiPathPredicateItem().setPattern("/product-service/product/**").
							setMatchStrategy(SentinelGatewayConstants.URL_MATCH_STRATEGY_PREFIX));
				}});
		ApiDefinition api2 = new ApiDefinition("order_api")
				.setPredicateItems(new HashSet<ApiPredicateItem>() {{
					add(new ApiPathPredicateItem().setPattern("/order-service/order"));
				}});
		definitions.add(api1);
		definitions.add(api2);
		GatewayApiDefinitionManager.loadApiDefinitions(definitions);
	}

    // Sentinel 自定义异常提醒，相当于一个友好的限流提示信息
	@PostConstruct
	public void initBlockHandlers() {
		BlockRequestHandler blockRequestHandler = new BlockRequestHandler() {
			public Mono<ServerResponse> handleRequest(ServerWebExchange serverWebExchange, Throwable throwable) {
				Map map = new HashMap<>();
				map.put("code", 001);
				map.put("message", "对不起,接口限流了");
				return ServerResponse.status(HttpStatus.OK).
						contentType(MediaType.APPLICATION_JSON_UTF8).
						body(BodyInserters.fromObject(map));
			}
		};
		GatewayCallbackManager.setBlockHandler(blockRequestHandler);
	}
}
```

#### 网关配置

```yaml
server:
  port: 8080 #端口
spring:
  application:
    name: api-gateway-server #服务名称
  redis:
    host: localhost
    pool: 6379
    database: 0
  cloud: #配置SpringCloudGateway的路由
    gateway:
      routes:
      - id: product-service
        uri: lb://service-product
        predicates:
        - Path=/product-service/**
```

#### Sentinel自定义异常提醒（上面配置类中）

1. 当触发限流后页面显示的是`Blocked by Sentinel: FlowException`。为了展示更加友好的限流提示， `Sentinel`支持自定义异常处理。
2. 您可以在 `GatewayCallbackManager` 注册回调进行定制：
    1.  `GatewayCallbackManager.setBlockHandler()` ：注册函数用于实现自定义的逻辑处理被限流的请求，对应接口为 `BlockRequestHandler`
    2. 默认实现为 `DefaultBlockRequestHandler` ，当被限流时会返回类似 于下面的错误信息： `Blocked by Sentinel: FlowException`

```java
	@PostConstruct
	public void initBlockHandlers() {
		BlockRequestHandler blockRequestHandler = new BlockRequestHandler() {
			public Mono<ServerResponse> handleRequest(ServerWebExchange serverWebExchange, Throwable throwable) {
				Map map = new HashMap<>();
				map.put("code", 001);
				map.put("message", "对不起,接口限流了");
				return ServerResponse.status(HttpStatus.OK).
						contentType(MediaType.APPLICATION_JSON_UTF8).
						body(BodyInserters.fromObject(map));
			}
		};
		GatewayCallbackManager.setBlockHandler(blockRequestHandler);
	}
```

## 网关的高可用

高可用`HA`（`High Availability`）是分布式系统架构设计中必须考虑的因素之一，它通常是指，通过设计减少系统不能提供服务的时间。我们都知道，单点是系统高可用的大敌，单点往往是系统高可用最大的风险和敌人，应该尽量在系统设计的过程中避免单点。方法论上，高可用保证的原则是**集群化**，或者 叫**冗余**：只有一个单点，挂了服务会受影响；如果有冗余备份，挂了还有其他`backup`能够顶上。

![image-20210712201811154](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210712201811154.png)

我们实际使用 `Spring Cloud Gateway` 的方式如上图，不同的客户端使用不同的负载将请求分发到后端的 `Gateway`，`Gateway` 再通过`HTTP`调用后端服务，最后对外输出。因此为了保证 `Gateway` 的高可用 性，前端可以同时启动多个 `Gateway` 实例进行负载，在 `Gateway` 的前端使用 `Nginx` 或者 `F5` 进行负载 转发以达到高可用性。

### 准备多个GateWay工程

```yaml
spring:
   application:
      name: api-gateway #指定服务名
cloud:
   gateway:
      routes:
       - id: product-service
         uri: lb://shop-service-product
         predicates:
          - Path=/product-service/**
         filters:
          - RewritePath=/product-service/(?<segment>.*), /$\{segment}
eureka:
   client:
      serviceUrl:
         defaultZone: http://eureka1:8761/eureka/
         registry-fetch-interval-seconds: 5 # 获取服务列表的周期：5s
   instance:
      preferIpAddress: true
      ip-address: 127.0.0.1
```

### 配置Nginx（配置nginx.config文件）

```sh
#配置多台服务器（这里只在一台服务器上的不同端口）
upstream gateway {
	server 127.0.0.1:8081;
	server 127.0.0.1:8080;
}
#请求转向mysvr 定义的服务器列表
location / {
	proxy_pass http://gateway;
}
```