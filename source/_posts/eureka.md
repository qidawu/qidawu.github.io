---
title: 注册中心 Eureka 客户端无损下线相关配置
date: 2021-09-20 17:48:49
updated:
tags: Java
typora-root-url: ..
---

# Eureka 服务注册发现流程

![eureka workflow](/img/spring/cloud/eureka-workflow.png)

![eureka cluster](/img/spring/cloud/eureka-cluster.png)

![eureka config](/img/spring/cloud/eureka-config.png)

# Eureka 配置修改

## Eureka client (consumer)

```bash
# Indicates whether this client should fetch eureka registry information from eureka server.
# 指示此 eureka client 是否应从 eureka server 获取注册表信息。（缓存到本地，后面会增量获取）
eureka.client.fetch-registry=true

# Indicates how often(in seconds) to fetch the registry information from the eureka server.
# 指示此 eureka client 从 eureka server 获取注册表信息的频率（以秒为单位）。可用于 eureka client 下线后，快速感知
eureka.client.registry-fetch-interval-seconds=30
```

## Eureka client (provider)

```bash
# Map of availability zone to list of fully qualified URLs to communicate with eureka server. Each value can be a single URL or a comma separated list of alternative locations. Typically the eureka server URLs carry protocol,host,port,context and version information if any. Example: http://ec2-256-156-243-129.compute-1.amazonaws.com:7001/eureka/ The changes are effective at runtime at the next service url refresh cycle as specified by eurekaServiceUrlPollIntervalSeconds.
eureka.client.service-url.default-zone: http://localhost:8000/eureka/

# Indicates whether or not this instance should register its information with eureka server for discovery by others. In some cases, you do not want your instances to be discovered whereas you just want do discover other instances.
# 指示此实例是否应向 eureka server 注册其信息以供其它服务发现。两种情况下会设置为 false：1、您不希望您的实例被发现，而您只想发现其他实例。2、eureka server 以单机模式运行（https://cloud.spring.io/spring-cloud-netflix/reference/html/#spring-cloud-eureka-server-standalone-mode）
eureka.client.register-with-eureka=true

# Indicates how often (in seconds) the eureka client needs to send heartbeats to eureka server to indicate that it is still alive. If the heartbeats are not received for the period specified in leaseExpirationDurationInSeconds, eureka server will remove the instance from its view, there by disallowing traffic to this instance. Note that the instance could still not take traffic if it implements HealthCheckCallback and then decides to make itself unavailable.
# 服务续约时间(心跳时间) —— 指示 eureka client 需要多长时间（以秒为单位）向 eureka server 发送心跳以表明它仍然活着。如果 eureka server 在 lease-expiration-duration-in-second 中指定的时间段内未收到心跳，eureka server 将从其视图中删除该实例，从而限流该实例。请注意，如果实例实现 HealthCheckCallback 然后决定使其自身不可用，则该实例仍然无法获取流量。
eureka.instance.lease-renewal-interval-in-seconds=30
```

## Eureka server

```bash
# Indicates the time in seconds that the eureka server waits since it received the last heartbeat before it can remove this instance from its view and there by disallowing traffic to this instance. Setting this value too long could mean that the traffic could be routed to the instance even though the instance is not alive. Setting this value too small could mean, the instance may be taken out of traffic because of temporary network glitches.This value to be set to atleast higher than the value specified in leaseRenewalIntervalInSeconds.
# 服务过期时间 —— 指示 eureka server 自收到最后一次心跳后等待的时间（以秒为单位），然后才能从其视图中删除此实例，并限流该实例。将此值设置得太长可能意味着即使实例挂了，流量也可以路由到实例。将此值设置得太小可能意味着，由于临时网络故障，eureka server 未及时收到心跳，实例可能会被限流。此值至少要设置为高于 lease-renewal-interval-in-seconds 中指定的值。
eureka.instance.lease-expiration-duration-in-second=90

# Gets the time interval with which the task that expires instances should wake up and run.
# 服务剔除 TimerTask 执行时间
eureka.server.eviction-interval-timer-in-ms=60 * 1000

# Gets the time interval with which the payload cache of the client should be updated.
# 一级缓存更新 TimerTask 执行时间，定时将 L2（readWriteCacheMap）覆盖掉 L1（readOnlyCacheMap）
eureka.server.response-cache-update-interval-ms=30 * 1000
```

# Eureka 配置验证

配置修改后，可以请求端点：/actuator/configprops，验证配置，例如：

## Eureka server

查找关键字：responseCacheUpdateIntervalMs：

![eureka server configprops](/img/spring/cloud/eureka-server-configprops.png)

## Eureka client

Eureka client (Consumer) 查询关键字：registryFetchIntervalSeconds

![eureka client configprops](/img/spring/cloud/eureka-client-configprops.png)

Eureka client (Consumer) 可以请求端点：/actuator/health 验证服务下线的感知结果：

![eureka client health](/img/spring/cloud/eureka-client-health.png)

# Eureka client 接口上下线

Eureka client 服务下线接口：/actuator/service-registry?status=DOWN

```bash
curl --location --request POST 'http://127.0.0.1:58061/actuator/service-registry?status=DOWN' \
--header 'Content-Type: application/vnd.spring-boot.actuator.v2+json;charset=UTF-8'
```

![eureka client down](/img/spring/cloud/eureka-client-down.png)

Eureka client 服务上线接口：/actuator/service-registry?status=UP

```bash
curl --location --request POST 'http://127.0.0.1:58061/actuator/service-registry?status=UP' \
--header 'Content-Type: application/vnd.spring-boot.actuator.v2+json;charset=UTF-8'
```

![eureka client up](/img/spring/cloud/eureka-client-up.png)

Eureka server 验证结果：

```bash
curl --location --request GET 'http://192.168.33.42:8000/eureka/apps' \
--header 'Accept: application/json'
```

# Eureka client 手动上下线

![eureka client down and up](/img/spring/cloud/eureka-client-down-and-up.png)

# 参考

《[解决微服务架构下流量有损问题，优雅上下线的实践和探索 | 阿里技术](https://mp.weixin.qq.com/s/eQzy3zvvEokNXYL637LNCg)》

《[eureka client 下线后，快速感知](https://www.cnblogs.com/zhangs1986/p/10529815.html)》

