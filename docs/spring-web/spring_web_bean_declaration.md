# 深入学习 Spring Web 开发 —— Bean 的声明

[TOC]

IoC 是 Spring 框架的最重要特性之一，对于 Spring IoC，我们能够最直观感受到的可能就是 Bean 的声明与注入，本文，我们先讲讲与 Bean 的声明相关的内容。

有多种方式可以声明 Spring Bean。

## @Component

通常可以在某个类上加上 @Component 注解，以标识自动将该类的实例注册到 Spring IoC 容器中：

```java
@Component
public class MyComponent {
}
```

@Controller、@Service、@Repository 是类似的，它们之间概念上的区别大于实际上的区别，通常不同作用的类使用不同的注解。

### @Controller

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

### @Service

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

### @Repository

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

## @Bean

### 与 @Configuration 一起使用

@Bean 一般与 @Configuration 一起使用，@Bean 比 @Component 更为灵活（虽然如此，但有时候 @Component 使用起来会更方便），使用它既可以注册自己编写的类，也可以注册第三方类：

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;
import org.susamlu.springweb.component.MyRestTemplate;

@Configuration
public class BeanConfig {

    @Bean
    public MyRestTemplate myRestTemplate() {
        return new MyRestTemplate();
    }

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

}
```

另外，通过 @Bean 也可以灵活配置我们将要创建的 Bean：

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

### 与 @Component 一起使用

@Bean 也可以与 @Component 一起使用，不过与上述方式不同的是，当直接调用该方法获取实例时，上述方式每次获取到的对象是相同的一个单例对象，而这种方式每次获取到的对象都是新建的对象：

```java
@Component
public class BeanConfig3 {

    @Bean
    public RestTemplate restTemplate2() {
        return new RestTemplate();
    }

}
```

```java
@Component
public class GetBeanTest {

    @Autowired
    private BeanConfig beanConfig;
    @Autowired
    private BeanConfig3 beanConfig3;

    @EventListener
    private void getBeans(ApplicationReadyEvent event) {
        RestTemplate restTemplate1 = beanConfig.restTemplate();
        RestTemplate restTemplate2 = beanConfig.restTemplate();
        RestTemplate restTemplate3 = beanConfig3.restTemplate2();
        RestTemplate restTemplate4 = beanConfig3.restTemplate2();
        System.out.println("restTemplate1 equals restTemplate2: " + restTemplate1.equals(restTemplate2));
        System.out.println("restTemplate3 equals restTemplate4: " + restTemplate3.equals(restTemplate4));
    }

}
```

上述代码输出的结果将是：

```html
restTemplate1 equals restTemplate2: true
restTemplate3 equals restTemplate4: false
```

### @Bean 的属性

#### name

@Bean 也存在 value 属性，同时还存在一个 name 属性，这两个属性的作用是一样的，都是用来定义 Bean 的名字的。value 和 name 属性是一个数组，也就是说一个 Bean 可以有多个名字，例如：

```java
@Configuration
public class BeanConfig4 {

    @Bean(name = {"restTemplateA", "restTemplateB"})
    public MyRestTemplate myRestTemplate() {
        return new MyRestTemplate();
    }

}
```

通过 @Bean 定义的 Bean 同样也遵循 `项目中不能存在两个名字一样的 Bean` 的规则，但在细节上，与通过 @Component 定义 Bean 有一些区别。

例如，在同一个类中，可以有多个 @Bean name 属性相同的方法，但只会有一个生效，比如下面的代码中，只有第一个方法定义的 Bean 被注册到了 Spring IoC 容器中，第二个方法定义的 Bean 被忽略了。

```java
@Configuration
public class BeanConfig5 {

    @Bean("restTemplateC")
    public MyRestTemplate myRestTemplate() {
        return new MyRestTemplate();
    }

