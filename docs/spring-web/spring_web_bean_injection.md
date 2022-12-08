# 深入学习 Spring Web 开发 —— Bean 的注入

[TOC]

上一篇文章，我们聊了与 Bean 的声明相关的内容，本文，我们重点聊聊与 Bean 的注入相关的内容。

## Bean 注入的方式

Bean 的注入，主要有三种方式，即：构造器注入、setter 注入和属性注入。

### 构造器注入

构造器注入，示例代码如下，对于下面的代码，应用启动后，InjectionComponent 的 myBean 属性会被顺利注入：

```java
@Component
public class InjectionComponent {

    private MyBean myBean;

    public InjectionComponent(MyBean myBean) {
        this.myBean = myBean;
    }

}
```

### setter 注入

setter 注入的示例代码如下（必须给 setter 方法增加 @Autowired 注解）：

```java
@Component
public class InjectionComponent2 {

    private MyBean myBean;

    @Autowired
    public void setMyBean(MyBean myBean) {
        this.myBean = myBean;
    }

}
```

### 属性注入

属性注入的示例代码如下：

```java
@Component
public class InjectionComponent3 {

    @Autowired
    private MyBean myBean;

}
```

## Bean 注入的注解

### @Autowared

从上面的代码例子，我们可以知道 @Autowared 是用来声明 Bean 的注入的。

#### @Autowared 的作用目标

@Autowared 是 JSR-330 javax.inject.Inject 的替代方案，可以作用在构造函数、方法、参数、字段和注解上。

##### 作用在构造函数上

- 如上面的例子 `构造器注入`，使用该方式进行注入时，@Autowared 不是必须的。
- 构造函数可以是公共的，也可以是私有的。
- 构造函数中的参数，必须在 Spring IoC 容器中可以找到，否则应用启动时将抛出异常，如：`Parameter 2 of constructor in org.susamlu.springweb.component.InjectionComponent required a bean of type 'org.springframework.cache.Cache' that could not be found.`。
- 如果存在多个构造函数，则必须存在一个无参构造函数，否则应用启动时将抛出异常，如：`Caused by: java.lang.NoSuchMethodException: org.susamlu.springweb.component.InjectionComponent.<init>()`。如果存在无参构造函数，则创建实例时，将使用该构造函数。
- 当存在多个构造函数时，如果在其中一个构造函数上加上 @Autowared，则上面一条规则会被打破，即此时可以不存在无参构造函数，且创建实例时，使用的不是无参构造函数，而是加上了 @Autowared 的构造函数。
- @Autowared 有一个 required 参数，默认为 true，表示构造函数中的全部参数都必须在 Spring IoC 容器中存在，否则应用启动时将抛出异常，如：`Parameter 2 of constructor in org.susamlu.springweb.component.InjectionComponent required a bean of type 'org.springframework.cache.Cache' that could not be found.`。
- 默认情况下，不能有多个构造函数存在 @Autowired 注解，否则，应用启动时将抛出异常，如：`Error creating bean with name 'injectionComponent': Invalid autowire-marked constructor: public org.susamlu.springweb.component.InjectionComponent(org.susamlu.springweb.component.MyBean,org.springframework.web.client.RestTemplate). Found constructor with 'required' Autowired annotation already: public org.susamlu.springweb.component.InjectionComponent(org.susamlu.springweb.component.MyBean)`。
- 如果多个构造函数都加上 @Autowired 注解，那么，这些注解的 required 必须全都设置为 false。这种情况下，表示这几个构造函数都是可选的，创建实例时，将选择能满足的参数最多的一个构造函数。在这基础上，会优先选择公共的构造函数。

如下面的代码，由于不存在 Cache 类的 Bean，而 MyBean 类和 RestTemplate 类的 Bean 是存在的，因此，应用启动时将使用构造函数 `InjectionComponent(MyBean myBean, RestTemplate restTemplate)` 来初始化 InjectionComponent 的 Bean。

```java
@Component
public class InjectionComponent {

    private MyBean myBean;
    private RestTemplate restTemplate;
    private Cache cache;

    @Autowired(required = false)
    public InjectionComponent(MyBean myBean) {
        this.myBean = myBean;
    }

    @Autowired(required = false)
    public InjectionComponent(MyBean myBean, RestTemplate restTemplate) {
        this.myBean = myBean;
        this.restTemplate = restTemplate;
    }

    @Autowired(required = false)
    public InjectionComponent(MyBean myBean, RestTemplate restTemplate, Cache cache) {
        this.myBean = myBean;
        this.restTemplate = restTemplate;
        this.cache = cache;
    }

}
```

[返回首页](https://susamlu.github.io/paitse)
[获取源码](https://github.com/susamlu/spring-web)