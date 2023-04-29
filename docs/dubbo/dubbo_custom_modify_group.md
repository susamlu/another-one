# Dubbo 自定义 ReferenceAnnotationBeanPostProcessor，以支持灵活配置 group

## 缘起

是日，公司小张在测试环境调用 Dubbo 服务遇到了点问题：

1. 蜡笔小新服务调用了用户服务和小泥鳅服务；
2. 用户服务的 Dubbo 分组只有一个：分组A，小泥鳅服务的 Dubbo 分组有好几个：分组A、分组B、分组C ...；
3. 小张需要与小泥鳅服务 `分组B` 上的实例进行联调；

需要如何配置呢？小张犯愁了。

## 事态发展

> 首先，要说明的是，公司中使用的 Dubbo 版本是 `com.alibaba:dubbo:2.6.3`。

### Dubbo 原生支持哪些配置

#### 全局配置

使用如下全局配置，用户服务和小泥鳅服务都会到 `分组B` 查找 service，最终会导致用户服务的 service 查找失败。

```properties
spring.dubbo.registry.group = 分组B
```

#### @Reference

使用 `@Reference` 配置分组，这样每个 service 都要配置一下，比较麻烦。

```kotlin
@Reference(group = "分组B")
private lateinit var serviceC: ServiceC
```

### 有没有更好的方式

于是，小张决心修改 Dubbo 底层，以更好支持这一需求。

#### 确定目标

目前的服务情况大概是这样的：

```html
用户服务 service：
com.xz.user.serviceA
com.xz.user.serviceB
小泥鳅服务 service：
com.xz.loach.serviceC
com.xz.loach.serviceD
```

如果支持这样的配置，就很好：

```
# 所有 com.xz.loach 下的 service，都到 分组B 去查找
dubbo.custom.groups = {"com.xz.loach.*": "分组B"}
```

#### 实现分析

通过追踪 `@Reference` 被引用的地方，最终找到下面这段代码，是很好的切入点：

```java
public class ReferenceAnnotationBeanPostProcessor extends InstantiationAwareBeanPostProcessorAdapter
        implements MergedBeanDefinitionPostProcessor, PriorityOrdered, ApplicationContextAware, BeanClassLoaderAware,
        DisposableBean {

    // ......

    private ReferenceBean<?> buildReferenceBean(Reference reference, Class<?> referenceClass) throws Exception {

        String referenceBeanCacheKey = generateReferenceBeanCacheKey(reference, referenceClass);

        ReferenceBean<?> referenceBean = referenceBeansCache.get(referenceBeanCacheKey);

        if (referenceBean == null) {

            ReferenceBeanBuilder beanBuilder = ReferenceBeanBuilder
                    .create(reference, classLoader, applicationContext)
                    .interfaceClass(referenceClass);

            referenceBean = beanBuilder.build();

            referenceBeansCache.putIfAbsent(referenceBeanCacheKey, referenceBean);

        }

        return referenceBean;

    }

    // ......
    
}
```

无奈的是，`buildReferenceBean` 是 `private` 的，它既不能被继承，又不能被代理，似乎只能祭出 `ASM` 了。但，小张想了想，想到了另一个损招：直接复制 `ReferenceAnnotationBeanPostProcessor` 的代码，把 `buildReferenceBean` 改成 `public`，然后将 `ReferenceAnnotationBeanPostProcessor` 整个替换掉。

# 终

最终的实现如下：

- 复制 `ReferenceAnnotationBeanPostProcessor`：

> `ReferenceAnnotationBeanPostProcessor` 又引入了几个默认作用域的类，所以最终复制了以下几个类。

```java
AbstractAnnotationConfigBeanBuilder.java
AnnotationPropertyValuesAdapter.java
ReferenceAnnotationBeanPostProcessor.java
ReferenceBeanBuilder.java
```

- 定义配置类：

```kotlin
@Configuration
class CustomDubboProperties {

    /**
     * groups 配置样例：
     * dubbo.custom.groups = {"com.xz.loach.*": "分组B"}
     */
    @Value("\${dubbo.custom.groups:}")
    lateinit var groups: String

}
```

- 自定义 `ReferenceAnnotationBeanPostProcessor`：

```kotlin
@Component
class CustomReferenceAnnotationBeanPostProcessor : ReferenceAnnotationBeanPostProcessor() {

    private val logger = LoggerFactory.getLogger(CustomReferenceAnnotationBeanPostProcessor::class.java)

    private lateinit var applicationContext: ApplicationContext

    private var groupMap: Map<String, String>? = null

    @Throws(BeansException::class)
    override fun setApplicationContext(applicationContext: ApplicationContext) {
        this.applicationContext = applicationContext
        super.setApplicationContext(applicationContext)
    }

    @Throws(Exception::class)
    override fun buildReferenceBean(reference: Reference?, referenceClass: Class<*>?): ReferenceBean<*>? {
        if (reference == null || referenceClass == null) {
            return super.buildReferenceBean(reference, referenceClass)
        }

        // 查找能匹配上的 group
        val antPathMatcher = AntPathMatcher()
        var group: String? = null
        getGroupMap().map {
            if (antPathMatcher.match(it.key, referenceClass.name)) {
                group = it.value
                return@map
            }
        }
        if (group.isNullOrBlank()) {
            return super.buildReferenceBean(reference, referenceClass)
        }

        // 替换 group
        val invocationHandler = Proxy.getInvocationHandler(reference)
        val memberValuesField = invocationHandler.javaClass.getDeclaredField("memberValues")
        memberValuesField.setAccessible(true)
        val memberValuesMap = memberValuesField[invocationHandler] as MutableMap<Any, Any>
        memberValuesMap["group"] = group!!

        logger.info("将[{}]的group替换为：{}", referenceClass.name, group)
        return super.buildReferenceBean(reference, referenceClass)
    }

    private fun getGroupMap(): Map<String, String> {
        if (groupMap != null) {
            return groupMap!!
        }

        val customDubboProperties = applicationContext.getBean(CustomDubboProperties::class.java)
        val groups = customDubboProperties.groups
        if (groups.isNullOrBlank()) {
            groupMap = mapOf()
            return groupMap!!
        }

        try {
            groupMap = JsonUtil.toBean(groups, Map::class.java) as Map<String, String>
        } catch (e: Exception) {
            logger.error("property \${dubbo.custom.groups} config error", e)
            groupMap = mapOf()
        }
        return groupMap!!
    }

}
```

- 使用自定义类替换 `ReferenceAnnotationBeanPostProcessor`：

```kotlin
@Component
class CustomDubboBeanFactoryPostProcessor : BeanFactoryPostProcessor {

    private val logger: Logger = LoggerFactory.getLogger(CustomDubboBeanFactoryPostProcessor::class.java)

    @Throws(BeansException::class)
    override fun postProcessBeanFactory(beanFactory: ConfigurableListableBeanFactory) {
        logger.info("使用自定义的CustomReferenceAnnotationBeanPostProcessor，替换com.alibaba:dubbo:2.6.3的ReferenceAnnotationBeanPostProcessor")
        val beanName = ReferenceAnnotationBeanPostProcessor.BEAN_NAME
        val beanDefinition = beanFactory.getBeanDefinition(beanName) as AbstractBeanDefinition
        beanDefinition.setBeanClass(CustomReferenceAnnotationBeanPostProcessor::class.java)
    }

}
```

大功告成。最终，小张高高兴兴地通过下面的配置，灵活配置不同服务的 group。再也不用担心复杂的测试环境了。

```properties
dubbo.custom.groups = {"com.xz.loach.*": "分组B"}
```