---
title: Spring事务机制
date: 2022/5/13 23:46:25
tags:
- Spring
categories:
- [Spring, Core]
description: Spring事务机制
---

# 事务支持
> Spring事务：
> 
> 被@Transaction注解的的方法(service)需要给别的类(controller)调用，事务才会生效。
> 
> 如果给本类另一个方法调用，另个方法也要被@Transaction注解！

## JDBC

```JAVA
//最原始的操作数据库的方法
public class Demo {
    public static void main(String[] args) throws ClassNotFoundException {
        Class.forName("com.mysql.jdbc.Driver");
        Connection connection = DriverManager.getConnection("url", "user", "password");
        PreparedStatement sql = connection.prepareStatement("sql");
        ResultSet resultSet = sql.executeQuery();
    }
}
```

```JAVA
public class JdbcTemplateDemo {
    public static void main(String[] args) {
        //准备数据源：Spring的内置数据源
        DriverManagerDataSource ds = new DriverManagerDataSource();
        ds.setDriverClassName("com.mysql.jdbc.Driver");
        ds.setUrl("jdbc:mysql://localhost:3306/eesy");
        ds.setUsername("root");
        ds.setPassword("1234");

        // 1.创建JdbcTemplate对象
        JdbcTemplate jt = new JdbcTemplate();
        // 给jt设置数据源
        jt.setDataSource(ds);
        // 2.执行操作
        jt.execute("insert into account(name,money)values('ccc',10p00)");
        // 3.查询操作
        List<Account> accounts = jt.queryForList("sql", Account.class);
    }
}
```

## XML配置事务(基于AOP)

```XML
<!-- Spring中基于XML的声明式事务控制配置步骤
    1、配置事务管理器
    2、配置事务的通知
            此时我们需要导入事务的约束 tx名称空间和约束，同时也需要aop的
            使用tx:advice标签配置事务通知
                属性：
                    id：给事务通知起一个唯一标识
                    transaction-manager：给事务通知提供一个事务管理器引用
    3、配置AOP中的通用切入点表达式
    4、建立事务通知和切入点表达式的对应关系
    5、配置事务的属性
           是在事务的通知tx:advice标签的内部
-->
<!-- 配置事务管理器 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"></property>
</bean>

<!-- 配置事务的通知-->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <!-- 配置事务的属性
           isolation：用于指定事务的隔离级别。默认值是DEFAULT，表示使用数据库的默认隔离级别。
           propagation：用于指定事务的传播行为。默认值是REQUIRED，表示一定会有事务，增删改的选择。查询方法可以选择SUPPORTS。
           属性取值：Required：必须的。说明必须要有事物，没有就新建事物。
                    supports：支持。说明仅仅是支持事务，没有事务就非事务方式执行。
                    mandatory：强制的。说明一定要有事务，没有事务就抛出异常。
                    required_new：必须新建事务。如果当前存在事物就挂起。
                    not_supported：不支持事务，如果存在事物就挂起。
                    never：绝不有事务。如果存在事物就抛出异常
           read-only：用于指定事务是否只读。只有查询方法才能设置为true。默认值是false，表示读写。
           timeout：用于指定事务的超时时间，默认值是-1，表示永不超时。如果指定了数值，以秒为单位。
           rollback-for：用于指定一个异常，当产生该异常时，事务回滚，产生其他异常时，事务不回滚。没有默认值。表示任何异常都回滚。
           no-rollback-for：用于指定一个异常，当产生该异常时，事务不回滚，产生其他异常时事务回滚。没有默认值。表示任何异常都回滚。
    -->
    <tx:attributes>
        <tx:method name="*" propagation="REQUIRED" read-only="false"/>
        <tx:method name="find*" propagation="SUPPORTS" read-only="true"></tx:method>
    </tx:attributes>
</tx:advice>

<!-- 配置aop-->
<aop:config>
    <!-- 配置切入点表达式-->
    <aop:pointcut id="pt1" expression="execution(* com.itheima.service.impl.*.*(..))"></aop:pointcut>
    <!--建立切入点表达式和事务通知的对应关系 -->
    <aop:advisor advice-ref="txAdvice" pointcut-ref="pt1"></aop:advisor>
</aop:config>
```



## 注解配置事务

### 注解混搭XML

```XML
<!-- spring中基于注解 的声明式事务控制配置步骤
    1、配置事务管理器
    2、开启spring对注解事务的支持
    3、在需要事务支持的地方使用@Transactional注解
-->
<!-- 配置事务管理器，也可以使用@Bean注解将输入管理器添加到IOC容器中 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"></property>
</bean>
<!-- 开启spring对注解事务的支持 使用注解@Transactional-->
<tx:annotation-driven transaction-manager="transactionManager"/>
```

### 纯注解

```JAVA
/*
 * spring中基于注解 的声明式事务控制配置步骤
 * 1、配置事务管理器，使用@Bean返回事务管理器
 * 2、开启spring对注解事务的支持，@EnableTransactionManagement
 * 3、在需要事务支持的地方使用@Transactional注解
*/
@Bean("transactionManager")
@Primary
public PlatformTransactionManager transactionManager() {
    return new DataSourceTransactionManager(dataSourceSinitekSirm());
}
```