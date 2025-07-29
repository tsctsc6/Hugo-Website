+++
date = '2025-07-22T15:58:51+08:00'
draft = false
title = 'Java 语言架构介绍'
categories = ['Sub Sections']
tags = ['Java']
+++

## 编译器
Java 的编译器，包含在 JDK(Java Development Kit) 中。 Java 由 Sun 公司于 1995 年推出，当时 JDK 的实现并没有开源。后来在开源社区的呼声下，部分开源了。后来 Oracle 收购了 Sun 公司后，确立 OpenJDK 项目，其他公司都可以自主实现 JDK 。目前有不少 OpenJDK ，比如 Oracle 的、微软的、华为的等等。

### Java 与 JDK 的版本命名
最开始，版本的命名是 Java 1.0 / JDK 1.0 ， Java 1.1  / JDK 1.1 ， Java 1.2  / JDK 1.2 ...更新的频率是两到三年一次。

直到 2017 年，Java 9 / JDK 9 发布，不但更改了版本命名方式，更新频率也加快到约半年一次。Java 9 的上一个版本， Java 1.8 也被称为 Java 8 .

目前，Java 的长期支持版本(LTS) 有：

* JDK 8：发布于 2014 年 3 月，支持至 2030 年，适合传统企业应用。
* JDK 11：发布于 2018 年 9 月，支持至 2032 年，广泛用于现代化开发。
* JDK 17：发布于 2021 年 9 月，支持至 2029 年，提供性能优化和新语言特性。
* JDK 21：发布于 2023 年 9 月，支持至 2031 年，引入虚拟线程和结构化并发等新特性。

### 在 Windows 上安装 Microsoft.OpenJDK
在终端中输入:

```shell
winget search Microsoft.OpenJDK
```

然后选择喜欢的版本安装，例如:

```shell
winget install Microsoft.OpenJDK.21
```

## IDE
推荐使用 JetBrains IntelliJ IDEA ，通过 JetBrains Toolbox 安装。

## 构建工具
目前 Java 的常用构建工具有两个： Maven 和 Gradle 。本文主要介绍 Maven 的使用方法。

IDEA 自带 Meven ，位于 `{IDEA 安装目录}\plugins\maven\lib\maven3\` 中，可执行文件是 `{IDEA 安装目录}\plugins\maven\lib\maven3\bin\mvn.cmd` 。

### 配置镜像源
对于中国大陆地区的程序员来说，经常遇到无法下载（或下载速度慢）依赖或第三方软件包的现象。解决这一现象的办法是，配置镜像源。

对于 IDEA 自带的 Maven ，需要修改 Maven 的 `conf\settings.xml` 文件，添加阿里云镜像：

```XML {name="IDEA 安装目录\plugins\maven\lib\maven3\confsettings.xml"}
<settings>
    <mirrors>
        <mirror>
            <id>aliyunmaven</id>
            <mirrorOf>central</mirrorOf>
            <name>阿里云公共仓库</name>
            <url>https://maven.aliyun.com/repository/public</url>
        </mirror>
    </mirrors>
</settings>
```

对于以其他方式安装的 Maven ，只需要找到 settings.xml 文件，修改即可。

### Maven 结构介绍
项目根目录的 `pom.xml` 文件是 Maven 项目核心配置文件，里面包含了项目名称、项目版本、项目依赖项等信息。

## 部署
JDK 会把 Java 代码编译为字节码，一个 `.java` 文件编译为一个 `.class` 文件。可以把整个项目打包为 `jar` 格式， `jar` 其实是一个压缩包，里面包含了一些 `.class` 文件以及其他配置文件。 `jar` 又分为包含依赖库的和不包含依赖库的。下面介绍包含依赖库的打包方式。

首先在 `pom.xml` 中加入:

```XML {name="pom.xml"}
<project>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>3.6.0</version>
                <configuration>
                    <!-- 指定主类 -->
                    <archive>
                        <manifest>
                            <mainClass>org.example.Main</mainClass>
                        </manifest>
                    </archive>
                    <!-- 打包所有依赖 -->
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase> <!-- 绑定到package阶段 -->
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

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
