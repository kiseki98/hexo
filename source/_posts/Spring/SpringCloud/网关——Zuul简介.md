---
title: 网关——Zuul简介
date: 2022/5/20 16:46:25
tags:
- SpringCloud
- 微服务
- Zuul
categories:
- [Spring, SpringCloud]
description: 网关——Zuul简介
---

# Zuul微服务网关

## 微服务网关概述

### 没有网关存在的问题

在学习完前面的知识后，微服务架构已经初具雏形。但还有一些问题：不同的微服务一般会有不同的`IP`和`Port`，客户端在访问这些微服务时必须记住几十甚至几百个地址，这对于客户端方来说太复杂也难以维护。如下图

![image-20210701210401666](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210701210401666.png)

**存在的问题**

1. 客户端会请求多个不同的服务，需要维护不同的请求地址，增加开发难度
2. 在某些场景下存在跨域请求的问题
3. 加大身份认证的难度，每个微服务需要独立认证

### 上述问题解决方案

![image-20210701211007135](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210701211007135.png)

因此，我们需要一个微服务网关，介于客户端与服务器之间的中间层，所有的外部请求都会先经过微服 务网关。客户端只需要与网关交互，只知道一个网关地址即可，这样简化了开发还有以下优点：

1. 易于监控
2. 易于认证 （微服务网关具备了用户识别认证，其他微服务不在认证）
3. 减少了客户端与各个微服务之间的交互次数

### 什么是微服务网关

1. `API` 网关是一个服务器，是系统对外的唯一入口。整合所有微服务，使微服务对外暴露唯一的请求路径
2. `API` 网关封装了系统内部架构，为每个客户端提供 一个定制的 `API`（比如给每个微服务设置特定请求前缀，根据前缀判断请求微服务）
3. `API `网关方式的核心要点是，所有的客户端和消费端都通过统一的网关接入微服务，在网关层处理所有的非业务功能。
4. 通常，网关也是提供 `REST/HTTP` 的访问 `API` 。服务端通过 `API-GW` 注册和管理服务。

### 作用与应用场景

1. 网关具有的职责，如
    1. **身份验证**
    2. 监控：监控当前请求和微服务是否正常
    3. **负载均衡**：如用户服务有多个微服务，网关会自动负载均衡
    4. 缓存
    5. **限流**
    6. 请求分片与管理
    7. 静态响应处理
2. 当然最主要的职责还是与**外界联系**。

### 常见API网关实现方式

1. `Zuul`
2. `Spring Cloud Gateway`
3. `Nginx（普通架构，非SpringCloud）`

### 基于Nginx实现API网关

**启动微服务 -> 配置`Nginx` -> 启动`Nginx`**

```lua
-- 下面意思是请求路径以/api-order开头的跳转到地址http://127.0.0.1:9001/;
location /api-order {
 	proxy_pass http://127.0.0.1:9001/;
}
location /api-product {
 	proxy_pass http://127.0.0.1:9002/;
}

```

## Zuul简介

`Zuul ` 是 `Netflix  `开源的微服务网关，它可以和 `Eureka、Ribbon、Hystrix` 等组件配合使用，`Zuul` 组件的核心是一系列的过滤器，这些过滤器可以完成以下功能：

1. **动态路由：**动态将请求路由到不同后端集群
2. **压力测试：**逐渐增加指向集群的流量，以了解性能
3. **负载分配：**为每一种负载类型分配对应容量，并弃用超出限定值的请求
4. 静态响应处理：边缘位置进行响应，避免转发到内部集群
5. 身份认证和安全: 识别每一个资源的验证要求，并拒绝那些不符的请求。

`Spring Cloud  ` 对 `Zuul` 进行了整合和增强

## Zuul搭建网关服务器

### 引入依赖

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-zuul</artifactId>
	<version>2.1.0.RELEASE</version>
</dependency>
```

### 配置并开启网关服务功能

```java
// 启动类加上注解@EnableZuulProxy
@EnableZuulProxy
public class Application {
    // 略...
}
```

### 配置文件（取消屏蔽cookie）

```yaml
server:
	port: 8080 #服务端口
spring:
	application:
		name: api-gateway #指定服务名	
