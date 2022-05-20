---
title: SpringMVC介绍
date: 2022/7/12 20:46:25
tags:
- Spring
- SpringMVC
categories:
- [Spring, SpringMVC]
description: SpringMVC介绍
---

# SpringMVC基本概念

## 三层架构和MVC

### 三层架构

1. **表现层**：也就是我们常说的 `web` 层。它负责接收客户端请求，向客户端响应结果，通常客户端使用`Http`协议请求`web`层，`web`需要接收`Http`请求，完成`Http`响应。
   1. 表现层包括展示层和控制层：控制层负责接收请求，展示层负责结果的展示。
   2. 表现层依赖业务层，接收到客户端请求一般会调用业务层进行业务处理并将处理结果响应给客户端。
   3. 表现层的设计一般都使用 `MVC` 模型。（ `MVC` 是表现层的设计模型）
2. **业务层**：也就是我们常说的`service`层。它负责业务逻辑处理。`web`层依赖业务层。业务层在业务处理时可能会依赖持久层，事务应该放到业务层来控制
3. **持久层**：也就是我们是常说的`dao`层。通俗的讲，持久层就是和数据库交互，对数据库表进行曾删改查的。

### MVC

`MVC `全名是 `Model View Controller`，是模型(`model`)－>视图(`view`)－>控制器(`controller`)的缩写，是一种用于设计创建 `Web` 应用程序表现层的模式。`MVC`中每个部分各司其职：

1. `Model`（模型）：通常指的就是我们的数据模型。作用一般情况下用于封装数据。
2. `View`（视图）：
   1. 通常指的就是我们的`JSP`或者`HTML`。作用一般就是展示数据的。
   2. 通常视图是依据模型数据创建的。
3. `Controller`（控制器）：
   1. 处理用户交互的部分。作用一般就是处理程序逻辑的。
   2. 举个例子：我们要保存一个用户的信息包含了姓名，性别，年龄。要求年龄必须是1~100之间的整数。姓名和性别不能为空。并且把数据填充到模型之中。此时除了`JS`的校验之外，服务器端也应该有**数据准确性**的校验，那么校验就是**控制器**的该做的。当校验失败后，由控制器负责把错误信息展示到页面。如果校验成功，控制器把数据填充到模型，并且调用业务层实现完整的业务需求。 

## SpringMVC概述

`SpringMVC`是一种基于 `Java `的实现 `MVC` 设计模型的请求驱动类型的轻量级 `Web` 框

`SpringMVC`的优势

1. 清晰的角色划分：
   1. 前端控制器`DispatcherServlet`
   2. 处理器映射器`HandlerMapping`
   3. 处理器适配器`HandlerAdapter`
   4. 视图解析器`ViewResolver`
   5. 处理器或页面控制器`Controller`
   6. **验证器** `Validator`
   7. 命令对象`Command` 请求参数绑定到的对象就叫命令对象
   8. 表单对象`Form Object` 提供给表单展示和提交到的对象就叫表单对象
2. 分工明确，而且扩展点相当灵活，可以很容易扩展，虽然几乎不需要。
3. 和`Spring` 其他框架无缝集成，是其它Web框架所不具备的。
4. 可适配，通过`HandlerAdapter`可以支持任意的类作为处理器。
5. 可定制性，`HandlerMapping、ViewResolver`等能够非常简单的定制。
6. 功能强大的数据验证、格式化、绑定机制。
7. 利用`Spring`提供的`Mock`对象能够非常简单的进行`Web`层单元测试。
8. 强大的`JSP`标签库，使`JSP`编写更容易。
9. 还有比如`Restful`风格的支持、简单的文件上传、约定大于配置的契约式编程支持、基于注解的零配置支持等等。

## SpringMVC对比Struts2

1. 共同点
   1. 它们都是表现层框架，都是基于 `MVC` 模型编写的。
   2. 它们的底层都离不开原始 `ServletAPI `
   3. 它们处理请求的机制都是一个**核心控制器**。
