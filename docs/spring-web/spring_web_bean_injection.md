# 深入学习 Spring Web 开发 —— Bean 的注入

[TOC]

上一篇文章，我们聊了与 Bean 的声明相关的内容，本文，我们重点聊聊与 Bean 的注入相关的内容。

## Bean 的注入方式

Bean 的注入方式，主要包含三种，即：构造函数注入、setter 注入和变量注入。

### 构造函数注入

构造函数注入，即通过构造函数定义需要注入的 Bean，通过这种方式进行注入，@Autowired 注解不是必须的。下面是一个示例代码，对于下面的代码，应用启动后，myBean 变量会被顺利注入：

```java
@Component
public class MyBean {
}
```

```java
@Component
public class InjectionComponent {

    private MyBean myBean;

    public InjectionComponent(MyBean myBean) {
        this.myBean = myBean;
    }

}
```

> @Autowired 注解通常用在 Spring 应用的 Bean 注入的声明中，下文我们会详细讲解。

### setter 注入

setter 注入，即通过 setter 方法定义需要注入的 Bean，setter 注入的示例代码如下，通过这种方式进行注入，@Autowired 注解是必须的：

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

### 变量注入

变量注入，即通过变量定义需要注入的 Bean，变量注入的示例代码如下，通过这种方式进行注入，@Autowired 注解也是必须的：

```java
@Component
public class InjectionComponent3 {

    @Autowired
    private MyBean myBean;

}
```

## Bean 注入的声明注解

### @Autowired

从上面的代码例子，我们可以知道 @Autowired 是用来声明 Bean 的注入的。

#### @Autowired 的作用目标

@Autowired 是 JSR-330 javax.inject.Inject 的替代方案，可以作用在构造函数、方法、参数、字段和注解上。

##### 作用在构造函数上

- 如上面的例子 `构造函数注入`，使用该方式进行注入时，@Autowired 不是必须的。
- 构造函数可以是公共的，也可以是私有的。
- 构造函数中的参数，必须在 Spring IoC 容器中可以找到，否则应用启动时将抛出异常。
- 如果存在多个构造函数，则必须存在一个无参构造函数，否则应用启动时将抛出异常。如果存在无参构造函数，则创建实例时，将使用该构造函数进行创建。
- 当存在多个构造函数时，如果在其中一个构造函数上加上 @Autowired，则上面一条规则会被打破，即此时可以不存在无参构造函数，且创建实例时，使用的不是无参构造函数，而是加上了 @Autowired 的构造函数。
- @Autowired 有一个 required 属性，默认为 true，表示构造函数中的全部参数都必须在 Spring IoC 容器中存在，否则应用启动时将抛出异常。
- 默认情况下，不能有多个构造函数存在 @Autowired 注解，否则，应用启动时将抛出异常。
- 如果多个构造函数都加上 @Autowired 注解，那么，这些注解的 required 属性必须全都设置为 false。这种情况下，表示这几个构造函数都是候选构造函数，创建实例时，将选择能够满足的参数最多的一个构造函数来进行对象的创建。在这基础上，会优先选择公共的构造函数。例如：如果存在一个包含 2 个参数和一个包含 3 个参数的构造函数，且这两个构造函数都能够被满足，但是包含 2 个参数的构造函数是公共的，包含 3 个参数的构造函数是私有的，那么，此时选择的将是包含 2 个参数的构造函数。
- 如果上面一条规则中的所有候选构造函数，没有一个可以被满足，并且存在无参构造函数，则创建实例时会选择无参构造函数。如果此时不存在无参构造函数，则应用启动时将抛出异常。

如下面的代码，由于不存在 Cache 类的实例，而 MyBean 类和 RestTemplate 类的实例是存在的，因此，应用启动时将使用构造函数 `InjectionComponent4(MyBean, RestTemplate)` 来初始化 InjectionComponent4 的实例。

