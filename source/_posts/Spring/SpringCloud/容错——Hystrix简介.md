---
title: 容错——Hystrix简介
date: 2022/5/20 14:46:25
tags:
- SpringCloud
- 微服务
- Hystrix
categories:
- [Spring, SpringCloud]
description: 容错——Hystrix简介
---

# Hystrix（Feign集成，通过AOP实现）

## 雪崩效应

![image-20210625170525718](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210625170525718.png)

一个请求可能要调用多个微服务，会形成非常复杂的调用链路，服务器支持的线程和并发数有限，由于服务的强依赖性，请求一直阻塞会导致服务器资源耗尽，形成雪崩效应。如服务C被阻塞**故障传播**，服务B迟迟等不到结果，服务A也是一样的，最终导致整个服务瘫痪、

**雪崩解决方案：**

1. 线程隔离(降级)：**降级还是会尝试请求服务**
    1. 隔离：为每个服务分配一个小线程池，一次请求卡住，还有其他线程可用，有连接超时判定
        1. 线程池满：调用立即拒绝，默认不采用排队，加速失败判定。用户请求不在直接访问服务，为是通过线程池空闲线程访问服务。线程池满进行降级处理
        2. 请求超时：直接降级处理
    2. 降级：**线程池满和请求超时触发服务降级**
        1. 服务降级：非核心不可用或弱可用（保证核心功能正常），**用户请求故障时不会阻塞**，不会无休止等待到系统崩溃
        2. 降级导致请求失败：不会导致阻塞，最多导致这个服务的线程池资源，对其它服务没影响

## 服务隔离

指将系统按照一定的原则划分为若干个服务模块，各个模块之间相对独立，无强依赖。当有故障发生时，能将问题和影响隔离在某个模块内部，不扩散风险，不波及其它模块，不影响整体的系统服务。

## 熔断降级

![屏幕截图 2021-06-28 194605](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE-2021-06-28-194605.png)
**熔断是降级的一种方式，降级是限流的一种方式**

### 熔断

这一概念来源于电子工程中的断路器 `CircuitBreaker`。在互联网系统中，当下游服务因访问压力过大而响应变慢或失败，上游服务为了保护系统整体的可用性，可以暂时切断对下游服务的调用。这种牺牲局部，保全整体的措施就叫做熔断。

### 降级

优先保证核心服务，当调用的服务因为**线程池满和请求超时触发服务降级**，在满足特定条件下触发熔断（见7.7），当某个服务熔断之后，服务器将不再被调用，此时客户端可以自己准备一个本地的 `fallback` 回调（降级），返回自定义数据，利于客观友好展示。

## Hystrix实现延迟和熔断原理

`Hystrix` 是由 `Netflix` 开源的一个延迟和容错库，用于隔离访问远程服务、服务或者第三方库，防止级联失败，提升系统的可用性与容错性。`Hystrix` 主要通过以下几点实现延迟和容错。

1. 包裹请求：使用`HystrixCommand` 包裹对依赖的调用逻辑，每个命令在独立线程中执行。这使用了设计模式中的`命令模式`。
2. 跳闸机制：当某服务的错误率超过一定的阈值时，`Hystrix `可以自动或手动跳闸，停止请求该服务一段时间。
3. 资源隔离：`Hystrix `为每个依赖都维护了一个小型的**线程池**（或者信号量）。如果该线程池已满，发往该依赖的请求就被立即拒绝，而不是排队等待，从而加速失败判定。
4. 监控：`Hystrix `可以近乎实时地监控运行指标和配置的变化，例如成功、失败、超时、以及被拒绝的请求等。
5. 回退机制：当请求失败、超时、被拒绝，或当断路器打开时，执行回退逻辑。回退逻辑由开发人员自行提供，例如返回一个缺省值。
6. 自我修复：断路器打开一段时间后，会自动进入“半开”状态。

## 对RestTemplate的支持

**调用的接口对应的服务不启动，模拟超时，这时候没有熔断，由于超时进入降级方法**

### 引用依赖

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

### 启用Hystrix

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableEurekaServer
@EnableFeignClients
// 启用Hystrix可以使用@SpringCloudApplication组合注解替代
@EnableHystrix
public class Application {

