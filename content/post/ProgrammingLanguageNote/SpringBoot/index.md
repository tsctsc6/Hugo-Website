+++
date = '2025-07-22T18:56:11+08:00'
draft = false
title = 'SpringBoot 架构介绍'
categories = ['Sub Sections']
+++

SpringBoot 是基于 Java 的，但是为了解决传统 Java（尤其 Java EE/Jakarta EE）在企业级应用开发、部署和运维中长期存在的一系列痛点和复杂性，打破了许多 java 的传统。故需要单独一篇文章。

## 从空白项目创建 SpringBoot 项目
### 初始化项目结构​
首先初始化一个 Maven 项目。

在 `./src/main/java` 中，创建软件包（其实就是目录），比如 `org.example` ，在其中创建 Java 类 `Main` 。这是 Spring Boot 主启动类​。

### 设置 pom.xml
#### ​​设置 Parent POM
在 `<project>` 标签内，添加如下 `<parent>` 配置。它继承 Spring Boot 的默认配置和依赖管理，让我们无需指定每个依赖的版本号。

```xml {name="pom.xml"}
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <!-- 使用最新的稳定版本，例如 3.2.4 -->
    <version>3.2.4</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
```

#### 添加依赖
在 `<project>` 标签内，添加你需要的 Spring Boot starter 依赖。最基本的 Web 项目添加 `spring-boot-starter-web` 。

```xml {name="pom.xml"}
<dependencies>
    <!-- Spring Boot Web Starter (包含内嵌Tomcat, Spring MVC, JSON支持等) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- (可选) Spring Boot Starter Test 用于测试 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <!-- (可选) Lombok (减少样板代码 Getter/Setter等), 需要在IDE中安装Lombok插件 -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <scope>provided</scope> <!-- 编译和测试时有效，运行时不需要 -->
    </dependency>
</dependencies>
```

#### 添加 Maven 插件
在 `<project>` 标签内，添加 Spring Boot Maven 插件。它允许我们使用 `mvn spring-boot:run` 启动应用，并打包成可执行 `jar` 。

```xml {name="pom.xml"}
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

### 编写主启动类
在刚刚创建的 `Main` 类中，编写以下代码：

```java {name="src/main/java/org/example/Main.java"}
package org.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication // 核心注解！组合了配置、自动配置、组件扫描
public class Main {
    public static void main(String[] args) {
        SpringApplication.run(Main.class, args); // 启动应用
    }
}
```

### 创建配置文件
在 `src/main/resources` 中创建文件 `application.yml` ，填写以下内容：

```yaml {name="src/main/resources/application.yml"}
server:
  port: 8080 # 默认是8080，你也可以改成其他端口如9090
spring:
  application:
    name: MyFirstSpringBootApp # 应用名称
```

至此，一个空白的 Spring Boot 项目已经创建。

## 部署
确保已经做了[添加 Maven 插件](#添加-maven-插件)步骤。

执行构建:

```shell
mvn clean package
```

> 如果使用 IDEA ，可以在右上角，新建配置 -> 新建配置 -> Maven , 设置命令为 `clean package` .

在项目根目录下的 `target` 目录下生成 `your-project-1.0-SNAPSHOT-jar-with-dependencies.jar` （包含所有依赖）。

运行:

```shell
java -jar ./target/your-project-1.0-SNAPSHOT-jar-with-dependencies.jar
```

可以在 `jar` 包的目录中，创建配置文件 `application.yml` 。