zuul:
	routes:
		product-service: # 这里是路由id，随意写
			path: /product-service/** # 这里是映射路径，转发请求路径前缀为/product-service到http://127.0.0.1:9002
			url: http://127.0.0.1:9002 # 映射路径对应的实际url地址
			sensitiveHeaders: # 默认zuul会屏蔽cookie，cookie不会传到下游服务，这里设置为空则取消默认的黑名单，如果设置了具体头信息则不会传到下游服务
```

## Zuul路由

根据请求的 `Url`，将请求分配到对应的微服务中进行处理

### 基础路由配置

### 面向服务路由配置（需要Eureka）

由于使用服务名简化了路由`Url`的配置，所以需要`Eureka`

**引入依赖**

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

**配置启动类**

```java
@SpringBootApplication
// 开启Zuul的网关功能
@EnableZuulProxy 
@EnableDiscoveryClient
public class ZuulServerApplication {
	public static void main(String[] args) {
		SpringApplication.run(ZuulServerApplication.class, args);
	}
}

```

**配置文件**

```yaml
# 配置eureka
eureka:
	client:
		serviceUrl:
			defaultZone: http://127.0.0.1:8761/eureka/
			registry-fetch-interval-seconds: 5 # 获取服务列表的周期：5s
	instance:
		preferIpAddress: true
		ip-address: 127.0.0.1
# 配置zuul
#配置路由规则
zuul:
	routes:
		product-service: # 这里是路由id，随意写
			path: /product-service/** # 这里是映射路径
			serviceId: shop-service-product # 指定转发的微服务名称
```

### 简化路由配置

除了配置文件之外与面向服务路由配置一致

```yaml
zuul:
	routes: # shop-service-product相当于服务名
		shop-service-product: /product-service/**
```

### 默认路由规则

在使用 `Zuul` 的过程中，上面讲述的规则已经大大的简化了配置项。但是当服务较多时，配置也是比较繁琐的。因此`Zuul`就指定了默认的路由规则：默认情况下，一切服务的映射路径就是**服务名本身**，例如服务名为： `shop-service-product` ，则默认的映射路径就是： `/shop-service/product/**`

## Zuul过滤器

`Zuul`过滤器：`Zuul`的核心是一系列过滤器，这些过滤器的作用如下

1. **身份验证与安全**：识别每个资源的验证要求，拒绝不符合的要求
2. 审查监控：再边缘位置追踪有意义的数据和统计结果，生成更精确的生产视图
3. **动态路由**：动态路由请求到不同的集群中
4. 压力测试：通过增加集群的流量了解性能
5. **负载分配**：为每种负载类型分配对应容量，并弃用超出现定制的请求
6. 静态响应处理器：在边缘位置建立部分响应，避免转发到集群内部
7. 多区域弹性：跨越`Aws Reign`进行请求路由，旨在`ELB`使用多样让系统的边缘更接近系统的使用者

### Zuul加入之后的架构

![image-20210701230745318](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210701230745318.png)

### Zuul过滤器接口

```java
public interface IZuulFilter {
    boolean shouldFilter();

    Object run() throws ZuulException;
}
```

### Zuul过滤器接口实现类（实际常继承这个类而非实现接口）

```java
public abstract class ZuulFilter implements IZuulFilter, Comparable<ZuulFilter> {
    private final AtomicReference<DynamicBooleanProperty> filterDisabledRef = new AtomicReference();

    public ZuulFilter() {
    }

    public abstract String filterType();

    public abstract int filterOrder();

    public boolean isStaticFilter() {
        return true;
    }

    public String disablePropertyName() {
        return "zuul." + this.getClass().getSimpleName() + "." + this.filterType() + ".disable";
    }

    public boolean isFilterDisabled() {
        this.filterDisabledRef.compareAndSet((Object)null, DynamicPropertyFactory.getInstance().getBooleanProperty(this.disablePropertyName(), false));
        return ((DynamicBooleanProperty)this.filterDisabledRef.get()).get();
    }

    public ZuulFilterResult runFilter() {
        ZuulFilterResult zr = new ZuulFilterResult();
        if (!this.isFilterDisabled()) {
            if (this.shouldFilter()) {
                Tracer t = TracerFactory.instance().startMicroTracer("ZUUL::" + this.getClass().getSimpleName());

                try {
                    Object res = this.run();
                    zr = new ZuulFilterResult(res, ExecutionStatus.SUCCESS);
                } catch (Throwable var7) {
                    t.setName("ZUUL::" + this.getClass().getSimpleName() + " failed");
                    zr = new ZuulFilterResult(ExecutionStatus.FAILED);
                    zr.setException(var7);
                } finally {
                    t.stopAndLog();
                }
            } else {
                zr = new ZuulFilterResult(ExecutionStatus.SKIPPED);
            }
        }

        return zr;
    }

    public int compareTo(ZuulFilter filter) {
        return Integer.compare(this.filterOrder(), filter.filterOrder());
    }

    public static class TestUnit {
        static Field field = null;
        @Mock
        private ZuulFilter f1;
        @Mock
        private ZuulFilter f2;

        public TestUnit() {
        }

        @Before
        public void before() {
            MockitoAnnotations.initMocks(this);
            MonitoringHelper.initMocks();
        }

        @Test
        public void testSort() {
            Mockito.when(this.f1.filterOrder()).thenReturn(1);
            Mockito.when(this.f2.filterOrder()).thenReturn(10);
            Mockito.when(this.f1.compareTo((ZuulFilter)Mockito.any(ZuulFilter.class))).thenCallRealMethod();
            Mockito.when(this.f2.compareTo((ZuulFilter)Mockito.any(ZuulFilter.class))).thenCallRealMethod();
            ArrayList<ZuulFilter> list = new ArrayList();
            list.add(this.f2);
            list.add(this.f1);
            Collections.sort(list);
            Assert.assertSame(this.f1, list.get(0));
        }

        @Test
        public void testShouldFilter() {
            class TestZuulFilter extends ZuulFilter {
                TestZuulFilter() {
                }

                public String filterType() {
                    return null;
                }

                public int filterOrder() {
                    return 0;
                }

                public boolean shouldFilter() {
                    return false;
                }

                public Object run() {
                    return null;
                }
            }

            TestZuulFilter tf1 = (TestZuulFilter)Mockito.spy(new TestZuulFilter());
            TestZuulFilter tf2 = (TestZuulFilter)Mockito.spy(new TestZuulFilter());
            Mockito.when(tf1.shouldFilter()).thenReturn(true);
            Mockito.when(tf2.shouldFilter()).thenReturn(false);

            try {
                tf1.runFilter();
                tf2.runFilter();
                ((TestZuulFilter)Mockito.verify(tf1, Mockito.times(1))).run();
                ((TestZuulFilter)Mockito.verify(tf2, Mockito.times(0))).run();
            } catch (Throwable var4) {
                var4.printStackTrace();
            }

        }

        @Test
        public void testIsFilterDisabled() {
            class TestZuulFilter extends ZuulFilter {
                TestZuulFilter() {
                }

                public String filterType() {
                    return null;
                }

                public int filterOrder() {
                    return 0;
                }

                public boolean isFilterDisabled() {
                    return false;
                }

                public boolean shouldFilter() {
                    return true;
                }

                public Object run() {
                    return null;
                }
            }

            TestZuulFilter tf1 = (TestZuulFilter)Mockito.spy(new TestZuulFilter());
            TestZuulFilter tf2 = (TestZuulFilter)Mockito.spy(new TestZuulFilter());
            Mockito.when(tf1.isFilterDisabled()).thenReturn(false);
            Mockito.when(tf2.isFilterDisabled()).thenReturn(true);

            try {
                tf1.runFilter();
                tf2.runFilter();
                ((TestZuulFilter)Mockito.verify(tf1, Mockito.times(1))).run();
                ((TestZuulFilter)Mockito.verify(tf2, Mockito.times(0))).run();
            } catch (Throwable var4) {
                var4.printStackTrace();
            }

        }

        @Test
        public void testDisabledPropNameOnInit() throws Exception {
            class TestZuulFilter extends ZuulFilter {
                final String filterType;

                public TestZuulFilter(String filterType) {
                    this.filterType = filterType;
                }

                public boolean shouldFilter() {
                    return false;
                }

                public Object run() {
                    return null;
                }

                public String filterType() {
                    return this.filterType;
                }

                public int filterOrder() {
                    return 0;
                }
            }

            TestZuulFilter filter = new TestZuulFilter("pre");
            Assert.assertFalse(filter.isFilterDisabled());
            AtomicReference<DynamicBooleanProperty> filterDisabledRef = (AtomicReference)field.get(filter);
            String filterName = ((DynamicBooleanProperty)filterDisabledRef.get()).getName();
            Assert.assertEquals("zuul.TestZuulFilter.pre.disable", filterName);
        }

        static {
            try {
                field = ZuulFilter.class.getDeclaredField("filterDisabledRef");
                field.setAccessible(true);
            } catch (Exception var1) {
                throw new RuntimeException(var1);
            }
        }
    }
}
```

### 自定义过滤器使用请求上下文

```java
// 通常是继承ZuulFilter,重写过滤器4个主要方法
public class D extends ZuulFilter {
    @Override
    public String filterType() {
        return null;
    }

    @Override
    public int filterOrder() {
        return 0;
    }

    @Override
    public boolean shouldFilter() {
        // 获取请求上下文
        RequestContext currentContext = RequestContext.getCurrentContext();
        HttpServletRequest request = currentContext.getRequest();
        return false;
    }

    // 执行过滤
    @Override
    public Object run() throws ZuulException {
        return null;
    }
}
```

## Zuul存在的问题

### 性能问题

1. `Zuul 1.x` 版本本质上就是一个同步 `Servlet`，采用多线程阻塞模型进行请求转发。
2. 简单讲，每来一个请求，`Servlet` 容器要为该请求分配一个线程专门负责处理这个请求，直到响应返回客户端这个线程才会被释放返回容器线程池。
3. 如果后台服务调用比较耗时，那么这个线程就会被阻塞，阻塞期间线程资源被占用，不能干其它事情。
4. `Servlet `容器线程池的大小有限，当前端请求量大，而后台慢服务比较多时，很容易耗尽容器线程池内的线程，造成容器无法接受新的请求。

### 不支持长连接

如`WebSocket`

## Zuul网关的替换方案

1. Zuul 2.x版本 
2. Spring GateWay