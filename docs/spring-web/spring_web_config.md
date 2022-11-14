# 深入学习 Spring Web 开发 —— 配置类

[TOC]

配置类的功能主要有两个：一是配置环境变量，二是配置 bean。

## @Configuration

配置类一般需要附带 @Configuration 注解。@Configuration 注解有两个属性，一个是 value，用来指定 Spring bean 的名称，一个是 proxyBeanMethods，用来指定是否需要为配置类的实例生成代理，从而使在应用中直接调用 @Bean 方法时也只返回单例对象，它的值默认为 true。

如下面两个类，默认情况下，restTemplate1 和 restTemplate2 是同一个对象，如果将 @Configuration 注解的 proxyBeanMethods 属性改为 false （@Configuration(proxyBeanMethods = false)），那么，restTemplate1 和 restTemplate2 就是不同的对象。

```java
@Configuration
public class FirstConfig {

    @Bean
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }

}
```

```java
@Slf4j
@Component
public class CallBeanMethod {

    @Autowired
    private FirstConfig firstConfig;

    @EventListener
    public void onStartup(ApplicationReadyEvent event) {
        RestTemplate restTemplate1 = firstConfig.getRestTemplate();
        RestTemplate restTemplate2 = firstConfig.getRestTemplate();
        log.info("Equals: {}", restTemplate1.equals(restTemplate2));
    }

}
```

## @PropertySource

