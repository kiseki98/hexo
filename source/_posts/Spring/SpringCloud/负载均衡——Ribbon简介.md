---
title: 负载均衡——Ribbon简介
date: 2022/5/20 12:46:25
tags:
- SpringCloud
- 微服务
- Ribbon
categories:
- [Spring, SpringCloud]
description: 负载均衡——Ribbon简介
---

# Ribbon（Eureka集成）

`Ribbon`是一个典型的客户端负载均衡

## 服务调用RestTemplate

1. 启用负载均衡：启动类写一个返回值为`RestTemplate`的方法，并加上`@Bean`和`@LoadBalance`注解
2. 基于`Ribbon`实现服务调用， 是通过拉取到的所有服务列表组成（服务名-请求路径的）映射关系。借助 `RestTemplate` 最终进行调用
3. 使用`RestTemplate`调用远程微服务，不再使用`Url`而是使用服务名替换`IP`地址

## 负载均衡

**客户端配置哪个服务采用哪种策略**

```yaml
# 下面是单独微服务的配置，也可以使用全局配置（避免一个微服务一个配置）
##需要调用的微服务名称
shop-service-product: # 服务名
 ribbon: #ribbon
   NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule # 负载均衡策略
```

1. 负载均衡类型：
    1. 服务端负载均衡：`Nginx（软件）`、`F5（硬件）`
    2. 客户端负载均衡：`Ribbon`
2. `Ribbon `负载均衡
    1. 原理：`Ribbon` 获取所有服务的地址（有相同服务名则可以负载均衡），根据内部负载均衡算法，获取本次请求的有效地址
    2. 负载均衡算法：实现顶级接口 `com.netflix.loadbalancer.IRule`
3. `Ribbon`负载均衡算法：
    1. 轮询策略：`com.netflix.loadbalancer.RoundRobinRule`
    2. 随机策略：`com.netflix.loadbalancer.RandomRule`
    3. 重试策略：`com.netflix.loadbalancer.RetryRule`
    4. 权重策略：`com.netflix.loadbalancer.WeightedResponseTimeRule  `会计算每个服务的权 重，越高的被调用的可能性越大
    5. 最佳策略：`com.netflix.loadbalancer.BestAvailableRule ` 遍历所有的服务实例，过滤掉故障实例，并返回请求数最小的实例返回
    6. 可用过滤策略：`com.netflix.loadbalancer.AvailabilityFilteringRule ` 过滤掉故障和请 求数超过阈值的服务实例，再从剩下的实例中轮询调用

## 策略选择

1. 如果每个机器配置一样，则建议不修改策略 (推荐)，默认轮循
2. 如果部分机器配置强，则可以改为权重策略`WeightedResponseTimeRule`

## 请求重试

```yaml
##需要调用的微服务名称
shop-service-product: # 服务名
 ribbon: #ribbon
 	ConnectTimeout: 250 # Ribbon的连接超时时间
 	ReadTimeout: 1000 # Ribbon的数据读取超时时间
 	OkToRetryOnAllOperations: true # 是否对所有操作都进行重试
 	MaxAutoRetriesNextServer: 1 # 切换实例的重试次数
 	MaxAutoRetries: 1 # 对当前实例的重试次数
```

1. 需要引入 `Spring` 重试的依赖 `spring-retry`
2. 请求超时时间（如250ms），消费者调用服务A超时没创建连接，立马调用服务B，服务A和B是同一服务的不同实例

## Ribbon源码分析

### 启动类引入spring.factories文件

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.cloud.netflix.ribbon.RibbonAutoConfiguration
```

### 配置类RibbonAutoConfiguration

注解 `@LoadBalance` 是对 `RestTemplate` 打一个标记

```java
@Configuration
@Conditional({RibbonAutoConfiguration.RibbonClassesConditions.class})
@RibbonClients
@AutoConfigureAfter(
    name = {"org.springframework.cloud.netflix.eureka.EurekaClientAutoConfiguration"}
)
// 装载此配置类之前先装载负载均衡配置 LoadBalancerAutoConfiguration 和 AsyncLoadBalancerAutoConfiguration
@AutoConfigureBefore({LoadBalancerAutoConfiguration.class, AsyncLoadBalancerAutoConfiguration.class})
@EnableConfigurationProperties({RibbonEagerLoadProperties.class, ServerIntrospectorProperties.class})
@ConditionalOnProperty(
    value = {"spring.cloud.loadbalancer.ribbon.enabled"},
    havingValue = "true",
    matchIfMissing = true
)
public class RibbonAutoConfiguration {
    // 略...
}
```

### （重要）配置类LoadBalancerAutoConfiguration

```java
// 配置类LoadBalancerAutoConfiguration的静态内部类LoadBalancerInterceptorConfig负载均衡配置
// 除了下面的负载均衡拦截器还有重试拦截器，具体看LoadBalancerAutoConfiguration源码
@Bean
@ConditionalOnMissingBean
// 重点，向RestTemplate添加一个请求拦截器，使用负载均衡后通过RestTemplate发送请求会经过新添加的那个拦截器
// LoadBalancerInterceptor负载均衡拦截器的
public RestTemplateCustomizer restTemplateCustomizer(final LoadBalancerInterceptor loadBalancerInterceptor) {
    return (restTemplate) -> {
        // 请求拦截器的List，从被@LoadBalance注解的Bean中获取
        List<ClientHttpRequestInterceptor> list = new ArrayList(restTemplate.getInterceptors());
        list.add(loadBalancerInterceptor);
        restTemplate.setInterceptors(list);
    };
}

// 在负载均衡拦截器intercept方法进行负载均衡
public class LoadBalancerInterceptor implements ClientHttpRequestInterceptor {
    public ClientHttpResponse intercept(final HttpRequest request, final byte[] body, final ClientHttpRequestExecution execution) {
        URI originalUri = request.getURI();
        String serviceName = originalUri.getHost();
        Assert.state(serviceName != null, "Request URI does not contain a valid hostname: " + originalUri);
        return (ClientHttpResponse)this.loadBalancer.execute(serviceName, this.requestFactory.createRequest(request, body, execution));
    }
}

// 负载均衡执行的代码
public class RibbonLoadBalancerClient implements LoadBalancerClient {
    public <T> T execute(String serviceId, LoadBalancerRequest<T> request, Object hint) throws IOException {
        // ILoadBalancer负载均衡器（负载均衡算法），获取待请求的服务地址
        ILoadBalancer loadBalancer = this.getLoadBalancer(serviceId);
        Server server = this.getServer(loadBalancer, hint);
        if (server == null) {
            throw new IllegalStateException("No instances available for " + serviceId);
        } else {
            RibbonLoadBalancerClient.RibbonServer ribbonServer = new RibbonLoadBalancerClient.RibbonServer(
                serviceId, server, this.isSecure(server, serviceId), this.serverIntrospector(serviceId).getMetadata(server)
            );
            return this.execute(serviceId, (ServiceInstance)ribbonServer, (LoadBalancerRequest)request);
        }
    }
}
```
