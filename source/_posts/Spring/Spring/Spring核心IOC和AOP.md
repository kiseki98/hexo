---
title: Spring核心IOC和AOP
date: 2022/5/13 22:46:25
tags:
- Spring
- IOC
- AOP
categories:
- [Spring, Core]
description: Spring核心IOC和AOP
---

# IOC的概念和作用（解耦、减少代码）

## 程序的耦合和解耦

> 耦合：程序间的依赖关系，包括类的依赖关系，方法间的依赖

## 解耦思路

通过反射创建对象而避免使用 new 关键字

## 工厂模式解耦

## 控制反转IOC（工厂模式和反射机制）

控制反转：，是一种**设计原则**，**减少代码**和**降低耦合度**，不用自己创建对象，只用描述对象怎么创建

**重要**：Spring IOC 负责创建对象，管理对象（ 通过依赖注入 DI），装配对象，配置对象， 管理对象的整个生命周期

![批注 2020-04-17 233410](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/ioc%E7%A4%BA%E6%84%8F%E5%9B%BE.png)

# AOP(XML方式)

## 什么是AOP(动态代理和反射机制)

AOP 可对业务逻辑的各部分进行隔离，使业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高开发的效率。简单来说就是不修改源码的基础上，对方法增强。可以降低耦合度

## AOP优势

1. 作用：在程序运行期间，不修改源码对已有方法进行增强。
2. 优势：减少重复代码，提高开发效率，维护方便

## AOP具体实现方式(动态代理)

### 动态代理

1. **特点**：字节码随用随创建，随用随加载
2. **作用**：不修改源码的基础上对方法增强
3. **类型**：JDK 的 Proxy（基于接口）和 Cglib 的 Enhancer（基于子类）

### 基于接口的动态代理

1. 如何创建代理对象：使用 Proxy 类中的 newProxyInstance()
2. 创建代理对象的要求：被代理类最少实现一个接口，如果没有则不能使用
3. newProxyInstance 方法的参数：
    1. ClassLoader：类加载器，它是用于加载代理对象字节码的。和被代理对象使用相同的类加载器。固定写法。
    2. Class[]：字节码数组，它是用于让代理对象和被代理对象有相同方法。固定写法。
    3. InvocationHandler：用于提供增强的代码逻辑，让我们写如何代理。一般都是写一个该接口的实现类，通常情况下都是匿名内部类。

```java
public class JDKProxy {
    public static void main(String[] args) {
        Test t = new Test(); // 要被代理的对象
        AccountService proxy = 
            (AccountService) Proxy.newProxyInstance(t.getClass().getClassLoader(), t.getClass().getInterfaces(), new 			 
            InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                // 增强的逻辑实现位置，在这里进行方法的增强
                Object invoke = method.invoke(t, args);
                return invoke; // 增强方法的返回值
            }
        });
        // 下面调用的方法决定了上面代理对象中的Method
        proxy.saveAccount(); // 使用代理对象调用增强的方法
    }
}
```



### 基于子类的动态代理

1. 如何创建代理对象：使用 Enhancer 类中的 create() 
2. 创建代理对象的要求：被代理类不能是最终类(不能被 final 修饰)
3. create 方法的参数：
    1. Class：字节码，它是用于指定被代理对象的字节码。
    2. Callback：用于提供增强的代码它是让我们写如何代理。
       1. 我们一般都是些一个该接口的实现类，通常情况下都是匿名内部类，但不是必须的。
       2. 此接口的实现类都是谁用谁写。
       3. 我们一般写的都是该接口的子接口实现类：MethodInterceptor

```java
public class Cglib {
    public static void main(String[] args) {
        Test t=new Test(); // 要被代理的对象
        Test enhancer = (Test) Enhancer.create(Test.class, new MethodInterceptor() {
            @Override
            public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                // 增强逻辑实现的位置
                Object invoke = method.invoke(t, args);
                return invoke; // 增强方法的返回值
            }
        });
        enhancer.saveAccount();
    }
}
```

## AOP相关术语

1. JoinPoint（连接点）：连接点代表一个应用程序的某个位置，表示在这个位置我们可以插入一个 AOP 切面(其实就是所有方法)
2. Pointcut（切入点）：是匹配连接点的表达式，所谓切入点是指我们要对哪些 JoinPoint 进行拦截的定义(连接点的子集)
3. Advice（通知/增强）：
    1. 所谓通知是指拦截到 JoinPoint 之后所要做的事情就是通知。
    2. 通知的类型：
        1. before：前置通知
        2. after-throwing：异常通知
        3. after-returning：后置通知
        4. after（final）：最终通知
