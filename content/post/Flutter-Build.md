+++
date = '2026-04-16T22:17:09+08:00'
lastmod = '2026-04-23T20:10:58+08:00'
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

Flutter 官方提供了 msix 的安装包打包方式，以及 Portable 的方式。这里讲一下使用 wix 打包为 msi 安装包的方式。

{{<link title="Building Windows apps with Flutter" link="https://docs.flutter.dev/platform-integration/windows/building" cover="auto">}}

根据上面的文章，发现 Flutter 需要打包进 msi 安装包的内容有 .exe ，若干 .dll 文件，以及一个 data 文件夹。 Flutter 应用程序需要依赖 Visual C++ Redistributable (VCRT) ，为了程序可以随意安装，把 VCRT 直接打包（msvcp140.dll, vcruntime140.dll, vcruntime140_1.dll）。

### 安装 wix

现在是 2026 年， wix 已经发布到 v7 大版本，是基于 .NET 来运行的。首先，安装 .NET SDK 10:

{{<link title="Download .NET 10.0 (Linux, macOS, and Windows) | .NET" link="https://dotnet.microsoft.com/en-us/download/dotnet/10.0" cover="auto">}}

然后使用以下命令安装 wix v7:

```shell
dotnet tool install --global wix
```

同意 eula 协议：

```shell
wix.exe eula accept wix7
```

### 复制 VCRT 的 dll

可以使用以下 Powershell 脚本复制 VCRT 的 dll 。

```powershell
$system32Path = "C:\Windows\System32"
$dlls = @("msvcp140.dll", "vcruntime140.dll", "vcruntime140_1.dll")
foreach ($dll in $dlls) {
    $source = Get-ChildItem -Path $system32Path -Filter $dll -Recurse | Select-Object -First 1
    if ($source) {
        Copy-Item $source.FullName -Destination "./" -Force
        Write-Host "Copied: $($source.FullName)"
    } else {
        Write-Error "$dll not found"
    }
}
```

### 编写 .wxs 文件

```xml {name="your-app-name.wxs"}
<Wix xmlns="http://wixtoolset.org/schemas/v4/wxs" 
     xmlns:ui="http://wixtoolset.org/schemas/v4/wxs/ui">

    <Package Name="your-app-name"
        Manufacturer="your-name" 
        Version="1.0.0"
        UpgradeCode="your-guid"
        Scope="perUser">

        <MediaTemplate EmbedCab="yes" />
    
        <ui:WixUI Id="WixUI_FeatureTree" InstallDirectory="INSTALLFOLDER" />

        <StandardDirectory Id="LocalAppDataFolder">
            <Directory Id="INSTALLFOLDER" Name="flutter_rust_app">
                <Directory Id="DataInstallDir" Name="data" />
            </Directory>
        </StandardDirectory>

        <StandardDirectory Id="ProgramMenuFolder">
            <Directory Id="InstallProgramMenuFolder" Name="flutter_rust_app" />
        </StandardDirectory>
        <StandardDirectory Id="DesktopFolder" />

        <Feature Id="Main" Title="Application" Description="Install application binaries"
            Display="expand" ConfigurableDirectory="INSTALLFOLDER"
            AllowAbsent="no">
            <ComponentGroupRef Id="AppBinaries" />
            <ComponentGroupRef Id="DataFiles" />

            <Feature Id="StartMenuShortcut" Title="Start Menu Shortcut" Level="1">
                <ComponentGroupRef Id="StartMenuShortcutGroup" />
            </Feature>
            <Feature Id="DesktopShortcut" Title="Desktop Shortcut" Level="1">
                <ComponentGroupRef Id="DesktopShortcutGroup" />
            </Feature>
        </Feature>
    </Package>

    <Fragment>
        <!-- Define the components for the application binaries -->
        <ComponentGroup Id="AppBinaries" Directory="INSTALLFOLDER">
            <Component>
                <File Id="MainExecutable" Source="flutter_rust_app.exe" KeyPath="yes" />
            </Component>
            <Component>
                <File Source="flutter_windows.dll" KeyPath="yes" />
            </Component>
            <Component>
                <File Source="rust_lib_flutter_rust_app.dll" KeyPath="yes" />
            </Component>
            <Component>
                <File Source="msvcp140.dll" KeyPath="yes" />
            </Component>
            <Component>
                <File Source="vcruntime140.dll" KeyPath="yes" />
            </Component>
            <Component>
                <File Source="vcruntime140_1.dll" KeyPath="yes" />
            </Component>
        </ComponentGroup>
        <ComponentGroup Id="DataFiles" Directory="DataInstallDir">
            <Files Include="data\**" />
        </ComponentGroup>

        <!-- Create startmenu shortcut -->
        <ComponentGroup Id="StartMenuShortcutGroup" Directory="InstallProgramMenuFolder">
            <Component Id="StartMenuShortcutComponent" Guid="*">
                <Shortcut Name="flutter_rust_app" 
                          Description="Launch flutter_rust_app"
                          Target="[!MainExecutable]" 
                          WorkingDirectory="INSTALLFOLDER" />
                <RemoveFolder Id="CleanProgramMenu" On="uninstall" />
                <RegistryValue Root="HKCU" Key="Software\tsctsc6\flutter_rust_app" Name="StartMenuShortcut" Type="integer" Value="1" KeyPath="yes" />
            </Component>
        </ComponentGroup>
        <!-- Create desktop shortcut -->
        <ComponentGroup Id="DesktopShortcutGroup" Directory="DesktopFolder">
            <Component Id="DesktopShortcutComponent" Guid="*">
                <Shortcut Name="flutter_rust_app" 
                          Description="Launch flutter_rust_app"
                          Target="[!MainExecutable]" 
                          WorkingDirectory="INSTALLFOLDER" />
                <RegistryValue Root="HKCU" Key="Software\tsctsc6\flutter_rust_app" Name="DesktopShortcut" Type="integer" Value="1" KeyPath="yes" />
            </Component>
        </ComponentGroup>
    </Fragment>
</Wix>
```

