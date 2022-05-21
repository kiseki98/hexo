---
title: 容错——Sentinel简介
date: 2022/5/20 15:46:25
tags:
- SpringCloud
- 微服务
- Sentinel
categories:
- [Spring, SpringCloud]
description: 容错——Sentinel简介
---

# Sentinel

随着微服务的流行，服务和服务之间的稳定性变得越来越重要。`Sentinel` 以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。

## Sentinel简介

![image-20210628225553115](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210628225553115.png)

`Sentinel`具有以下特征：

1. 丰富的应用场景：`Sentinel` 承接了阿里巴巴近10年的双十一大促流量的核心场景，例如秒杀（即 突发流量控制在系统容量可以承受的范围）、消息削峰填谷、集群流量控制、实时熔断下游不可用应用等。
2. 完备的实时监控：`Sentinel   ` 同时提供实时的监控功能。您可以在控制台中看到接入应用的单台机器秒级数据，甚至500台以下规模的集群的汇总运行情况。
3. 广泛的开源生态：`Sentinel` 提供开箱即用的与其它开源框架/库的整合模块，例如与 `Spring Cloud、Dubbo、gRPC` 的整合。您只需要引入相应的依赖并进行简单的配置即可快速地接入 `Sentinel`。
4. 完善的 `SPI` 扩展点：`Sentinel ` 提供简单易用、完善的 `SPI` 扩展接口。您可以通过实现扩展接口来快速地定制逻辑。例如定制规则管理、适配动态数据源等。

## Sentinel和Hystrix区别

|                |                          Sentinel                           |         Hystrix          |           resilience4j           |
| :------------: | :---------------------------------------------------------: | :----------------------: | :------------------------------: |
|    隔离策略    |                信号量隔离（并发线程数限流）                 |  线程池隔离/信号量隔离   |      线程池隔离/信号量隔离       |
|  熔断降级策略  |               基于响应时间、异常比率、异常数                |       基于异常比率       |      基于异常比率、响应时间      |
|  实时统计实现  |                    滑动窗口（LeapArray）                    | 滑动窗口（基 于 RxJava） |          Ring Bit Buffe          |
|  动态规则配置  |                        动态规则配置                         |       动态规则配置       |             有限支持             |
|    有限支持    |                         多个扩展点                          |        插件的形式        |            接口的形式            |
| 基于注解的支持 |                              √                              |            √             |                √                 |
|      限流      |             基于 `QPS`，支持基于调用关系的限流              |        有限的支持        |          `Rate Limiter`          |
|    流量整形    |           支持预热模式、匀速器模式、预热排队模式            |          不支持          |    简单的 `Rate Limiter` 模式    |
| 系统自适应保护 |                              √                              |            ×             |                ×                 |
|     控制台     | 提供开箱即用的控制台，可配置规则、 查看秒级监控、机器发现等 |      简单的监控查看      | 不提供控制台，可对接其它监控系统 |

## Sentinel名词解释

`Sentinel` 可以简单的分为 `Sentinel` 核心库和 `Dashboard`。核心库不依赖 `Dashboard` ，但是结合 `Dashboard` 可以取得最好的效果。

使用 `Sentinel` 来进行熔断保护，主要分为几个步骤：

1. 定义资源：可以是任何东西，一个服务，服务里的方法，甚至是一段代码
2. 定义规则：`Sentinel` 支持以下几种规则，`Sentinel `的所有规则都可以在内存态中动态地查询及修改，修改之后立即生效
    1. 流量控制规则
    2. 熔断降级规则
    3. 系统保护规则
    4. 来源访问控制规则
    5. 热点参数规则
3. 检验规则是否生效

**释义：**先把可能需要保护的资源定义好，之后再配置规则（动态控制，`Hystrix` 不支持）。也可以理解为，只要有了资源，我们就可以在任何时候灵活地定义各种流量控制规则。在编码的时候，只需要考虑这个代码是否需要保护，如果需要保护，就将之定义为一个资源。

## 使用Sentinel控制台

### 获取控制台

您可以从官方网站中下载最新版本的控制台`jar`包， `https://github.com/alibaba/Sentinel/releases/download/1.6.3/sentinel-dashboard-1.6.3.jar`

### 启动控制台

执行下面命令后，浏览器访问8080端口，登录 `Sentinel` 控制台，账号密码默认都是 `sentinel`

```shell
# -Dserver.port=8080 用于指定 Sentinel 控制台端口为 8080 
# -Dserver.name 用于指定项目名称
# -jar 指定jar包
java -Dserver.port=8080 -Dcsp.sentinel.dashboard.server=localhost:8080 -Dproject.name=sentineldashboard -jar sentinel-dashboard-1.6.3.jar
```

### 客户端接入控制台

**引入依赖**

```xml
<dependency>
	<groupId>com.alibaba.csp</groupId>
	<artifactId>sentinel-transport-simple-http</artifactId>
</dependency>
```

## Sentinel的依赖和配置

### 引入依赖

```xml
<!--父工程引用-->
<dependency>
	<groupId>com.alibaba.cloud</groupId>
	<artifactId>spring-cloud-alibaba-dependencies</artifactId>
	<version>2.1.0.RELEASE</version>
	<type>pom</type>
	<scope>import</scope>
</dependency>
<!--子工程引用-->
<dependency>
	<groupId>com.alibaba.cloud</groupId>
	<artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

### 客户端接入Sentinel配置

```yaml
spring:
	cloud:
		sentinel:
			transport:
				dashboard: localhost:8080 # 配置控制台请求路径