    @Bean("restTemplateC")
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

}
```

但，如果多个 @Bean name 属性相同的方法定义在不同的类中，那么在应用启动的时候就会抛出异常。如有下面两个类：

```java
@Configuration
public class BeanConfig6 {

    @Bean("restTemplateD")
    public MyRestTemplate myRestTemplate() {
        return new MyRestTemplate();
    }

}
```

```java
@Configuration
public class BeanConfig7 {

    @Bean("restTemplateD")
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

}
```

应用启动时将抛出如下的异常信息：

```html
The bean 'restTemplateD', defined in class path resource [org/susamlu/springweb/config/BeanConfig7.class], could not be registered. A bean with that name has already been defined in class path resource [org/susamlu/springweb/config/BeanConfig6.class] and overriding is disabled.
```

#### autowire

autowire 属性是用来定义实例变量的注入的，在 Spring 5.1 版本已置为弃用（本文使用的 Spring 版本为 5.3.22），所以在此不打算详细讲解。至于为何弃用，代码注释的说法是：1. 可以通过方法参数的解析进行实例变量的注入，2. 可以通过 @Autowired 进行实例变量的注入，因此，不建议再使用该属性。

对于第三方类，我们可以通过定义方法参数的方式，处理变量的注入问题，如：

```java
@Configuration
public class BeanConfig8 {

    @Bean
    public RestTemplate customRestTemplate2(ClientHttpRequestFactory requestFactory) {
        RestTemplate restTemplate = new RestTemplate();
        restTemplate.setRequestFactory(requestFactory);
        return restTemplate;
    }

    @Bean
    public ClientHttpRequestFactory requestFactory() {
        return new BufferingClientHttpRequestFactory(new SimpleClientHttpRequestFactory());
    }

}
```

对于我们自己编写的类，则直接在需要注入的变量处加上 @Autowired 注解即可：

```java
public class MyRestTemplate2 {

    @Autowired
    private RestTemplate restTemplate;

}
```

```java
@Configuration
public class BeanConfig9 {

    @Bean
    public MyRestTemplate2 myRestTemplate2() {
        return new MyRestTemplate2();
    }

}
```

如此，在 @Bean 方法中返回的对象，即便是通过 new 的方式创建的，变量也能正常注入。

#### autowireCandidate

autowireCandidate 的值默认为 true，表示通过 @Bean 方法注册的 Bean 是可以被注入到其他实例变量中的，如果将其改为 false，则表示不可被注入。如下面的代码：

```java
@Configuration
public class BeanConfig8 {

    @Bean
    public RestTemplate customRestTemplate2(ClientHttpRequestFactory requestFactory) {
        RestTemplate restTemplate = new RestTemplate();
        restTemplate.setRequestFactory(requestFactory);
        return restTemplate;
    }

    @Bean(autowireCandidate = false)
    public ClientHttpRequestFactory requestFactory() {
        return new BufferingClientHttpRequestFactory(new SimpleClientHttpRequestFactory());
    }

}
```

由于通过 requestFactory() 方法注册的 Bean 被禁止了被注入，因此，应用启动的时候会抛出下面的异常信息：

```html
Parameter 0 of method customRestTemplate2 in org.susamlu.springweb.config.BeanConfig8 required a bean of type 'org.springframework.http.client.ClientHttpRequestFactory' that could not be found.
```

#### initMethod

initMethod 可以指定在 Bean 初始化完成前需要执行的方法，该方法必须定义在 Bean 的类中，且该方法必须是无参的。下面的代码展示了 initMethod 的使用方式：

```java
public class MyRestTemplate3 {

    private void init() {
        System.out.println("call MyRestTemplate3#init()");
    }

}
```

```java
@Configuration
public class BeanConfig10 {

