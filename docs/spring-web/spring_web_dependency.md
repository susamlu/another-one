# 深入学习 Spring Web 开发 —— 依赖引入

[TOC]

上一篇文章介绍了如何快速搭建一个 Spring Web 项目，本文重点聊聊项目的依赖是如何引入的。

我们前面提到，搭建 Spring Web 项目时，只需要继承 `spring-boot-starter-parent` 并指定它的版本，接着引入 `spring-boot-starter-web` ，且无需指定 `spring-boot-starter-web` 的版本，即可把 Spring Web 项目所需要的全部依赖引进来，具体是如何做到的呢？

这里会涉及到 Maven 的 parent 和 dependencyManagement 标签，我们先讲讲这两个标签的作用。

## Maven 标签

### parent

在 Maven 项目中，可以通过继承的方式，让子项目继承父项目所定义的内容，如：继承 groupId、version、properties、dependencies 等。

下面的例子中，`my-app-child` 只需要继承 `my-app-parent` ，即可引入父项目的全部依赖。即父项目 `my-app-parent` 引入了 `maven-artifact` 和 `maven-core` 两个依赖，并指定了它们的版本。

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

子项目只需继承 `my-app-parent` 并指定它的版本，就引入了这两个依赖，并且依赖的版本也跟父项目所指定的版本一样。

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

有时候子项目并不需要引入父项目的全部依赖，只需要引入部分依赖，但又希望在父项目中统一定义依赖的版本，`dependencyManagement` 标签可以帮我们完成这个事情。

下面的例子中，`my-app-child` 引入了 `maven-core` 依赖，父项目仅仅只是预定义了依赖的版本。也就是说，父项目指定了 `maven-artifact` 和 `maven-core` 两个依赖的版本，但并没有引入这两个依赖。

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

在子项目中只引入了 `maven-core` 依赖，即 `maven-artifact` 是没有被引入的，且子项目无需指定 `maven-core` 依赖的版本，该依赖的版本就与父项目所指定的版本一样。

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

## Spring Web 依赖的引入

### spring-boot-starter-parent

2.7.2 版本的 `spring-boot-starter-parent` 继承自父项目 `spring-boot-dependencies`。

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

`spring-boot-dependencies` 通过 dependencyManagement 标签预先指定了各个 `starter` 和其它各个依赖的版本。尤其是，将 `spring-boot-starter-web` 的版本指定为 2.7.2 。

```xml
<!-- spring-boot-dependencies -->
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.7.2</version>
    <packaging>pom</packaging>
    <name>spring-boot-dependencies</name>
    <!-- ... -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.apache.activemq</groupId>
                <artifactId>activemq-amqp</artifactId>
                <version>${activemq.version}</version>
            </dependency>
            <dependency>
                <groupId>org.apache.activemq</groupId>
                <artifactId>activemq-blueprint</artifactId>
                <version>${activemq.version}</version>
            </dependency>
            <!-- ... -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
                <version>2.7.2</version>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-webflux</artifactId>
                <version>2.7.2</version>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-websocket</artifactId>
                <version>2.7.2</version>
            </dependency>
            <!-- ... -->
        </dependencies>
    </dependencyManagement>
    <!-- ... -->
</project>
```

### spring-boot-starter-web

2.7.2 版本的 `spring-boot-starter-web` 将 Spring Web 项目需要的全部依赖，如 `spring-web` 等引入了进来，并指定了它们的版本：

```xml
<!-- spring-boot-starter-web -->
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"
         xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.7.2</version>
    <name>spring-boot-starter-web</name>
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

因此，在我们继承 2.7.2 版本的 `spring-boot-starter-parent` ，并引入 `spring-boot-starter-web` 之后，就相当于引入了 2.7.2 版本的 `spring-boot-starter-web` ，2.7.2 版本的 `spring-boot-starter-web` 又将该版本所需要的特定版本的依赖引入了进来，从而 Spring Web 项目就能够在 `spring-boot-starter-parent` 和 `spring-boot-starter-web` 的共同作用下，将项目所需要的依赖和依赖版本全部定义好。并且，在这个过程中，我们不需要的依赖，如 `spring-boot-starter-webflux` 等，是没有被引入进来的，这样就达到了既方便又灵活的效果。

> 读到这里，不知道读者有没有这样的疑问：我继承的是 `spring-boot-starter-parent` 项目，为什么却把 `spring-boot-dependencies` 的内容也继承了？这其实是由继承的传递性造成的，即继承的特性导致了子项目除了会继承父项目的内容，同时也会继承所有其它祖先项目的内容。

[返回首页](https://susamlu.github.io/paitse)
[获取源码](https://github.com/susamlu/spring-web)