```

### Sentinel的降级配置

**通过 `Sentinel` 控制台配置降级规则**

服务重启后，控制台内配置的规则会失效，可以在本地配置，`Sentinel`控制台可以获取到这些规则

![image-20210630205155214](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210630205155214.png)

```yaml
# 通过文件读取限流规则
spring: 
	cloud:
		sentinel:
			datasource:
				ds1:
					file:
						file: classpath: flowrule.json # 表示从resources目录下的flowrule.json中获取郭泽
						data-type: json
						rule-type: flow

```

**`flowrule.json`**
> 一条限流规则主要由下面几个因素组成：配置文件添加如下配置,设置值在Java类RuleConstant中

```json
[
  {
    "resource": "orderFindById", // resource：资源名，即限流规则的作用对象
    "controlBehavior": 0,		// controlBehavior: 流量控制效果（直接拒绝、Warm Up、匀速排队）
    "count": 1,					// count: 限流阈值
    "grade": 1,					// grade: 限流阈值类型（QPS 或并发线程数）
    "limitApp": "default",		// limitApp: 流控针对的调用来源，若为 default 则不区分调用来源
    "strategy": 0				// strategy: 调用关系限流策略
  }
]
```



## 通用资源保护

```java
public class Application {

    @Autowired
    private RestTemplate restTemplate;

    /*
    * blockHandler 熔断调用的降级方法
    * fallback 抛出异常调用的降级方法
    * value 自定义资源的名称，不设置：全限定类名.方法名
    * */
    @SentinelResource(blockHandler = "breakMethod", fallback = "fallBack")
    @GetMapping("test")
    public ResponseEntity get(Long id) {
        ResponseEntity fromRemote = feignClient.getFromRemote(id);
        ResponseEntity url = restTemplate.getForObject("url", ResponseEntity.class);
        return fromRemote;
    }


    // 指定熔断方法
    public ResponseEntity breakMethod(Long id) {
        return new ResponseEntity(HttpStatus.UNAUTHORIZED);
    }

    // 指定抛出异常降级方法
    public ResponseEntity fallBack(Long id) {
        return new ResponseEntity(HttpStatus.UNAUTHORIZED);
    }
}
```

## 对RestTemplate支持

`8.6`的例子里，如果很多地方用到 `RestTemplate`，每个受保护资源都要用 `@SentinelResource(blockHandler = "breakMethod", fallback = "fallBack")`会很麻烦，`Sentinel `提供一个更加智能的保护措施

### 配置RestTemplate

```java
public class Test {

    /*
    *  Sentinel支持RestTemplate的服务调用使用Sentinel方法
    *  在构造RestTemplate对象的时候只需要加上@SentinelRestTemplate即可
    *  使用RestTemplate调用的方法都会被熔断保护
    *  资源名：原来需要在使用了RestTemplate的接口的@SentinelResource注解指定value(有默认值)作为资源名
    *		  现在使用了配置的RestTemplate，资源名默认方式：请求方式:微服务名称:请求路径，像下面两个
    *   httpmethod:schema://host:port/path ：协议、主机、端口和路径
    *   httpmethod:schema://host:port ：协议、主机和端口
    * 要求：@SentinelRestTemplate指定的熔断或者抛出异常降级类的方法是static，返回值类型为SentinelClientHttpResponse，固定4个参数
    * */
    @Bean
    @LoadBalanced
    @SentinelRestTemplate(fallback = "handleFallback",fallbackClass = ExceptionUtil.class,blockHandler = "handleBlock",blockHandlerClass = ExceptionUtil.class)
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

### 配置熔断\抛出异常降级类和方法

```java
/**
* 熔断降级和抛出异常降级
*/
public class ExceptionUtil {
	// 限流熔断业务逻辑
	public static SentinelClientHttpResponse handleBlock(
        HttpRequest request,
        byte[] body,
        ClientHttpRequestExecution execution, 
        BlockException ex) {
		System.err.println("Oops: " + ex.getClass().getCanonicalName());
		return new SentinelClientHttpResponse("限流熔断降级");
	}	

    
    // 异常熔断业务逻辑
	public static SentinelClientHttpResponse handleFallback(
        HttpRequest request, 
        byte[] body,
        ClientHttpRequestExecution execution, 
        BlockException ex) {
		System.err.println("fallback: " + ex.getClass().getCanonicalName());
		return new SentinelClientHttpResponse("异常熔断降级");
	}
}
```

## 对Feign的支持（和Hystrix基本一致）

### 引入依赖

```xml
<dependency>
	<groupId>com.alibaba.cloud</groupId>
	<artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
	</dependency>
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency
```

### 开启Sentinel支持

```yaml
feign:
	sentinel:
		enabled: true
```

### 配置FeignClient

```java
// 声明需要调用的微服务
@FeignClient(name = "服务提供者的名称", fallback = FallBack.class)
public interface IFeignClient {
    // 配置需要调用的微服务接口
    @GetMapping("/server/getInfo/{id}")
    ResponseEntity getFromRemote(@PathVariable("id") Long id);
}

// 定义FeignClient实现类
@Component
public class FallBack implements IFeignClient{
    @Override
    public ResponseEntity getFromRemote(Long id) {
        return new ResponseEntity(HttpStatus.UNAUTHORIZED);
    }
}
```