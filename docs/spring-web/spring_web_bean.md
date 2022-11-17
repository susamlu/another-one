# 深入学习 Spring Web 开发 —— Bean 的声明与注入

[TOC]

IoC 是 Spring 框架的最重要特性之一，而对于 Spring IoC，我们能够最直观感受到的可能就是 Bean 的声明与注入，本文，我们将围绕该主题进行讲解。

## Bean 的声明

有多种方式可以声明 Spring Bean。

### @Component

通常可以在某个类上加上 @Component 注解，以标识自动将该类注册到 Spring IoC 容器中：

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

这几个注解有一个相同的属性：value，它是用来定义 Bean 的名字的。默认情况下，value 值为空字符串，此时，Bean 以类的全限定类名作为名字，如果 value 值不为空，则 Bean 以 value 值作为名字。

### @Bean

@Bean 一般与 @Configuration 一起使用，：

### @Import

### @ImportResource

### 手动注册

> BeanDefinitionRegistry

### 一切引入了 @Component 的注解

## Bean 的注入

[返回首页](https://susamlu.github.io/paitse)
[获取源码](https://github.com/susamlu/spring-web)