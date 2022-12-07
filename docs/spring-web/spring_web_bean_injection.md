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



[返回首页](https://susamlu.github.io/paitse)
[获取源码](https://github.com/susamlu/spring-web)