+++
date = '2026-04-16T22:17:09+08:00'
lastmod = '2026-04-17T19:39:16+08:00'
draft = false
title = 'Flutter 构建指南'
categories = ['Main Sections']
tags = ['Flutter']
+++

Flutter 是由 Google 开发的一款开源、高性能的 UI 软件开发工具包（SDK）。自 2017 年发布以来，它已经从一个简单的移动端框架进化成了支持移动端（Android/iOS）、 Web 、桌面（Windows/macOS/Linux）以及嵌入式设备的全平台解决方案。

Flutter 的优点有：

* 使用 Dart 编程语言，一个在执行模型和内存模型上为了 UI 渲染做了优化的语言。
* 使用自绘引擎（Impeller & Skia）。
* 最终产物体积较小。

本文是 Flutter 在各个平台的构建指南。 Host 是 Windows 。

## Windows

## Android

### 环境配置

#### Java

安装 Java ，配置环境变量 JAVA_HOME 。如果 Java 版本太新，可能也会报错“Java version xx or higher is required. To override this check set SKIP_JDK_VERSION_CHECK”。

#### Android-SDK

直接安装 Android Studio 比较省事，但是为此安装一个 IDE ，代价有点高。所以下面介绍纯命令行工具的安装。

首先下载 Android-SDK 。在下面的连接中，找到并下载“仅限命令行工具”。

{{<link title="下载 Android Studio 和应用工具 - Android 开发者 | Android Developers" link="https://developer.android.com/studio#command-tools" cover="auto">}}

然后解压到任意目录中，路径不要有空格。解压后结构应为：

```
cmdline-tools\
 ├── bin\
 │   ├── sdkmanager.bat
 │   ├── avdmanager.bat
 │   ├── ...
 ├── lib\
 │   ├── ...
```

配置环境变量 ANDROID_HOME="\<floder-path\>"。

尝试运行命令 `sdkmanager --list` 。如果报错：“Error: Could not determine SDK root. Error: Either specify it explicitly with --sdk_root= or move this package into its expected location: \<sdk\>\cmdline-tools\latest\”

就把目录结构改为：

```
cmdline-tools\
 ├── latest\
     ├── bin\
     │   ├── sdkmanager.bat
     │   ├── avdmanager.bat
     │   ├── ...
     ├── lib\
     │   ├── ...
```

默认情况下，安卓相关的东西，都会下载到 `用户文件夹\.android\` 文件夹下。

设置环境变量 ANDROID_USER_HOME="D:\\.user-home\\.android" ，之后安卓相关的构建缓存会在这里。

安装相关套件（以 Android 34 为例）：

```powershell
sdkmanager `
  "platform-tools" `
  "platforms;android-34" `
  "build-tools;34.0.0"
  "ndk;26.1.10909125"
```

也可以不安装，在构建 Flutter 项目的时候，会自动安装对应的套件。

### 修改项目构建设置

在新建一个 Flutter 项目后，有一个 android 文件夹。

#### 镜像配置

构建 Android 项目需要下载大量的依赖，所以要配置好镜像。

修改 ./android/gradle.properties 文件，使用操作系统证书库，防止“不信任证书”的错误：

```txt {name="./android/gradle.properties"}
systemProp.javax.net.ssl.trustStores=
systemProp.javax.net.ssl.trustStoreType=Windows-ROOT
```

---

第一次运行要下载 Gradle 。

