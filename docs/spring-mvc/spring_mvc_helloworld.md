# 从零搭建 Spring MVC 项目 —— HelloWorld

阅读本文前，你需要掌握 Java 和 Maven 的基础知识。

搭建一个 Spring MVC HelloWorld 项目，只需要简单的几步：

## 1. 继承 Spring Boot 项目

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.7.2</version>
    <relativePath/>
</parent>
```

## 2. 引入 Spring MVC 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

## 3. 编写 Controller 代码

```java
@RestController
public class HelloWorldController {

    @GetMapping("/hello")
    public String hello(@RequestParam(value = "name", defaultValue = "World") String name) {
        return String.format("Hello %s!", name);
    }

}
```

## 4. 编写启动类

```java
@SpringBootApplication
public class HelloWorldApplication {

    public static void main(String[] args) {
        SpringApplication.run(HelloWorldApplication.class, args);
    }

}
```

## 5. 运行启动类

运行启动类，在浏览器中输入：`http://localhost:8080/hello?name=小穆` ，即可看到如下效果：

<img src="../images/spring_mvc_helloworld_0.png" width="100%" style="border: solid 1px #dce6f0; border-radius: 0.3rem;">

[返回首页](https://susamlu.github.io/paitse)
[获取源码](https://github.com/susamlu/spring-mvc)