---
layout: post
title:  "SpringCloud-Eureka配置"
date:   2020-9-2
categories: SpringCloud
tags:  SpringCloud
author: Clear Li
---







































# Eureka配置

## pom配置

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>org.example</groupId>
  <artifactId>SpringCloudLearn9001</artifactId>
  <version>1.0-SNAPSHOT</version>

  <name>SpringCloudLearn9001</name>
  <!-- FIXME change it to the project's website -->
  <url>http://www.example.com</url>


  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
    <!-- SpringCloud版本，是最新的F系列 -->
    <spring-cloud.version>Finchley.RELEASE</spring-cloud.version>
    <mybatis.starter.version>1.3.2</mybatis.starter.version>
    <mapper.starter.version>2.0.2</mapper.starter.version>
    <druid.starter.version>1.1.9</druid.starter.version>
    <mysql.version>5.1.32</mysql.version>
    <pageHelper.starter.version>1.2.3</pageHelper.starter.version>
    <leyou.latest.version>1.0.0-SNAPSHOT</leyou.latest.version>
    <fastDFS.client.version>1.26.1-RELEASE</fastDFS.client.version>
  </properties>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.3.RELEASE</version>
  </parent>
  <!-- 管理依赖 -->
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-dependencies</artifactId>
        <version>${spring-cloud.version}</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>
  <dependencies>
    <!--SpringCloud eureka-server -->
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.9</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
  <!-- 注意： 这里必须要添加， 否者各种依赖有问题 -->
  <repositories>
    <repository>
      <id>spring-milestones</id>
      <name>Spring Milestones</name>
      <url>https://repo.spring.io/libs-milestone</url>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </repository>
  </repositories>


  <build>
    <pluginManagement><!-- lock down plugins versions to avoid using Maven defaults (may be moved to parent pom) -->
      <plugins>
        <!-- clean lifecycle, see https://maven.apache.org/ref/current/maven-core/lifecycles.html#clean_Lifecycle -->
        <plugin>
          <artifactId>maven-clean-plugin</artifactId>
          <version>3.1.0</version>
        </plugin>
        <!-- default lifecycle, jar packaging: see https://maven.apache.org/ref/current/maven-core/default-bindings.html#Plugin_bindings_for_jar_packaging -->
        <plugin>
          <artifactId>maven-resources-plugin</artifactId>
          <version>3.0.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.8.0</version>
        </plugin>
        <plugin>
          <artifactId>maven-surefire-plugin</artifactId>
          <version>2.22.1</version>
        </plugin>
        <plugin>
          <artifactId>maven-jar-plugin</artifactId>
          <version>3.0.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-install-plugin</artifactId>
          <version>2.5.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-deploy-plugin</artifactId>
          <version>2.8.2</version>
        </plugin>
        <!-- site lifecycle, see https://maven.apache.org/ref/current/maven-core/lifecycles.html#site_Lifecycle -->
        <plugin>
          <artifactId>maven-site-plugin</artifactId>
          <version>3.7.1</version>
        </plugin>
        <plugin>
          <artifactId>maven-project-info-reports-plugin</artifactId>
          <version>3.0.0</version>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>
</project>

```

## Server   yml配置

```
###eureka 服务端口号
server:
  port: 9100
###服务注册名称
eureka:
  instance:
    hostname: 127.0.0.1
  ###客户端调用地址
  client:
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:8100/eureka/
    ###因为该应用为注册中心，不会注册自己
    register-with-eureka: true
    ###因为自己为注册中心 ,不会去在该应用中的检测服务
    fetch-registry: true
  server:
    #测试关闭自我保护机制保证 不可用服务及时提出
    enable-self-preservation: false
    eviction-interval-timer-in-ms: 2000
spring:
  application:
    name: app-clear-eureka

```

注意保护机制的配置，在生产环境下建议关闭自我保护机制。但是在线上环境要打开

## Client yml配置

```
###eureka 服务端口号
server:
  port: 8200
