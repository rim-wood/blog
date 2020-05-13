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

路由过滤器允许以某种方式修改传入的HTTP请求或传出的HTTP响应。路由过滤器适用于特定路由。Spring Cloud Gateway包括许多内置的GatewayFilter工厂。
下面一一列举：
详细的使用说明请参考[测试代码](https://github.com/spring-cloud/spring-cloud-gateway/tree/master/spring-cloud-gateway-core/src/test/java/org/springframework/cloud/gateway/filter/factory)

## 6.1 AddRequestHeader GatewayFilter Factory

添加请求头的过滤器，有两个参数，name和value。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_header_route
        uri: https://example.org
        filters:
        - AddRequestHeader=X-Request-red, blue
```
将在请求头里面加入 X-Request-red:blue 这样的参数进去，下游就可以通过新的请求头拿到对应的值。

当然也可以使用参数注入的方式，例如下面这样，将path路径上的参数，放入到header中：

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_header_route
        uri: https://example.org
        predicates:
        - Path=/red/{segment}
        filters:
        - AddRequestHeader=X-Request-Red, Blue-{segment}
```

## 6.2. The AddRequestParameter GatewayFilter Factory

添加参数的过滤器，有两个参数，name和value。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_parameter_route
        uri: https://example.org
        filters:
        - AddRequestParameter=red, blue
```
这会将red = blue添加到所有匹配请求的下游请求的查询字符串中。

同样的也可以采用参数注入的方式，例如下面这样，将Host的参数，放入到查询字符串中：

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_parameter_route
        uri: https://example.org
        predicates:
        - Host: {segment}.myhost.org
        filters:
        - AddRequestParameter=foo, bar-{segment}
```

## 6.3. The AddResponseHeader GatewayFilter Factory

添加返回头过滤器，有两个参数，name和value。
跟上面差不多，不写了。

## 6.4. The DedupeResponseHeader GatewayFilter Factory

重复参数返回头删除过滤器，有两个参数 name 和 strategy ，name是返回头中的参数名称，strategy是策略，包含三种策略：RETAIN_FIRST (默认), RETAIN_LAST, RETAIN_UNIQUE.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: dedupe_response_header_route
        uri: https://example.org
        filters:
        - DedupeResponseHeader=Access-Control-Allow-Credentials Access-Control-Allow-Origin
```

这个例子表示当网关CORS逻辑和下游逻辑都将它们添加时，这将删除Access-Control-Allow-Credentials和Access-Control-Allow-Origin响应标头的重复值。

## 6.5. The Hystrix GatewayFilter Factory

Hystrix 是Netflix开源的断路器组件。Hystrix GatewayFilter允许你向网关路由引入断路器，保护你的服务不受级联故障的影响，并允许你在下游故障时提供fallback响应。

要在项目中启用Hystrix网关过滤器，需要添加对 spring-cloud-starter-netflix-hystrix的依赖

Hystrix GatewayFilter Factory 需要一个name参数，即HystrixCommand的名称。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: hystrix_route
        uri: https://example.org
        filters:
        - Hystrix=myCommandName
```

这将剩余的过滤器包装在命令名为“myCommandName”的HystrixCommand中。

hystrix过滤器还可以接受可选的fallbackUri 参数。目前，仅支持forward: 预设的URI，如果调用fallback，则请求将转发到与URI匹配的控制器。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: hystrix_route
        uri: lb://backing-service:8088
        predicates:
        - Path=/consumingserviceendpoint
        filters:
        - name: Hystrix
          args:
            name: fallbackcmd
            fallbackUri: forward:/incaseoffailureusethis
        - RewritePath=/consumingserviceendpoint, /backingserviceendpoint
```

当调用hystrix fallback时，这将转发到/incaseoffailureusethis。
*注意，这个示例还演示了（可选）通过目标URI上的'lb`前缀,使用Spring Cloud Netflix Ribbon 客户端负载均衡。*

主要场景是使用fallbackUri 到网关应用程序中的内部控制器或处理程序。但是，也可以将请求重新路由到外部应用程序中的控制器或处理程序，如：

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: ingredients
        uri: lb://ingredients
        predicates:
        - Path=//ingredients/**
        filters:
        - name: Hystrix
          args:
            name: fetchIngredients
            fallbackUri: forward:/fallback
      - id: ingredients-fallback
        uri: http://localhost:9994
        predicates:
        - Path=/fallback
```

在上面例子中，gateway应用程序中没有 fallback 实现，但是另一个应用程序中有一个接口实现，注册为“http://localhost:9994”。

在将请求转发到fallback的情况下，Hystrix Gateway过滤还支持直接抛出Throwable 。它被作为ServerWebExchangeUtils.HYSTRIX_EXECUTION_EXCEPTION_ATTR属性添加到ServerWebExchange中，可以在处理网关应用程序中的fallback时使用。

对于外部控制器/处理程序方案，可以添加带有异常详细信息的header。可以在下面6.7的 FallbackHeaders GatewayFilter Factory section.中找到有关它的更多信息。

hystrix配置参数（如 timeouts）可以使用全局默认值配置，也可以使用Hystrix wiki中所述属性进行配置。

要为上面的示例路由设置5秒超时，将使用以下配置：

```yaml
hystrix.command.fallbackcmd.execution.isolation.thread.timeoutInMilliseconds: 5000
```

## 6.6. Spring Cloud CircuitBreaker GatewayFilter Factory

Spring Cloud CircuitBreaker 使用Spring Cloud CircuitBreaker API将网关路由包装在断路器中。
Spring Cloud CircuitBreaker支持可与Spring Cloud Gateway一起使用的两个库Hystrix和Resilience4J。
由于Netflix已将Hystrix置于仅维护模式，因此建议您使用Resilience4J。

要启用Spring Cloud CircuitBreaker过滤器，您需要在类路径上放置spring-cloud-starter-circuitbreaker-reactor-resilience4j或spring-cloud-starter-netflix-hystrix。

以下示例配置了一个Spring Cloud CircuitBreaker GatewayFilter：
```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: circuitbreaker_route
        uri: https://example.org
        filters:
        - CircuitBreaker=myCircuitBreaker
```
要配置断路器，请参阅所使用的基础断路器实现的配置。
- [Resilience4J Documentation](https://cloud.spring.io/spring-cloud-circuitbreaker/reference/html/spring-cloud-circuitbreaker.html)
- [Hystrix Documentation]()

Spring Cloud CircuitBreaker过滤器还可以接受可选的fallbackUri参数。
当前，仅支持转发：计划的URI。如果调用了fallback，则请求将转发到与URI匹配的控制器。
以下示例配置了这种fallback
```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: circuitbreaker_route
        uri: lb://backing-service:8088
        predicates:
        - Path=/consumingServiceEndpoint
        filters:
        - name: CircuitBreaker
          args:
            name: myCircuitBreaker
            fallbackUri: forward:/inCaseOfFailureUseThis
        - RewritePath=/consumingServiceEndpoint, /backingServiceEndpoint
```

用java代码实现

```java
@Bean
public RouteLocator routes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("circuitbreaker_route", r -> r.path("/consumingServiceEndpoint")
            .filters(f -> f.circuitBreaker(c -> c.name("myCircuitBreaker").fallbackUri("forward:/inCaseOfFailureUseThis"))
                .rewritePath("/consumingServiceEndpoint", "/backingServiceEndpoint")).uri("lb://backing-service:8088")
        .build();
}
```
当调用断路器回退时，此示例转发到/ inCaseofFailureUseThis URI。
*请注意，此示例还演示了（可选）Spring Cloud Netflix Ribbon负载平衡（由目标URI上的lb前缀定义）。*

主要方案是使用fallbackUri在网关应用程序内定义内部控制器或处理程序。
但是，您还可以将请求重新路由到外部应用程序中的控制器或处理程序，如下所示：

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: ingredients
        uri: lb://ingredients
        predicates:
        - Path=//ingredients/**
        filters:
        - name: CircuitBreaker
          args:
            name: fetchIngredients
            fallbackUri: forward:/fallback
      - id: ingredients-fallback
        uri: http://localhost:9994
        predicates:
        - Path=/fallback
```


## 6.7. The FallbackHeaders GatewayFilter Factory

FallbackHeaders允许在转发到外部应用程序中的FallbackUri的请求的header中添加Hystrix异常详细信息，如下所示：

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: ingredients
        uri: lb://ingredients
        predicates:
        - Path=//ingredients/**
        filters:
        - name: CircuitBreaker
          args:
            name: fetchIngredients
            fallbackUri: forward:/fallback
      - id: ingredients-fallback
        uri: http://localhost:9994
        predicates:
        - Path=/fallback
        filters:
        - name: FallbackHeaders
          args:
            executionExceptionTypeHeaderName: Test-Header
```

在本例中，在运行HystrixCommand发生执行异常后，请求将被转发到 localhost:9994应用程序中的 fallback终端或程序。异常类型、消息（如果可用）cause exception类型和消息的头，将由FallbackHeaders filter添加到该请求中。

通过设置下面列出的参数值及其默认值，可以在配置中覆盖headers的名称：

- executionExceptionTypeHeaderName ("Execution-Exception-Type")
- executionExceptionMessageHeaderName ("Execution-Exception-Message")
- rootCauseExceptionTypeHeaderName ("Root-Cause-Exception-Type")
- rootCauseExceptionMessageHeaderName ("Root-Cause-Exception-Message")

Hystrix 如何实现的更多细节可以参考 [Hystrix GatewayFilter Factory section](https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.2.RELEASE/reference/html/#hystrix)或者[Spring Cloud CircuitBreaker Factory section.](https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.2.RELEASE/reference/html/#spring-cloud-circuitbreaker-filter-factory).

## 6.8. The MapRequestHeader GatewayFilter Factory
MapRequestHeader过滤器有fromHeader和toHeader两个参数。它创建一个新的命名头（toHeader），然后从传入的HTTP请求中将其值从现有的命名头（fromHeader）中提取出来。
如果输入标头不存在，则过滤器不起作用。
如果新的命名报头已经存在，则其值将使用新值进行扩充。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: map_request_header_route
        uri: https://example.org
        filters:
        - MapRequestHeader=Blue, X-Request-Red
```
这会将X-Request-Red：<values>标头添加到下游请求中，这个值就是来自传入HTTP请求的Blue标头的值。

## 6.9. The PrefixPath GatewayFilter Factory

PrefixPath GatewayFilter 只有一个 prefix 参数.
```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: prefixpath_route
        uri: https://example.org
        filters:
        - PrefixPath=/mypath
```
这会将/mypath作为所有匹配请求的路径的前缀。因此，对/hello的请求将发送到/mypath/hello。

## 6.10. The PreserveHostHeader GatewayFilter Factory

该filter没有参数。设置了该Filter后，当客户端通过Gateway访问服务时，服务中获取到Host头。如果经过此过滤器，则服务端获取到的是客户端请求网关的Host；如果未经过此过滤器，则服务端获取到的是网关请求服务端的Host。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: preserve_host_route
        uri: https://example.org
        filters:
        - PreserveHostHeader
```

## 6.11. The RequestRateLimiter GatewayFilter Factory

RequestRateLimiter使用RateLimiter实现是否允许继续执行当前请求。如果不允许继续执行，则返回HTTP 429 - Too Many Requests （默认情况下）。

这个过滤器可以配置一个可选的keyResolver 参数和rate limiter参数（见下文）。

keyResolver 是 KeyResolver 接口的实现类.在配置中，按名称使用SpEL引用bean。#{@myKeyResolver} 是引用名为'myKeyResolver'的bean的SpEL表达式。以下代码显示了KeyResolver接口

```java
public interface KeyResolver {
    Mono<String> resolve(ServerWebExchange exchange);
}
```

KeyResolver接口允许使用可插拔策略来派生限制请求的key。在未来的里程碑版本中，将有一些KeyResolver实现。

KeyResolver的默认实现是PrincipalNameKeyResolver，它从ServerWebExchange检索Principal并调用Principal.getName()。

默认情况下，如果KeyResolver 没有获取到key，请求将被拒绝。此行为可以使用 spring.cloud.gateway.filter.request-rate-limiter.deny-empty-key (true or false) 和 spring.cloud.gateway.filter.request-rate-limiter.empty-key-status-code属性进行调整。

*该过滤器不能使用快捷表示法，例如*
```yaml
spring.cloud.gateway.routes[0].filters[0]=RequestRateLimiter=2, 2, #{@userkeyresolver}
```

### 6.11.1 Redis RateLimiter

redis实现基于[Stripe](https://stripe.com/blog/rate-limiters)完成的工作。 它需要使用spring-boot-starter-data-redis-active。

算法使用的是令牌桶算法[Token Bucket Algorithm](https://en.wikipedia.org/wiki/Token_bucket).

redis-rate-limiter.replenishRate是您希望允许用户每秒执行多少请求，而不会丢弃任何请求。 这是令牌桶填充的速率。

redis-rate-limiter.burstCapacity是用户在一秒钟内允许执行的最大请求数。 这是令牌桶可以容纳的令牌数。 将此值设置为零将阻止所有请求。

redis-rate-limiter.requestedTokens属性是一个请求要花费多少个令牌。这是每个请求从存储桶中获取的令牌数，默认为1。

通过在replenishRate和burstCapacity中设置相同的值来实现稳定的速率。通过将burstCapacity设置为高于replenishRate，可以允许临时突发流量。在这种情况下，需要在两次突发之间允许速率限制器留出一段时间（根据replenishRate），因为连续2次突发将导致请求被丢弃（HTTP 429 - Too Many Requests）。

通过将replenishRate设置为所需的请求数，将requestTokens设置为以秒为单位的时间跨度并将burstCapacity设置为replenishRate和requestedToken的乘积，例如1，可以达到以下1个请求的速率限制。
设置replenishRate = 1，requestedTokens = 60和burstCapacity = 60将导致限制为每分钟1个请求。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: requestratelimiter_route
        uri: https://example.org
        filters:
        - name: RequestRateLimiter
          args:
            redis-rate-limiter.replenishRate: 10
            redis-rate-limiter.burstCapacity: 20
            redis-rate-limiter.requestedTokens: 1
```

以下示例在Java中配置KeyResolver：

```java
@Bean
KeyResolver userKeyResolver() {
    return exchange -> Mono.just(exchange.getRequest().getQueryParams().getFirst("user"));
}
```

这定义了每个用户10的请求速率限制。
允许突发20，但是在下一秒中，只有10个请求可用。
这个userKeyResolver是获取user请求参数的简单方法（请注意，不建议在生产环境中使用此参数）。

您还可以将速率限制器定义为实现RateLimiter接口的Bean。在配置中，可以使用SpEL按名称引用Bean。
＃{@ myRateLimiter}是一个SpEL表达式，它引用名为myRateLimiter的bean。

以下清单定义了一个速率限制器，该限制器使用前面清单中定义的KeyResolver：

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: requestratelimiter_route
        uri: https://example.org
        filters:
        - name: RequestRateLimiter
          args:
            rate-limiter: "#{@myRateLimiter}"
            key-resolver: "#{@userKeyResolver}"
```

## 6.12. The RedirectTo GatewayFilter Factory

RedirectTo GatewayFilter Factory 包含status 参数和url参数。 状态参数必须是300系列的重定向http状态码，例如301。地址必须有效，这是Location标头的值。
对于相对重定向，您应该使用uri：no：// op作为路由定义的uri。
下面的清单配置一个RedirectTo
```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: prefixpath_route
        uri: https://example.org
        filters:
        - RedirectTo=302, https://acme.org
```
这将发送带有Location：https：//acme.org标头的状态302以执行重定向。

## 6.13. The RemoveRequestHeader GatewayFilter Factory

RemoveRequestHeader 过滤器只有一个参数 name，用来删除请求头header中的参数

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: removerequestheader_route
        uri: https://example.org
        filters:
        - RemoveRequestHeader=X-Request-Foo
```
这将删除X-Request-Foo标头，然后将其发送到下游。

## 6.14. RemoveResponseHeader GatewayFilter Factory

RemoveResponseHeader 过滤器只有一个参数 name，用来删除返回头header中的参数

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: removeresponseheader_route
        uri: https://example.org
        filters:
        - RemoveResponseHeader=X-Response-Foo
```
这将从响应中删除X-Response-Foo标头，然后将其返回到网关客户端。
要删除任何类型的敏感标头，您应该为可能要执行此操作的所有路由配置此过滤器.
另外，您可以使用s pring.cloud.gateway.default-filters 一次配置此过滤器，并将其应用于所有路由。

## 6.15. The RemoveRequestParameter GatewayFilter Factory

RemoveRequestParameter 过滤器有一个参数 name ，它用来删除的查询参数。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: removerequestparameter_route
        uri: https://example.org
        filters:
        - RemoveRequestParameter=red
```

这将删除red参数，然后再将其发送到下游。

## 6.16. The RewritePath GatewayFilter Factory

RewritePath 过滤器有两个参数 regexp 正则规则 和 replacement 参数，这使用Java正则表达式来灵活地重写请求路径.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: rewritepath_route
        uri: https://example.org
        predicates:
        - Path=/red/**
        filters:
        - RewritePath=/red/(?<segment>/?.*), /$\{segment}
```

对于/red/blue的请求路径，这会在发出下游请求之前将路径设置为/blue。请注意，由于YAML规范，应将$替换为$\。

## 6.17. RewriteLocationResponseHeader GatewayFilter Factory

RewriteLocationResponseHeader 过滤器用于修改返回头中的Location，通常用于摆脱后端特定的细节。
它有stripVersionMode, locationHeaderName, hostValue, protocolsRegex四个参数
```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: rewritelocationresponseheader_route
        uri: http://example.org
        filters:
        - RewriteLocationResponseHeader=AS_IN_REQUEST, Location, ,
``` 

对于POST api.example.com/some/object/name的请求，如果返回头中的location的值为object-service.prod.example.net/v2/some/object/id，将被重写为api.example.com/some/object/id。

stripVersionMode参数具有以下可能的值：NEVER_STRIP，AS_IN_REQUEST（默认值）和ALWAYS_STRIP，用来是否剥离出/v1或者/v2这类接口版本路径

- NEVER_STRIP：即使原始请求路径不包含任何版本，也不会剥离该版本。
- AS_IN_REQUEST 仅当原始请求路径不包含任何版本时，才剥离该版本。
- ALWAYS_STRIP 即使原始请求路径包含版本，也始终剥离版本。

hostValue参数（如果提供）用于替换响应Location标头的 host:port 部分。如果未提供，则使用主机请求标头的值。

protocolRegex参数必须是有效的正则表达式字符串，协议名称与之匹配。如果不匹配，则过滤器不执行任何操作。默认值为http | https | ftp | ftps

## 6.18. The RewriteResponseHeader GatewayFilter Factory

RewriteResponseHeader 过滤器包含 name, regexp和 replacement 参数.。通过使用Java正则表达式灵活地重写响应头的值

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: rewriteresponseheader_route
        uri: https://example.org
        filters:
        - RewriteResponseHeader=X-Response-Red, , password=[^&]+, password=***
```

对于一个/42?user=ford&password=omg!what&flag=true的header值，在做下游请求时将被设置为/42?user=ford&password=***&flag=true，由于YAML规范，请使用 $\替换 $

## 6.19. The SaveSession GatewayFilter Factory

SaveSession GatewayFilter Factory将调用转发到下游之前强制执行WebSession::save 操作。这在使用 Spring Session 之类时特别有用，需要确保会话状态在进行转发调用之前已保存。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: save_session
        uri: https://example.org
        predicates:
        - Path=/foo/**
        filters:
        - SaveSession
```

如果您将Spring Security与Spring Session集成在一起，并想确保安全性详细信息已转发到远程进程，那么这一点至关重要。

## 6.20. The SecureHeaders GatewayFilter Factory

根据此[博客文章](https://blog.appcanary.com/2017/http-security-headers.html)中的建议，SecureHeaders GatewayFilter工厂将许多headers添加到响应中。

添加了以下标头（以其默认值显示):
- X-Xss-Protection:1 (mode=block)
- Strict-Transport-Security (max-age=631138519)
- X-Frame-Options (DENY)
- X-Content-Type-Options (nosniff)
- Referrer-Policy (no-referrer)
- Content-Security-Policy (default-src 'self' https:; font-src 'self' https: data:; img-src 'self' https: data:; object-src 'none'; script-src https:; style-src 'self' https: 'unsafe-inline)'
- X-Download-Options (noopen)
- X-Permitted-Cross-Domain-Policies (none)

要更改默认值，请在spring.cloud.gateway.filter.secure-headers命名空间中设置适当的属性。
可以使用以下属性：
- xss-protection-header
- strict-transport-security
- x-frame-options
- x-content-type-options
- referrer-policy
- content-security-policy
- x-download-options
- x-permitted-cross-domain-policies

要禁用默认值，请设置spring.cloud.gateway.filter.secure-headers.disable属性，并用逗号分隔值。

以下示例显示了如何执行此操作：

```yaml
spring.cloud.gateway.filter.secure-headers.disable=x-frame-options,strict-transport-security
```

## 6.21. The SetPath GatewayFilter Factory

SetPath GatewayFilter 采用 template路径参数。它提供了一种通过允许路径的模板化segments来操作请求路径的简单方法。使用Spring Framework中的URI模板，允许多个匹配segments。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setpath_route
        uri: https://example.org
        predicates:
        - Path=/red/{segment}
        filters:
        - SetPath=/{segment}
```

对于一个 /red/blue 请求，在做下游请求前，路径将被设置为/blue

## 6.22. The SetRequestHeader GatewayFilter Factory

SetRequestHeader 有两个参数 name 和 value,用来修改请求头中参数的值

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setrequestheader_route
        uri: https://example.org
        filters:
        - SetRequestHeader=X-Request-Red, Blue
```

该GatewayFilter用给定名称替换（而不是添加）所有header。因此，如果下游服务器响应 X-Request-Red:1234，则将其替换为 X-Request-Red:Blue，这是下游服务将收到的内容。

SetRequestHeader知道用于匹配路径或主机的URI变量。URI变量可以在值中使用，并在运行时扩展。
以下示例配置使用变量的SetRequestHeader GatewayFilter：

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setrequestheader_route
        uri: https://example.org
        predicates:
        - Host: {segment}.myhost.org
        filters:
        - SetRequestHeader=foo, bar-{segment}
```

## 6.23. The SetResponseHeader GatewayFilter Factory

与SetRequestHeader大同小异，只是这是修改返回头的参数的值

## 6.24. The SetStatus GatewayFilter Factory

SetStatus GatewayFilter Factory 包括唯一的 status 参数.必须是一个可用的 Spring HttpStatus。它可以是整数值404或字符串枚举NOT_FOUND。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setstatusstring_route
        uri: https://example.org
        filters:
        - SetStatus=BAD_REQUEST
      - id: setstatusint_route
        uri: https://example.org
        filters:
        - SetStatus=401
```
无论哪种情况，响应的HTTP状态都设置为401。

您可以将SetStatus GatewayFilter配置为在响应的header中从代理请求返回原始HTTP状态代码。

如果使用以下属性配置header，则会将其添加到响应中：

```yaml
spring:
  cloud:
    gateway:
      set-status:
        original-status-header-name: original-http-status
```

## 6.25. The StripPrefix GatewayFilter Factory

StripPrefix GatewayFilter Factory 包括一个parts参数。 parts参数指示在将请求发送到下游之前，要从请求中去除的路径的层数。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: nameRoot
        uri: https://nameservice
        predicates:
        - Path=/name/**
        filters:
        - StripPrefix=2
```

通过网关对/name/blue/red的请求时，对nameservice的请求看起来像 http://nameservice/red。

## 6.26. The Retry GatewayFilter Factory

Retry GatewayFilter Factory包括 retries, statuses, methods和 series 参数.

- retries: 应尝试的重试次数
- statuses: 应该重试的HTTP状态代码，用org.springframework.http.HttpStatus标识
- methods: 应该重试的HTTP方法，用 org.springframework.http.HttpMethod标识
- series: 要重试的一系列状态码，用 org.springframework.http.HttpStatus.Series标识
- exceptions：应该重试的引发异常的列表。
- backoff：为重试配置的指数补偿。在firstBackoff *（factor ^ n）的退避间隔之后执行重试，其中n是迭代。如果配置了maxBackoff，则将应用的最大退避限制为maxBackoff。如果basedOnPreviousValue为true，则使用prevBackoff * factor计算退避量。

如果启用了以下默认值，则为“重试”筛选器配置：

- retries: Three times
- series: 5XX series
- methods: GET method
- exceptions: IOException and TimeoutException
- backoff: disabled

例子

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: retry_test
        uri: http://localhost:8080/flakey
        predicates:
        - Host=*.retry.com
        filters:
        - name: Retry
          args:
            retries: 3
            statuses: BAD_GATEWAY
            methods: GET,POST
            backoff:
              firstBackoff: 10ms
              maxBackoff: 50ms
              factor: 2
              basedOnPreviousValue: false
```

当将重试过滤器与带有forward转发URL一起使用时，应仔细编写目标端点，以便在发生错误的情况下，它不会做任何可能导致响应发送到客户端并提交的操作。
例如，如果目标端点是带注释的控制器，则目标控制器方法不应返回带有错误状态代码的ResponseEntity。
相反，它应该引发Exception或发出错误信号（例如，通过Mono.error（ex）返回值），可以将重试过滤器配置为通过重试来处理。

当将重试过滤器与任何具有主体的HTTP方法一起使用时，主体将被缓存，并且网关将受到内存的限制。
正文缓存在ServerWebExchangeUtils.CACHED_REQUEST_BODY_ATTR定义的请求属性中。
对象的类型是org.springframework.core.io.buffer.DataBuffer。

## 6.27. The RequestSize GatewayFilter Factory

当请求大小大于允许的限制时，RequestSize GatewayFilter Factory可以限制请求不到达下游服务。过滤器以RequestSize作为参数，这是定义请求的允许大小限制(以字节为单位)。

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: request_size_route
        uri: http://localhost:8080/upload
        predicates:
        - Path=/upload
        filters:
        - name: RequestSize
          args:
            maxSize: 5000000
```

当请求因大小而被拒绝时， RequestSize GatewayFilter Factory 将响应状态设置为413 Payload Too Large，并带有额外的header errorMessage 。下面是一个 errorMessage的例子。

```text
errorMessage : Request size is larger than permissible limit. Request size is 6.0 MB where permissible limit is 5.0 MB
```

如果未在路由定义中作为过滤器参数提供，则默认请求大小将设置为5 MB。

## 6.28. Modify a Request Body GatewayFilter Factory

您可以使用ModifyRequestBody筛选器筛选器来修改请求主体，然后将其由网关向下游发送。

只能使用Java DSL来配置此过滤器

```java
@Bean
public RouteLocator routes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("rewrite_request_obj", r -> r.host("*.rewriterequestobj.org")
            .filters(f -> f.prefixPath("/httpbin")
                .modifyRequestBody(String.class, Hello.class, MediaType.APPLICATION_JSON_VALUE,
                    (exchange, s) -> return Mono.just(new Hello(s.toUpperCase())))).uri(uri))
        .build();
}

static class Hello {
    String message;

    public Hello() { }

    public Hello(String message) {
        this.message = message;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}
```

## 6.29. Modify a Response Body GatewayFilter Factory

您可以使用ModifyResponseBody筛选器来修改响应正文，然后将其发送回客户端。

只能使用Java DSL来配置此过滤器

```java
@Bean
public RouteLocator routes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("rewrite_response_upper", r -> r.host("*.rewriteresponseupper.org")
            .filters(f -> f.prefixPath("/httpbin")
                .modifyResponseBody(String.class, String.class,
                    (exchange, s) -> Mono.just(s.toUpperCase()))).uri(uri)
        .build();
}
```

## 6.30. Default Filters

要添加过滤器并将其应用于所有路由，可以使用spring.cloud.gateway.default-filters。
此属性采用过滤器列表。
以下例子定义了一组默认过滤器

```yaml
spring:
  cloud:
    gateway:
      default-filters:
      - AddResponseHeader=X-Response-Default-Red, Default-Blue
      - PrefixPath=/httpbin
```

# 7. 全局过滤器

GlobalFilter接口与GatewayFilter具有相同的签名。是有条件地应用于所有路由的特殊过滤器。

## 7.1. Combined Global Filter and GatewayFilter Ordering



## 7.2. Forward Routing Filter

## 7.3. The LoadBalancerClient Filter

## 7.4. The ReactiveLoadBalancerClientFilter

## 7.5. The Netty Routing Filter

## 7.6. The Netty Write Response Filter

## 7.7. The RouteToRequestUrl Filter

## 7.8. The Websocket Routing Filter

## 7.9. The Gateway Metrics Filter

## 7.10. Marking An Exchange As Routed

# 8. 请求头过滤器



# 9. TLS and SSL



# 10. 配置



# 11. 路由原数据配置



# 12. Http超时配置



# 13. Reactor Netty 访问日志



# 14. CORS 配置



# 15. Actuator API


# 16. 故障排除



# 17. 开发者想到



# 18. 使用 Spring MVC 或 Webflux创建一个简单的路由


# 19. 配置属性表

