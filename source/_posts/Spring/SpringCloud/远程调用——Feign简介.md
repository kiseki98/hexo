---
title: 远程调用——Feign简介
date: 2022/5/20 13:46:25
tags:
- SpringCloud
- 微服务
- Feign
categories:
- [Spring, SpringCloud]
description: 远程调用——Feign简介
---

# Feign（停止维护）：整合了Ribbon、Hystrix

## 特性

原来存在的问题：使用 `DiscoveryClient` 拉取服务列表，使用 `RestTemplate` 在 `URL` 中拼接服务名，调用其他服务接口，参数拼接复杂

1. 声明式，模板化的 `HTTP` 客户端
2. `Feign` 可帮助我们更加便捷，优雅的调用 `HTTP API`
3. 在 `SpringCloud` 中，使用 `Feign` 非常简单——创建一个接口，并在接口上添加一些注解，代码就完成了。
4. `Feign  `支持多种注解，例如 `Feign` 自带的注解或者`JAX-RS`注解等。
5. `SpringCloud` 对 `Feign` 进行了增强，使 `Feign` 支持了 `SpringMVC` 注解，并整合了 `Ribbon` 和 `Eureka`， 从而让 `Feign` 的使用更加方便。

## 使用Feign（服务调用）

### 导入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

### 配置接口

```java
// @FeignClient声明需要调用的微服务
@FeignClient(name = "服务提供者的名称")
public interface IFeignClient {
    // 配置需要调用的微服务接口,需要和调用的接口一致
    @GetMapping("/server/getInfo/{id}")
    ResponseEntity getFromRemote(@PathVariable("id") Long id);
}
```

### 启用Feign

```java
// 启动类使用注解
@EnableFeignClients
```

### 使用Feign接口调用远程接口

```java
@SpringBootApplication
@EnableFeignClients
public class Application {
    @Autowired
    private IFeignClient feignClient;

    @GetMapping("test")
    public ResponseEntity get() {
        ResponseEntity fromRemote = feignClient.getFromRemote(1L);
        return fromRemote;
    }
}
```

## Feign的配置

### 基本配置

```yaml
feign:
	client:
		config:
			feignName: ##定义FeginClient的名称
				connectTimeout: 5000 # 建立链接的超时时长，相当于Request.Options
				readTimeout: 5000 # 读取超时时长，相当于Request.Options
				loggerLevel: full # 配置Feign的日志级别，相当于代码配置方式中的Logger
				errorDecoder: com.example.SimpleErrorDecoder # Feign的错误解码器，相当于代码配置方式中的ErrorDecoder
				retryer: com.example.SimpleRetryer # 配置重试，相当于代码配置方式中的Retryer
				requestInterceptors: # 配置拦截器，相当于代码配置方式中的RequestInterceptor
					- com.example.FooRequestInterceptor
					- com.example.BarRequestInterceptor
				decode404: false
```

### 请求压缩

`Feign` 支持对请求和响应进行`GZIP`压缩，减少通信过程中的性能损耗

```yaml
feign:
	compression:
		request:
			enabled: true # 开启请求压缩
			mime-types: text/html,application/xml,application/json # 设置压缩的数据类型
			min-request-size: 2048 # 设置触发压缩的大小下限KB
		response:
			enabled: true # 开启响应压缩

```

### 日志配置

```yaml
feign:
  client:
    config:
	  shop-service-product: # 需要调用的微服务名称
	    loggerLevel: FULL # NONE：不输出日志，性能最高 BASIC：适合生产追踪问题 HEADERS：BASIC基础上记录请求和响应头信息 FULL：记录所有
  logging:
	  level:
	    cn.itcast.order.fegin.ProductFeginClient: debug
```

## 源码分析

### Feign的自动配置

所有的配置都在`@EnableFeignClients`上

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
@Import({FeignClientsRegistrar.class})
public @interface EnableFeignClients {
    String[] value() default {};

    String[] basePackages() default {};

    Class<?>[] basePackageClasses() default {};

    Class<?>[] defaultConfiguration() default {};

