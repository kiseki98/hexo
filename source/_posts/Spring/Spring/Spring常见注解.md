---
title: Spring常见注解
date: 2022/5/13 20:46:25
tags:
- Spring
- SpringCore
categories:
- [Spring, Core]
description: Spring常见注解
---

# Spring自动装配方式(不可以装配简单类型)

1. `no`：默认的方式是不进行自动装配，通过显式设置`ref`属性来进行装配。
2. `byType`：通过`Bean`的类型装配
3. `byName`：通过`Bean`的`name`装配
4. `construct`：这个方式类似于`byType`， 但是要提供给构造器参数，如果没有确定的带参数的构造器参数类型，将会抛出异常。
5. `autodetect`：首先尝试使用`constructor`来自动装配，如果无法工作，则使用`byType`方式。

# 泛用性注解

## bean的注入
> @Repository：一般持久层使用
> 
> @Service：一般业务层使用
> 
> @Controller：一般控制层使用
> 
> @Component：不适用于上面三种的情况使用
> 
> 这四个注解和bean标签作用一致，都是将bean注入到容器内

## @Bean
> 将方法的返回值作为bean放入IOC容器中，上面4种是将bean的创建交给Spring管理，有自己的生命流程而@Bean是将用户创建的bean直接注入IOC容器，id默认是方法名

## @Configuration
> 指定一个类是配置类

## @Import
> 用于导入其他配置类，导入类可以不被@Configuration注解

## @ComponentScan
> 指定Spring容器创建时扫描的包路径

## @Primary
> 自动装配时当出现多个bean候选者时，被注解为@Primary的bean将作为首选者，否则将抛出异常。

## @Conditional 注解
> 满足一定条件才会注入容器中，派生的注解有以下几个
> 
> @ConditionalOnBean({Maker.class})：表示容器存在类型为Maker的bean才把被注解的类注入容器内
> 
> @ConditionalOnMissingBean({Maker.class})：表示容器不存在类型为Maker的Bean才把被注解的类注入容器内
> 
> @ConditionalOnProperty("spring.cloud.service-registry.auto-registration.enabled)：存在配置参会注入被注解的类
> 
> @ConditionalOnClass({Maker.class})：存在Maker这个类就注入被此注解修饰的类

# 生命周期和作用域相关注解

## @Scope
> 和bean标签的scope属性作用一样

## @Lazy
> 表示bean懒加载，什么时候使用什么时候创建，不在容器创建时创建

## @PostConstruct
> 和bean标签的init-method属性类似，都是bean完成属性注入后调用的方法，在构造方法和init-method之间得到调用。要求方法是实例方法、无形参、无返回值

## @PreDestroy
> 和bean标签的destroy-method属性类似，都是bean销毁前调用的方法，注解的方法在destroy-method方法调用后执行.要求方法是实例方法、无形参、无返回值

## 在bean初始化后或者销毁前执行特定操作的方法
1. 通过实现 InitializingBean/DisposableBean接口来定制初始化之后/销毁之前的操作方法
2. 通过 bean标签的init-method/destroy-method属性指定初始化之后 /销毁之前调用的操作方法
3. 在指定方法上加上@PostConstruct 或@PreDestroy指定该方法是在初始化之后还是销毁之前调用



# 属性注入注解

## @PropertySource
> 用于指定properties文件的位置，文件中数据就会被Spring以键值对存到一个位置，注入某个变量中使用@Value注解 

## @Value
> 给基本类型变量注入值，value属性可以私用SPEL表达式，如@Value("${user.name}")，需要@PropertySource注解指定properties文件的位置

## @Autowired
> 和bean标签内的property标签相似，默认按类型注入，如果有多个类型匹配，会以变量名作为`Bean`的`id`在`IOC`容器中查找，找不到`Bean`的`id`对应或者找到多个`id`相同的报异常。如果要允许`null`可以使用`@Autowired(required=false)` 。可以作用在方法、构造方法、字段、参数上面。注入属性使用的`AutowiredAnnotationBeanPostProcessor`后置处理器

## @Qualifier
> 一般不能单独使用，一般配合@Autowired使用，用于指定bean的id，两注解组合作用同@Resource，给方法的参数注入时可以单独使用。如果@Bean注解的方法有形参，以形参名为bean的id从IOC容器中查找，作用在字段、方法、参数、类上面

## @Resource
> 默认按bean的id注入，如果找不到会按照byType，如果找不到或者找到多个会报错。可设置根据id还是type查找，@Resource(name=bean名字)，@Resource(type=bean的class)查找注入属性使用的CommonAnnotationBeanPostProcessor后置处理器


# 动态刷新
## @RefreshScope
作用在类上，使用配置中心动态修改配置文件，文件内容发生修改，属性也会被修改