###服务注册名称
eureka:
  instance:
    hostname: 127.0.0.1
    ##配置心跳时间间隔  
    lease-renewal-interval-in-seconds: 1
    ##配置服务端心跳等待时间上线，两秒内没收到则剔除这个服务
    lease-expiration-duration-in-seconds: 2
  ###客户端调用地址

  client:
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:8100/eureka/,http://${eureka.instance.hostname}:9100/eureka/
    ###因为该应用为注册中心，不会注册自己
    register-with-eureka: true
    ###检测应用中的服务
    fetch-registry: true
spring:
  application:
    name: app-clear-order


```

# zk配置

### pom文件

 

```
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>org.example</groupId>
  <artifactId>springCloud-zk</artifactId>
  <version>1.0-SNAPSHOT</version>

  <name>springCloud-zk</name>
  <!-- FIXME change it to the project's website -->
  <url>http://www.example.com</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
    <!-- SpringCloud版本，是最新的F系列 -->
    <spring-cloud.version>Finchley.RELEASE</spring-cloud.version>
    <mybatis.starter.version>1.3.2</mybatis.starter.version>
    <mapper.starter.version>2.0.2</mapper.starter.version>
    <druid.starter.version>1.1.9</druid.starter.version>
    <mysql.version>5.1.32</mysql.version>
    <pageHelper.starter.version>1.2.3</pageHelper.starter.version>
    <leyou.latest.version>1.0.0-SNAPSHOT</leyou.latest.version>
    <fastDFS.client.version>1.26.1-RELEASE</fastDFS.client.version>
  </properties>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.3.RELEASE</version>
  </parent>
  <!-- 管理依赖 -->
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-dependencies</artifactId>
        <version>${spring-cloud.version}</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>
  <dependencies>
    <!--SpringCloud eureka-server -->
    <!-- SpringBoot整合Web组件 -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.9</version>
      <scope>test</scope>
    </dependency>
    <!-- SpringBoot整合eureka客户端 -->
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
      <exclusions>
        <!--移除3.5.3-->
        <exclusion>
          <artifactId>zookeeper</artifactId>
          <groupId>org.apache.zookeeper</groupId>
        </exclusion>
      </exclusions>
    </dependency>
    <dependency>
      <groupId>org.apache.zookeeper</groupId>
      <artifactId>zookeeper</artifactId>
      <version>3.4.9</version>
    </dependency>

  </dependencies>
  <!-- 注意： 这里必须要添加， 否者各种依赖有问题 -->
  <repositories>
    <repository>
      <id>spring-milestones</id>
      <name>Spring Milestones</name>
      <url>https://repo.spring.io/libs-milestone</url>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </repository>
  </repositories>


  <build>
    <pluginManagement><!-- lock down plugins versions to avoid using Maven defaults (may be moved to parent pom) -->
      <plugins>
        <!-- clean lifecycle, see https://maven.apache.org/ref/current/maven-core/lifecycles.html#clean_Lifecycle -->
        <plugin>
          <artifactId>maven-clean-plugin</artifactId>
          <version>3.1.0</version>
        </plugin>
        <!-- default lifecycle, jar packaging: see https://maven.apache.org/ref/current/maven-core/default-bindings.html#Plugin_bindings_for_jar_packaging -->
        <plugin>
          <artifactId>maven-resources-plugin</artifactId>
          <version>3.0.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.8.0</version>
        </plugin>
        <plugin>
          <artifactId>maven-surefire-plugin</artifactId>
          <version>2.22.1</version>
        </plugin>
        <plugin>
          <artifactId>maven-jar-plugin</artifactId>
          <version>3.0.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-install-plugin</artifactId>
          <version>2.5.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-deploy-plugin</artifactId>
          <version>2.8.2</version>
        </plugin>
        <!-- site lifecycle, see https://maven.apache.org/ref/current/maven-core/lifecycles.html#site_Lifecycle -->
        <plugin>
          <artifactId>maven-site-plugin</artifactId>
          <version>3.7.1</version>
        </plugin>
        <plugin>
          <artifactId>maven-project-info-reports-plugin</artifactId>
          <version>3.0.0</version>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>
