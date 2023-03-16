# 深入学习 Spring Web 开发 —— BeanDefinition（下）

[TOC]

上一篇文章，我们探讨了什么是 BeanDefinition，本文，我们主要探讨 BeanDefinition 是如何起作用的。

## BeanDefinition 是如何起作用的

在前面的文章中，我们已经了解过项目中的 Bean 是如何被一步步收集起来的（见：[深入学习 Spring Web 开发 —— 应用启动](spring_web_run.html)），收集起来之后就会被注册为 BeanDefinition。BeanDefinition 除了可以通过自动注册得到，也可以通过手动注册得到（见：[深入学习 Spring Web 开发 —— Bean 的声明](spring_web_bean_declaration.html)）。下面，我们先了解一下 BeanDefinition 的注册是如何进行的。

### Bean 的注册过程

我们先回忆一下前面的文章中提到过的 Bean 的手动注册：

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

顺着这段代码，我们从 registerBeanDefinition() 方法开始，一步步跟进，最终就可以了解清楚整个注册过程了。