2. 区别
   1. `SpringMVC` 的入口是`Servlet`, 而 `Struts2` 是 `Filter`
   2. `SpringMVC` 是基于方法设计的，而`Struts2`是基于类，`Struts2`每次执行都会创建一个动作类。会稍微比 `SpringMVC` 慢些。
   3. `SpringMVC` 使用更加简洁,同时还支持 `JSR303`, 处理 `Ajax` 的请求更方便(`JSR303` 是一套 `JavaBean` 参数校验的标准，如 `Hibernate` 的 `validate`框架，它定义了很多常用的校验注解，将这些注解加在我们 `JavaBean` 的属性上面，可以根据需要校验。)
   4. `Struts2` 的`OGNL`表达式使页面的开发效率相比 `SpringMVC`  更高些，但执行效率并没有比`JSTL`提升，尤其是`Struts2`的表单标签，远没有`HTML`执行效率高

# SpringMVC入门

## XML配置SpringMVC（web.xml）

```xml
<!DOCTYPE web-app PUBLIC
      	"-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
        "http://java.sun.com/dtd/web-app_2_3.dtd" >
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
    http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
         version="3.0">
    <display-name>Archetype Created Web Application</display-name>
    <!--配置spring监听器，默认只加载web-inf下的applicationContext.xml配置文件,一般不要去移动配置文件统一资源管理，所以指去找指定位置的配置文件-->
    <!--用来初始化spring的-->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <!--配置一个request监听,作用是后台aop日志里面获取访问ip-->
    <listener>
        <listener-class>org.springframework.web.context.request.RequestContextListener</listener-class>
    </listener>
    <!--重新配置spring配置文件路径,listener没有子标签，监听器来加载applicationContext.xml文件-->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath*:applicationContext.xml,classpath*:springSecurity.xml</param-value>
    </context-param>
    <!--这里配置前端控制器加载springmvc.xml-->
    <servlet>
        <servlet-name>dispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!--加载springmvc配置文件-->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
    <!--配置解决中文乱码的过滤器-->
    <filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    <!--配置错误页面，发生错误跳转到该页面-->
    <error-page>
        <error-code>403</error-code>
        <location>/403.jsp</location>
    </error-page>
</web-app>
```

## 注解配置SpringMVC

`tomcat`通过 `SPI` （不是反射）找到实现 `WebApplicationInitializer` 接口的类，直接调用 `onStartup()`，完成配置类的加载和配置，不需要`XML`

```java
public class MyWebApplicationInitializer implements WebApplicationInitializer {
    @Override
    public void onStartup(ServletContext servletCxt) {
        // Load Spring web application configuration
        AnnotationConfigWebApplicationContext ac = new AnnotationConfigWebApplicationContext();
        ac.register(AppConfig.class);
        ac.refresh();
        // Create and register the DispatcherServlet
        DispatcherServlet servlet = new DispatcherServlet(ac);
        ServletRegistration.Dynamic registration = servletCxt.addServlet("app", servlet);
        registration.setLoadOnStartup(1);
        registration.addMapping("/app/*");
    }
}
```

## SpringMVC配置文件(springmvc.xml)

```xml
<!--开启注解扫描-->
<context:component-scan base-package="com.duhao"></context:component-scan>

<!--配置视图解析器-->
<bean id="internalResourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <!--告诉资路径-->
    <property name="prefix" value="/pages/"></property>
    <!--告诉解析器文件后缀名-->
    <property name="suffix" value=".jsp"></property>
</bean>

<!--配置自定义类型转换器-->
<bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
    <property name="converters">
        <set>
            <bean class="com.duhao.utils.StringToDate"></bean>
        </set>
    </property>
</bean>

<!-- 配置文件上传解析器 --><!-- id的值是固定的--><!-- 设置上传文件的最大尺寸为5MB -->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <property name="maxUploadSize">
        <value>5242880</value>
    </property>
</bean>

<!--告诉前端控制器那些静态资源不拦截,我这里在web.xml当中配置servletmapping,default为*.这样的形式-->
<mvc:resources mapping="/js/**" location="/js/**"></mvc:resources>
<mvc:resources mapping="*" location="/js/**"></mvc:resources>

<!--开启注解支持,加入自定义的类型转换器conversion-service="conversionService"-->
<mvc:annotation-driven conversion-service="conversionService"></mvc:annotation-driven>
```