    @Bean(initMethod = "init")
    public MyRestTemplate3 myRestTemplate3() {
        return new MyRestTemplate3();
    }

}
```

#### destroyMethod

destroyMethod 可以指定在 Bean 销毁前需要执行的方法，该方法必须定义在 Bean 的类中，且该方法必须是公共且无参的。

默认情况下，destroyMethod 的值为 (inferred)，表示会自动推断 Bean 销毁前需要执行的方法，即如果 Bean 中存在公共无参方法 close() 或 shutdown() 时，Bean 销毁前将会自动执行它。如果 Bean 中同时存在 close() 和 shutdown() 方法，则只会执行 close() 方法。

```java
public class MyRestTemplate4 {

    public void close() {
        System.out.println("call MyRestTemplate4#close()");
    }

}
```

```java
public class MyRestTemplate5 {

    public void shutdown() {
        System.out.println("call MyRestTemplate5#shutdown()");
    }

}
```

```java
@Configuration
public class BeanConfig11 {

    @Bean
    public MyRestTemplate4 myRestTemplate4() {
        return new MyRestTemplate4();
    }

    @Bean
    public MyRestTemplate5 myRestTemplate5() {
        return new MyRestTemplate5();
    }

}
```

也可以显式指定该方法：

```java
public class MyRestTemplate6 {

    public void destroy() {
        System.out.println("call MyRestTemplate6#destroy()");
    }

}
```

```java
@Configuration
public class BeanConfig11 {

    @Bean(destroyMethod = "destroy")
    public MyRestTemplate6 myRestTemplate6() {
        return new MyRestTemplate6();
    }

}
```

如果需要取消应用的默认推断行为，需要将 destroyMethod 的值设置为空字符串：

```java
@Configuration
public class BeanConfig11 {

    @Bean(destroyMethod = "")
    public MyRestTemplate7 myRestTemplate7() {
        return new MyRestTemplate7();
    }

}
```

这种方式不会影响 DisposableBean 的 destroy() 方法的执行。

## @Import

@Import 可以引入配置类、ImportSelector 和 ImportBeanDefinitionRegistrar 的实现类，也可以引入常规的其他组件类。通过它既可以引入自己编写的类，也可以引入第三方类。通过 @Import 引入的类的实例会被自动注册到 Spring IoC 容器中。

@Import 有一个数组类型的 value 属性，我们可以向这个属性传递一个或多个 class：

```java
@Configuration
@Import({JdbcProperties.class, MyRestTemplate7.class})
public class BeanConfig12 {
}
```

@Import 可以重复多次引入同一个类（当然这是没有必要的），这个类对应的实例最终只会被注册一次。

> ImportSelector 和 ImportBeanDefinitionRegistrar 是用来做什么的？它们都是接口，ImportSelector 有一个 selectImports() 方法，ImportBeanDefinitionRegistrar 有一个 registerBeanDefinitions() 方法，这两个方法一个是用来筛选类，一个是用来注册类的，我们可以通过实现这两个接口，从而自定义类的筛选和注册逻辑。

## @ImportResource

@ImportResource 与 @Import 类似，都是为了引入 Spring Bean。@Import 可以直接引入待引入的类，而 @ImportResource 能够引入的是外部文件，如 xml 文件或 groovy 文件。@ImportResource 有三个属性，其中两个是 value 和 locations，这两个属性的作用是一样的，都是用来指定引入资源的位置的。另外一个属性是 reader，如果我们希望 @ImportResource 能解析 xml 和 groovy 以外的文件，那么只要自定义 BeanDefinitionReader，并将自定义的类传值给 reader 属性即可，具体如何自定义，在此不做详细讲解。下面是一个 @ImportResource 的使用例子：

```java
public class MyBean {

    private String field;

    public void setField(String field) {
        this.field = field;
    }

}
```

```xml
<!-- bean.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <bean class="org.susamlu.springweb.component.MyBean">
        <property name="field" value="sample-value"></property>
    </bean>
