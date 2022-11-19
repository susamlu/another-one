# 深入学习 Spring Web 开发 —— Bean 的声明与注入

[TOC]

IoC 是 Spring 框架的最重要特性之一，而对于 Spring IoC，我们能够最直观感受到的可能就是 Bean 的声明与注入，本文，我们将围绕该主题进行讲解。

## Bean 的声明

有多种方式可以声明 Spring Bean。

### @Component

通常可以在某个类上加上 @Component 注解，以标识自动将该类的实例注册到 Spring IoC 容器中：

```java
@Component
public class MyComponent {
}
```

@Controller、@Service、@Repository 是类似的，它们之间概念上的区别大于实际上的区别，通常不同作用的类使用不同的注解。

如果这个类是一个 controller 类，那么一般使用 @Controller 注解（或者 @RestController 注解）：

```java
@Controller
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping("/api/users/{userId}")
    public User findUser(@PathVariable(name = "userId") Long userId) {
        return userService.findUser(userId);
    }

}
```

如果这个类是一个 service 类，那么一般使用 @Service 注解：

```java
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    public User findUser(Long userId) {
        return userRepository.findById(userId);
    }

}
```

如果这个类是一个 dao 类，那么一般使用 @Repository 注解：

```java
@Repository
public class UserRepository {

    public User findById(Long userId) {
        return new User(userId, "小穆");
    }

}
```

这几个注解有一个相同的属性：value，它是用来定义 Bean 的名字的。默认情况下，value 值为空字符串，此时，Bean 以首字母小写的类名作为名字，如果 value 值不为空字符串，则 Bean 以 value 值作为名字。如下面两个写法，其中 Bean 的名字分别为：`myComponent` 和 `oneComponent`。

```java
@Component
public class MyComponent {
}
```

```java
@Component(value = "oneComponent")
public class MyComponent2 {
}
```

项目中是不能存在两个名字一样的 Bean 的，假设有下面两个类：

```java
@Component
public class MyComponent {
}
```

```java
@Component(value = "myComponent")
public class MyComponent2 {
}
```

那么，应用启动的时候将会抛出异常信息：

```html
Annotation-specified bean name 'myComponent' for bean class [org.susamlu.springweb.component.MyComponent2] conflicts with existing, non-compatible bean definition of same name and class [org.susamlu.springweb.component.MyComponent]
```

### @Bean

#### 与 @Configuration 一起使用

@Bean 一般与 @Configuration 一起使用，@Bean 比 @Component 更为灵活（虽然如此，但有时候 @Component 使用起来会更方便），使用它既可以注册当前项目类的实例，也可以注册第三方类的实例：

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;
import org.susamlu.springweb.component.MyRestTemplate;

@Configuration
public class BeanConfig {

    @Bean
    public MyRestTemplate getMyRestTemplate() {
        return new MyRestTemplate();
    }

    @Bean
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }

}
```

@Bean 的灵活之处也在于，这种方式可以灵活配置我们将要创建的 Bean：

```java
@Configuration
public class BeanConfig2 {

    @Bean
    public RestTemplate customRestTemplate() {
        RestTemplate restTemplate = new RestTemplate();
        restTemplate.setRequestFactory(new BufferingClientHttpRequestFactory(restTemplate.getRequestFactory()));
        return restTemplate;
    }

}
```

#### 与 @Component 一起使用

#### @Bean 的属性

##### name

@Bean 也存在 value 属性，同时还存在一个 name 属性，这两个属性的作用是一样的，都是用来定义 Bean 的名字的。value 和 name 属性是一个数组，也就是说一个 Bean 可以有多个名字，例如：

```java
@Configuration
public class BeanConfig3 {

    @Bean(name = {"restTemplateA", "restTemplateB"})
    public MyRestTemplate myRestTemplate() {
        return new MyRestTemplate();
    }

}
```

### @Import

### @ImportResource

### 手动注册

> BeanDefinitionRegistry

### 一切引入了 @Component 的注解

## Bean 的注入

[返回首页](https://susamlu.github.io/paitse)
[获取源码](https://github.com/susamlu/spring-web)