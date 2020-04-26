---
layout: post
title:  "配置Springboot在Nginx下的session共享问题"
date:   2020-4-26
categories: 高可用
tags:  Nginx
author: Clear Li
---











在分布式下首要问题就是要解决session不一致的问题。要让用户每次访问都访问同一个session。下面是解决办法









### 1 nginx做的负载均衡)

用Nginx 做的负载均衡可以添加ip_hash这个配置，

从而使同一个ip的请求发到同一台服务器。

这种方法，没有做到负载均衡，而是一直访问同一个服务器

### 2 利用数据库同步session

这种以前我使用过。使用数据库同步将会给数据库带来巨大的压力，且速度将会很慢。

是非常不好的一种方法，切身体会。

### 3利用cookie同步session数据

具体就是将session的数据存入cookie，这种方法有两方面问题，第一占用带宽，因为cookie要发送给客户端。

第二不安全，将用户数据放入cookie很容易被截取。

### 4使用redis存session

这种是目前来说比较好的一种模式。速度方面也是很快的。由于spring boot已经封装好了，因此具体操作也很简单

首先在pom文件中引入依赖

```xml
 <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-redis</artifactId>
    </dependency>
    <!--spring session 与redis应用基本环境配置,需要开启redis后才可以使用，不然启动Spring boot会报错 -->
    <dependency>
      <groupId>org.springframework.session</groupId>
      <artifactId>spring-session-data-redis</artifactId>
    </dependency>
```

在spring的配置文件中加入redis的连接配置

```
spring.redis.database=0
spring.redis.host=192.168.206.22
spring.redis.port=6379
#spring.redis.password=123456
spring.redis.pool.max-idle=8
spring.redis.pool.min-idle=0
spring.redis.pool.max-active=8
spring.redis.pool.max-wait=-1
spring.redis.timeout=5000
```

创建共享session的配置文件

```java
package com.clear.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.data.redis.connection.jedis.JedisConnectionFactory;
import org.springframework.session.data.redis.config.annotation.web.http.EnableRedisHttpSession;

//这个类用配置redis服务器的连接
//maxInactiveIntervalInSeconds为SpringSession的过期时间（单位：秒）
@EnableRedisHttpSession(maxInactiveIntervalInSeconds = 1800)
public class SessionConfig {

    // 冒号后的值为没有配置文件时，制动装载的默认值
    @Value("${spring.redis.host:localhost}")
    String HostName;
    @Value("${redis.port:6379}")
    int Port;

    @Bean
    public JedisConnectionFactory connectionFactory() {
        JedisConnectionFactory connection = new JedisConnectionFactory();
        connection.setPort(Port);
        connection.setHostName(HostName);
        return connection;
    }
}


```

创建初始化session的类

```java
public class SessionInitializer extends AbstractHttpSessionApplicationInitializer {
    public SessionInitializer() {
        super(SessionConfig.class);
    }
}

```

配置nginx的ip来测试一番

![image-20200426161037051](/img/image-20200426161037051.png)

![image-20200426223922658](/img/image-20200426223922658.png)

![image-20200426223950344](/img/image-20200426223950344.png)

共享session成功