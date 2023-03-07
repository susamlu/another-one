# 深入学习 Spring Web 开发 —— BeanDefinition（上）

[TOC]

本文主要探讨什么是 BeanDefinition，下一篇文章，我们将探讨 BeanDefinition 是如何起作用的。

## BeanDefinition 是什么

下面，我们先看一下 BeanDefinition 的源码：

```java
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {

    String SCOPE_SINGLETON = ConfigurableBeanFactory.SCOPE_SINGLETON;
    String SCOPE_PROTOTYPE = ConfigurableBeanFactory.SCOPE_PROTOTYPE;

    int ROLE_APPLICATION = 0;
    int ROLE_SUPPORT = 1;
    int ROLE_INFRASTRUCTURE = 2;

    void setParentName(@Nullable String parentName);

    @Nullable
    String getParentName();

    void setBeanClassName(@Nullable String beanClassName);

    @Nullable
    String getBeanClassName();

    void setScope(@Nullable String scope);

    @Nullable
    String getScope();

    void setLazyInit(boolean lazyInit);

    boolean isLazyInit();

    void setDependsOn(@Nullable String... dependsOn);

    @Nullable
    String[] getDependsOn();

    void setAutowireCandidate(boolean autowireCandidate);

    boolean isAutowireCandidate();

    void setPrimary(boolean primary);

    boolean isPrimary();

    void setFactoryBeanName(@Nullable String factoryBeanName);

    @Nullable
    String getFactoryBeanName();

    void setFactoryMethodName(@Nullable String factoryMethodName);

    @Nullable
    String getFactoryMethodName();

    ConstructorArgumentValues getConstructorArgumentValues();

    default boolean hasConstructorArgumentValues() {
        return !getConstructorArgumentValues().isEmpty();
    }

    MutablePropertyValues getPropertyValues();

    default boolean hasPropertyValues() {
        return !getPropertyValues().isEmpty();
    }

    void setInitMethodName(@Nullable String initMethodName);

    @Nullable
    String getInitMethodName();

    void setDestroyMethodName(@Nullable String destroyMethodName);

    @Nullable
    String getDestroyMethodName();

    void setRole(int role);

    int getRole();

    void setDescription(@Nullable String description);

    @Nullable
    String getDescription();

    ResolvableType getResolvableType();

    boolean isSingleton();

    boolean isPrototype();

    boolean isAbstract();

    @Nullable
    String getResourceDescription();

    @Nullable
    BeanDefinition getOriginatingBeanDefinition();

}
```

可以看到 BeanDefinition 是一个接口，它继承了 AttributeAccessor 和 BeanMetadataElement，其中 AttributeAccessor 的源码如下：

```java
public interface AttributeAccessor {

    void setAttribute(String name, @Nullable Object value);

    @Nullable
    Object getAttribute(String name);

    @SuppressWarnings("unchecked")
    default <T> T computeAttribute(String name, Function<String, T> computeFunction) {
        Assert.notNull(name, "Name must not be null");
        Assert.notNull(computeFunction, "Compute function must not be null");
        Object value = getAttribute(name);
        if (value == null) {
            value = computeFunction.apply(name);
            Assert.state(value != null,
                    () -> String.format("Compute function must not return null for attribute named '%s'", name));
            setAttribute(name, value);
        }
        return (T) value;
    }

    @Nullable
    Object removeAttribute(String name);

    boolean hasAttribute(String name);

    String[] attributeNames();

}
```

AttributeAccessor 主要提供了访问和修改属性的方法。

> 需要注意的是，这里的属性是额外加给 BeanDefinition 的属性，而不是它所代表的对象本身的属性。

BeanMetadataElement 的源码如下：

```java
public interface BeanMetadataElement {

    @Nullable
    default Object getSource() {
        return null;
    }

}
```

BeanMetadataElement 只提供了一个方法，它是用来获取 Bean 元数据的配置源的（一般指向 Bean 对象的 class 文件的磁盘目录）。

了解完这两个接口，我们再看回到 BeanDefinition 的源码，其中包含了几个静态变量：

```java
    String SCOPE_SINGLETON = ConfigurableBeanFactory.SCOPE_SINGLETON;
    String SCOPE_PROTOTYPE = ConfigurableBeanFactory.SCOPE_PROTOTYPE;

    int ROLE_APPLICATION = 0;
    int ROLE_SUPPORT = 1;
    int ROLE_INFRASTRUCTURE = 2;
```

SCOPE_SINGLETON、SCOPE_PROTOTYPE 用于表示对象是单例还是多例的，ROLE_ 开头的几个常量跟 ComponentDefinition 有关，具体的作用在此不做分析，有兴趣的读者可以自行研究。

接下来，我们可以看到下面的方法，这些方法跟我们如何定义这个 BeanDefinition 有关，我们前面讲到的用来定义 Bean 特性的一些内容，如：@Scope、@Lazy、@DependsOn、@Primary、initMethod、destroyMethod、是否是候选类等，在这里都可以看到。

```java
    void setScope(@Nullable String scope);

    @Nullable
    String getScope();

    void setLazyInit(boolean lazyInit);

    boolean isLazyInit();

    void setDependsOn(@Nullable String... dependsOn);

    @Nullable
    String[] getDependsOn();

    void setAutowireCandidate(boolean autowireCandidate);

    boolean isAutowireCandidate();

    void setPrimary(boolean primary);

    boolean isPrimary();

    void setInitMethodName(@Nullable String initMethodName);

    @Nullable
    String getInitMethodName();

    void setDestroyMethodName(@Nullable String destroyMethodName);

    @Nullable
    String getDestroyMethodName();

    boolean isSingleton();

    boolean isPrototype();
```

再接着，我们看到下面的方法，基本上是一些辅助性方法和对 Bean 对象本身的基本内容进行管理的方法（如：对类名、构造函数参数值、属性值等进行管理的方法）。

```java
    void setParentName(@Nullable String parentName);

    @Nullable
    String getParentName();

    void setBeanClassName(@Nullable String beanClassName);

    @Nullable
    String getBeanClassName();

    void setFactoryBeanName(@Nullable String factoryBeanName);

    @Nullable
    String getFactoryBeanName();

    void setFactoryMethodName(@Nullable String factoryMethodName);

    @Nullable
    String getFactoryMethodName();

    ConstructorArgumentValues getConstructorArgumentValues();

    default boolean hasConstructorArgumentValues() {
        return !getConstructorArgumentValues().isEmpty();
    }

    MutablePropertyValues getPropertyValues();

    default boolean hasPropertyValues() {
        return !getPropertyValues().isEmpty();
    }

    void setDescription(@Nullable String description);

    @Nullable
    String getDescription();

    void setRole(int role);

    int getRole();

    ResolvableType getResolvableType();

    boolean isAbstract();

    @Nullable
    String getResourceDescription();

    @Nullable
    BeanDefinition getOriginatingBeanDefinition();
```

## 总结

本文，我们简单介绍了 BeanDefinition 的基本内容，下一篇文章，我们将探讨它是如何起作用的。