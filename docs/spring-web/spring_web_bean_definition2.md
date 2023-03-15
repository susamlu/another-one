# 深入学习 Spring Web 开发 —— BeanDefinition（下）

[TOC]

上一篇文章探讨什么是 BeanDefinition，本文，我们主要探讨 BeanDefinition 是如何起作用的。

## BeanDefinition 是如何起作用的

在前面的文章中，我们已经了解过项目中的 Bean 是如何被一步步收集起来的（见：[深入学习 Spring Web 开发 —— 应用启动](spring-web/spring_web_run.html)），收集起来之后就会被注册为 BeanDefinition。BeanDefinition 除了可以通过自动注册得到，我们也可以进行主动注册（见：[深入学习 Spring Web 开发 —— Bean 的声明](spring-web/spring_web_bean_declaration.html)）。在此，我们先了解 Bean 的注册是如何进行的。

### Bean 的注册过程