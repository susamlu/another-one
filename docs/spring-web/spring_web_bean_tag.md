# 深入学习 Spring Web 开发 —— Bean 的附加注解

[TOC]

前面的文章，我们介绍了 Bean 的声明与注入，关于 Bean 还有一些辅助性注解是非常重要的，本文，我们重点聊聊这些辅助性注解。

## @Scope

@Scope 用于指定 Bean 的作用域。

比如，我们可以使用 ConfigurableBeanFactory.SCOPE_SINGLETON，标注 Bean 的作用域为 Singleton，即该 Bean 是单例的：

```java
@Configuration
public class BeanConfig {

    @Bean
    @Scope(ConfigurableBeanFactory.SCOPE_SINGLETON)
    public MyBean singletonBean() {
        return new MyBean();
    }

}
```

也可以通过 ConfigurableBeanFactory.SCOPE_PROTOTYPE，标注 Bean 的作用域为 Prototype，即该 Bean 是多例的（每次访问都会获取一个新的对象）：

```java
@Configuration
public class BeanConfig {

    @Bean
    @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
    public MyBean prototypeBean() {
        return new MyBean();
    }

}
```

```java
@Component
public class GetBeanTest {

    @Autowired
    private BeanConfig beanConfig;

    @EventListener
    private void getBeans(ApplicationReadyEvent event) {
        MyBean singletonBean1 = beanConfig.singletonBean();
        MyBean singletonBean2 = beanConfig.singletonBean();
        MyBean prototypeBean1 = beanConfig.prototypeBean();
        MyBean prototypeBean2 = beanConfig.prototypeBean();
        System.out.println("singletonBean1 equals singletonBean2: " + singletonBean1.equals(singletonBean2));
        System.out.println("prototypeBean1 equals prototypeBean2: " + prototypeBean1.equals(prototypeBean2));
    }

}
```

上面代码的输出结果为：

```html
singletonBean1 equals singletonBean2: true
prototypeBean1 equals prototypeBean2: false
```

@Scope 除了能定义这两种作用域，对于 Web 应用，还能定义以下三种作用域：Request、Session、Application，表示对象的生命周期是在请求、会话，还是在应用内。下面是具体的代码例子：

```java
public class BeanConfig {

    @Bean
    @Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
    public MyBean requestBean() {
        return new MyBean();
    }

    @Bean
    @Scope(value = WebApplicationContext.SCOPE_SESSION, proxyMode = ScopedProxyMode.TARGET_CLASS)
    public MyBean sessionBean() {
        return new MyBean();
    }

    @Bean
    @Scope(value = WebApplicationContext.SCOPE_APPLICATION, proxyMode = ScopedProxyMode.TARGET_CLASS)
    public MyBean applicationBean() {
        return new MyBean();
    }

}
```

需要注意的是，这三种作用域的 Bean，定义时 proxyMode 是必须要指定的，否则应用启动时将抛出异常。

## @Lazy

## @DependsOn

## @Primary

[返回首页](https://susamlu.github.io/paitse)
[获取源码](https://github.com/susamlu/spring-web)