    @Autowired
    private IFeignClient feignClient;

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

// @SpringCloudApplication注解包含@SpringBootApplication、@EnableDiscoveryClient、@EnableCircuitBreaker
@SpringBootApplication
@EnableDiscoveryClient
@EnableCircuitBreaker
public @interface SpringCloudApplication {
}
```

### 配置熔断触发降级逻辑Controller层

```java
@RestController
public class DemoController {

    // 1.配置熔断的降级方法，下面是基于AOP的HystrixCommandAspect里面，
    @HystrixCommand(fallbackMethod = "getFallBack")
    @GetMapping("test")
    public ResponseEntity get() {
        ResponseEntity fromRemote = restTemplate.getForObject("http://shop-serviceproduct/product/1", Product.class);
        return fromRemote;
    }
    
    // 2.指定接口专门熔断降级方法，返回值要和被保护的方法一致，方法参数一致
    public ResponseEntity getFallBack() {
        ResponseEntity fromRemote = new ResponseEntity();
        return fromRemote;
    }
}

// 熔断的原理，AOP实现熔断
@Aspect
public class HystrixCommandAspect {

    // 切入点，被@HystrixCommand声明的方法
    @Pointcut("@annotation(com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand)")
    public void hystrixCommandAnnotationPointcut() {
    }

    @Pointcut("@annotation(com.netflix.hystrix.contrib.javanica.annotation.HystrixCollapser)")
    public void hystrixCollapserAnnotationPointcut() {
    }
}
```

### 基于RestTemplate的统一降级配置

```java
@RestController
// 指定通用熔断降级方法，返回值与接口一致，降级方法没有参数，@HystrixCommand不需要单独指定了
@DefaultProperties(defaultFallback = "getFallBack")
public class DemoController {

    @GetMapping("test1")
    public ResponseEntity get() {
        ResponseEntity fromRemote = restTemplate.getForObject("http://shop-serviceproduct/product/1", Product.class);
        return fromRemote;
    }
    
     @GetMapping("test2")
    public ResponseEntity get() {
        ResponseEntity fromRemote = restTemplate.getForObject("http://shop-serviceproduct/product/2", Product.class);
        return fromRemote;
    }
    
