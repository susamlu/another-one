# 深入学习 Spring Web 开发 —— Bean 的附加注解

[TOC]

前面的文章，我们介绍了 Bean 的声明与注入，关于 Bean 还有一些辅助性注解是非常重要的，本文，我们重点聊聊这些辅助性注解。

## @Scope

@Scope 用于指定 Bean 的作用域。

比如，通过 @Scope 可以标注 Bean 的作用域为 singleton（这也是 Spring Bean 的默认作用域），如下面代码得到的 Bean 就是单例的：

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

也可以通过 @Scope 标注 Bean 的作用域为 prototype，下面代码得到的 Bean 是多例的（每次访问都会获取一个新的对象）：

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
    private ScopeBeanConfig scopeBeanConfig;

    @EventListener
    private void getBeans(ApplicationReadyEvent event) {
        MyBean singletonBean1 = scopeBeanConfig.singletonBean();
        MyBean singletonBean2 = scopeBeanConfig.singletonBean();
        MyBean prototypeBean1 = scopeBeanConfig.prototypeBean();
        MyBean prototypeBean2 = scopeBeanConfig.prototypeBean();
        System.out.println("singletonBean1 与 singletonBean2 是同一个对象: " + ((singletonBean1.equals(singletonBean2) ? "是" : "不是")));
        System.out.println("prototypeBean1 与 prototypeBean2 是同一个对象: " + (prototypeBean1.equals(prototypeBean2) ? "是" : "不是"));
    }

}
```

上面代码演示了多次请求获取同一个 Bean 时，得到的对象是否是同一个，它的输出结果为：

```html
singletonBean1 与 singletonBean2 是同一个对象: 是
prototypeBean1 与 prototypeBean2 是同一个对象: 不是
```

@Scope 除了能定义这两种作用域，对于 Web 应用，还能定义以下三种作用域：request、session、application，表示对象的生命周期是在请求内、会话内，还是在应用内。下面是具体的代码例子：

```java
public class ScopeBeanConfig {

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

需要注意的是，这三种作用域的 Bean，定义时 proxyMode 是必须要指定的，否则应用启动时将抛出异常。另外，上面几个作用域的标注，也可以用下面几个注解替代：

```java
public class ScopeBeanConfig {

    @Bean
    @RequestScope
    public MyBean requestBean2() {
        return new MyBean();
    }

    @Bean
    @SessionScope
    public MyBean sessionBean2() {
        return new MyBean();
    }

    @Bean
    @ApplicationScope
    public MyBean applicationBean2() {
        return new MyBean();
    }

}
```

## @Lazy

@Lazy 用于指定 Bean 的延迟加载，即用 @Lazy 标注过的 Bean 不会在应用启动的时候就初始化，会延迟到需要用到的时候才初始化。示例代码如下：

```java
@Component
@Lazy
public class MyLazyBean {

    public MyLazyBean() {
        System.out.println("init MyLazyBean");
    }

}
```

需要注意的是，如果代码的某处注入了该 Bean，那么只要注入该 Bean 的类在应用启动时被初始化了，该 Bean 也会在此时被初始化（自动注入时会创建该 Bean 的对象），如：

```java
@Autowired
private MyLazyBean myLazyBean;
```

如果要保证该 Bean 确实可以延迟加载，可以使用下面的方法获取：

```java
@Autowired
private ApplicationContext applicationContext;

...

applicationContext.getBean(MyLazyBean.class);
```

## @DependsOn

@DependsOn 用来指定 Bean 之间的依赖关系，如下面的例子：

```java
@Component
@DependsOn({"myDependsOnBeanB"})
public class MyDependsOnBeanA {

    public MyDependsOnBeanA() {
        System.out.println("init MyDependsOnBeanA");
    }

}
```

```java
@Component
public class MyDependsOnBeanB {

    public MyDependsOnBeanB() {
        System.out.println("init MyDependsOnBeanB");
    }

}
```

myDependsOnBeanA 依赖于 myDependsOnBeanB，因此它会在 myDependsOnBeanB 加载完成之后才被加载。

## @Primary

@Primary 用于指定依赖注入的首选 Bean，即指定有多个类型相同的 Bean 时，优先注入的是哪个。关于 @Primary 在上一篇文章中已经介绍，本文不再赘述。

## 总结

本文主要讲解了 Spring Bean 的几个附加注解的使用方法，它们都是我们平时开发中比较常用的几个注解，希望这些知识对读者会有所帮助。

[返回首页](https://susamlu.github.io/paitse)
[获取源码](https://github.com/susamlu/spring-web)