</project>

```

注意zk版本关系

### yml配置

```
###订单服务的端口号
server:
  port: 8002
###服务别名----服务注册到注册中心名称
spring:
  application:
    name: zk-member
  cloud:
    zookeeper:
      connect-string: 127.0.0.1:2181

```



# consul配置

启动consul命

consul agent -dev -ui -node=cy

-dev开发服务器模式启动，-node结点名为cy，-ui可以用界面访问，默认能访问。

 

测试访问地址:http://localhost:8500 

## pom文件

```
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>org.example</groupId>
  <artifactId>springCloud-zk</artifactId>
  <version>1.0-SNAPSHOT</version>

  <name>springCloud-zk</name>
  <!-- FIXME change it to the project's website -->
  <url>http://www.example.com</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
    <!-- SpringCloud版本，是最新的F系列 -->
    <spring-cloud.version>Finchley.RELEASE</spring-cloud.version>
    <mybatis.starter.version>1.3.2</mybatis.starter.version>
    <mapper.starter.version>2.0.2</mapper.starter.version>
    <druid.starter.version>1.1.9</druid.starter.version>
    <mysql.version>5.1.32</mysql.version>
    <pageHelper.starter.version>1.2.3</pageHelper.starter.version>
    <leyou.latest.version>1.0.0-SNAPSHOT</leyou.latest.version>
    <fastDFS.client.version>1.26.1-RELEASE</fastDFS.client.version>
  </properties>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.3.RELEASE</version>
  </parent>
  <!-- 管理依赖 -->
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-dependencies</artifactId>
        <version>${spring-cloud.version}</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>
  <dependencies>
    <!--SpringCloud eureka-server -->
    <!-- SpringBoot整合Web组件 -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.9</version>
      <scope>test</scope>
    </dependency>
    <!-- SpringBoot整合eureka客户端 -->
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-consul-discovery</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>


  </dependencies>
  <!-- 注意： 这里必须要添加， 否者各种依赖有问题 -->
  <repositories>
    <repository>
      <id>spring-milestones</id>
      <name>Spring Milestones</name>
      <url>https://repo.spring.io/libs-milestone</url>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </repository>
  </repositories>


  <build>
    <pluginManagement><!-- lock down plugins versions to avoid using Maven defaults (may be moved to parent pom) -->
      <plugins>
        <!-- clean lifecycle, see https://maven.apache.org/ref/current/maven-core/lifecycles.html#clean_Lifecycle -->
        <plugin>
          <artifactId>maven-clean-plugin</artifactId>
          <version>3.1.0</version>
        </plugin>
        <!-- default lifecycle, jar packaging: see https://maven.apache.org/ref/current/maven-core/default-bindings.html#Plugin_bindings_for_jar_packaging -->
        <plugin>
          <artifactId>maven-resources-plugin</artifactId>
          <version>3.0.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.8.0</version>
        </plugin>
        <plugin>
          <artifactId>maven-surefire-plugin</artifactId>
          <version>2.22.1</version>
        </plugin>
        <plugin>
          <artifactId>maven-jar-plugin</artifactId>
          <version>3.0.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-install-plugin</artifactId>
          <version>2.5.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-deploy-plugin</artifactId>
          <version>2.8.2</version>
        </plugin>
        <!-- site lifecycle, see https://maven.apache.org/ref/current/maven-core/lifecycles.html#site_Lifecycle -->
        <plugin>
          <artifactId>maven-site-plugin</artifactId>
          <version>3.7.1</version>
        </plugin>
        <plugin>
          <artifactId>maven-project-info-reports-plugin</artifactId>
          <version>3.0.0</version>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>
</project>

```

## yml配置

```
server:
  port: 8502
spring:
  application:
    name: consul-member
  ####consul注册中心地址
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        hostname: 192.168.1.102

```

