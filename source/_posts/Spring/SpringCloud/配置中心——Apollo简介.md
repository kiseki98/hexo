---
title: 配置中心——Apollo简介
date: 2022/5/20 19:46:25
tags:
- SpringCloud
- 微服务
- 配置中心
categories:
- [Spring, SpringCloud]
description: 配置中心——Apollo简介
---

# 开源配置中心Apollo

`SpringCloudConfig`存在的问题：

1. 配置文件存放在`git`服务器上，`git`服务器也需要维护成本
2. 在`git`上修改了配置文件，还需要手动访问开放端点，手动刷新

## Apollo概述

`Apollo`是携程框架部门研发的开源配置管理中心，能够集中化管理应用不同环境、不同集群的配置，配置修改后能够实时推送到应用端，并且具备规范的权限、流程治理等特性。服务端基于`SpringBoot`和`SpringCloud`开发，打包后可以直接运行，不需要额外安装`Tomcat`等应用容器。 正是基于配置的特殊性，所以`Apollo`从设计之初就立志于成为一个有治理能力的配置发布平台，目前提供了以下的特性：

1. 统一管理不同环境、不同集群的配置
    1. `Apollo`提供了一个统一界面集中式管理不同环境`environment`、不同集群`cluster`、 不同命名空间`namespace`的配置
    2. 同一份代码部署在不同的集群，可以有不同的配置，比如`Zookeeper`的地址等
    3. 通过命名空间`namespace`可以很方便地支持多个不同应用共享同一份配置，同时还允许 应用对共享的配置进行覆盖
2. 配置修改实时生效（热发布）
3. 版本发布管理：所有的配置发布都有版本概念，从而可以方便地支持配置的回滚
4. 灰度发布：支持配置的灰度发布，比如点了发布后，只对部分应用实例生效，等观察一段时间没问题后再推给所有应用实例
5. 权限管理、发布审核、操作审计
    1. 应用和配置的管理都有完善的权限管理机制，对配置的管理还分为了编辑和发布两个环节， 从而减少人为的错误
    2. 所有的操作都有审计日志，可以方便地追踪问题
6. 客户端配置信息监控：可以在界面上方便地看到配置在被哪些实例使用
7. 提供`Java`和`.Net`原生客户端
    1. 提供了Java和.Net的原生客户端，方便应用集成
    2. 支持`SpringPlaceholder`，`Annotation`和`SpringBoot`的`ConfigurationProperties`，方便应用使用
    3. 同时提供了`Http`接口，非`Java`和`.Net`应用也可以方便地使用
8. 提供开放平台API
    1. `Apollo`自身提供了比较完善的统一配置管理界面，支持多环境、多数据中心配置管理、权限、流程治理等特性。不过`Apollo`出于通用性考虑，不会对配置的修改做过多限制，只要符合基本的格式就能保存，不会针对不同的配置值进行针对性的校验，如数据库用户名、密码，`Redis`服务地址等
    2. 对于这类应用配置，`Apollo`支持应用方通过开放平台`API`在`Apollo`进行配置的修改和发布，并且具备完善的授权和权限控制
9. 部署简单
    1. 配置中心作为基础服务，可用性要求非常高，这就要求`Apollo`对外部依赖尽可能地少
    2. 目前唯一的外部依赖是`MySQL`，所以部署非常简单，只要安装好`Java`和`MySQL`就可以让 `Apollo`跑起来
    3. `Apollo`还提供了打包脚本，一键就可以生成所有需要的安装包，并且支持自定义运行时参数

## Apollo实现方式

![image-20210721231717234](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210721231717234.png)

上图简要描述了`Apollo`客户端的实现原理

1. 客户端和服务端保持了一个**长连接**，从而能第一时间获得配置更新的推送
2. 客户端
    1. 定时拉取配置，长连接`socket`的补偿机制，防止长连接出现问题获取不到数据
    2. 客户端从`Apollo`配置中心服务端获取到应用的最新配置后，会保存在内存中和本地文件系统（遇到服务不可用网络不通的时候，能从本地恢复配置）
