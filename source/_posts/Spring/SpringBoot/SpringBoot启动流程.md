---
title: SpringBoot启动流程 
date: 2022-04-23 20:20:00 
tags: SpringBoot
categories:
- [Spring, SpringBoot]
description: SpringBoot启动流程简短介绍
---

# SpringBoot启动流程

## SpringBoot 启动流程
![SpringBoot自动配置过程](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/SpringBoot%E8%87%AA%E5%8A%A8%E9%85%8D%E7%BD%AE%E8%BF%87%E7%A8%8B.png)
### SpringApplication 的构造方法

### SpringBoot 启动类 run() 方法源码

```java

@SpringBootApplication
public class DemoApplication {
    /**
     * 1.SpringBoot 启动类默认的写法，可以修改
     * @param args
     */
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}

public class SpringApplication {

    //创建一个新的实例，这个应用程序的上下文将要从指定的来源加载Bean
    public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
        // 资源初始化资源加载器，默认为null
        this.resourceLoader = resourceLoader;
        // 断言主要加载资源类不能为 null，否则报错
        Assert.notNull(primarySources, "PrimarySources must not be null");
        // 初始化主要加载资源类集合并去重
        this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
        // 推断当前 WEB 应用类型，一共有三种：NONE,SERVLET,REACTIVE
        this.webApplicationType = WebApplicationType.deduceFromClasspath();
        // 设置应用上线文初始化器,从"META-INF/spring.factories"读取ApplicationContextInitializer类的实例名称集合并去重，并进行set去重。（一共7个）
        setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
        // 设置监听器,从"META-INF/spring.factories"读取ApplicationListener类的实例名称集合并去重，并进行set去重。（一共11个）
        setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
        // 推断主入口应用类，通过当前调用栈，获取Main方法所在类，并赋值给mainApplicationClass
        this.mainApplicationClass = deduceMainApplicationClass();
    }


    public ConfigurableApplicationContext run(String... args) {
        // 1、创建并启动计时监控类
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();
        // 2、初始化应用上下文和异常报告集合
        ConfigurableApplicationContext context = null;
        Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
        // 3、设置系统属性“java.awt.headless”的值，默认为true，用于运行headless服务器，进行简单的图像处理，多用于在缺少显示屏、键盘或者鼠标时的系统配置，很多监控工具如jconsole 需要将该值设置为true
        configureHeadlessProperty();
        // 4、创建所有spring运行监听器并发布应用启动事件，简单说的话就是获取SpringApplicationRunListener类型的实例（EventPublishingRunListener对象），并封装进SpringApplicationRunListeners对象，然后返回这个SpringApplicationRunListeners对象。说的再简单点，getRunListeners就是准备好了运行时监听器EventPublishingRunListener。
        SpringApplicationRunListeners listeners = getRunListeners(args);
        listeners.starting();
        try {
            // 5、初始化默认应用参数类
            ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
            // 6、根据运行监听器和应用参数来准备spring环境
            ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
            // 将要忽略的bean的参数打开
            configureIgnoreBeanInfo(environment);
            // 7、创建banner打印类
            Banner printedBanner = printBanner(environment);
            // 8、创建应用上下文，可以理解为创建一个容器
            context = createApplicationContext();
            // 9、准备异常报告器，用来支持报告关于启动的错误
            exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
                    new Class[]{ConfigurableApplicationContext.class}, context);
            // 10、准备应用上下文，该步骤包含一个非常关键的操作，将启动类注入容器，为后续开启自动化提供基础
            prepareContext(context, environment, listeners, applicationArguments, printedBanner);
            // 11、刷新应用上下文
            refreshContext(context);
            // 12、应用上下文刷新后置处理，做一些扩展功能
            afterRefresh(context, applicationArguments);
            // 13、停止计时监控类
            stopWatch.stop();
            // 14、输出日志记录执行主类名、时间信息
            if (this.logStartupInfo) {
                new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
            }
            // 15、发布应用上下文启动监听事件
            listeners.started(context);
            // 16、执行所有的Runner运行器
            callRunners(context, applicationArguments);
        } catch (Throwable ex) {
            handleRunFailure(context, ex, exceptionReporters, listeners);
            throw new IllegalStateException(ex);
        }
        try {
            //17、发布应用上下文就绪事件
            listeners.running(context);
        } catch (Throwable ex) {
            handleRunFailure(context, ex, exceptionReporters, null);
            throw new IllegalStateException(ex);
        }
        //18、返回应用上下文
        return context;
    }
}
```

## SpringApplication 的构造方法

