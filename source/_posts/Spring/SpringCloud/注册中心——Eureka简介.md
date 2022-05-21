---
title: 注册中心——Eureka简介
date: 2022/5/20 11:46:25
tags:
- SpringCloud
- 微服务
- Eureka
categories:
- [Spring, SpringCloud]
description: 注册中心——Eureka简介
---

# Eureka（停止维护）

## Eureka配置

### 基础架构
1. 功能：提供服务发现和注册功能
2. `EurekaServer`：
    1. 启动器`@EnableEurekaServer`
    2. 微服务注册给`eureka：eureka.client.service-url.defaultZone=http://127.0.0.1:10086/eureka`，多个地址注册使用`,`分割
3. `EurekaClient`：
    1. 启动器`@EnableDiscoverClient`
    2. 微服务注册给`eureka：eureka.client.service-url.defaultZone=http://127.0.0.1:10086/eureka`
4. 自定义服务实例`Id`
    1. 配置`eureka.instance.instance-id = "服务示例Id"`

### 高可用
1. `Eureka`相互注册：只要`Eureka`注册给对方，就相互有自己服务注册信息，`Eureka`集群相互共享注册信息。`Eureka`相互注册围成圈保证高可用
2. 服务提供者`Provider`：
    1. `server`向`EurekaServer`注册服务，并且完成服务续约工作
    2. 服务注册：向`Eureka`发送`Rest`请求，要求`register-with-eureka: true`默认为`true`，携带自己元数据信息，`Eureka`把元数据存到双层`Map`结构中
    3. 服务续约：向`Eureka`发送`Rest`请求，告诉`EurekaServer`自己还活着
        1. 服务续约间隔：`eureka.instance.lease-renewal-interval-in-seconds=30`，默认30秒
        2. 服务失效时间：`eureka.instance.lease-expiration-duration-in-seconds=90`，默认90秒。超时没收到心跳，`Eureka`把服务从服务列表中剔除
3. 服务消费者`Consumer`：
    1. 拉取服务：`eureka.client.fetch-registry=true`，默认为`true`不用配置
    2. 服务列表拉取间隔：`eureka.client.registry-fetch-interval-seconds=30`，默认30秒
4. 失效剔除和自我保护：
    1. 服务下线：服务正常关闭，发送请求告诉 `Eureka` 自己下线。`Eureka` 将服务设置为下线状态
    2. 失效剔除间隔：服务不一定正常下线，`eureka.server.eviction-interval-timer-in-ms: 60000`，默认每60秒剔除失效服务
    3. 自我保护：当服务未按时续约心跳，`Eureka `会统计最近 `15min` 的心跳失效服务实例是否超过`85%`，超过则把注册信息保护起来不剔除失效服务；`eureka.server.enable-self-preservation: false`，关闭自我保护
5. `Eureka`的元数据（服务注册的信息）
    1. 使用`DiscoveryClient`获取元数据的工具类
    2. `getInstances()`：获取服务实例
    3. `getServices()`：获取服务名称列表
       
## Eureka源码解析

### ImportSelector

1. `ImportSelector`接口是`Spring`导入外部配置的核心接口，在`SpringBoot`的自动化配置和`@EnableXXX`(功能性注解)中起到了决定性的作用。
2. 当在`@Configuration`标注的`Class`上使用`@Import`引入了一个 `ImportSelector`实现类后，会把实现类中返回的`Class`名称都定义为`Bean`。

```java
public interface ImportSelector {
    String[] selectImports(AnnotationMetadata var1);
}
```

### DeferredImportSelector

1. `DeferredImportSelector` 接口继承 `ImportSelector`，他和 `ImportSelector` 的区别在于装载`Bean`的时机
2. `DeferredImportSelector`需要等所有的`@Configuration`都执行完毕后才会进行装载

```java
public interface DeferredImportSelector extends ImportSelector {
    //...省略
}
```

## EurekaServer自动配置原理

### 启动类注解引入spring.factories文件