```java
@Component
public class InjectionComponent4 {

    private MyBean myBean;
    private RestTemplate restTemplate;
    private Cache cache;

    @Autowired(required = false)
    public InjectionComponent4(MyBean myBean) {
        this.myBean = myBean;
    }

    @Autowired(required = false)
    public InjectionComponent4(MyBean myBean, RestTemplate restTemplate) {
        this.myBean = myBean;
        this.restTemplate = restTemplate;
    }

    @Autowired(required = false)
    public InjectionComponent4(MyBean myBean, RestTemplate restTemplate, Cache cache) {
        this.myBean = myBean;
        this.restTemplate = restTemplate;
        this.cache = cache;
    }

}
```

##### 作用在方法上

当 @Autowired 作用在某个方法时，该方法可以有任意的名称和任意数量的参数，setter 方法只是其中的一种特殊形式，该方法也可以是私有的：

```java
@Component
public class InjectionComponent5 {

    private MyBean myBean;

    @Autowired
    private void myBean(MyBean myBean) {
        this.myBean = myBean;
    }

}
```

##### 作用在参数上

虽然 Spring 框架 5.0 之后支持在构造函数和方法的参数上加上 @Autowired 注解，但是 Spring 框架中的大部分代码都不支持该方式的注入，目前只有 Spring Test 部分支持。如下面的代码，在实际运行的时候，myBean 是注入不了的：

```java
@Component
public class InjectionComponent6 {

    private MyBean myBean;

    public void init(@Autowired MyBean myBean) {
        this.myBean = myBean;
    }

}
```

如果是在测试代码中，被 @Autowired 标注的参数却是可以成功注入的。如下面的测试代码，myBean 是可以被注入的：

```java
@SpringBootTest
public class InjectionTest {

    @Test
    public void test(@Autowired MyBean myBean) {
        assert myBean != null;
    }

}
```

##### 作用在字段上

示例见前文的 `变量注入`。

#### Collection 和 Map 的注入

使用 @Autowire 还可以注入 Collection 和 Map 类型的变量。

##### Collection 的注入

如下面的代码，OrderBean 和 OrderBean2 的实例都会被注入到 InjectionComponent7 的 orderBeans 中，可以通过实现 Ordered 接口为类的实例提供排序功能，order 小的排在前面。

```java
@Component
public class OrderBean extends BaseOrderBean implements Ordered {

    @Override
    public int getOrder() {
        return 1;
    }

}
```

```java
@Component
public class OrderBean2 extends BaseOrderBean implements Ordered {

    @Override
    public int getOrder() {
        return 2;
    }

}
```

```java
@Component
public class InjectionComponent7 {

    @Autowired
    private List<BaseOrderBean> orderBeans;

}
```

上面的代码中，OrderBean 的 order 为 1，OrderBean2 的 order 为 2，因此，它们的实例被注入到 orderBeans 时，OrderBean 的实例将排在 OrderBean2 的实例前面。

也可以通过 @Order 实现排序：

```java
public class OrderBean3 {

    private String field;

    public void setField(String field) {
        this.field = field;
    }

}
```

```java
@Configuration
public class BeanConfig {

    @Bean
    @Order(1)
    public OrderBean3 orderBean3() {
        OrderBean3 orderBean = new OrderBean3();
        orderBean.setField("order 1");
        return orderBean;
    }

    @Bean
    @Order(2)
    public OrderBean3 orderBean4() {
        OrderBean3 orderBean = new OrderBean3();
        orderBean.setField("order 2");
        return orderBean;
    }

}
```

```java
@Component
public class InjectionComponent8 {

    @Autowired
    private List<OrderBean3> orderBeans;

}
```

上面的代码中，实例 orderBean3 和 orderBean4 被注入到 orderBeans 时，orderBean3 将排在 orderBean4 前面。（orderBean3() 方法创建的实例为 orderBean3，orderBean4() 方法创建的实例为 orderBean4。）

##### Map 的注入

下面的代码中，全部 BaseOrderBean 的实例都会注入到 orderBeanMap 中，Bean 的名字将作为 Map 的键，Bean 的实例将作为 Map 的值。