## SpringMVC组件

处理器映射器、处理器适配器、视图解析器称为`SpringMVC`的三大组件

1. 前端控制器 `DispatcherServlet`：用户请求到达前端控制器，它就相当于`MVC`模式中的`C`，`DispatcherServlet`是整个流程控制的中心，由它调用其它组件处理用户的请求，`DispatcherServlet `的存在降低了组件之间的耦合性。
2. 处理器映射器`HandlerMapping`：根据用户请求找到`Handler`，`SpringMVC`提供了不同的映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等。
3. 处理器`Handler`：它就是我们开发中要编写的具体业务控制器。由`DispatcherServlet`把用户请求转发到 `Handler`。由`Handler`对具体的用户请求进行处理。
4. 处理器适配器`HandlAdapter`：通过`HandlerAdapter`对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。
5. 视图解析器`ViewResolver：View` ：负责将处理结果生成`View`，`ViewResolver`首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成`View`视图对象，最后对`View`进行渲染将处理结果通过页面展示给用户。
6. 视图`View`：`SpringMVC`框架提供了很多的`View`视图类型的支持，包括：`FreeMarkerView`、`PDFView`等。我们最常用的视图就是`JSP`。

## SpringMVC执行流程

![04](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/04.bmp)

1. 当启动`Tomcat`服务器的时候，因为配置了`<load-on-startup>`标签，所以会创建 `DispatcherServlet` 对象，就会加载`springmvc.xml`配置文件
2. 开启了注解扫描，那么`HelloController`对象就会被创建
3. 从`index.jsp`发送请求，请求会先到达`DispatcherServlet`核心控制器，根据配置`@RequestMapping`注解找到执行的具体方法
4. 根据执行方法的返回值，再根据配置的**视图解析器**，去指定的目录下查找指定名称的`JSP`文件
5. `Tomcat`服务器渲染页面，做出响应

## 请求参数封装

1. 请求参数的绑定说明
   1. 绑定机制
      1. 表单提交的数据都是`key=value`格式的 `username=haha&password=123`
      2. `SpringMVC`的参数绑定过程是把表单提交的请求参数，自动作为控制器中方法的参数进行绑定的
      3. 要求：提交表单的`name`和参数的`名称`是相同的
   2. 支持的数据类型
      1. 基本数据类型和字符串类型
      2. 实体类型`JavaBean`
      3. 集合数据类型（`List`、`Map`集合等）
2. 基本数据类型和字符串类型
   1. 提交表单的`name`和参数的名称是相同的
   2. 区分大小写
3. 实体类型`JavaBean`
   1. 提交表单的`name`和`JavaBean`中的属性名称需要一致
   2. 如果一个`JavaBean`类中包含其他的引用类型，那么表单的`name`属性需要编写成：对象.属性 例如：`address.name`

# SpringMVC使用

`Restful`风格的`URL`

1. 请求路径一样，可以根据不同的请求方式去执行后台的不同方法（请求方式`Post、Get、Put、Delete`）
2. `Restful`风格的`URL`优点：结构清晰、符合标准、易于理解、扩展方便

## 注解使用

1. `@RequestMapping`	
   1. `@RequestMapping`的作用是建立请求`URL`和处理方法之间的对应关系
   2. `@RequestMapping`可以作用在方法和类上
      1.  作用在类上：第一级的访问目录
      2. 作用在方法上：第二级的访问目录
      3. 细节：路径可以不编写 / 表示应用的根目录开始，`${ pageContext.request.contextPath }`也可以省略不写，但是路径上不能写 /
   3. `@RequestMapping`的属性
      1. `path` ：指定请求路径的`URL`
      2. `value` ：和path属性是一样的
      3. `mthod`：指定该方法的请求方式（`Post、Get、Put、Delete`）
      4. `params` 指定限制请求参数的条件
      5. `headers` 发送的请求中必须包含的请求头