由于启动类注包含`@EnableAutoConfiguration`，所以`Eureka`首先找到`spring.factories`配置文件的配置类`EurekaServerAutoConfiguration`

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  org.springframework.cloud.netflix.eureka.server.EurekaServerAutoConfiguration
```

### 配置类EurekaServerAutoConfiguration

容器存在类型 `Marker` 的 `Bean` 才会把 `EurekaServerAutoConfiguration` 注入到容器内，配置才会生效

```java
@Configuration(
    proxyBeanMethods = false
)
@Import({EurekaServerInitializerConfiguration.class})
// 重点是下面这个注解的值Maker.class
@ConditionalOnBean({Marker.class})
@EnableConfigurationProperties({EurekaDashboardProperties.class, InstanceRegistryProperties.class})
@PropertySource({"classpath:/eureka/server.properties"})
public class EurekaServerAutoConfiguration implements WebMvcConfigurer {
    private static final String[] EUREKA_PACKAGES = new String[]{"com.netflix.discovery", "com.netflix.eureka"};
    @Autowired
    private ApplicationInfoManager applicationInfoManager;
    @Autowired
    private EurekaServerConfig eurekaServerConfig;
    @Autowired
    private EurekaClientConfig eurekaClientConfig;
    @Autowired
    private EurekaClient eurekaClient;
    @Autowired
    private InstanceRegistryProperties instanceRegistryProperties;
    public static final CloudJacksonJson JACKSON_JSON = new CloudJacksonJson();
    
    // 完成Eureka控制台相关功能
    @Bean
    @ConditionalOnProperty(
        prefix = "eureka.dashboard",
        name = {"enabled"},
        matchIfMissing = true
    )
    public EurekaController eurekaController() {
        return new EurekaController(this.applicationInfoManager);
    }
    
    // 初始化jersey配置，扫描EUREKA_PACKAGES包的内容，主要扫描注解@Path、@Provider、@Produces，发布web接口让EurekaClient调用（注册、续约、下线）
    @Bean
    public Application jerseyApplication(Environment environment, ResourceLoader resourceLoader) {
        ClassPathScanningCandidateComponentProvider provider = new ClassPathScanningCandidateComponentProvider(false, environment);
        provider.addIncludeFilter(new AnnotationTypeFilter(Path.class));
        provider.addIncludeFilter(new AnnotationTypeFilter(Provider.class));
        Set<Class<?>> classes = new HashSet();
        String[] var5 = EUREKA_PACKAGES;
        int var6 = var5.length;

        for(int var7 = 0; var7 < var6; ++var7) {
            String basePackage = var5[var7];
            Set<BeanDefinition> beans = provider.findCandidateComponents(basePackage);
            Iterator var10 = beans.iterator();

            while(var10.hasNext()) {
                BeanDefinition bd = (BeanDefinition)var10.next();
                Class<?> cls = ClassUtils.resolveClassName(bd.getBeanClassName(), resourceLoader.getClassLoader());
                classes.add(cls);
            }
        }

        Map<String, Object> propsAndFeatures = new HashMap();
        propsAndFeatures.put("com.sun.jersey.config.property.WebPageContentRegex", "/eureka/(fonts|images|css|js)/.*");
        DefaultResourceConfig rc = new DefaultResourceConfig(classes);
        rc.setPropertiesAndFeatures(propsAndFeatures);
        return rc;
    }
}
```

### 注解@EnableEurekaServer

存在`@EnableEurekaServer`才会注入`Maker`才能使`EurekaServerAutoConfiguration`生效（注入容器）

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import({EurekaServerMarkerConfiguration.class})
public @interface EnableEurekaServer {
}
// -----------------------------------------------------------------------------------------------------------------------------
@Configuration(
    proxyBeanMethods = false
)
public class EurekaServerMarkerConfiguration {
    public EurekaServerMarkerConfiguration() {
    }

    @Bean
    public EurekaServerMarkerConfiguration.Marker eurekaServerMarkerBean() {
        return new EurekaServerMarkerConfiguration.Marker();
    }

    class Marker {
        Marker() {
        }
    }
}
```

## EurekaClient自动配置原理

### 启动类注解引入spring.factories文件

由于启动类注包含`@EnableAutoConfiguration`，所以`Eureka`首先找到`spring.factories`配置文件的配置类`EurekaServerAutoConfiguration`

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.cloud.netflix.eureka.config.EurekaClientConfigServerAutoConfiguration,\
org.springframework.cloud.netflix.eureka.config.DiscoveryClientOptionalArgsConfiguration,\
org.springframework.cloud.netflix.eureka.EurekaClientAutoConfiguration,\
org.springframework.cloud.netflix.ribbon.eureka.RibbonEurekaAutoConfiguration,\
org.springframework.cloud.netflix.eureka.EurekaDiscoveryClientConfiguration,\
org.springframework.cloud.netflix.eureka.reactive.EurekaReactiveDiscoveryClientConfiguration,\
org.springframework.cloud.netflix.eureka.loadbalancer.LoadBalancerEurekaAutoConfiguration

org.springframework.cloud.bootstrap.BootstrapConfiguration=\
org.springframework.cloud.netflix.eureka.config.EurekaConfigServerBootstrapConfiguration

```

### 配置类EurekaClientAutoConfiguration

`EurekaDiscoveryClientConfiguration`向容器内注入`DiscoveryClient`，用此调用`EurekaServer`