```java
@Component
public class InjectionComponent9 {

    @Autowired
    private Map<String, BaseOrderBean> orderBeanMap;

}
```

结合前面的代码，orderBeanMap 中会存在两个实例，一个是 OrderBean 的实例 orderBean，一个是 OrderBean2 的实例 orderBean2。

#### @Primary

@Autowired 根据 Bean 的类型找到它，并将它注入到变量中，如果一个类型有两个 Bean，那么注入的时候将不知道要注入的是哪一个。如果用 @Primary 对 Bean 进行了标注，那么注入的时候就会选择该 Bean 进行注入。

```java
public class MyBean3 {

    private String field;

    public MyBean3(String field) {
        this.field = field;
    }

}
```

```java
@Configuration
public class BeanConfig2 {

    @Primary
    @Bean
    public MyBean3 primaryTestBean1() {
        return new MyBean3("primaryTestBean1");
    }

    @Bean
    public MyBean3 primaryTestBean2() {
        return new MyBean3("primaryTestBean2");
    }

}
```

```java
@Component
public class InjectionComponent10 {

    @Autowired
    private MyBean3 myBean;

}
```

上面的代码中，方法 primaryTestBean1() 上标注有 @Primary 注解，因此，InjectionComponent10 的 myBean 变量注入时，选择的 Bean 将是通过 primaryTestBean1() 方法创建的 Bean。

#### @Qualifier

一个类型有多个 Bean 的时候，也可以通过 @Qualifier 进行标注，以让变量注入的时候根据 Bean 的名字找到 Bean。

```java
@Configuration
public class BeanConfig3 {

    @Bean
    public MyBean3 qualifierTestBean1() {
        return new MyBean3("qualifierTestBean1");
    }

    @Bean
    public MyBean3 qualifierTestBean2() {
        return new MyBean3("qualifierTestBean2");
    }

}
```

```java
@Component
public class InjectionComponent11 {

    @Qualifier("qualifierTestBean1")
    @Autowired
    private MyBean3 myBean;

}
```

上面的代码中，@Qualifier 的 value 值指定为 qualifierTestBean1，因此，InjectionComponent11 的 myBean 变量注入时，选择的 Bean 将是 qualifierTestBean1。

@Qualifier 还有另一个用法，就是在声明 Bean 和注入 Bean 的地方都使用该注解进行标注，那么注入的时候就会自动定位到对应的 Bean。

```java
@Configuration
public class BeanConfig4 {

    @Qualifier
    @Bean
    public MyBean3 qualifierTestBean3() {
        return new MyBean3("qualifierTestBean3");
    }

    @Bean
    public MyBean3 qualifierTestBean4() {
        return new MyBean3("qualifierTestBean4");
    }

}
```

```java
@Component
public class InjectionComponent12 {

    @Qualifier
    @Autowired
    private MyBean3 myBean;

}
```

上面的代码中，方法 qualifierTestBean3() 上标注有 @Qualifier 注解，InjectionComponent12 的 myBean 变量上也标注有 @Qualifier，因此，InjectionComponent12 的 myBean 变量注入时，选择的 Bean 将是通过 qualifierTestBean3() 方法创建的 Bean。

### @Resource

@Resource 也是用来声明 Bean 注入的注解。

- 对于 @Resource，默认情况下，优先使用名字查找待注入的 Bean（当然，根据名字查找的 Bean，类型也是要匹配上的）；如果找不到，则使用类型查找待注入的 Bean。
- @Resource 可以标注在方法和字段上。
- 标注在方法上时，Bean 类型为方法参数的类型，Bean 名字优先使用方法的名字（如果是 setter 方法，会去掉 “set”，并将首字母小写），如果通过方法名字找不到 Bean，则再使用参数的名字。（@Resource 标注的方法只能存在一个参数。）
- 标注在字段上时，Bean 类型为字段类型，Bean 名字为字段名字。

代码示例如下：

```java
public class ResourceBean {

    private String field;

    public ResourceBean(String field) {
        this.field = field;
    }

}
```

