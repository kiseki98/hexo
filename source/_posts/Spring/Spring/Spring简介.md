---
title: Spring简介
date: 2022/5/13 13:46:25
tags:
- Spring
categories:
- [Spring, Core]
description: Spring简介
---

# 概述

**Spring**是**分层的Java SE/EE应用 full-stack轻量级开源框架**，以**IOC**（Inverse Of Control：反转控制）和**AOP**（Aspect Oriented Programming：面向切面编程）为内核。提供了展现层 `Spring MVC` 和持久层 `Spring JDBC` 以及业务层事务管理等应用技术，能整合众多著名的第三方框架和类库，是使用最多的开源框架。

# 优势

1. **方便解耦，简化开发**：通过 `Spring` 提供的 `IOC` 容器，可以将对象间的依赖关系交由 `Spring` 进行控制，避免硬编码所造成的过度程序耦合。用户也不必再为单例模式类、属性文件解析等这些很底层的需求编写代码，可以更专注于上层的应用。
2. **AOP编程的支持**：通过 `Spring` 的 `AOP` 功能，方便进行面向切面的编程，许多不容易用传统 `OOP` 实现的功能可以通过 `AOP` 轻松应付。
3. **声明式事务的支持**：可以将我们从单调烦闷的事务管理代码中解脱出来，通过声明式方式灵活的进行事务的管理，提高开发效率和质量。
4. **方便程序的测试**：可以用非容器依赖的编程方式进行几乎所有的测试工作，测试不再是昂贵的操作，而是随手可做的事情。
5. **方便集成各种优秀框架**：`Spring ` 可以降低各种框架的使用难度，提供了对各种优秀框架（`Struts、Hibernate、Hessian、Quartz`）的直接支持。降低`JavaEE API`的使用难度 `Spring` 对 `JavaEE API`（如 `JDBC`、 `JavaMail`、远程调用等）进行了薄薄的封装层，使这些 `API` 的使用难度大为降低。

# Spring容器ApplicationContext的实现类

```java
public class Text {
    public void testAccount() {
        // 它是从类的根路径下加载配置文件
        ApplicationContext ac1 = new ClassPathXmlApplicationContext("bean.xml"); 
        // 当我们使用注解配置容器对象时，需要使用此类来创建Spring容器。它用来读取注解。
        ApplicationContext ac2 = new AnnotationConfigApplicationContext(AppConfig.class); 
        // 它是从磁盘路径上加载配置文件 
        ApplicationContext ac3 = new FileSystemXmlApplicationContext(); 
        AccountService accountService = (AccountService) ac1.getBean("accountService");
    }
}
```
# XML方式使用Spring
        
## 基于XML配置

```xml
<!-- 使用默认无参构造方法-->
<bean id="" class="" scope=""  lazy-init="" init-method="" destory-method=""/>

<!-- 与@Bean不同在于方法可以是静态的-->
<!-- 静态工厂注入，将工厂类的静态方法创建的bean注入IOC容器-->
<bean id="" class="工厂类" factory-method=""/>

<!-- 实例工厂注入，将工厂类的私有方法创建的bean注入IOC容器，所以工厂也要注入IOC容器-->
<bean id="factory" class="工厂类"/> 
<bean id="" factory-bean="factory" factory-method=""/>

<!-- 需要在扫描包的类上加上@Service，@Component，@Controller，@Repository等注解-->
<component-scan packge=""/>  

<!-- 属性注入 可以通过有参构造注入或者set方法注入属性(还有静态工厂和实例工厂注入)-->
<!-- 通过有参构造方法注入属性，但是要求所有参数constructor-arg都必须写上-->
<bean id="user" class="com.duhao.User">
    <constructor-arg type="" value=""/>
    <constructor-arg type="" value=""/>
</bean>
<!-- 通过set方法设置属性，需要注入什么就写什么，要求有set方法，下面是视图解析器-->
<bean id="internalResourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/pages/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

## bean标签的属性说明
1. singleton：默认值，**单例**，容器创建时创建，容器销毁时死亡
2. prototype：原型（多例），什么时候需要，什么时候创建（懒加载），销毁通过GC回收
3. request：作用于 web 应用的请求范围
4. session：web项目中, Spring 创建一个 bean 的对象,将对象存入到 session 域中
5. global session：web 项目中,应用在集群环境如果没有**集群环境**那么 global Session 相当于 session

### scope
### lazy-init
> 懒加载，使用时才创建

### init-method
> 生命周期的回调方法。完成属性注入后调用

### destroy-method
> 生命周期的回调方法。销毁bean之前调用


