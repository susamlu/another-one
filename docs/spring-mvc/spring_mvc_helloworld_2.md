# 深入浅出 Spring MVC —— HelloWorld (进阶篇)

上一篇文章已经介绍了如何快速搭建一个 Spring MVC 项目，本文将对上文中提及的几个技术点，进行深入的讲解。

我们前面提到，搭建 Spring MVC 项目时，只需要继承 `spring-boot-starter-parent` ，并引入 `spring-boot-starter-web` ，即可把 Spring MVC
项目所需要的全部依赖引进来，并且我们不需要指定依赖的版本，这是如何做到的呢？

这里会涉及到 Maven 的 parent 和 dependencyManagement 标签，我们先讲讲这两个标签的作用。

## Maven 标签

### parent

在 Maven 项目中，可以通过继承的方式，让子项目继承父项目所定义的内容，如：groupId、version、properties、dependencies 等。

下面的例子中，`my-app-child` 只需要继承 `my-app-parent` ，即可引入父项目的全部依赖。即父项目 `my-app-parent` 引入了 `maven-artifact` 和 `maven-core`
两个依赖，并指定了它们的版本，子项目只需继承 `my-app-parent` ，就引入了这两个依赖，并且依赖的版本也跟父项目所指定的版本一样。

```xml
<!-- my-app-parent -->
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.mycompany.app</groupId>
    <artifactId>my-app-parent</artifactId>
    <version>1.0-SNAPSHOT</version>

    <name>my-app-parent</name>
    <url>http://www.example.com</url>

    <properties>
        <mavenVersion>3.0</mavenVersion>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.apache.maven</groupId>
            <artifactId>maven-artifact</artifactId>
            <version>${mavenVersion}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.maven</groupId>
            <artifactId>maven-core</artifactId>
            <version>${mavenVersion}</version>
        </dependency>
    </dependencies>

</project>
```

```xml
<!-- my-app-child -->
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.mycompany.app</groupId>
        <artifactId>my-app-parent</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>my-app-child</artifactId>

</project>
```

### dependencyManagement

有时候子项目并不需要引入父项目的全部依赖，只需要引入部分依赖，但又希望在父项目中统一定义好依赖的版本，`dependencyManagement` 标签可以帮我们完成这个事情。

下面的例子中，`my-app-child` 引入了 `maven-core` 依赖，父项目仅仅只是预定义了依赖的版本。也就是说，父项目指定了 `maven-artifact` 和 `maven-core`
两个依赖的版本，但并没有引入依赖，在子项目中只引入了 `maven-core` 依赖，即 `maven-artifact` 是没有被引入的，且子项目无需指定 `maven-core` 依赖的版本，该依赖的版本就与父项目所指定的版本一样。

```xml
<!-- my-app-parent -->
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.mycompany.app</groupId>
    <artifactId>my-app-parent</artifactId>
    <version>1.0-SNAPSHOT</version>

    <name>my-app-parent</name>
    <url>http://www.example.com</url>

    <properties>
        <mavenVersion>3.0</mavenVersion>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.apache.maven</groupId>
                <artifactId>maven-artifact</artifactId>
                <version>${mavenVersion}</version>
            </dependency>
            <dependency>
                <groupId>org.apache.maven</groupId>
                <artifactId>maven-core</artifactId>
                <version>${mavenVersion}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>

</project>
```

```xml
<!-- my-app-child -->
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.mycompany.app</groupId>
        <artifactId>my-app-parent</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>my-app-child</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.apache.maven</groupId>
            <artifactId>maven-core</artifactId>
        </dependency>
    </dependencies>

</project>
```

## spring-boot-starter-parent

`spring-boot-starter-parent` 继承自父项目 `spring-boot-dependencies`，`spring-boot-dependencies` 通过 dependencyManagement 标签预先指定了 Spring Boot 项目的全部依赖版本。尤其是，将 `spring-boot-starter-web` 的版本指定为 2.7.2 。

```xml
<!-- spring-boot-starter-parent -->
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-dependencies</artifactId>
        <version>2.7.2</version>
    </parent>
    <!-- ... -->
</project>
```

```xml
<!-- spring-boot-dependencie -->
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.7.2</version>
    <packaging>pom</packaging>
    <name>spring-boot-dependencies</name>
    <description>Spring Boot Dependencies</description>
    <url>https://spring.io/projects/spring-boot</url>
    <!-- ... -->
    <dependencyManagement>
        <dependencies>
            <!-- ... -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
                <version>2.7.2</version>
            </dependency>
            <!-- ... -->
        </dependencies>
    </dependencyManagement>
    <!-- ... -->
</project>
```

## spring-boot-starter-web

2.7.2 版本的 `spring-boot-starter-web` 将 `spring-webmvc` 等项目引入了进来，并指定了各依赖的版本：

```xml
<!-- spring-boot-starter-web -->
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"
         xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <!-- ... -->
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.7.2</version>
    <name>spring-boot-starter-web</name>
    <description>Starter for building web, including RESTful, applications using Spring MVC. Uses Tomcat as the default
        embedded container
    </description>
    <url>https://spring.io/projects/spring-boot</url>
    <!-- ... -->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <version>2.7.2</version>
            <scope>compile</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-json</artifactId>
            <version>2.7.2</version>
            <scope>compile</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <version>2.7.2</version>
            <scope>compile</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>5.3.22</version>
            <scope>compile</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.3.22</version>
            <scope>compile</scope>
        </dependency>
    </dependencies>
</project>
```

在 `spring-boot-starter-parent` 和 `spring-boot-starter-web` 的共同作用下，就完成了 Spring MVC 项目全部依赖和依赖版本的定义。

## @SpringBootApplication 与 SpringApplication

不知道读者在编写 Spring Boot 项目的时候，有没有思考过启动类中为何需要同时使用 @SpringBootApplication 和 SpringApplication，它们分别的作用又是什么？下面让我们一起来一探究竟。

[返回首页](https://susamlu.github.io/paitse)
[获取源码](https://github.com/susamlu/spring-mvc)