```java
@Configuration
public class BeanConfig5 {

    @Bean
    public ResourceBean resourceBean1() {
        return new ResourceBean("resourceBean1");
    }

    @Bean
    public ResourceBean resourceBean2() {
        return new ResourceBean("resourceBean2");
    }

}
```

```java
@Component
public class InjectionComponent13 {

    @Resource
    private ResourceBean resourceBean1;

    private ResourceBean resourceBean2;

    @Resource
    public void setResourceBean2(ResourceBean resourceBean) {
        this.resourceBean2 = resourceBean;
    }

}
```

上面的代码中，变量 resourceBean1 的类型为 ResourceBean，名字为 resourceBean1，@Resource 注解将根据变量的名字找到对应的 Bean 进行注入。方法 setResourceBean2() 待注入的 Bean 的类型为 ResourceBean，名字为 resourceBean2（setResourceBean2 去掉 “set”，并将首字母小写），@Resource 注解将根据这个名字找到对应的 Bean 进行注入。

@Resource 有一个 name 属性，可以直接指定 Bean 的名字：

```java
@Component
public class InjectionComponent14 {

    @Resource(name = "resourceBean1")
    private ResourceBean myBean1;

    private ResourceBean myBean2;

    @Resource(name = "resourceBean2")
    public void setMyBean2(ResourceBean resourceBean) {
        this.myBean2 = resourceBean;
    }

}
```

上面的代码中，变量 myBean1 和 方法 setMyBean2() 都通过 @Resource 的 name 属性指定了待注入 Bean 的名字，@Resource 将根据该名字找到对应的 Bean 进行注入。

另外，@Resource 也可以配合 @Primary 或 @Qualifier 一起使用。

### @Inject

要使用 @Inject，需要先引入 jar 包：

```xml
<dependency>
    <groupId>javax.inject</groupId>
    <artifactId>javax.inject</artifactId>
    <version>1</version>
</dependency>
```

@Inject 可以标注在构造函数、方法和字段上。@Inject 可以通过类型进行注入：

```java
@Component
public class InjectionComponent15 {

    @Inject
    private MyBean myBean1;

    private MyBean myBean2;

    @Inject
    public void setMyBean(MyBean myBean) {
        this.myBean2 = myBean;
    }

    private MyBean myBean3;

    @Inject
    public InjectionComponent15(MyBean myBean) {
        this.myBean3 = myBean;
    }

}
```

上面的代码中，变量 myBean1、方法 setMyBean() 和 构造函数 InjectionComponent15()，均通过类型来进行 Bean 的注入。

@Inject 也可以通过名字进行注入：

```java
@Component
public class InjectionComponent16 {

    @Inject
    private ResourceBean resourceBean1;

    private ResourceBean resourceBean2;

    @Inject
    public void setResourceBean(ResourceBean resourceBean2) {
        this.resourceBean2 = resourceBean2;
    }

    private ResourceBean resourceBean3;

    @Inject
    public InjectionComponent16(ResourceBean resourceBean2) {
        this.resourceBean3 = resourceBean2;
    }

}
```

上面的代码中，变量 resourceBean1、方法 setResourceBean() 和 构造函数 InjectionComponent16()，均通过名字来进行 Bean 的注入。

同样地，@Inject 也可以配合 @Primary 或 @Qualifier 一起使用。

## 小结

@Autowired、@Resource、@Inject 有什么异同点？显然，它们都可以用来声明 Bean 的注入。它们的不同点主要集中在：1. 可以标注的地方有所差异，2. 拥有的属性不同，3. 注入的方式略有差异。而它们最大的不同，笔者认为是，@Autowired 是 Spring 自定义的注解（@Primary、@Qualifier 也是 Spring 自定义的注解），@Resource 是 Jakarta 定义的注解，而 @Inject 则是在 JSR-330 中定义的注解，即它们的作用非常类似，最大的区别在于制定者的不同。

[返回首页](https://susamlu.github.io/paitse)
[获取源码](https://github.com/susamlu/spring-web)