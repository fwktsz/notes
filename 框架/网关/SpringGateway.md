 # Spring Cloud Gateway

## 1. 术语


Route: The basic building block of the gateway. It is defined by an ID, a destination URI, a collection of predicates, and a collection of filters. A route is matched if the aggregate predicate is true. 
- 路由，一组路由规则的基础组成模块。包含一个自定义的id，一个uri，一组断言和拦截器的集合。如果 这组断言匹配，则执行这个路由


- Predicate: This is a Java 8 Function Predicate. The input type is a Spring Framework ServerWebExchange. This lets you match on anything from the HTTP request, such as headers or parameters.

- 断言：匹配  http请求的一些路径或者参数


- Filter: These are instances of GatewayFilter that have been constructed with a specific factory. Here, you can modify requests and responses before or after sending the downstream request.

- 拦截器： 就是常用的filter。

 代码片段如下：

 ```
 spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - Cookie=mycookie,mycookievalue
 ```



## 2. 工作原理

![原理图](./img/spring_cloud_gateway_diagram.png)

通过断言进行http 请求的 路径已经参数的匹配，并对对应的请求进行拦截与转发。
客户端向Spring Cloud Gateway发出请求。如果网关处理程序映射确定一个请求匹配一个路由，它将被发送到网关Web处理程序。此处理程序通过特定于请求的过滤器链运行请求。虚线分隔过滤器的原因是，过滤器可以在发送代理请求之前和之后运行逻辑。执行所有“预”筛选逻辑。然后发出代理请求。发出代理请求后，运行“post”筛选器逻辑。

## 3. 如何使用：

### 3.1 yml配置：

- 简化配置：
```
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - Cookie=mycookie,mycookievalue
```
- 完全展开参数：

```
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

## Route Predicate Factories

### 路由断言种类与配置

Spring Cloud Gateway 的路由匹配，是基于Spring WebFlux HandlerMapping的。Spring Cloud Gateway包括许多内置的路由断言工厂。所有这些断言都匹配HTTP请求的不同属性。您可以将多个路由断言工厂与逻辑和语句组合在一起。

1. The After Route Predicate Factory

 After路由断言工厂接受一个参数，一个datetime(它是一个java ZonedDateTime)。此断言匹配发生在指定日期时间之后的请求。配置after路由的示例如下:

 ```
 spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - After=2017-01-20T17:42:47.789-07:00[America/Denver]
 ```

2. The Before Route Predicate Factory

3. The Between Route Predicate Factory

4. The Cookie Route Predicate Factory

5. The Header Route Predicate Factory

6. The Host Route Predicate Factory

7. The Host Route Predicate Factory

8. The Path Route Predicate Factory

    路径路由断言工厂接受两个参数:一个Spring PathMatcher模式列表和一个名为matchOptionalTrailingSeparator的可选标志。下面的例子配置了一个路径路由断言:

```
spring:
  cloud:
    gateway:
      routes:
      - id: path_route
        uri: https://example.org
        predicates:
        - Path=/red/{segment},/blue/{segment}
```
如果请求路径是，这个路由匹配，例如:/red/1或/red/blue或/blue/green。

这个断言提取URI模板变量(例如在前面的例子中定义的segment)作为名称和值的映射，并将其与在ServerWebExchangeUtils.URI_TEMPLATE_VARIABLES_ATTRIBUTE中定义的键放在ServerWebExchange.getAttributes()中。这些值可以被GatewayFilter工厂使用

可以使用一个实用方法(称为get)来简化对这些变量的访问。下面的例子展示了如何使用get方法:

```
    Map<String, String> uriVariables = ServerWebExchangeUtils.getPathPredicateVariables(exchange);

    String segment = uriVariables.get("segment");
```

9. The Query Route Predicate Factory

Query路由断言工厂接受两个参数:一个必需的参数和一个可选的regexp(它是一个Java正则表达式)。下面的例子配置了一个查询路由断言:

```
spring:
  cloud:
    gateway:
      routes:
      - id: query_route
        uri: https://example.org
        predicates:
        - Query=green
