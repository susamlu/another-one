# Spring Web 开发：从入门到精通 —— Validation

通过 Validation 注解，可以方便地定义需要以何种方式来校验接口参数。

## 快速开始

### 1. 引入 Validation 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

### 2. 使用校验注解

> @Data 为 lombok 的注解，在此不做详细说明。

使用校验注解，以定义需要以何种方式来校验接口参数。

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

### 3. 声明校验参数

在 Controller 中，声明接口方法中需要校验的参数。

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
        List<String> errorMessages = exception.getFieldErrors()
                .stream()
                .map(fieldError -> String.format("%s %s", fieldError.getField(), 
                        fieldError.getDefaultMessage()))
                .sorted()
                .collect(Collectors.toList());

        Map<String, Object> errorMessageMap = CollectionUtils.newLinkedHashMap(2);
        errorMessageMap.put("code", HttpStatus.BAD_REQUEST.value());
        errorMessageMap.put("message", errorMessages);

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

## 常用注解

下面列举了，在日常的开发当中，我们经常需要使用到的 Validation 注解。这些注解，主要定义在 javax.validation 包下。

### @NotNull、@NotEmpty 与 @NotBlank

| 注解 | 作用对象 | 含义 |
| :--- | :--- | :--- |
| @NotNull | 可以作用于任意对象 | 对象不能为 null |
| @NotEmpty | 只能作用于 String、Collection、Map 和数组 | 对象不能为 null ，且对象的长度不能为 0 |
| @NotBlank | 只能作用于 String | 字符串不能为 null ，且字符串的长度不能为 0 |

示例代码：

```java
@Data
public class NotAnnotationRequest {

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

### @Size 与 @Min、@Max

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
| flags | 标识正则表达式的模式，包含：<br/>Pattern.Flag.UNIX_LINES、<br/>Pattern.Flag.CASE_INSENSITIVE、<br/>Pattern.Flag.COMMENTS、<br/>Pattern.Flag.MULTILINE、<br/>Pattern.Flag.DOTALL、<br/>Pattern.Flag.UNICODE_CASE、<br/>Pattern.Flag.CANON_EQ<br/>共7种模式 |
| message | 错误提示信息（message 不是 @Pattern 特有的，所有 Validation 注解都包含该参数） |

示例代码：

```java
@Data
public class PatternAnnotationRequest {

    @Pattern(regexp = "^1[3456789]\\d{9}$", message = "手机号码格式不正确")
    private String mobile;

}
```

### @Valid 与 @Validated

@Valid 和 @Validated 都可以用来声明校验参数，但它们并不完全一样，它们的区别主要包含以下几个方面：

- @Valid 是定义在 javax 包下的标准注解，@Validated 是 Spring 官方定义的注解。
- @Valid 可以作用于方法、字段、构造函数、参数和运行时使用的类型，@Validated 可作用于类型、方法和参数。
- @Valid 可以用来定义嵌套校验，@Validated 无法单独完成嵌套校验的定义。
- @Validated 支持分组校验，@Valid 不支持分组校验。

#### 使用 @Valid 声明校验参数

除了可以使用 @Validated 声明校验参数，也可以使用 @Valid 对需要校验的参数进行声明。

```java
@RestController
public class UserController {

    @PostMapping("/api/users")
    public UserRequest createUser(@RequestBody @Valid UserRequest userRequest) {
        return userRequest;
    }

}
```

#### 嵌套校验

可以使用 @Validated + @Valid，或 @Valid + @Valid 的方式共同定义嵌套校验。

> 使用 @Valid 指定需要校验的接口参数；使用 @Validated 或 @Valid 声明 Controller 的接口方法中，需要校验的参数。

使用校验注解，指定需要校验的接口参数：

```java
@Data
public class CompanyRequest {

    @NotBlank
    private String name;

    @Valid
    @NotNull
    private AddressRequest address;

}
```

```java
@Data
public class AddressRequest {

    @NotBlank
    private String country;

    @NotBlank
    private String province;

    @NotBlank
    private String city;

}
```

声明 Controller 的接口方法中，需要校验的参数：

```java
@RestController
public class NestValidController {