默认情况下， Gradle 会自动下载到 `用户文件夹\.gradle\` 文件夹下。

设置环境变量 GRADLE_USER_HOME="D:\\.user-home\\.gradle" ，之后下载的 Gradle 会安装到这里。

{{< tabs >}}

<!-- tab 从镜像源安装 Gradle -->

```txt {name="./android/gradle/wrapper/gradle-wrapper.properties"}
distributionUrl=https://mirrors.aliyun.com/macports/distfiles/gradle/gradle-x.x.x-all.zip
```

<!-- tab 从本地安装 Gradle -->

```txt {name="./android/gradle/wrapper/gradle-wrapper.properties"}
distributionUrl=file:///C:/file/path/gradle-x.x.x-all.zip
```

{{</ tabs >}}

---

构建需要下载 maven 包。

{{< tabs >}}

<!-- tab 从镜像源下载 maven 包 -->

```txt {name="./android/gradle.properties"}
......
repositories.grails.default = https://maven.aliyun.com/repository/public
```

<!-- tab 通过代理下载 maven 包 -->

```txt {name="./android/gradle.properties"}
......
systemProp.http.proxyHost=proxy.example.com
systemProp.http.proxyPort=8080
systemProp.https.proxyHost=proxy.example.com
systemProp.https.proxyPort=8080
systemProp.http.proxyUser=username
systemProp.http.proxyPassword=password
```

{{</ tabs >}}

---

设置 Gradle 仓库源：

```Kotlin {name="./android/settings.gradle.kts",hl_lines=["3-5", "12-24"]}
pluginManagement {
    repositories {
        maven { url = uri("https://maven.aliyun.com/repository/public") }
        maven { url = uri("https://maven.aliyun.com/repository/google") }
        maven { url = uri("https://maven.aliyun.com/repository/gradle-plugin") }
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}

dependencyResolutionManagement {
    // 关键：强制优先使用这里的配置
    repositoriesMode.set(RepositoriesMode.PREFER_SETTINGS)
    repositories {
        maven { url = uri("https://storage.flutter-io.cn/download.flutter.io") }
        maven { url = uri("https://maven.aliyun.com/repository/google") }
        maven { url = uri("https://maven.aliyun.com/repository/central") }
        maven { url = uri("https://maven.aliyun.com/repository/public") }
        maven { url = uri("https://maven.aliyun.com/repository/gradle-plugin") }
        google()
        mavenCentral()
    }
}
```

#### APK 签名配置

生成数字签名（Keystore）：

```shell
keytool -genkey -v -keystore your-path\\your-keystore.jks -storetype JKS -keyalg RSA -keysize 4096 -validity 10000 -alias your-alias
```

> [!info]
> `keytool` 由 Java 提供。

在 ./android/ 目录下新建 `key.properties` 文件：

```txt {name="./android/key.properties"}
storePassword=你的密码
keyPassword=你的密码
keyAlias=your-alias
storeFile=your-path\\your-keystore.jks
```

> [!warn]
> 不要把 `./android/key.properties` 文件提交到代码库中。

修改 ./android/app/build.gradle.kts ：

```Kotlin {name="./android/app/build.gradle.kts",hl_lines=["1-8", "18-20", "25-29", "35-39"]}
import java.util.Properties
import java.io.FileInputStream

val keystoreProperties = Properties()
val keystorePropertiesFile = rootProject.file("key.properties")
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(keystorePropertiesFile.inputStream())
}

android {
    // ...
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_21
        targetCompatibility = JavaVersion.VERSION_21
    }

    kotlinOptions {
        // 原本是 jvmTarget = JavaVersion.VERSION_17.toString()
        // java 版本要和上面的内容对应
        jvmTarget = "21"
    }

    signingConfigs {
        create("release") {
            keyAlias = keystoreProperties["keyAlias"] as? String ?: ""
            keyPassword = keystoreProperties["keyPassword"] as? String ?: ""
            val storeFilePath = keystoreProperties["storeFile"] as? String
            storeFile = if (storeFilePath != null) file(storeFilePath) else null
            storePassword = keystoreProperties["storePassword"] as? String ?: ""
        }
    }

    buildTypes {
        release {
            signingConfig = signingConfigs.getByName("release")
            // 启用 R8 混淆
            isMinifyEnabled = true
            isShrinkResources = true
            proguardFiles(getDefaultProguardFile("proguard-android-optimize.txt"), "proguard-rules.pro")
        }
    }
}
```

### 构建

使用命令构建：

```shell
flutter build apk --release --split-per-abi -v
```

> [!info]
> `--split-per-abi` 是指根据不同的 CPU 架构（ABI）拆分 APK。

> [!info]
> `-v` 是指打印出所有底层日志。
