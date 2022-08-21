# 从零搭建 Spring MVC 项目 —— HttpMessageConverter

## Long 转 String，日期转时间戳

### 1. 初始代码

```java
@Data
public class UserRequest {

    private Long id;

    private String name;

    private Date createTime;

    private Date updateTime;

}
```

```java
@RestController
public class UserController {

    @PostMapping("/api/users")
    public UserRequest createUser(@RequestBody UserRequest userRequest) {
        userRequest.setId(IdUtil.getSnowflakeNextId());
        return userRequest;
    }

}
```

### 2. 运行结果(1)

使用 curl 访问接口：

```shell
curl --location --request POST 'http://localhost:8080/api/users' \
--header 'Content-Type: application/json' \
--data-raw '{
    "name": "小穆",
    "createTime": 1661004235959,
    "updateTime": 1661004235959
}'
```

返回结果如下：

```json
{
    "id": 1560991269656014848,
    "name": "小穆",
    "createTime": "2022-08-20T14:03:55.959+00:00",
    "updateTime": "2022-08-20T14:03:55.959+00:00"
}
```

目前存在两个问题：

1. 返回的 id 位数过长，前端接收到的结果会发生溢出，后面几位数字会被转成 0；
2. 返回的时间格式对前端不友好，前端同事希望收到的是时间戳。

如何解决？继续往下看。

### 3. 使用 HttpMessageConverter 转换数据

> Long.class 只能转换包装类型 Long 的数据，Long.TYPE 只能转换基础类型 long 的数据。

```java
@Configuration
public class WebConfiguration implements WebMvcConfigurer {

    @Override
    public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
        MappingJackson2HttpMessageConverter jackson2HttpMessageConverter = new MappingJackson2HttpMessageConverter();
        converters.add(0, jackson2HttpMessageConverter);

        ObjectMapper objectMapper = jackson2HttpMessageConverter.getObjectMapper();
        SimpleModule simpleModule = new SimpleModule();
        simpleModule.addSerializer(Long.class, ToStringSerializer.instance);
        simpleModule.addSerializer(Long.TYPE, ToStringSerializer.instance);
        simpleModule.addSerializer(Date.class, DateSerializer.instance);
        objectMapper.registerModule(simpleModule);
    }

}
```

### 4. 运行结果(2)

使用 curl 访问接口：

```shell
curl --location --request POST 'http://localhost:8080/api/users' \
--header 'Content-Type: application/json' \
--data-raw '{
    "name": "小穆",
    "createTime": 1661004235959,
    "updateTime": 1661004235959
}'
```

返回结果如下，上面遇到的问题顺利解决：

```json
{
  "id": "1561221986730184704",
  "name": "小穆",
  "createTime": 1661004235959,
  "updateTime": 1661004235959
}
```

## 进阶学习

### HttpMessageConverter 原理

#### 启动过程

#### 调用过程

#### 思考题：

1. 使用 configureHandlerExceptionResolvers，还是 extendHandlerExceptionResolvers？
2. converters.add(jackson2HttpMessageConverter) 与 converters.add(0, jackson2HttpMessageConverter) 有何区别？

### 自定义 Serializer

## 扩展学习

EnableWebMvc, WebMvcAutoConfiguration, WebMvcConfigurationSupport, WebMvcConfigurer, WebMvcConfigurerAdapter