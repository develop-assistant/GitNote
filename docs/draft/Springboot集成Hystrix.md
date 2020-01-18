# Springboot集成Hystrix

**pom.xml**

```xml
        <dependency>
            <groupId>com.netflix.hystrix</groupId>
            <artifactId>hystrix-core</artifactId>
            <version>1.5.18</version>
        </dependency>
        <dependency>
            <groupId>com.netflix.hystrix</groupId>
            <artifactId>hystrix-metrics-event-stream</artifactId>
            <version>1.5.18</version>
        </dependency>
        <dependency>
            <groupId>com.netflix.hystrix</groupId>
            <artifactId>hystrix-javanica</artifactId>
            <version>1.5.18</version>
        </dependency>
```

**HystrixConfig.java**

```java
@Configuration
public class HystrixConfig {

    /**
     * 用来拦截处理HystrixCommand注解
     * @return
     */
    @Bean
    public HystrixCommandAspect hystrixCommandAspect() {
        return new HystrixCommandAspect();
    }

    /**
     * 用来向监控中心Dashboard发送stream信息
     * @return
     */
    @Bean
    public ServletRegistrationBean hystrixMetricsStreamServlet() {
        ServletRegistrationBean registration = new ServletRegistrationBean(new HystrixMetricsStreamServlet());
        registration.addUrlMappings("/hystrix.stream");
        return registration;
    }
}
```

**OrderService1.java**

```java
@Slf4j
@Service
public class OrderService1 {

    /**
     * 超时降级策略
     * @param order
     * @return
     */
    @HystrixCommand(commandKey = "/orders/create",
            commandProperties = {
                    @HystrixProperty(name="execution.timeout.enabled",value="true"),
                    @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value="3000")
            },
            fallbackMethod = "createOrderFallbackMethodTimeout")
    public String createOrder(@RequestBody String order){

        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
//            e.printStackTrace();
        }
        return "success";
    }

    /**
     * 超时降级策略 createOrder超时降级
     * @param order
     * @return
     */
    public String createOrderFallbackMethodTimeout(String order){
        log.info("--------超时降级策略执行--------");
        return "Hystrix time out";
    }
}
```

**OrderService2.java**

```java
@Slf4j
@Service
public class OrderService2 {

    /**
     * 限流策略：信号量方式
     * @param order
     * @return
     */
    @HystrixCommand(commandKey = "/orders/insert",
            commandProperties = {
                    @HystrixProperty(name="execution.isolation.strategy",value="SEMAPHORE"),
                    @HystrixProperty(name="execution.isolation.semaphore.maxConcurrentRequests",value="3"),
            },
            fallbackMethod = "insertOrderFallbackMethodSemaphore")
    public String insertOrder(@RequestBody String order){

        return "success";
    }

    /**
     * 限流策略：信号量方式 insertOrder限流降级
     * @param order
     * @return
     */
    public String insertOrderFallbackMethodSemaphore( String order){
        log.info("--------信号量限流降级策略执行--------");
        return "Hystrix semaphore";
    }
}

```

**OrderService3.java**

```java
@Slf4j
@Service
public class OrderService3 {

    /**
     * 限流策略：线程池方式
     * @param order
     * queueSizeRejectionThreshold 排队线程数量阈值，默认为5，达到时拒绝
     * @return
     */
    @HystrixCommand(commandKey = "/orders/add",
            commandProperties = {
                    @HystrixProperty(name="execution.isolation.strategy",value="THREAD")
            },
            threadPoolKey = "addOrderThreadPool",
            threadPoolProperties = {
                    @HystrixProperty(name="coreSize",value="3"),
                    @HystrixProperty(name="maxQueueSize",value="5"),
                    @HystrixProperty(name="queueSizeRejectionThreshold",value="5")
            },
            fallbackMethod = "addOrderFallbackMethodThread")
    public String addOrder(@RequestBody String order){

        return "success";
    }

    /**
     *  限流策略：线程池方式 addOrder限流降级
     * @param order
     * @return
     */
    public String addOrderFallbackMethodThread(String order){
        log.info("--------线程池限流降级策略执行--------");
        return "Hystrix threadPool";
    }
}
```

**测试**

```java
@Slf4j
@SpringBootTest
class AntsCacheApplicationTests {

    @Autowired
    OrderService1 orderService1;
    @Autowired
    OrderService2 orderService2;
    @Autowired
    OrderService3 orderService3;

    @Test
    void contextLoads() throws InterruptedException {

        // 测试Hystrix超时降级策略
        orderService1.createOrder("orderService1");

        // 测试Hystrix信号量降级策略
        CountDownLatch countDownLatch = new CountDownLatch(5);
        ExecutorService exec = Executors.newCachedThreadPool();

        for (int i=0; i<5; i++) {
            exec.execute(() -> {
                try {
                    orderService2.insertOrder("orderService2");
                } finally {
                    countDownLatch.countDown();
                }
            });
        }
        countDownLatch.await();
        exec.shutdown();

        // 测试Hystrix线程池降级策略
        CountDownLatch countDownLatch1 = new CountDownLatch(10);
        ExecutorService exec1 = Executors.newCachedThreadPool();

        for (int i=0; i<10; i++) {
            exec1.execute(() -> {
                try {
                    orderService3.addOrder("orderService3");
                } finally {
                    countDownLatch1.countDown();
                }
            });
        }
        countDownLatch1.await();
        exec1.shutdown();
    }

}
```

**结果**

```
2020-01-18 19:27:54.422  INFO 67897 --- [ HystrixTimer-1] com.idcmind.ants.hystrix.OrderService1   : --------超时降级策略执行--------
2020-01-18 19:27:54.436  INFO 67897 --- [pool-3-thread-1] com.idcmind.ants.hystrix.OrderService2   : --------信号量限流降级策略执行--------
2020-01-18 19:27:54.436  INFO 67897 --- [pool-3-thread-4] com.idcmind.ants.hystrix.OrderService2   : --------信号量限流降级策略执行--------
2020-01-18 19:27:54.454  INFO 67897 --- [pool-4-thread-8] com.idcmind.ants.hystrix.OrderService3   : --------线程池限流降级策略执行--------
2020-01-18 19:27:54.454  INFO 67897 --- [pool-4-thread-1] com.idcmind.ants.hystrix.OrderService3   : --------线程池限流降级策略执行--------

```