    Class<?>[] clients() default {};
}
```

### 配置类FeignClientsRegistrar

```java
// Spring中实现ImportBeanDefinitionRegistrar接口Spring会自动调用registerBeanDefinitions方法
class FeignClientsRegistrar implements ImportBeanDefinitionRegistrar, ResourceLoaderAware, EnvironmentAware {
    // 下面方法被Spring自动调用
    public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
        this.registerDefaultConfiguration(metadata, registry);
        // 重点方法，扫描并配置注解@FeignClient声明的类，注册到Spring容器中
        this.registerFeignClients(metadata, registry);
    }
}
```

### 创建并注册FeignClientFactoryBean对象

```java
public class FeignClientFactoryBean implements FactoryBean<Object>, InitializingBean, ApplicationContextAware, BeanFactoryAware {
    // 实现FactoryBean的重要方法
    public Object getObject() {
        return this.getTarget();
    }
    
    // getObject进入下面
    <T> T getTarget() {
            Targeter targeter = (Targeter)this.get(context, Targeter.class);
            return targeter.target(this, builder, context, new HardCodedTarget(this.type, this.name, url));
        } else {
            if (!this.name.startsWith("http")) this.url = "http://" + this.name;
            else this.url = this.name;
            this.url = this.url + this.cleanPath();
            return this.loadBalance(builder, context, new HardCodedTarget(this.type, this.name, this.url));
        }
    }
}

// getTarget最终进入下面
 public <T> T target(Target<T> target) {
            return this.build().newInstance(target);
 }
// newInstance进入下面返回FeignInvocationHandler
public <T> T newInstance(Target<T> target) {
        Map<String, MethodHandler> nameToHandler = this.targetToHandlersByName.apply(target);
        Map<Method, MethodHandler> methodToHandler = new LinkedHashMap();
        List<DefaultMethodHandler> defaultMethodHandlers = new LinkedList();
        Method[] var5 = target.type().getMethods();
        int var6 = var5.length;
        for(int var7 = 0; var7 < var6; ++var7) {
            Method method = var5[var7];
            if (method.getDeclaringClass() != Object.class) {
                if (Util.isDefault(method)) {
                    DefaultMethodHandler handler = new DefaultMethodHandler(method);
                    defaultMethodHandlers.add(handler);
                    methodToHandler.put(method, handler);
                } else methodToHandler.put(method, nameToHandler.get(Feign.configKey(target.type(), method)));
            }
        }

    	// 返回FeignInvocationHandler代理对象
        InvocationHandler handler = this.factory.create(target, methodToHandler);
        T proxy = Proxy.newProxyInstance(target.type().getClassLoader(), new Class[]{target.type()}, handler);
        Iterator var12 = defaultMethodHandlers.iterator();

        while(var12.hasNext()) {
            DefaultMethodHandler defaultMethodHandler = (DefaultMethodHandler)var12.next();
            defaultMethodHandler.bindTo(proxy);
        }
        return proxy;
}
```

### 源码解析

1. 扫描 `@FeignClient` 声明的接口
2. 创建 `@FeignClient` 声明接口的动态代理对象 `FeignInvocationHandler`
3. `@FeignClient` 声明的接口最终通过代理对象调用远程接口

## JMetter模拟高并发

### 调整Tomcat参数

1. `server.tomcat.max-threads=20` 调整 `Tomcat` 最大线程数，决定了应用服务同时可以处理多少个 `HTTP` 请求，`Tomcat `默认为200
2. `server.tomcat.accept-count=80 `调整最大等待数，当 `HTTP` 请求数达到 `Tomcat` 的最大线程数时，新的HTTP请求到来，`Tomcat `会将该请求放在等待队列中，队列也满了，就会拒绝请求
3. `server.tomcat.max-connections=100 ` 最大连接数

### 存在问题和解决方案

**问题：**

双击`bin`目录下的`jmetter.bat`，现实情况是可能存在网络波动，或者一个接口处理时间偏长的情况（占用线程时间变长，空闲线程减少），没有空闲线程，导致其他请求一直在等待，线程满了进入队列（请求积压），队列满了拒绝掉（系统崩溃）

**解决方案：**

为了解决上面问题，使用服务隔离的方法【线程池隔离、信号量（计数器）隔离】，线程池隔离给每个方法分配一个小的线程池，信号量隔离是给每个方法设定一个阈值。以此减少积压请求