    @PostMapping("/api/companies")
    public CompanyRequest createCompany(@RequestBody @Validated CompanyRequest companyRequest) {
        return companyRequest;
    }

}
```

#### 分组校验

分组校验可以使用在参数定义相同、接口校验不同的场景中。

定义分组：

```java
public interface CreateGroup extends Default {
}
```

```java
public interface UpdateGroup extends Default {
}
```

使用校验注解，指定需要校验的接口参数：

```java
@Data
public class OrganizationRequest {

    @NotBlank(groups = CreateGroup.class)
    @Null(groups = UpdateGroup.class)
    private String orgCode;

    @NotBlank
    private String orgName;

}
```

声明 Controller 的接口方法中，需要校验的参数：

```java
@RestController
public class GroupValidController {

    @PostMapping("/api/organizations")
    public OrganizationRequest createOrganization(
            @RequestBody @Validated(CreateGroup.class) OrganizationRequest organizationRequest) {
        return organizationRequest;
    }

    @PutMapping("/api/organizations")
    public OrganizationRequest updateOrganization(
            @RequestBody @Validated(UpdateGroup.class) OrganizationRequest organizationRequest) {
        return organizationRequest;
    }

}
```

## 其他注解

除了常用的注解，javax.validation 下还定义了下面这些注解，以供我们在有需要时选择使用。

### @Null

| 注解 | 作用对象 | 含义 |
| :--- | :--- | :--- |
| @Null | 可以作用于任意对象 | 对象必须为 null |

示例代码：

```java
@Data
public class NullAnnotationRequest {

    @Null
    private Object nullObject;

}
```

### @Email

| 注解 | 作用对象 | 含义 |
| :--- | :--- | :--- |
| @Email | 只能作用于 String | 字符串必须为合法的 email |

示例代码：

```java
@Data
public class EmailAnnotationRequest {

    @Email
    private String email;

}
```

### 布尔类型注解

| 注解 | 作用对象 | 含义 |
| :--- | :--- | :--- |
| @AssertTrue | 可以作用于 boolean 和 Boolean | 对象必须为 true |
| @AssertFalse | 可以作用于 boolean 和 Boolean | 对象必须为 false |

示例代码：

```java
@Data
public class BooleanAnnotationRequest {

    @AssertTrue
    private Boolean trueParam;

    @AssertFalse
    private boolean falseParam;

}
```

### 数字类型注解

| 注解 | 作用对象 | 含义 |
| :--- | :--- | :--- |
| @DecimalMin | 可以作用于数字类型和字符串 | 表示数字的最小值 |
| @DecimalMax | 可以作用于数字类型和字符串 | 表示数字的最大值 |
| @Digits | 可以作用于数字类型和字符串 | 包含两个变量：integer 和 fraction，其中 integer 表示整数位的最大位数；fraction 表示小数位的最大位数 |
| @Negative | 可以作用于数字类型 | 表示必须为负数 |
| @NegativeOrZero | 可以作用于数字类型 | 表示必须为负数或零 |
| @Positive | 可以作用于数字类型 | 表示必须为正数 |
| @PositiveOrZero | 可以作用于数字类型 | 表示必须为正数或零 |

示例代码：

```java
@Data
public class NumberAnnotationRequest {

    @DecimalMin("1.5")
    @DecimalMax("2.5")
    private BigDecimal limitedBigDecimal;

    @Digits(integer = 1, fraction = 2)
    private BigDecimal digits;

    @Negative
    private short negative;

    @NegativeOrZero
    private int negativeOrZero;

    @Positive
    private long positive;

    @PositiveOrZero
    private BigDecimal positiveOrZero;

}
```

### 时间类型注解

| 注解 | 作用对象 | 含义 |
| :--- | :--- | :--- |
| @Past | 可以作用于时间类型 | 表示必须为过去的时间 |
| @PastOrPresent | 可以作用于时间类型 | 表示必须为过去的时间或现在 |
| @Future | 可以作用于时间类型 | 表示必须为将来的时间 |
| @FutureOrPresent | 可以作用于时间类型 | 表示必须为将来的时间或现在 |

示例代码：

```java
@Data
public class TimeAnnotationRequest {

    @Past
    private Date past;

    @PastOrPresent
    private Date pastOrPresent;

    @Future
    private Date future;

    @FutureOrPresent
    private Date futureOrPresent;

}
```

[返回首页](https://susamlu.github.io/paitse)
[获取源码](https://github.com/susamlu/spring-web)