</beans>
```

```java
@Configuration
@ImportResource("classpath:bean.xml")
public class BeanConfig13 {
}
```

## 手动注册

手动注册 BeanDefinition，一般可以使用 DefaultListableBeanFactory 的 registerBeanDefinition() 方法进行注册，注册的时候通常可以借助 GenericBeanDefinition 或 BeanDefinitionBuilder 来完成。

### GenericBeanDefinition

借助 GenericBeanDefinition 完成 BeanDefinition 的注册，示例代码如下：

```java
public class MyBean2 {

    private String field;

    public void setField(String field) {
        this.field = field;
    }

    public void doSomething() {
        System.out.println("from my bean, field: " + field);
    }

}
```

```java
public class GenericBeanDefinitionExample {

    public static void main(String[] args) {
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
        GenericBeanDefinition beanDefinition = new GenericBeanDefinition();
        beanDefinition.setBeanClass(MyBean2.class);
        beanDefinition.getPropertyValues().addPropertyValue("field", "sample-value");
        beanFactory.registerBeanDefinition("myBean", beanDefinition);

        MyBean2 bean = beanFactory.getBean(MyBean2.class);
        bean.doSomething();
    }

}
```

### BeanDefinitionBuilder

借助 BeanDefinitionBuilder 完成 BeanDefinition 的注册，示例代码如下：

```java
public class BeanDefinitionBuilderExample {

    public static void main(String[] args) {
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
        BeanDefinitionBuilder builder = BeanDefinitionBuilder.rootBeanDefinition(MyBean2.class)
                .addPropertyValue("field", "sample-value");
        beanFactory.registerBeanDefinition("myBean", builder.getBeanDefinition());

        MyBean2 bean = beanFactory.getBean(MyBean2.class);
        bean.doSomething();
    }

}
```

为了在 Spring Boot 应用启动的过程中注册 BeanDefinition，可以通过实现 BeanFactoryPostProcessor 或 BeanDefinitionRegistryPostProcessor 接口来完成。

### BeanFactoryPostProcessor

通过实现 BeanFactoryPostProcessor 接口注册 BeanDefinition，示例代码如下：

```java
@Configuration
public class BeanConfig14 implements BeanFactoryPostProcessor {

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        BeanDefinitionBuilder builder = BeanDefinitionBuilder.rootBeanDefinition(MyBean2.class)
                .addPropertyValue("field", "sample-value");
        ((DefaultListableBeanFactory) beanFactory)
                .registerBeanDefinition("myBean2", builder.getBeanDefinition());
    }

}
```

### BeanDefinitionRegistryPostProcessor

通过实现 BeanDefinitionRegistryPostProcessor 接口注册 BeanDefinition，示例代码如下：

```java
@Configuration
public class BeanConfig15 implements BeanDefinitionRegistryPostProcessor {

    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
        BeanDefinitionBuilder builder = BeanDefinitionBuilder.rootBeanDefinition(MyBean2.class)
                .addPropertyValue("field", "sample-value");
        registry.registerBeanDefinition("myBean3", builder.getBeanDefinition());
    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        // do nothing
    }

}
```

## 一切引入了 @Component 的注解

除了上面的方式可以注册 Spring Bean，还有其他方式吗？

在前面的文章中，我们分析过 Spring Boot 应用在启动的时候，会将含有 @Component 注解的全部类注册到 Spring IoC 容器中。而 Spring 框架的特性是，只要某个注解 `注解B` 引入了另一个注解 `注解A`，那么某个类在使用了这个注解 `注解B` 的时候其实也引入了另一个注解 `注解A`，因此，只要一个类使用了 `一切引入了 @Component 的注解`，它就会被注册到 Spring IoC 容器中。

而这些注解就包括了：@ControllerAdvice、@RestControllerAdvice、@JsonComponent 等。

## 小结

本文系统介绍了声明 Spring Bean 的各种方式，相信读者在了解完这些内容后，对于在什么场景下使用什么方式以及如何进行 Bean 的声明，就再也不用范迷糊了。

[返回首页](https://susamlu.github.io/paitse)
[获取源码](https://github.com/susamlu/spring-web)