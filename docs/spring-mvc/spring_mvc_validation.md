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

> @Data 为 lombok 的注解，在此不做详细说明。

```java
@Data
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
@RestController
public class UserController {

    @PostMapping("/api/users")
    public UserRequest createUser(@RequestBody @Validated UserRequest userRequest) {
        return userRequest;
    }

}
```

### 4. 捕捉全局异常

> 这一步不是必须的，目的是为了能方便查看校验结果。

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
@Data
public class InvertAnnotationRequest {

    @NotNull
    private Object notNullObject;

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

### @Size 与 @Max、@Min

| 注解 | 作用对象 | 含义 |
| :--- | :--- | :--- |
| @Size | 可以作用于 String、Collection、Map 和数组 | 包含两个变量：min 和 max，其中 min 的默认值为 0 ，表示对象的最小长度；max 的默认值为 Integer.MAX_VALUE ，表示对象的最大长度 |
| @Min | 可以作用于 BigDecimal、BigInteger，byte、short、int、long 以及它们的包装类型 | 表示数字的最小值 |
| @Max | 可以作用于 BigDecimal、BigInteger，byte、short、int、long 以及它们的包装类型 | 表示数字的最大值 |

示例代码：

```java
@Data
public class RangeAnnotationRequest {

    @Size(min = 1, max = 5)
    private String limitedString;

    @Size(min = 1, max = 5)
    private Collection<Integer> limitedCollection;

    @Size(min = 1, max = 5)
    private Map<String, String> limitedMap;

    @Size(min = 1, max = 5)
    private int[] limitedArray;

    @Min(1)
    @Max(2)
    private Integer limitedNumber;

}
```

### @Pattern

| 参数 | 含义 |
| :--- | :--- |
| regexp | 正则表达式 |
| flags | 标识正则表达式的模式，包含：Pattern.Flag.UNIX_LINES、Pattern.Flag.CASE_INSENSITIVE、Pattern.Flag.COMMENTS、Pattern.Flag.MULTILINE、Pattern.Flag.DOTALL、Pattern.Flag.UNICODE_CASE、Pattern.Flag.CANON_EQ 共7种模式 |
| message | 错误提示信息（该参数不是 @Pattern 特有，所有 Validation 注解都包含该参数） |

示例代码：

```java
@Data
public class PatternAnnotationRequest {

    @Pattern(regexp = "^1[3456789]\\d{9}$", message = "手机号码格式不正确")
    private String mobile;

}
```

### @Valid 与 @Validated

## 扩展内容

[返回首页](https://susamlu.github.io/paitse)
[获取源码](https://github.com/susamlu/spring-mvc)
