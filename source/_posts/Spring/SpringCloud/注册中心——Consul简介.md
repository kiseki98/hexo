---
title: 注册中心——Consul简介
date: 2022/5/20 11:47:25
tags:
- SpringCloud
- 微服务
- Consul
categories:
- [Spring, SpringCloud]
description: 注册中心——Consul简介
---

# Consul

## Consul基本概念

### Consul的优势

1. 使用 `Raft` 算法来保证一致性， 比复杂的 `Paxos` 算法更直接，相比较而言，`zookeeper` 采用的是 `Paxos`， 而 `etcd` 使用的则是 `Raft`
2. 支持多数据中心，内外网的服务采用不同的端口进行监听。 多数据中心集群可以避免单数据中心的单点故障，而其部署则需要考虑网络延迟,，分片等情况等。 `zookeeper` 和 `etcd` 均不提供多数据中心功能的支持。
3. 支持健康检查。 `etcd` 不提供此功能。
4.  支持 `http` 和 `dns` 协议接口。 `zookeeper` 的集成较为复杂, `etcd` 只支持 `http` 协议。
5. 官方提供 `web` 管理界面, `etcd` 无此功能。 综合比较, `Consul` 作为服务注册和配置管理的新星, 比较值得关注和研究。

### Consul的特性

1. 服务发现
2. 健康检查
3. `Key/Value`存储
4. 多数据中心

## Eureka和Consul的区别

1. 一致性
    1. `Consul ` 强一致性 `CP`
        1. 服务注册相比 `Eureka` 会稍慢一些。因为 `Consul` 的 `Raft` 协议要求必须过半数的节点都写入成功才认为注册成功
        2.  `Leader  ` 挂掉时，重新选举期间整个 `Consul` 不可用。保证了强一致性但牺牲了可用性
    2. `Eureka `保证高可用和最终一致性 `AP`
        1. 服务注册相对要快，因为不需要等注册信息 `replicate` 到其他节点，也不保证注册信息是否 `replicate` 成功
        2. 当数据出现不一致时，虽然A,，B上的注册信息不完全相同，但每个 `Eureka` 节点依然能够正常对外提供服务，这会出现查询服务信息时如果请求A查不到，但请求B就能查到。如此保证了可用性但牺牲了一致性。
2. 开发语言和使用
    1. `Eureka ` 就是个 `Servlet` 程序，跑在 `Servlet` 容器中
    2. `Consul `则是 `Go` 编写而成，安装启动即可

## Windows使用Consul

### 开发者模式快速启动

```shell
## 进入Consul安装目录 以开发者模式快速启动，-client表示可访问当前服务的IP地址，浏览打开IP:8500端口访问控制台
## agent：启动一个consul的守护进程 -dev是开发者模式，-client，-server
consul agent -dev -client=0.0.0.0
```

### 服务注册与发现

可以使用 `PostMan` 发送 `Http` 请求注册服务和获取服务信息

## 服务注册与发现

### 引入依赖

```properties
# 服务发现依赖坐标
spring-cloud-starter-consul-discovery
# 健康检查依赖坐标
spring-cloud-starter-actuator
```

### 服务配置

```yaml
spring:
 cloud:
   consul: #consul相关配置其中 spring.cloud.consul 中添加consul的相关配置
     host: 192.168.74.101 #ConsulServer请求地址
     port: 8500 #ConsulServer端口
     discovery:
       register: true #是否注册，消费者可以不注册也可以拉取服务列表
       instance-id: ${spring.application.name}-1  #实例ID，唯一ID可使用${spring.application.name}:${spring.cloud.client.ipAddress}
       service-name: ${spring.application.name} #服务实例名称
       port: ${server.port} #服务实例端口
       healthCheckPath: /actuator/health #健康检查路径
       healthCheckInterval: 15s #健康检查时间间隔
       prefer-ip-address: true #开启ip地址注册
       ip-address: ${spring.cloud.client.ip-address} #当前微服务的请求ip
```

### 服务发现

`SpringCloud` 对 `Consul` 进行进一步处理向其中集成了 `Ribbon` 的支持消费者拉取服务信息和 `Eureka`一致，可以使用 `DiscoveryClient（获取元数据，拉取服务列表）`和 `RestTemplate` 调用服务

## Consul高可用集群搭建

### 启动Consul命令

```shell
consul agent -dev -client=0.0.0.0
# agent：启动一个consul守护进程
# -dev：开发者模式
# -client（主要）：不做实质性的consul工作，是和server数据通讯和交互的代理，是转发所有RPC到server的代理
# -server（主要）：实际的consul服务
```

**主要成员**

