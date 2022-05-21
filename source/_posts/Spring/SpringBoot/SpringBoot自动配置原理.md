---
title: SpringBoot自动配置原理
date: 2022/7/3 20:46:25
tags:
- Spring
- SpringBoot
categories:
- [Spring, SpringBoot]
description: SpringBoot自动配置原理
---

# SpringBoot自动装配原理

## 自动配置原理解析

从`spring-boot-starter-dependencies`的`pom.xml`中我们可以发现，一部分坐标的版本、依赖管理、插件管理已经定义好，所以我们的`SpringBoot`工程继承`spring-boot-starter-parent`后已经具备版本锁定等配置了。所以起步依赖的作用就是进行依赖的传递。

## 关键注解

1. `@SpringBootApplication`
   1. `@SpringBootConfiguration`：等同与`@Configuration`，既标注该类是`Spring`的一个配置类，代替`xml`文件
   2. `@EnableAutoConfiguration`：`SpringBoot`自动配置功能开启
   3. `@ComponentScan`：配置输入扫描包
2. `@ProperSource("classpath：db.properties")`：指定外部属性文件，如数据库连接配置文件
3. `@Value("${jdbc.password}")`：给类的成员注入值
4. `@ConfigurationProperties(prifix="配置文件前缀")`：被注解类成员自动按前缀注入属性
5. `@EnableConfigurationProperties(class=类名)`：启用数据源类。作为该类成员通过`@AutoWrid`注入
6. `SpringBoot`属性注入4种方式：
   1. `@AutoWrid`：配合`@ConfigurationProperties(prifix="配置文件前缀")`，注入该注解类到使用类中
   2. 构造方法注入：启用数据源类，构造方法参数为数据源类
   3. `@Bean`方法形参注入：在`@Bean`注解方法的形参上放数据源类，并且启用数据源
   4. `@Bean`方法使用@ConfigurationProperties(prifix="配置文件前缀")：要求方法返回值类要有`set`方法

## SpringBoot自动配置原理

![SpringBoot自动配置原理](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/SpringBoot%E8%87%AA%E5%8A%A8%E9%85%8D%E7%BD%AE%E5%8E%9F%E7%90%86.gif)

1. 自动配置的基础
   1. 当在`@Configuration`标注的`Class`上使用`@Import`引入了一个 `ImportSelector`接口实现类后
   2. 会把实现类方法`selectImports()`返回的值，`Class`名称都注入到容器中。
   3. 通过`ConfigurationClassParser`的`processImports()`加载外部配置类，找到`ImportSelector`的实现类调用`selectImports()`获取返回的类，返回的类如果同样继承`ImportSelector`，则继续递归调用`processImports()`
2. 自动配置原理
   1. 通过启动类注解`@SpringBootApplication`引入`@EnableAutoConfiguration`注解
   2. 注解`@EnableAutoConfiguration`引入`@Import({AutoConfigurationImportSelector.class})`
   3. `AutoConfigurationImportSelector`其中的`selectImports()`调用`getCandidateConfigurations()`，而`getCandidateConfigurations()`调用`getAutoConfigurationEntry()`
   4. 而`getAutoConfigurationEntry()`调用`loadSpringFactories()`扫描所有`jar`包下面的`spring.factories`文件根据注解`key`获取对应的`value`并返回
   5. `ImportSelector`接口是`Spring`导入外部配置的核心接口，在`SpringBoot`的自动化配置和`@EnableXXX`(功能性注解)中起到了决定性的作用。

```java
public interface ImportSelector {
    // 返回值的class名同样认为时一个被@Configuration注解的类
    String[] selectImports(AnnotationMetadata var1);
}
```

```java
// 实现类AutoConfigurationImportSelector
public String[] selectImports(AnnotationMetadata annotationMetadata) {
    if (!this.isEnabled(annotationMetadata)) {
        return NO_IMPORTS;
    } else {
        AutoConfigurationImportSelector.AutoConfigurationEntry autoConfigurationEntry =
            // 调用getAutoConfigurationEntry方法
            this.getAutoConfigurationEntry(annotationMetadata);
        return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
    }
}
// 调用getAutoConfigurationEntry方法
protected AutoConfigurationImportSelector.AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
      if (!this.isEnabled(annotationMetadata)) {
          return EMPTY_ENTRY;
      } else {
          AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
          // 重点在这
          List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
          configurations = this.removeDuplicates(configurations);
          Set<String> exclusions = this.getExclusions(annotationMetadata, attributes);
          this.checkExcludedClasses(configurations, exclusions);
          configurations.removeAll(exclusions);
          configurations = this.getConfigurationClassFilter().filter(configurations);
          this.fireAutoConfigurationImportEvents(configurations, exclusions);
          return new AutoConfigurationImportSelector.AutoConfigurationEntry(configurations, exclusions);
      }
  }
// 调用getCandidateConfigurations，重点spring.factories
protected List<String> getCandidateConfigurations(
    AnnotationMetadata metadata, AnnotationAttributes attributes
) {
        List<String> configurations = SpringFactoriesLoader.loadFactoryNames(
            this.getSpringFactoriesLoaderFactoryClass(), 
            this.getBeanClassLoader()
        );
        Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.");
        return configurations;
    }
```

`spring.factories`的`key`是注解，`value`是对应的配置类，表示配置类加上该注解则将配置的`value`通过`selectImports()`注入到容器

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.cloud.gateway.config.GatewayClassPathWarningAutoConfiguration,\
org.springframework.cloud.gateway.config.GatewayAutoConfiguration,\
org.springframework.cloud.gateway.config.GatewayHystrixCircuitBreakerAutoConfiguration,\
org.springframework.cloud.gateway.config.GatewayResilience4JCircuitBreakerAutoConfiguration,\
org.springframework.cloud.gateway.config.GatewayLoadBalancerClientAutoConfiguration,\
org.springframework.cloud.gateway.config.GatewayNoLoadBalancerClientAutoConfiguration,\
org.springframework.cloud.gateway.config.GatewayMetricsAutoConfiguration,\
org.springframework.cloud.gateway.config.GatewayRedisAutoConfiguration,\
org.springframework.cloud.gateway.discovery.GatewayDiscoveryClientAutoConfiguration,\
org.springframework.cloud.gateway.config.SimpleUrlHandlerMappingGlobalCorsAutoConfiguration,\
org.springframework.cloud.gateway.config.GatewayReactiveLoadBalancerClientAutoConfiguration
```

