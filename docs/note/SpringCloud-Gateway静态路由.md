# SpringCloud-Gateway静态路由

## 1. 为什么引入API网关

使用 API 网关后的优点如下：

- 易于监控。可以在网关收集监控数据并将其推送到外部系统进行分析。
- 易于认证。可以在网关上进行认证，然后再将请求转发到后端的微服务，而无须在每个微服务中进行认证。
- 减少了客户端与各个微服务之间的交互次数。



## 2. 搭建环境

首先搭建一个微服务基本测试环境，服务发现组件选择consul。

### .1. 服务提供者

pom.xml

```xml
		<dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    </dependencies>

		<dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.1.4.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Greenwich.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

pplication.yml

```yaml
spring:
  application:
    name: idc-cloud-provider
  cloud:
    consul:
      host: localhost
      port: 8500
server:
  port: 2001

```

ProviderApplication.java

```java
@EnableDiscoveryClient
@SpringBootApplication
public class ProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(ProviderApplication.class, args);
    }
}
```

ProviderController.java

```java
@RestController
public class ProviderController {

    @Autowired
    DiscoveryClient discoveryClient;

    @GetMapping("/provider")
    public String provider() {
        String services = "Services: " + discoveryClient.getServices();
        System.out.println(services);
        return services;
    }
}
```

访问: http://localhost:2001/provider 

结果如下:

```
Services: [consul, idc-cloud-consumer, idc-cloud-gateway, idc-cloud-provider]
```



### 2.2. 服务消费者

Pom.xml增加

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```

application.yml

```yaml
spring:
  application:
    name: idc-cloud-consumer
  cloud:
    consul:
      host: localhost
      port: 8500
server:
  port: 2101

```

ConsumerApplication.java

```java
@EnableFeignClients
@EnableDiscoveryClient
@SpringBootApplication
public class ConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }
}
```

RemoteService.java

```java
@FeignClient("idc-cloud-provider")
public interface RemoteService {

    /**
     * 方法名随意，url路径匹配即可
     * @return
     */
    @GetMapping("/provider")
    String test();
}
```

ConsumerController.java

```java
@RestController
public class ConsumerController {

    @Autowired
    RemoteService remoteService;

    @GetMapping("/consumer")
    public String consumer() {
        String result = remoteService.test();
        return result;
    }
}

```

访问: http://localhost:2101/consumer 

输出如下:

```
Services: [consul, idc-cloud-consumer, idc-cloud-gateway, idc-cloud-provider]
```



## . 静态路由

Pom.xml

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webflux</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
            <scope>compile</scope>
        </dependency>
    </dependencies>
```

Application.yml

```yaml
spring:
  application:
    name: idc-cloud-gateway
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
        - id: consumer
          uri: lb://idc-cloud-consumer
          predicates:
            - Path=/c/**
          filters:
            - StripPrefix=1
server:
  port: 8100
```

IdcGatewayApplication.java

```java
@SpringBootApplication
@EnableDiscoveryClient
public class IdcGatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(IdcGatewayApplication.class, args);
    }
}
```

访问: http://localhost:8100/p/provider

输出结果

```
Services: [consul, idc-cloud-consumer, idc-cloud-gateway, idc-cloud-provider]
```

访问: http://localhost:8100/c/consumer

输出结果

```
Services: [consul, idc-cloud-consumer, idc-cloud-gateway, idc-cloud-provider]
```

可见静态路由配置已生效。