1. `client`
    1. 职责：不做实质性的 `consul` 工作，是和 `server` 数据通讯和交互，把微服务信息以内部 `RPC` 到 `server` 的代理
    2. 搭建：一个 `client` 对应一个微服务（相当于绑定到微服务作为一个整体），且部署在一台机器上，通过 `client` 以 `RPC` 发送数据到 `server`
    3. 数量：不限，一个微服务对应一个 `client`，占用资源小
2. `server`
    1. 职责：实际的 `consul` 服务
    2. 数量：推荐3-5个 `server` 搭建集群，多个可能涉及消息同步，会很慢

![image-20210623205040517](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/image-20210623205040517.png)

### Gossip流言协议（Client）

**例如 Leader 选举后，事件广播，所有的`Consul`的`Agent`节点都会参与到这个协议中（client、server），多节点中数据复制，获取到新的Leader信息**

![Gossip协议](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/Gossip%E5%8D%8F%E8%AE%AE.gif)

### Raft一致性算法（Server）

保证 `Server` 集群数据强一致性，数据实时同步

**组成**

1. `Leader`：是 `Server` 集群唯一可处理请求的
2. `Follower`：选民，被动接受数据
3. `Candidate`：候选人，可以被选取为`Leader`

**流程**

1. 选主
    1. 候选人向 `Follower` 发送数据，询问是否可以选主
    2. `Follower` 相应确认信息，候选人选为 `Leader`，负责接受 `client` 数据
2. 数据同步
    1. `Leader` 接收数据，发送到 `Follower`，`Follower `响应 `Leader`，表示自己收到
    2. `Leader` 接受到 `Follower` 数据，响应给 `client`，表示自己接受到信息（`Leader` 和 `Follow` 都接受数据才算接收到）

### 集群搭建

**安装`Consul Server`**

```sh
##从官网下载最新版本的Consul服务
wget https://releases.hashicorp.com/consul/1.5.3/consul_1.5.3_linux_amd64.zip
##使用unzip命令解压
unzip consul_1.5.3_linux_amd64.zip
##将解压好的consul可执行命令拷贝到/usr/local/bin目录下
cp consul /usr/local/bin
##测试一下
consul
```

**启动`Consul Server`**

```sh
##登录s1虚拟机，以server形式运行
consul agent -server -bootstrap-expect 3 -data-dir /etc/consul.d -node=server-1 -bind=192.168.74.101 -ui -client 0.0.0.0 &
##登录s2 虚拟机，以server形式运行
consul agent -server -bootstrap-expect 2 -data-dir /etc/consul.d -node=server-2 -bind=192.168.74.102 -ui -client 0.0.0.0 & 
##登录s3 虚拟机，以server形式运行
consul agent -server -bootstrap-expect 2 -data-dir /etc/consul.d -node=server-3 -bind=192.168.74.103 -ui -client 0.0.0.0 &

# -server： 以server身份启动。
# -bootstrap-expect：集群要求的最少server数量，当低于这个数量，集群即失效。
# -data-dir：data存放的目录，更多信息请参阅consul数据同步机制
# -node：节点名称
# -bind：监听的ip地址。绑定当台服务的地址（一台机器可能存在多个网卡，多个ip）
# -ui：开启web管理界面
# -client：客户端的ip地址，0.0.0.0表示不限制
# & ：在后台运行，此为linux脚本语
```

**本地启动`Consul Client`**

```shell
##在本地电脑中使用client形式启动consul
consul agent -client=0.0.0.0  -data-dir /etc/consul.d -node=client-1
```

**加入集群**

```sh
##s2，s3，client加入consul集群，以s1为leader
consul join 192.168.74.101

##查看consul集群节点信息
consul members

## 5.4.2在项目中配置consul的client的地址完成注册
```

## Consul常见问题

### Consul节点和微服务注销

1. 使用 `PostMan` 调用`HTTP API`
    1. 注销任意节点和服务：`/catalog/deregister`
    2. 注销当前节点的服务：`/agent/service/deregister/:service_id`
2. 在节点所在机器使用命令
    1. 本机：`consul leave`
    2. 其他机器：`consul force leave 节点Id`

### 健康检查和故障转移

1. 问题描述
    1. 在集群环境下，健康检查是由微服务注册到的 `Client` 来处理的，那么如果这个 `Client` 挂掉了（微服务和 `Client` 在同一台机器上），那么此节点的健康检查就处于无人管理的状态。
    2. 从实际应用看，节点上的服务可能既要被发现，又要发现别的服务，如果节点挂掉了，仅提供被发现的功能（已经注册了）实际上服务还是不可用的。
2. 解决方案
    1.  对微服务和 `Client` 的整体做集群（多个实例和多个`Client`）来解决问题
    2. 当然发现别的微服务也可以不使用本机节点，可以通过访问一个 `Nginx` 实现的若干`Consul`节点的负载均衡来实现。