2. `@RequestParam`
   1. 作用：把请求中的指定名称的参数传递给`controller`中的形参赋值
   2. 属性
      1. `value`：请求参数中的名称
      2. `required`：请求参数中是否必须提供此参数，默认值是`true`
3. `@RequestBody`
   1. 作用：用于获取请求体的内容（`Post`和`Put`有请求体）
   2. 属性：`required`，是否必须有请求体，默认值是`true`
4. `@PathVariable`
   1. 作用：拥有绑定`Url`中的占位符的。例如：`Url`中有`/delete/{id}`，`{id}`就是占位符
   2. 属性：
      1. `value`：指定url中的占位符名称
      2. `required`：是否必须有占位符
5. `@RequestHeader`
   1. 作用：获取指定请求头的值
   2. 属性：
      1. `value`：请求头的名称
      2. `require`：是否必须包含此请求头
6. `@CookieValue`
   1. 作用：用于获取指定`cookie`的名称的值
   2. 属性`value`：表示`cookie`的名称
7. `@ModelAttribute`
   1. 作用
      1. 出现在方法上：表示当前方法会在控制器方法执行前先执行。
      2. 出现在参数上：获取指定的数据给参数赋值。
   2. 应用场景：当提交表单数据不是完整的实体数据时，保证没有提交的字段使用数据库原来的数据。
8. `@SessionAttributes`
   1. 作用：用于多次执行控制器方法间的参数共享
   2. 属性`value`：指定存入属性的名称
   3. 数据存入`session`域中

## SpringMVC响应

响应数据，根据`controller`方法的返回值类型分类

### String作为返回值

1. `Controller`方法返回字符串可以指定逻辑视图的名称，根据视图解析器为物理视图的地址。
2. 如`return "success"`，解析为`sucess.jsp`，跳转到`success`页面
3. `SpringMVC`提供的转发和重定向（返回值是string）
   1. 转发`forward`：`return "forward:/user/findAll";`
   2. 重定向`redirect`：`return "redirect:/user/findAll";`

### void作为返回值

1. 如果控制器的方法返回值编写成`void`，执行程序报`404`的异常，默认查找`JSP`页面（默认`JSP`默认找和方法名相同的）没有找到。 默认会跳转到`@RequestMapping(value="/initUpdate") initUpdate`的页面。
2. 可以使用请求转发或者重定向跳转到指定的页面

### ModelAndView作为返回值

```java
@RequestMapping("hello")
public ModelAndView sayHello() {
    ModelAndView mv = new ModelAndView();
    mv.setViewName("success");
    mv.addObject("u","这里是测试model是否存进request域当中" );
    return mv;
}
```

### Model作为方法参数（原理其实是以ModelAndView作为返回值）

```java
@RequestMapping("testRequest/{id}")
public String testRequest(@Pathvariable("id") int id,Model model) {
    //可以给request域添加数据，jsp可以解析request域中的数据
    model.addAttribute("u", "这里是测试model是否存进request域当中");
    return "success";
}
```

### ResponseEntity作为返回值

```java
// 作用和@Response一样，都是将JavaBean转化为Json格式响应浏览器，但是在返回时可以设置响应状态码（404，401）
@RequestMapping("save")
public ResponseEntity<String> save() {
    return ResponseEntity.ok("响应数据");
    return ResponseEntity.notFound().build();
}
```

## 拦截器

### 自定义拦截器

```java
public class MyInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)  {
        return false;//根据返回值决定是否放行
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView)  {
              //后置，可以重定向，给request域中设置数据
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)  {

    }
}
```

### 注册拦截器

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new  MyInterceptor()).addPathPatterns("/secure/*");
    }
}
```