### getSpringFactoriesInstances() 方法

> getSpringFactoriesInstances(class) 方法从"META-INF/spring.factories"读取所有的类名称，并找到class的名称对应要加载的类的类名集合

## SpringBoot 的启动流程

> 一切从 SpringApplication.run() 开始，最终返回一个 ConfigurableApplicationContext

### 创建定时器stopWatch并启动

### 获取并运行listeners

```java
public class SpringApplication {
    //创建Spring监听器
    private SpringApplicationRunListeners getRunListeners(String[] args) {
        Class<?>[] types = new Class<?>[]{SpringApplication.class, String[].class};
        return new SpringApplicationRunListeners(logger,
                getSpringFactoriesInstances(SpringApplicationRunListener.class, types, this, args));
    }
}

class SpringApplicationRunListeners {
    SpringApplicationRunListeners(Log log, Collection<? extends SpringApplicationRunListener> listeners) {
        this.log = log;
        this.listeners = new ArrayList<>(listeners);
    }

    void starting(ConfigurableBootstrapContext bootstrapContext, Class<?> mainApplicationClass) {
        this.doWithListeners("spring.boot.application.starting", (listener) -> {
            listener.starting(bootstrapContext);
        }, (step) -> {
            if (mainApplicationClass != null) {
                step.tag("mainApplicationClass", mainApplicationClass.getName());
            }
        });
    }
}

public class EventPublishingRunListener implements SpringApplicationRunListener, Ordered {
    // 此处的监听器可以看出是事件发布监听器，主要用来发布启动事件
    @Override
    public void starting() {
        // 这里是创建application事件‘applicationStartingEvent’
        this.initialMulticaster.multicastEvent(new ApplicationStartingEvent(this.application, this.args));
    }

    // applicationStartingEvent是SpringBoot框架最早执行的监听器，在该监听器执行started方法时，会继续发布事件，主要是基于Spring的事件机制
    @Override
    public void multicastEvent(final ApplicationEvent event, @Nullable ResolvableType eventType) {
        ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));
        //获取线程池，如果为空则同步处理。这里线程池为空，还未初始化
        Executor executor = getTaskExecutor();
        for (ApplicationListener<?> listener : getApplicationListeners(event, type)) {
            if (executor != null) {
                //异步发送事件
                executor.execute(() -> invokeListener(listener, event));
            } else {
                //同步发送事件
                invokeListener(listener, event);
            }
        }
    }
}
```

### 初始化默认应用参数

```java
public class DefaultApplicationArguments implements ApplicationArguments {
    ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);

    public DefaultApplicationArguments(String... args) {
        Assert.notNull(args, "Args must not be null");
        this.source = new Source(args);
        this.args = args;
    }
}
```

### 根据运行监听器和应用参数来准备Spring环境

```java
public class SpringApplication {
    // 详细环境的准备
    private ConfigurableEnvironment prepareEnvironment(SpringApplicationRunListeners listeners,
                                                       ApplicationArguments applicationArguments) {
        // 获取或者创建应用环境
        ConfigurableEnvironment environment = getOrCreateEnvironment();
        // 配置应用环境，配置propertySource和activeProfiles
        configureEnvironment(environment, applicationArguments.getSourceArgs());
        // listeners环境准备，广播ApplicationEnvironmentPreparedEvent
        ConfigurationPropertySources.attach(environment);
        listeners.environmentPrepared(environment);
        // 将环境绑定给当前应用程序
        bindToSpringApplication(environment);
        // 对当前的环境类型进行判断，如果不一致进行转换
        if (!this.isCustomEnvironment) {
            environment = new EnvironmentConverter(getClassLoader()).convertEnvironmentIfNecessary(environment,
                    deduceEnvironmentClass());
        }
        // 配置propertySource对它自己的递归依赖
        ConfigurationPropertySources.attach(environment);
        return environment;
    }

    // 获取或者创建应用环境，根据应用程序的类型可以分为servlet环境、标准环境(特殊的非web环境)和响应式环境
    private ConfigurableEnvironment getOrCreateEnvironment() {
        // 存在则直接返回
        if (this.environment != null) {
            return this.environment;
        }
        // 根据webApplicationType创建对应的Environment
        switch (this.webApplicationType) {
            case SERVLET:
                return new StandardServletEnvironment();
            case REACTIVE:
                return new StandardReactiveWebEnvironment();
            default:
                return new StandardEnvironment();
        }
    }

    // 配置应用环境
    protected void configureEnvironment(ConfigurableEnvironment environment, String[] args) {
        if (this.addConversionService) {
            ConversionService conversionService = ApplicationConversionService.getSharedInstance();
            environment.setConversionService((ConfigurableConversionService) conversionService);
        }
        // 配置property sources
        configurePropertySources(environment, args);
        // 配置profiles
        configureProfiles(environment, args);
    }
}
```