### 打包

首先构建 Flutter 应用程序：

```shell
flutter build windows --release -v
```

把写好的 wxs 移动到输出文件夹下(./build/windows/x64/runner/Release)，复制 VCRT 的 dll 。

添加相关扩展：

```shell
wix extension add WixToolset.UI.wixext
```

打包：

```shell
wix build your-app-name.wxs -ext WixToolset.UI.wixext
```

> [!info]
> 默认输出的 msi 的名称和 .wxs 文件的名称相同。

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

把 "\<floder-path\>/cmdline-tools/latest/bin" 加入到环境变量 path 中。

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

若是想移除套件，可以先使用 `sdkmanager --list` 查看已经安装的套件，再使用 `sdkmanager --uninstall "platforms;android-xx"` 移除套件。

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

```Kotlin {name="./android/settings.gradle.kts", hl_lines=["3-5", "12-24"]}
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

修改 Java 版本：

```Kotlin {name="./android/app/build.gradle.kts", hl_lines=["4-5", "9-11"]}
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
}
```

#### flutter_rust_bridge 的配置

使用 flutter_rust_bridge 创建项目后，会有 /rust_builder/android 文件夹。为了使构建工具版本的统一，以及配置镜像源，修改以下文件：

```gradle {name="./rust_builder/android/build.gradle", hl_lines=["3-7", "15-19", "30-40"]}
buildscript {
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

rootProject.allprojects {
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

android {
    if (project.android.hasProperty("namespace")) {
        namespace 'com.flutter_rust_bridge.rust_lib_flutter_rust_app'
    }

    compileSdk = flutter.compileSdkVersion
    ndkVersion = flutter.ndkVersion

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_21
        targetCompatibility = JavaVersion.VERSION_21
    }

    defaultConfig {
        minSdkVersion flutter.minSdkVersion
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

```Kotlin {name="./android/app/build.gradle.kts", hl_lines=["1-8", "13-17", "23-27"]}
import java.util.Properties
import java.io.FileInputStream

val keystoreProperties = Properties()
val keystorePropertiesFile = rootProject.file("key.properties")
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(keystorePropertiesFile.inputStream())
}

android {
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
