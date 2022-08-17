# 从零搭建 Spring MVC 项目 —— Validation

## 快速开始

### 1. 引入 Validation 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>2.0.1.Final</version>
</dependency>
```

### 2. 编写校验代码

```java
public class UserRequest {

    @NotBlank
    private String name;

    @Min(0)
    @Max(200)
    private Integer age;

}
```

### 3. 指定校验参数

```java
public UserRequest createUser(@RequestBody @Validated UserRequest userRequest) {
    return userRequest;
}
```

### 4. 捕捉全局异常

> 这一步不是必须的，主要是为了能方便查看校验结果。

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, Object>> handleMethodArgumentNotValidException(
            MethodArgumentNotValidException exception) {
        List<FieldError> fieldErrors = exception.getFieldErrors();
        List<String> errorMessages = new ArrayList<>(fieldErrors.size());
        for (FieldError fieldError : fieldErrors) {
            errorMessages.add(fieldError.getField() + " " + fieldError.getDefaultMessage());
        }

        Map<String, Object> errorMessageMap = CollectionUtils.newLinkedHashMap(2);
        errorMessageMap.put("code", HttpStatus.BAD_REQUEST.value());
        errorMessageMap.put("message", errorMessages.stream().sorted().collect(Collectors.toList()));
        return ResponseEntity.badRequest().body(errorMessageMap);
    }

}
```

### 5. 查看校验效果

使用 curl 访问接口：

```shell
curl --location --request POST 'http://localhost:8080/api/users' \
--header 'Content-Type: application/json' \
--data-raw '{
    "name": "",
    "age": -1
}'
```

最终的校验结果如下：

```json
{
    "code": 400,
    "message": [
        "age 最小不能小于0",
        "name 不能为空"
    ]
}
```

## 进阶内容

### @NotNull、@NotEmpty 与 @NotBlank

| 注解 | 作用对象 | 含义 |
| :--- | :--- | :--- |
| @NotNull | 可以作用于任意对象 | 对象不能为 null |
| @NotEmpty | 只能作用于 String、Collection、Map 和数组 | 对象不能为 null ，且对象的长度不能为 0 |
| @NotBlank | 只能作用于 String | 字符串不能为 null ，且字符串的长度不能为 0 |

示例代码：

```java
public class InvertAnnotationRequest {

    @NotNull
    private Object notNullObject;

    @NotNull
    private String notNullString;

    @NotNull
    private Collection<Integer> notNullCollection;

    @NotNull
    private Map<String, String> notNullMap;

    @NotNull
    private int[] notNullArray;

    @NotEmpty
    private String notEmptyString;

    @NotEmpty
    private Collection<Integer> notEmptyCollection;

    @NotEmpty
    private Map<String, String> notEmptyMap;

    @NotEmpty
    private int[] notEmptyArray;

    @NotBlank
    private String notBlankString;

}
```

## 其他内容

[返回首页](https://susamlu.github.io/another-one)
[获取源码](https://github.com/susamlu/spring-mvc)
