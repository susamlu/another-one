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

AttributeAccessor 主要提供了访问和修改对像属性的方法。BeanMetadataElement 的源码如下：