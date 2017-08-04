---
title: SpringBoot学习-基本使用
date: 2016-12-09 19:30:02
toc: true
tags:
   - SpringBoot
categories:
   - SpringBoot
---
在您第1次接触和学习Spring框架的时候，是否因为其繁杂的配置而退却了？在你第n次使用Spring框架的时候，是否觉得一堆反复黏贴的配置有一些厌烦？那么您就不妨来试试使用Spring Boot来让你更易上手，更简单快捷地构建Spring应用！
Spring Boot让我们的Spring应用变的更轻量化。比如：你可以仅仅依靠一个Java类来运行一个Spring引用。你也可以打包你的应用为jar并通过使用java -jar来运行你的Spring Web应用。
<!-- more -->
# 使用SpringBoot
## 开始
废话不多讲，为什么使用SpringBoot，就一个字：“爽”。快速入门，精简配置，开箱即用，独立运行这
些优点绝对会让你爱上它，当然最重要的一点就是微服务
## 构建项目
使用maven构建，一般继承 spring-boot-starter-parent 项目来获取合适的默认设置只需要简单
地设置 parent 为：
```java
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.4.0.RELEASE</version>
    </parent>
```
该父项目提供以下特性：
1. 默认编译级别为Java 1.6
2. 源码编码为UTF-8
3. 一个依赖管理节点,允许你省略普通依赖的 <version> 标签,继承自 spring-boot-dependencies POM。
4. 合适的资源过滤
5. 合适的插件配置（ exec插件， surefire， Git commit ID， shade）
6. 针对 application.properties 和 application.yml 的资源过滤
### 改变属性
可以使用 <properties> 标签，例如
```java
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>//项目编码格式
    <java.version>1.8</java.version>//改变java编译版本
    <docker.image.prefix>willmin</docker.image.prefix>//其他属性配置
    <docker.plugin.version>0.4.12</docker.plugin.version>
</properties>
```
### 打包插件
让springboot应用独立运行，需要将应用导成可执行的jar，可以利用Spring Boot Maven插件，在<plugins>中配置
在pom中写入：
```java
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```
### 依赖项目列表
在pom中可以轻易引用springboot的各种依赖，所有的starters遵循一个相似的命名模式： spring-boot-starter-* 
* 是一种特殊类型的应用程序，该命名结构旨在帮你找到需要的starter。
表 13.1. Spring Boot application starters

| 名称    |  描述 |
|-------------------------------------------------------|------|
|  **spring-boot-starter** | 核心Spring Boot starter， 包括自动配置支持， 日志和YAML |
|  **spring-boot-starter-actuator** | 生产准备的特性， 用于帮你监控和管理应用 |
|  **spring-boot-starter-amqp** | 对"高级消息队列协议"的支持， 通过 spring-rabbit 实现 |
|  **spring-boot-starter-aop** | 对面向切面编程的支持， 包括 spring-aop 和AspectJ |
|  **spring-boot-starter-test** | 对常用测试依赖的支持， 包括JUnit, Hamcrest和Mockito， 还有 spring-test 模块 |
|  **spring-boot-starter-web** | 对全栈web开发的支持， 包括Tomcat和 spring-webmvc |
|  **spring-boot-starter-websocket** | 对WebSocket开发的支持 |
|  **spring-boot-starter-mail** | 对 javax.mail 的支持  |
|  **spring-boot-starter-mobile** | 对 spring-mobile 的支持 |
|  **spring-boot-starter-mustache** | 对Mustache模板引擎的支持 |
|  **spring-boot-starter-redis** | 对REDIS键值数据存储的支持， 包括 spring-redis |
|  **spring-boot-starter-security** | 对 spring-security 的支持 |
|  **spring-boot-starter-jdbc** | 对JDBC数据库的支持 |
|  **spring-boot-starter-data-jpa** | 对"Java持久化API"的支持， 包括 spring-data-jpa ， spring-orm 和Hibernate |
|  **spring-boot-starter-data-rest** | 对通过REST暴露Spring Data仓库的支持， 通过 spring-data-rest-webmvc 实现 |
|  spring-boot-starter-thymeleaf | 对Thymeleaf模板引擎的支持， 包括和Spring的集成 |
|  spring-boot-starter-velocity | 对Velocity模板引擎的支持 |
|  spring-boot-starter-batch | 对Spring Batch的支持， 包括HSQLDB数据库 |
|  spring-boot-starter-cloudconnectors | 对Spring Cloud Connectors的支持， 简化在云平台下(例如，Cloud Foundry和Heroku)服务的连接 |
|  spring-boot-starter-dataelasticsearch | 对Elasticsearch搜索和分析引擎的支持， 包括 spring-data-elasticsearch |
|  spring-boot-starter-datagemfire | 对GemFire分布式数据存储的支持， 包括 spring-data-gemfire |
|  spring-boot-starter-datamongodb | 对MongoDB NOSQL数据库的支持， 包括 spring-data-mongodb |
|  spring-boot-starter-data-solr | 对Apache Solr搜索平台的支持， 包括 spring-data-solr |
|  spring-boot-starter-freemarker | 对FreeMarker模板引擎的支持 |
|  spring-boot-starter-groovytemplates | 对Groovy模板引擎的支持 |
|  spring-boot-starter-hateoas | 对基于HATEOAS的RESTful服务的支持， 通过 spring-hateoas 实现 |
|  spring-boot-starter-hornetq | 对"Java消息服务API"的支持， 通过HornetQ实现 |
|  spring-boot-starter-integration | 对普通 spring-integration 模块的支持 |
|  spring-boot-starter-jersey | 对Jersey RESTful Web服务框架的支持 |
|  spring-boot-starter-jtaatomikos | 对JTA分布式事务的支持， 通过Atomikos实现 |
|  spring-boot-starter-jta-bitronix | 对JTA分布式事务的支持， 通过Bitronix实现 |
|  spring-boot-starter-socialfacebook | 对 spring-social-facebook 的支持 |
|  spring-boot-starter-sociallinkedin | 对 spring-social-linkedin 的支持 |
|  spring-boot-starter-socialtwitter | 对 spring-social-twitter 的支持 |
|  spring-boot-starter-ws | 对Spring Web服务的支持 |
|  spring-boot-starter-jetty | 导入Jetty HTTP引擎（ 作为Tomcat的替代）|
|  spring-boot-starter-log4j | 对Log4J日志系统的支持 |
|  spring-boot-starter-logging | 导入Spring Boot的默认日志系统（ Logback）|
|  spring-boot-starter-tomcat | 导入Spring Boot的默认HTTP引擎（ Tomcat）|

### 代码结构
springboot应用并不要求任何特殊的代码结构，我一般是这么写的

```javascript
|-main
    |-java/com/icepear/项目名
        |-web
            |-Controller
        |-service
            |-impl
                |-实现
            |-接口
        |-domain
            |-enums
            |-interfaces
                |-Repository.java
            |-实体.java
    |-docker
        |-Dockerfile
    |-resources
        |-application.yml
```