### 创建Banner的打印类

```java
public class SpringApplication {
    private Banner printBanner(ConfigurableEnvironment environment) {
        if (this.bannerMode == Banner.Mode.OFF) {
            return null;
        }
        ResourceLoader resourceLoader = (this.resourceLoader != null) ? this.resourceLoader : new DefaultResourceLoader(getClassLoader());
        SpringApplicationBannerPrinter bannerPrinter = new SpringApplicationBannerPrinter(resourceLoader, this.banner);
        if (this.bannerMode == Mode.LOG) {
            return bannerPrinter.print(environment, this.mainApplicationClass, logger);
        }
        return bannerPrinter.print(environment, this.mainApplicationClass, System.out);
    }
}
```

### 创建应用的上下文:根据不同的应用类型初始化不同的上下文应用类

```java
public class SpringApplication {
    protected ConfigurableApplicationContext createApplicationContext() {
        Class<?> contextClass = this.applicationContextClass;
        if (contextClass == null) {
            try {
                switch (this.webApplicationType) {
                    case SERVLET:
                        contextClass = Class.forName(DEFAULT_SERVLET_WEB_CONTEXT_CLASS);
                        break;
                    case REACTIVE:
                        contextClass = Class.forName(DEFAULT_REACTIVE_WEB_CONTEXT_CLASS);
                        break;
                    default:
                        contextClass = Class.forName(DEFAULT_CONTEXT_CLASS);
                }
            } catch (ClassNotFoundException ex) {
                throw new IllegalStateException("Unable create a default ApplicationContext, please specify an ApplicationContextClass", ex);
            }
        }
        return (ConfigurableApplicationContext) BeanUtils.instantiateClass(contextClass);
    }
}
```

### 准备异常报告器

```java
public class SpringApplication {
    public ConfigurableApplicationContext run(String... args) {
        exceptionReporters =
                getSpringFactoriesInstances(SpringBootExceptionReporter.class, new Class[]{
                        ConfigurableApplicationContext.class
                }, context);
    }
}

```

### 准备应用上下文

```java
public class SpringApplication {
    private void prepareContext(DefaultBootstrapContext bootstrapContext, ConfigurableApplicationContext context, ConfigurableEnvironment environment, SpringApplicationRunListeners listeners, ApplicationArguments applicationArguments, Banner printedBanner) {
        // 应用上下文的environment
        context.setEnvironment(environment);
        // 应用上下文后处理
        this.postProcessApplicationContext(context);
        // 为上下文应用所有初始化器，执行容器中的applicationContextInitializer(spring.factories的实例)，将所有的初始化对象放置到context对象中 
        this.applyInitializers(context);
        // 触发所有SpringApplicationRunListener监听器的ContextPrepared事件方法。添加所有的事件监听器
        listeners.contextPrepared(context);
        bootstrapContext.close(context);
        // 记录启动日志 
        if (this.logStartupInfo) {
            this.logStartupInfo(context.getParent() == null);
            this.logStartupProfileInfo(context);
        }

        // 注册启动参数bean，将容器指定的参数封装成bean，注入容器 
        ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
        beanFactory.registerSingleton("springApplicationArguments", applicationArguments);

        // 设置banner 
        if (printedBanner != null) {
            beanFactory.registerSingleton("springBootBanner", printedBanner);
        }

        if (beanFactory instanceof AbstractAutowireCapableBeanFactory) {
            ((AbstractAutowireCapableBeanFactory) beanFactory).setAllowCircularReferences(this.allowCircularReferences);
            if (beanFactory instanceof DefaultListableBeanFactory) {
                ((DefaultListableBeanFactory) beanFactory).setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
            }
        }

        if (this.lazyInitialization) {
            context.addBeanFactoryPostProcessor(new LazyInitializationBeanFactoryPostProcessor());
        }

        // 加载所有资源，指的是启动器指定的参数
        Set<Object> sources = this.getAllSources();
        Assert.notEmpty(sources, "Sources must not be empty");
        // 将bean加载到上下文中
        this.load(context, sources.toArray(new Object[0]));
        // 触发所有SpringApplicationRunListener监听器的contextLoaded事件方法
        listeners.contextLoaded(context);
    }
}

```

### 刷新应用上下文

