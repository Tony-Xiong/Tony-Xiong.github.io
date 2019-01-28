---
layout: post
title: "微服务架构相关技术分类"
date: 2019-01-28 10:18:00 +0800
categories: jekyll update
---

#### 网关
- Zuul
- Spring-Cloud-Gateway
- Kong

#### RPC
- Dubbo 配合zookeeper或者nacos使用，推荐使用nacos
- Ribbon 配合eureka使用
- Feign 配合eureka使用
- Motan
- ServerComb 需要配合华为的相关技术

#### MQ
- RocketMQ
- kafka
- RabbitMQ

#### Job
- Quartz
- Elastic-Job-Lite
- Elastic-Job-Cloud
- XXL-Job

#### 注册中心
- zookeeper
- Eureka
- Consul
- Ectd
- Alibaba Nacos Discovery

#### 配置中心
- Apollo
- Disconf
- Spring-cloud-config
- Alibaba Nacos Config

#### Tracing服务追踪
- CAT
- SkyWalking
- Zipkin
- Pinpoint

#### 服务器容器
- tomcat
- netty
- jetty
- Ngnix

#### java/J2EE
- Java 并发
- Java 集合
- spring

#### web层
- Spring MVC
- Spring WebFlux

#### ORM
- MyBatis
- Hiberante
- Spring-Data-JPA

#### 连接池
- Durid
- HikariCP

#### 数据库中间件

#### 数据库

##### SQL
- MySQL
- Oracle
- postgreSQL
- DB2
- H2
##### NoSQL
- redis
- mongoDB

#### 搜索
- Lucene
- Elastic-Search
- Solr

#### 服务熔断
- Hystrix
- Alibaba Sentinel

#### 其他