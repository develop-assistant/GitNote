# SpringCloud-Gateway过滤器



## 自定义过滤器

```java
/**
 * 此过滤器功能为计算请求完成时间
 */
public class ElapsedFilter implements GatewayFilter, Ordered {

    private static final String ELAPSED_TIME_BEGIN = "elapsedTimeBegin";

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        exchange.getAttributes().put(ELAPSED_TIME_BEGIN, System.currentTimeMillis());
        return chain.filter(exchange).then(
                Mono.fromRunnable(() -> {
                    Long startTime = exchange.getAttribute(ELAPSED_TIME_BEGIN);
                    if (startTime != null) {
                        System.out.println(exchange.getRequest().getURI().getRawPath() + ": " + (System.currentTimeMillis() - startTime) + "ms");
                    }
                })
        );
    }

    /*
     *过滤器存在优先级，order越大，优先级越低
     */
    @Override
    public int getOrder() {
        return Ordered.LOWEST_PRECEDENCE;
    }
}
```



```java
@Configuration
public class FilterConfig {


    /**
     * http://localhost:8100/filter/provider
     * @param builder
     * @return
     */
    @Bean
    public RouteLocator customerRouteLocator(RouteLocatorBuilder builder) {
        // @formatter:off
        return builder.routes()
                .route(r -> r.path("/filter/**")
                        .filters(f -> f.stripPrefix(1)
                                .filter(new ElapsedFilter()))
                        .uri("lb://idc-cloud-provider")
                        .order(0)
                        .id("filter")
                )
                .build();
        // @formatter:on
    }

}
```



```yaml
server:
  port: 8100
spring:
  application:
    name: idc-cloud-gateway
  redis:
    host: localhost
    port: 6379
    timeout: 6000ms  # 连接超时时长（毫秒）
    jedis:
      pool:
        max-active: 1000  # 连接池最大连接数（使用负值表示没有限制）
        max-wait: -1ms      # 连接池最大阻塞等待时间（使用负值表示没有限制）
        max-idle: 10      # 连接池中的最大空闲连接
        min-idle: 5       # 连接池中的最小空闲连接
  cloud:
    consul:
      host: localhost
      port: 8500
    gateway:
      discovery:
        locator:
          enabled: true # gateway可以通过开启以下配置来打开根据服务的serviceId来匹配路由,默认是大写
      routes:
        - id: provider  # 路由 ID，保持唯一
          uri: lb://idc-cloud-provider # uri指目标服务地址，lb代表从注册中心获取服务
          predicates: # 路由条件。Predicate 接受一个输入参数，返回一个布尔值结果。该接口包含多种默认方法来将 Predicate 组合成其他复杂的逻辑（比如：与，或，非）
            - Path=/p/**
          filters:
            - StripPrefix=1 # 过滤器StripPrefix，作用是去掉请求路径的最前面n个部分截取掉。StripPrefix=1就代表截取路径的个数为1，比如前端过来请求/test/good/1/view，匹配成功后，路由到后端的请求路径就会变成http://localhost:8888/good/1/view

```



测试(网关port=8100，idc-cloud-provider服务port=2001)

```
➜  ~ curl http://localhost:8100/filter/provider
Services: [consul, idc-cloud-gateway, idc-cloud-provider]

输出
/provider: 1ms
```