```java
public class SpringApplication {
    public void refresh() throws BeansException, IllegalStateException {
        synchronized (this.startupShutdownMonitor) {
            StartupStep contextRefresh = this.applicationStartup.start("spring.context.refresh");

            // Prepare this context for refreshing.
            // 刷新上下文环境，初始化上下文环境，对系统的环境变量或者系统属性进行准备和校验
            prepareRefresh();

            // Tell the subclass to refresh the internal bean factory.
            // 初始化beanFactory
            ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

            // Prepare the bean factory for use in this context.
            // 为上下文准备beanFactory，对beanFactory的各种功能进行填充，如@autowired，设置spel表达式解析器，设置编辑注册器，添加applicationContextAwareProcessor处理器等等
            prepareBeanFactory(beanFactory);

            try {
                // Allows post-processing of the bean factory in context subclasses.
                // 提供子类覆盖的额外处理，即子类处理自定义的beanFactoryPostProcess
                postProcessBeanFactory(beanFactory);

                StartupStep beanPostProcess = this.applicationStartup.start("spring.context.beans.post-process");
                // Invoke factory processors registered as beans in the context.
                // 激活各种beanFactory处理器   
                invokeBeanFactoryPostProcessors(beanFactory);

                // Register bean processors that intercept bean creation.
                // 注册拦截bean创建的bean处理器，即注册beanPostProcessor     
                registerBeanPostProcessors(beanFactory);
                beanPostProcess.end();

                // Initialize message source for this context.
                // 初始化上下文中的资源文件如国际化文件的处理  
                initMessageSource();

                // Initialize event multicaster for this context.
                // 初始化上下文事件广播器  
                initApplicationEventMulticaster();

                // Initialize other special beans in specific context subclasses.
                // 给子类扩展初始化其他bean
                onRefresh();

                // Check for listener beans and register them.
                // 在所有的bean中查找listener bean,然后注册到广播器中
                registerListeners();

                // Instantiate all remaining (non-lazy-init) singletons.
                // 初始化剩余的非懒惰的bean，即初始化非延迟加载的bean 
                finishBeanFactoryInitialization(beanFactory);

                // Last step: publish corresponding event.
                // 发完成刷新过程，通知声明周期处理器刷新过程，同时发出ContextRefreshEvent通知别人
                finishRefresh();
            }

            catch (BeansException ex) {
                if (logger.isWarnEnabled()) {
                    logger.warn("Exception encountered during context initialization - " +
                            "cancelling refresh attempt: " + ex);
                }

                // Destroy already created singletons to avoid dangling resources.
                destroyBeans();

                // Reset 'active' flag.
                cancelRefresh(ex);

                // Propagate exception to caller.
                throw ex;
            }

            finally {
                // Reset common introspection caches in Spring's core, since we
                // might not ever need metadata for singleton beans anymore...
                resetCommonCaches();
                contextRefresh.end();
            }
        }
    }
}
```

### 应用上下文刷新后置处理

> 当前方法的代码是空的，可以做一些自定义的后置处理操作

```java
public class SpringApplication {
    protected void afterRefresh(ConfigurableApplicationContext context, ApplicationArguments args) {
    }
}
```

### 发布应用上下文启动完成事件：触发所有SpringApplicationRunListener监听器的started事件方法

```java
public class SpringApplication {
    void started(ConfigurableApplicationContext context, Duration timeTaken) {
        this.doWithListeners("spring.boot.application.started", (listener) -> {
            listener.started(context, timeTaken);
        });
    }
}
```

### 执行所有Runner执行器：执行所有applicationRunner和CommandLineRunner两种运行器

```java
public class SpringApplication {
    private void callRunners(ApplicationContext context, ApplicationArguments args) {
        List<Object> runners = new ArrayList<>();
        runners.addAll(context.getBeansOfType(ApplicationRunner.class).values());
        runners.addAll(context.getBeansOfType(CommandLineRunner.class).values());
        AnnotationAwareOrderComparator.sort(runners);
        for (Object runner : new LinkedHashSet<>(runners)) {
            if (runner instanceof ApplicationRunner) {
                callRunner((ApplicationRunner) runner, args);
            }
            if (runner instanceof CommandLineRunner) {
                callRunner((CommandLineRunner) runner, args);
            }
        }
    }
}
```

### 发布应用上下文就绪事件：触发所有SpringApplicationRunnerListener将挺起的running事件方法

```java
public class SpringApplication {
    void running(ConfigurableApplicationContext context) {
        for (SpringApplicationRunListener listener : this.listeners) {
            listener.running(context);
        }
    }
}
```