---
banner: http://icepear.oss-cn-shenzhen.aliyuncs.com/springboot/1/spring.jpg
title: spring cloud gateway 2.2.2 中文文档
date: 2020-05-12 21:30:02
toc: true
tags: 
- Spring Cloud Gateway
categories:
- springcloud
---
本文档基于官网()[https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.2.RELEASE/reference/html/] 2.2.2.RELEASE版本进行翻译，中间按照自己的理解并不是全文照搬。
<!--more-->

# 1. 怎样引入 Spring Cloud Gateway
要在项目中引入Spring Cloud Gateway，需要引用 group org.springframework.cloud 和 artifact id为spring-cloud-starter-gateway starter。最新的Spring Cloud Release 构建信息，请参阅(Spring Cloud Project page)[https://spring.io/projects/spring-cloud]。

如果应用了该starter，但由于某种原因不希望启用网关，请进行设置spring.cloud.gateway.enabled=false。

```
警告：
Spring Cloud Gateway基于Spring Boot 2.x，Spring WebFlux和Project Reactor构建。
当您使用Spring Cloud Gateway时，许多您熟悉的同步库（例如，Spring Data和Spring Security）和模式可能不适用。
如果您对这些项目不熟悉，建议您在使用Spring Cloud Gateway之前先阅读它们的文档以熟悉一些新概念。
```
```
警告：
Spring Cloud Gateway依赖Spring Boot和Spring Webflux提供的Netty runtime。它不能在传统的Servlet容器中工作或构建为WAR
```

# 2. 核心关键字

- **Route 路由**：gateway的基本构建模块。它由ID、目标URI、断言集合和过滤器集合组成。如果聚合断言结果为真，则匹配到该路由。
- **Predicate 断言**：这是一个Java 8 Function Predicate。输入类型是 Spring Framework ServerWebExchange。这允许开发人员可以匹配来自HTTP请求的任何内容，例如Header或参数。
- **Filter 过滤器**：这些是使用特定工厂构建的 Spring FrameworkGatewayFilter实例。所以可以在返回请求之前或之后修改请求和响应的内容。

# 3. 工作原理

下图从总体上概述了Spring Cloud Gateway的工作方式：

!()[https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.2.RELEASE/reference/html/images/spring_cloud_gateway_diagram.png]

1. 客户端向Spring Cloud Gateway发出请求。
2. 如果网关处理程序映射确定请求与路由匹配，则将其发送到网关Web处理程序。
3. 该处理程序通过特定于请求的过滤器链来运行请求。
3. 筛选器由虚线分隔的原因是，筛选器可以在发送代理请求之前和之后运行逻辑。
5. 所有“前置”过滤器逻辑均被执行。
6. 然后发出代理请求。
7. 发出代理请求后，将运行“后”过滤器逻辑。

在没有端口的路由中定义的URI，HTTP和HTTPS URI的默认端口值分别为80和443。

# 4. 配置路由断言工厂和过滤器工厂的方式

有两种配置断言和过滤器的方法,下面的大多数示例都使用快捷方式。名称和参数名称将在每个部分的第一个或两个句子中以代码形式列出。参数通常按快捷方式配置所需的顺序列出。

## 4.1 快捷方式
快捷方式配置由过滤器名称识别，后跟一个等号（=），然后是由逗号分隔的参数值（，）例如

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - Cookie=mycookie,mycookievalue
```

## 4.2 完全扩展的参数。
完全扩展的参数看起来更像带有名称/值对的标准Yaml配置。通常，将有一个名称键和一个args键。args键是用于配置谓词或过滤器的键值对的映射
```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - name: Cookie
          args:
            name: mycookie
            regexp: mycookievalue
```

# 5. 路由断言工厂

## 5.1. The After Route Predicate Factory

After Route Predicate Factory采用一个参数——日期时间。在该日期时间之后发生的请求都将被匹配。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - After=2017-01-20T17:42:47.789-07:00[America/Denver]
```

## 5.2. The Before Route Predicate Factory

Before Route Predicate Factory采用一个参数——日期时间。在该日期时间之前发生的请求都将被匹配。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: before_route
        uri: https://example.org
        predicates:
        - Before=2017-01-20T17:42:47.789-07:00[America/Denver]
```

## 5.3. The Between Route Predicate Factory

Between 路由断言 Factory有两个参数，datetime1和datetime2。在datetime1和datetime2之间的请求将被匹配。datetime2参数的实际时间必须在datetime1之后。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: between_route
        uri: https://example.org
        predicates:
        - Between=2017-01-20T17:42:47.789-07:00[America/Denver], 2017-01-21T17:42:47.789-07:00[America/Denver]
```

## 5.4. The Cookie Route Predicate Factory

Cookie 路由断言有两个参数，cookie名称和regexp正则表达式。请求包含次cookie名称且正则表达式为真的将会被匹配。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: cookie_route
        uri: https://example.org
        predicates:
        - Cookie=chocolate, ch.p
```


## 5.5. The Header Route Predicate Factory

Header 路由断言有两个参数，header名称和regexp正则表达式。请求包含次header名称且正则表达式为真的将会被匹配。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: header_route
        uri: https://example.org
        predicates:
        - Header=X-Request-Id, \d+
```

## 5.6. The Host Route Predicate Factory

Host 路由断言包括一个参数：主机名模式列表patterns。使用Ant路径匹配规则，.作为分隔符

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: https://example.org
        predicates:
        - Host=**.somehost.org,**.anotherhost.org
```

## 5.7. The Method Route Predicate Factory

Method 路由断言只包含一个参数: 需要匹配的HTTP请求方式

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: method_route
        uri: https://example.org
        predicates:
        - Method=GET,POST
```

## 5.8. The Path Route Predicate Factory

路径路由断言使用两个参数：Spring PathMatcher模式列表和一个称为matchOptionalTrailingSeparator的可选标志。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: path_route
        uri: https://example.org
        predicates:
        - Path=/red/{segment},/blue/{segment}
```
如果请求路径为例如/red/1或/red/blue或/blue/green，则此路由匹配。
要获取segment的值可以采用

```java
Map<String, String> uriVariables = ServerWebExchangeUtils.getPathPredicateVariables(exchange);

String segment = uriVariables.get("segment");
```

## 5.9. The Query Route Predicate Factory

Query 路由断言 Factory 有2个参数: 必选项 param 和可选项 regexp.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: query_route
        uri: https://example.org
        predicates:
        - Query=green
```
如果请求包含green查询参数，则上述路由匹配。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: query_route
        uri: https://example.org
        predicates:
        - Query=red, gree.
```
如果请求包含red查询参数，并且值包含gree正则规则的，则上述路由匹配。

## 5.10. The RemoteAddr Route Predicate Factory

RemoteAddr 路由断言的参数为 一个CIDR符号（IPv4或IPv6）字符串的列表，最小值为1，例如192.168.0.1/16（其中192.168.0.1是IP地址并且16是子网掩码）。


```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: remoteaddr_route
        uri: https://example.org
        predicates:
        - RemoteAddr=192.168.1.1/24
```

如果请求的远程地址是例如192.168.1.10，则此路由匹配。

## 5.11. The Weight Route Predicate Factory

权重路由断言的参数为：group 和 weight ，权重按组计算

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: weight_high
        uri: https://weighthigh.org
        predicates:
        - Weight=group1, 8
      - id: weight_low
        uri: https://weightlow.org
        predicates:
        - Weight=group1, 2
```

这条路线会将约80％的流量转发至weighthigh.org，并将约20％的流量转发至weightlow.org。

### 5.11.1 修改远程地址的解析方式
默认情况下，RemoteAddr 路由断言使用传入请求中的remote address。如果Spring Cloud Gateway位于代理层后面，也就是说网关前面还配置了一个Nginx之类的反向代理，则可能与实际客户端IP地址不匹配。

可以通过设置自定义RemoteAddressResolver来自定义解析远程地址的方式。Spring Cloud Gateway网关附带一个非默认远程地址解析程序，它基于X-Forwarded-For header, XForwardedRemoteAddressResolver.

XForwardedRemoteAddressResolver 有两个静态构造函数方法，采用不同的安全方法：

XForwardedRemoteAddressResolver:：TrustAll返回一个RemoteAddressResolver，它始终采用X-Forwarded-for头中找到的第一个IP地址。这种方法容易受到欺骗，因为恶意客户端可能会为解析程序接受的“x-forwarded-for”设置初始值。

XForwardedRemoteAddressResolver:：MaxTrustedIndex获取一个索引，会在X-Forwarded-For信息里面，从右到左选择信任最多maxTrustedIndex个ip，因为X-Forwarded-For是越往右是越接近gateway的代理机器ip，所以是越往右的ip，信任度是越高的。
那么如果前面只是挡了一层Nginx的话，如果只需要Nginx前面客户端的ip，则maxTrustedIndex取1，就可以比较安全地获取真实客户端ip,如果在访问Spring Cloud Gateway之前需要两个受信任的基础代理点，那么应该使用2。

例如索引是下面这样的值：

```text
X-Forwarded-For: 0.0.0.1, 0.0.0.2, 0.0.0.3
```

那么当我们使用下面的maxTrustedIndex值将生成以下远程地址:

|maxTrustedIndex|result|
| - | - |
| [Integer.MIN_VALUE,0] | (invalid, IllegalArgumentException during initialization) |
| 1 | 0.0.0.3 |
| 2 | 0.0.0.2 |
| 3 | 0.0.0.1 |
| [4, Integer.MAX_VALUE] | 0.0.0.1 |

Java 配置方式:
GatewayConfig.java
```java
RemoteAddressResolver resolver = XForwardedRemoteAddressResolver
    .maxTrustedIndex(1);

...

.route("direct-route",
    r -> r.remoteAddr("10.1.1.1", "10.10.1.1/24")
        .uri("https://downstream1")
.route("proxied-route",
    r -> r.remoteAddr(resolver, "10.10.1.1", "10.10.1.1/24")
        .uri("https://downstream2")
)
```

# 6. 路由过滤器


```yaml

```

# 7. 全局过滤器


```yaml

```

# 8. 请求头过滤器


```yaml

```

# 9. TLS and SSL


```yaml

```

# 10. 配置


```yaml

```

# 11. 路由原数据配置


```yaml

```

# 12. Http超时配置


```yaml

```

# 13. Reactor Netty 访问日志


```yaml

```

# 14. CORS 配置


```yaml

```

# 15. Actuator API


```yaml

```

# 16. 故障排除


```yaml

```

# 17. 开发者想到


```yaml

```

# 18. 使用 Spring MVC 或 Webflux创建一个简单的路由


```yaml

```

# 19. 配置属性表


```yaml

```