4. Introduction（引介）：引介是一种特殊的通知，在不修改类代码的前提下, Introduction 可以在运行期为类动态地添加一些方法或 Field
5. Target（目标对象）：代理的目标对象（需要被增强的对象）
6. Weaving（织入）：是指把增强应用到目标对象来创建新的代理对象的过程（织入增强方法的过程）
7. Spring采用动态代理织入，而 AspectJ 采用编译期织入和类装载期织入
8. Proxy（代理）：一个类被AOP织入增强后，就产生一个代理类（增强后的对象）
9. Aspect（切面）：是切入点和通知（引介）的结合（就是JoinPoint，Advice，PointCut所在的类）

## 代理的选择

在 Spring 中会根据目标类是否实现了接口来决定采用哪种动态代理的方式。 Aop 代理对象（代理后是一个新的对象）是在 Spring 容器创建之后就创建的，不是在 getBean 方法时创建的

几种不同类型的自动代理

1. BeanNameAutoProxyCreator
2. DefaultAdvisorAutoProxy
3. CreatorMetadataAutoproxying

## Spring的AOP使用

AOP 切入点表达式（拦截切入点，进行增强）

1. execution
2. within 拦截指定包
3. this 代理对象类型（全限定类名）
4. target 代理的目标对象（全限定类名）
5. args 方法的参数 bean 以 bean 的名字为。。。才进行拦截以上除了 execution ，加上@表示只有被。。注解才拦截（全限定类型名）

### XML方式使用AOP

```xml
<bean id="logger" class="com.duhao.utils.Logger"></bean>
<aop:config>
    <aop:aspect id="logAdvice" ref="logger">
        <aop:before method="beforePrintLog" pointcut-ref="active"></aop:before>
        <aop:after-returning method="afterPrintLog" pointcut-ref="active"></aop:after-returning>
        <aop:after-throwing method="catchPrintLog" pointcut-ref="active"></aop:after-throwing>
        <aop:after method="finallyPrintLog" pointcut-ref="active"></aop:after>
        <!--环绕通知相当于整个增强方法，环绕通知参数可以写ProceedingJoinPoint接口，Spring会为我们提供实现类，
		   调用proceed()方法（相当于切入点方法）该方法之前为前置通知，之后为后置通知，类推-->
        <aop:around method="arroundPrintLog" pointcut-ref="active"></aop:around>
        <!--配置切入点，那么在上面四个通知标签中直接利用pointcut-ref引入，而不需要用pointcut标签来写切入点表达式-->
        <aop:pointcut id="active" expression="execution(* com.duhao.service.impl.*.*(..))"></aop:pointcut>
    </aop:aspect>
</aop:config>
```



### 注解方式使用AOP

首先开启注解 AOP 支持，在 Config 类上加上注解 @EnableAspectJAutoProxy，表示开启注解 AOP

```JAVA
@Component
@Aspect
public class Logger {
    //切入点表达式，切入点表达式的名称是下面方法名，被通知引入要加双引号
    @Pointcut("execution(* com.duhao.service.impl.*.*(..))")
    public void pointcut() {
        //execution最小粒度是方法，当然也可以是类，但是不建议
    }

    @Before("pointcut()")
    public void beforePrintLog() {
        System.out.println("logger类中的printLog方法执行了。。。。前置通知");
    }

    @AfterReturning("pointcut()")
    public void afterPrintLog() {
        System.out.println("logger类中的printLog方法执行了。。。。后置通知");
    }

    @AfterThrowing("pointcut()")
    public void catchPrintLog() {
        System.out.println("logger类中的printLog方法执行了。。。。异常通知");
    }

    @After("pointcut()")
    public void finallyPrintLog() {
        System.out.println("logger类中的printLog方法执行了。。。。最终通知");
    }

    @Around("pointcut()")
    public Object arroundPrintLog(ProceedingJoinPoint proceedingJoinPoint) {
        Object rtValue = null;
        try {
            Object[] args = proceedingJoinPoint.getArgs();
            //procee()方法之前就是前置通知，之后就是后置通知，catch里面就是异常通知，finally里就是最终通知
            System.out.println("logger类中的printLog方法执行了。。。。前置通知");
            rtValue = proceedingJoinPoint.proceed(args);
            System.out.println("logger类中的printLog方法执行了。。。。后置通知");
            return rtValue;
        } catch (Throwable throwable) {
            throw new RuntimeException("logger类中的printLog方法执行了。。。。异常通知");
        } finally {
            System.out.println("logger类中的printLog方法执行了。。。。最终通知");
        }
    }
}
```
