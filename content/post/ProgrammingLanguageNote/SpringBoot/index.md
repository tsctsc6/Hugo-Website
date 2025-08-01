+++
date = '2025-07-22T18:56:11+08:00'
draft = false
title = 'SpringBoot 架构介绍'
categories = ['Sub Sections']
tags = ['Java', 'Web', 'MVC']
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

## SpringBoot 项目的一般结构
在 Spring Boot 应用中，标准的 MVC（Model-View-Controller）架构是推荐的分层结构，以下是典型的分层架构及其职责：

```txt
src/main/java
└── org.example
    ├── Application.java              // 主启动类
    │
    ├── controller                    // 控制层 (处理HTTP请求)
    │   └── UserController.java
    │
    ├── service                       // 服务层 (业务逻辑)
    │   ├── UserService.java          // 接口
    │   └── impl
    │       └── UserServiceImpl.java  // 实现类
    │
    ├── repository                    // 数据访问层 (访问数据库)
    │   └── UserRepository.java
    │
    ├── model                         // 实体/模型层
    │   ├── entity
    │   │   └── UserEntity.java       // 数据库实体
    │   └── dto
    │       ├── UserRequest.java      // 请求DTO
    │       └── UserResponse.java     // 响应DTO
    │
    ├── config                        // 配置层
    │   └── SwaggerConfig.java
    │
    └── exception                     // 异常处理
        └── GlobalExceptionHandler.java
```

### Controller 层 (控制器层)
职责​​：

* 接收 HTTP 请求
* 参数校验
* 调用 Service 层处理业务逻辑
* 返回 HTTP 响应

示例代码​​：

```java
@RestController
@RequestMapping("/api/users")
public class UserController
{
    private final UserService userService;

    // 构造函数注入 Service
    public UserController(UserService userService)
    {
        this.userService = userService;
    }

    @PostMapping
    public ResponseEntity<UserResponseDto> createUser(@Valid @RequestBody UserRequestDto request)
    {
        return ResponseEntity.status(HttpStatus.CREATED).body(userService.createUser(request));
    }

    @GetMapping("/{id}")
    public ResponseEntity<UserResponseDto> getUser(@PathVariable Long id)
    {
        return ResponseEntity.ok(userService.getUserById(id));
    }
}
```

### Service 层 (服务层)
职责​​：

* 封装业务逻辑
* 处理事务管理
* 协调多个 Repository 操作

示例代码​​：

```java
public interface UserService
{
    UserResponse createUser(UserRequest request);
    UserResponse getUserById(Long id);
}

@Service
public class UserServiceImpl implements UserService
{
    private final UserRepository userRepository;
    private final ModelMapper modelMapper;

    @Autowired
    public UserServiceImpl(UserRepository userRepository, ModelMapper modelMapper)
    {
        this.userRepository = userRepository;
        this.modelMapper = modelMapper;
    }

    @Override
    @Transactional
    public UserResponseDto createUser(UserRequestDto requestDto)
    {
        UserEntity user = modelMapper.map(requestDto, UserEntity.class);
        userRepository.save(user);
        return modelMapper.map(user, UserResponseDto.class);
    }

    @Override
    public UserResponse getUserById(Long id)
    {
        UserEntity user = userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User not found"));
        return modelMapper.map(user, UserResponse.class);
    }
}
```

### Repository 层 (数据访问层)
职责​​：

* 数据库操作接口
* 封装数据访问逻辑
* 提供 CRUD 操作

示例代码​​：

```java
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;

@Repository
public class UserRepository
{
    private final JdbcTemplate jdbcTemplate;

    public UserRepository(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    // 创建用户
    public int createUser(String username, String email)
    {
        String sql = "INSERT INTO users (username, email) VALUES (?, ?)";
        return jdbcTemplate.update(sql, username, email);
    }

    // 根据ID查询用户
    public User findUserById(int id)
    {
        String sql = "SELECT * FROM users WHERE id = ?";
        return jdbcTemplate.queryForObject(sql, new Object[]{id}, (rs, rowNum) ->
            new User(
                rs.getInt("id"),
                rs.getString("username"),
                rs.getString("email")
            )
        );
    }

    // 获取所有用户
    public List<User> findAllUsers()
    {
        String sql = "SELECT * FROM users";
        return jdbcTemplate.query(sql, (rs, rowNum) ->
            new User(
                rs.getInt("id"),
                rs.getString("username"),
                rs.getString("email")
            )
        );
    }
}
```

> 以上代码需要 Spring Boot JDBC 依赖：
> ```
>     <dependency>
>         <groupId>org.springframework.boot</groupId>
>         <artifactId>spring-boot-starter-jdbc</artifactId>
>         <version>3.2.4</version> <!-- 使用与您的 Spring Boot 匹配的版本 -->
>     </dependency>
> ```

### Model 层 (模型层)
职责​​：

* 定义实体类型

示例代码​​：

```java
@Entity
@Table(name = "users")
public class UserEntity
{
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String name;
    
    @Column(unique = true, nullable = false)
    private String email;
    
    // Getters/Setters
    ...
}
```

### DTO
DTO 是 Data Transfer Object (数据传输对象)的缩写。DTO用于在不同层之间传输数据，比如 Controller 方法的入参、 Controller 方法的返回值、 Service 方法的的入参、 Service 方法的的返回值等等。

### 全局异常处理
示例代码​​：

```java
@ControllerAdvice
public class GlobalExceptionHandler
{
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<?> handleNotFound(ResourceNotFoundException ex)
    {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(ex.getMessage());
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<?> handleValidationExceptions(MethodArgumentNotValidException ex)
    {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getAllErrors().forEach(error -> {
            String fieldName = ((FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });
        return ResponseEntity.badRequest().body(errors);
    }
}
```