    // 2.指定统一熔断降级方法，返回值要和被保护的方法一致，方法参数一致
    public ResponseEntity getFallBack() {
        ResponseEntity fromRemote = new ResponseEntity();
        return fromRemote;
    }
}
```

## 对Feign的支持

**调用的接口对应的服务不启动，模拟超时，这时候没有熔断，由于超时进入降级方法**

### 引入依赖

### Feign配置开始Hystrix支持

```yaml
feign:
	hystrix: #在feign中开启hystrix熔断
		enabled: true
```

### 配置FeignClient接口和降级触发方法

缺陷是每个方法都需要定义降级方法，不能像 `RestTemplate` 一样，定义统一降级方法

```java
// 配置FeignClient接口，声明需要调用的微服务，声明降级方法的实现类（需要实现当前接口，降级方法为实现类对应方法）
@FeignClient(name = "服务提供者的名称", fallback = FallBack.class)
public interface IFeignClient {
    // 配置需要调用的微服务接口
    @GetMapping("/server/getInfo/{id}")
    ResponseEntity getFromRemote(@PathVariable("id") Long id);
}

// 声明FeignClient的降级方法，@Component需要注入到Spring容器内
@Component
public class FallBack implements IFeignClient{
    @Override
    public ResponseEntity getFromRemote(Long id) {
        return new ResponseEntity(HttpStatus.UNAUTHORIZED);
    }
}
```

## Hystrix的断路器

### 断路器三种状态

1. `CLOSED`：关闭状态（闭路），所有请求都能正常访问
2. `OPEN`：开启状态（断路），所有请求都拒绝，进入降级方法，默认持续5秒
3. `HALF OPEN`：半开状态，`OPEN `状态切换到 `HALF OPEN`，放行一个请求，成功切换到 `CLOSED`，否则切换到 `OPEN`

![image-20210628212401472](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210628212401472.png)

### 断路器配置

```yaml
hystrix:
	command:
		default:
			excation:
				isolation:
					thread:
						timeoutInMillissecond=1000 # 超时时间默认1s，规定时间内没有获取远程接口的数据，就调用降级方法
			circuitBreaker:
				requestVolumeThreshold=5 # 触发熔断的最小请求次数，默认20
				sleepWindowInMilliseconds=10000 # 熔断多少秒后去尝试请求
				errorThresholdPercentage=50 # 触发熔断的失败请求最小占比，默认50%
```

### 断路器的隔离策略

![image-20210628223515784](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210628223515784.png)

**线程池隔离策略：**

使用一个线程池来处理当前的请求，设置任务返回处理超时时间，堆积的请求堆积入线程池队列。这种方式需要为每个依赖的服务申请线程池，有一定的资源消耗，好处是可以应对突发流量（流量洪峰来临时，处理不完可将数据存储到线程池队里慢慢处理）

**信号量隔离策略：**

使用一个原子计数器（或信号量）来记录当前有多少个线程在运行，请求来先判断计数器的数值，若超过设置的最大线程个数则**丢弃**该请求，若不超过则执行计数操作，请求来计数器+1，请求返回计数器-1，这种方式是严格的控制线程且立即返回模式，无法应对突发流量（流量洪峰来临时，处理的线程超过数量，其他的请求会直接返回，不继续去请求依赖的服务）

**隔离策略配置**

```yaml
hystrix:
	command:
		default:
			execution:
				isolation:
					strategy : # 配置隔离策略
						ExecutionIsolationStrategy.SEMAPHORE # 信号量隔离
						ExecutionIsolationStrategy.THREAD #c线程池隔离
					maxConcurrentRequests : 10 #最大信号量上限
```

### Hystrix指执行过程

![hystrix执行过程](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/hystrix%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B.jpg)

## Hystrix DashBoard监控平台

我们知道，当请求失败，被拒绝，超时的时候，都会进入到降级方法中。但进入降级方法并不意味着断路器已经被打开。那么如何才能了解断路器中的状态呢？访问`http://localhost:9001/hystrix`进入监控平台

### 引入依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>
```

### 启用Hystrix DashBoard

```java
// 启动类加上@EnableHystrixDashboard注解，激活Hystrix DashBoard监控平台
@EnableHystrixDashboard
public class OrderApplication {
	public static void main(String[] args) {
		SpringApplication.run(OrderApplication.class, args);
	}
}
```

### 使用Hystrix DashBoard

进入`http://localhost:9001/hystrix`，输入`http://localhost:9001/actuator/hystrix.stream` 需要监控的服务（除了端口，固定写法），切换微服务麻烦，后面`Turbine`聚合监控更方便

![image-20210628205941974](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210628205941974.png)

**不同颜色，代表不同的指标**

![image-20210628210305474](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210628210305474.png)

## Hystrix聚合监控Turbine（需要创建微服务）

在微服务架构体系中，每个服务都需要配置`Hystrix DashBoard`监控。如果每次只能查看单个实例的监控数据，就需要不断切换监控地址，这显然很不方便。要想看这个系统的`Hystrix Dashboard`数据就需 要用到`Hystrix Turbine`。`Turbine`是一个聚合`Hystrix` 监控数据的工具，他可以将所有相关微服务的 `Hystrix` 监控数据聚合到一起，方便使用。引入`Turbine`后，整个监控系统架构如下：

![image-20210628210806692](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210628210806692.png)

### 搭建Turbine服务

### 引入依赖

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-turbine</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>
```

### 配置多个微服务Hystrix监控

```yaml
server:
	port: 8031
spring:
	application:
		name: microservice-hystrix-turbine # 服务名称
eureka:
	client:
		service-url:
			defaultZone: http://localhost:8761/eureka/ # 注册中心，从eureka获取服务列表，拿到数据
	instance:
		prefer-ip-address: true
turbine:
	appConfig: shop-service-order # 要监控的微服务列表，多个用,分隔
	clusterNameExpression: "'default'"
```

### 配置启动类

```java
@SpringBootApplication
// 启用turbine
@EnableTurbine
// 需要使用Hystrix页面监控平台，所以需要下面注解
@EnableHystrixDashboard
public class TurbineServerApplication {
	public static void main(String[] args) {
		SpringApplication.run(TurbineServerApplication.class, args);
	}
}
```

### 使用Turbine

浏览器访问搭建的`Turbine`微服务 `http://localhost:8031/hystrix` 展示`Hystrix Dashboard` ，并在`URL`位置输入 `http://local host:8031/turbine.stream`，动态根据`turbine.stream`数据展示多个微服务