3.  应用程序从`Apollo`客户端获取最新的配置、订阅配置更新通知

## 搭建Apollo环境

### 环境要求

1. `Java`语言环境
    1. `Apollo`服务端：1.8+
    2. `Apollo`客户端：1.7+
2. `MySQL`数据库环境
    1. 版本：5.6.5+
    2. `Apollo`的表结构对 `timestamp` 使用了多个`default`声明，所以需要5.6.5以上版本

### 搭建服务端环境

#### 下载Apollo

通过官网提供的下载连接下载安装包

#### 配置数据库

`Apollo`服务端共需要两个数据库： `ApolloPortalDB` 和 `ApolloConfigDB` ，我们把数据库、表的创建和样例数据都分别准备了`sql`文件（见`SpringCloud`目录下的`Apollo`文件夹），只需要导入数据库即可

#### 配置数据库连接

`Apollo`服务端需要知道如何连接到你前面创建的数据库，所以需要编辑`demo.sh`（见`SpringCloud`目录下的`Apollo`文件夹）），修改`ApolloPortalDB` 和`ApolloConfigDB`相关的数据库信息

```sh
#apollo config db info
apollo_config_db_url=jdbc:mysql://localhost:3306/ApolloConfigDB?
characterEncoding=utf8
apollo_config_db_username=用户名
apollo_config_db_password=密码（如果没有密码，留空即可）
# apollo portal db info
apollo_portal_db_url=jdbc:mysql://localhost:3306/ApolloPortalDB?
characterEncoding=utf8
apollo_portal_db_username=用户名
apollo_portal_db_password=密码（如果没有密码，留空即可
```

#### 启动脚本（deme.sh）

启动脚本（见`SpringCloud`目录下的`Apollo`文件夹的`demo.sh`）会在本地启动3个服务，分别使用8070， 8080， 8090端口，请确保这3个端口当前没有被使用。分别启动`Eureka`、`Apollo Portal`（管理台页面）、其他辅助应用

```sh
./demo.sh start
```

当看到如下输出后，就说明启动成功了！

```sh
==== starting service ====
Service logging file is ./service/apollo-service.log
Started [10768]
Waiting for config service startup.......
Config service started. You may visit http://localhost:8080 for service status 
now!
Waiting for admin service startup....
Admin service started
==== starting portal ====
Portal logging file is ./portal/apollo-portal.log
Started [10846]
Waiting for portal startup......
Portal started. You can visit http://localhost:8070 now!
```

#### 访问管理台

通过浏览器打开 `http://ip:8070` 即可访问`Apollo`配置中心的前端页面，输入默认用户名密码`apollo/admin`即可登录到应用中

![image-20210721234538165](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210721234538165.png)

### 客户端集成

#### 引入依赖

`Apollo`的客户端`jar`包已经上传到中央仓库，应用在实际使用时只需要按照如下方式引入即可

```xml
<dependency>
    <groupId>com.ctrip.framework.apollo</groupId>
    <artifactId>apollo-client</artifactId>
    <version>1.1.0</version>
</dependency>
```

#### SpringBoot集成

`SpringBoot`支持通过`application.properties/bootstrap.properties`来配置，该方式能使配置在更早的阶段注入，比如使用 `@ConditionalOnProperty` 的场景或者是有一些`spring-boot-starter`在启动阶段就需要读取配置做一些事情，所以对于`SpringBoot`环境建议通过以下方式来接入`Apollo`(需要0.10.0及以上版本）

使用方式很简单，只需要在`application.yml/bootstrap.yml`中按照如下样例配置即可

```yaml
apollo:
  bootstrap: # 开启Apollo
    enabled: true
  meta: http://192.168.74.101:8080 # eureka的路径，之前启动的demo.sh的三个端口中的8080，实际是eureka
app:
  id: test01 # 自己在Apollo中配置中心的appid
```