```
如果请求包含 green 查询参数，则上述路由匹配。

```
spring:
  cloud:
    gateway:
      routes:
      - id: query_route
        uri: https://example.org
        predicates:
        - Query=red, gree.
```
如果请求中包含一个红色的查询参数，且该参数的值匹配gree，则上述路由匹配。Regexp，所以绿色和greet是匹配的。

10. The RemoteAddr Route Predicate Factory

RemoteAddr路由断言工厂接受一个源列表(最小大小为1)，它是cidr表示法(IPv4或IPv6)字符串，例如192.168.0.1/16(其中192.168.0.1是一个IP地址，16是一个子网掩码)。配置RemoteAddr路由断言的示例如下:
```
spring:
  cloud:
    gateway:
      routes:
      - id: remoteaddr_route
        uri: https://example.org
        predicates:
        - RemoteAddr=192.168.1.1/24
```

11. The Weight Route Predicate Factory

Weight路由断言工厂接受两个参数:group和Weight(一个int类型)。权重按每组计算。配置权重路由断言的示例如下:

```
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

这条路线将把80%的流量转发到weighthigh.org, 20%的流量转发到weighlow.org 

这个断言可以做灰度发布。



## GatewayFilter Factories

路由过滤器允许以某种方式修改传入的HTTP请求或传出的HTTP响应。路由过滤器的作用域是特定的路由。Spring Cloud Gateway包括许多内置的GatewayFilter factory。


1. The AddRequestHeader GatewayFilter Factory

AddRequestHeader GatewayFilter工厂接受一个名称和值参数。下面的例子配置了一个AddRequestHeader GatewayFilter:

```
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_header_route
        uri: https://example.org
        filters:
        - AddRequestHeader=X-Request-red, blue
```

这个清单为所有匹配的请求将X-Request-red:blue标头添加到下游请求的标头。

AddRequestHeader能够识别用于匹配路径或主机的URI变量。URI变量可以在值中使用，并在运行时展开。下面的例子配置了一个使用变量的AddRequestHeader GatewayFilter:

```
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

2. The AddRequestParameter GatewayFilter Factory

3. The AddResponseHeader GatewayFilter Factory

4. The DedupeResponseHeader GatewayFilter Factory

5. The Hystrix GatewayFilter Factory

  **Netflix让Hystrix进入了维护模式。我们建议您使用springcloud CircuitBreaker Gateway Filter和Resilience4J一起使用，因为对Hystrix的支持将在未来的版本中被移除**

  Hystrix是Netflix的一个库，它实现了断路器模式。Hystrix GatewayFilter允许您在网关路由中引入断路器，保护您的服务免于级联故障，并允许您在下游故障时提供后备响应。
要在项目中启用Hystrix GatewayFilter实例，请从Spring Cloud Netflix中添加Spring - Cloud -start - Netflix - Hystrix依赖项。
Hystrix GatewayFilter工厂需要一个名称参数，它是HystrixCommand的名称。以配置Hystrix网关为例 ：

```
spring:
  cloud:
    gateway:
      routes:
      - id: hystrix_route
        uri: https://example.org
        filters:
        - Hystrix=myCommandName
```

这将使用命令名myCommandName在HystrixCommand中包装其余的过滤器。

Hystrix过滤器还可以接受一个可选的fallbackUri参数。目前，只支持forward: schemed uri。

如果调用了回调，则请求被转发到由URI匹配的控制器。下面的示例配置了这样的回调:

```
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


当调用Hystrix回调时，这将转发到/incaseoffailureusethis URI。注意，这个例子还演示了(可选的)Spring Cloud Netflix Ribbon负载均衡(在目标URI上定义了lb前缀)。

主要的场景是在网关应用程序中使用fallbackUri到内部控制器或处理程序。然而，你也可以在外部应用程序中将请求重新路由到控制器或处理程序，如下所示:

```
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


6. Spring Cloud CircuitBreaker GatewayFilter Factory

7. The FallbackHeaders GatewayFilter Factory

8. The MapRequestHeader GatewayFilter Factory

9. The PrefixPath GatewayFilter Factory

10. The PreserveHostHeader GatewayFilter Factory

11. The RequestRateLimiter GatewayFilter Factory

  限流的功能：

    RequestRateLimiter GatewayFilter工厂使用一个RateLimiter实现来确定当前请求是否被允许继续。如果不是，则返回HTTP 429 - Too Many Requests(默认情况下)的状态。

    这个过滤器接受一个可选的keyResolver参数和特定于速率限制器的参数(在本节后面描述)。

    keyResolver是一个实现keyResolver接口的bean。在配置中，使用SpEL通过名称引用bean。#{@myKeyResolver}是一个SpEL表达式，引用了一个名为myKeyResolver的bean。下面的清单显示了KeyResolver接口

```
public interface KeyResolver {
    Mono<String> resolve(ServerWebExchange exchange);
}
```

KeyResolver接口允许可插入策略派生出用于限制请求的key。在未来的里程碑版本中，将会有一些KeyResolver实现。

KeyResolver的默认实现是PrincipalNameKeyResolver，它从ServerWebExchange检索Principal并调用Principal. getname()。默认情况下，如果KeyResolver没有找到密钥，请求将被拒绝。

您可以通过设置如下属性
spring.cloud.gateway.filter.request-rate-limit .deny-empty-key (true或false)和
spring.cloud.gateway.filter.request-rate-limiter.empty-key-status-code 。

11.1 The Redis RateLimiter

  Redis的实现是基于在Stripe完成的工作。它需要引入 Spring-Boot-starter-data-redis-reactive 依赖。

  算法：令牌桶算法

- redis-rate-limiter.replenishRate  允许用户每秒执行多少请求

- redis-rate-limiter.burstCapacity  在一秒钟内允许做的最大请求数

- redis-rate-limiter.requestedTokens 请求所花费的令牌数,默认1

一个稳定的速率是通过设置相同的值在补ishrate和burst容量。可以通过设置burstCapacity高于recharge rate来允许临时突发。在这种情况下，速率限制需要被允许在突发之间有一段时间(根据补充速率)，因为两个连续的突发将导致请求丢失(HTTP 429 -太多的请求)。

下面的清单配置了redis-rate-limiter:波纹管1请求/ s速率限制是通过设置replenishRate希望的请求数量,requestedTokens秒的时间间隔和burstCapacity replenishRate和requestedTokens的产物,如设置replenishRate = 1, requestedTokens = 60和burstCapacity = 60将导致限制1请求/分钟。

```
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

下面的例子在Java中配置KeyResolver:

```
@Bean
KeyResolver userKeyResolver() {
    return exchange -> Mono.just(exchange.getRequest().getQueryParams().getFirst("user"));
}
```

这定义了每个用户10个请求速率限制。允许爆发20次请求，但是在下一秒，只有10次请求可用。KeyResolver是一个简单的工具，它可以获取用户请求参数(注意，不建议在生产环境中使用此方法)。

您还可以将速率限制器定义为实现RateLimiter接口的bean。在配置中，可以使用SpEL通过名称引用bean。#{@myRateLimiter}是一个SpEL表达式，它引用了一个名为myRateLimiter的bean。下面的清单定义了一个速率限制器，它使用了上一个清单中定义的KeyResolver

```
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

12. The RedirectTo GatewayFilter Factory

13. The RemoveRequestHeader GatewayFilter Factory

14. RemoveResponseHeader GatewayFilter Factory

15. The RemoveRequestParameter GatewayFilter Factory

16. The RewritePath GatewayFilter Factory

17. RewriteLocationResponseHeader GatewayFilter Factory

18. The RewriteResponseHeader GatewayFilter Factory

19. The SaveSession GatewayFilter Factory



#  Global Filters
