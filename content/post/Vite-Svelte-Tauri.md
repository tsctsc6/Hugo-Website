+++
date = '2025-06-08T19:30:28+08:00'
lastmod = '2025-12-09T16:37:10+08:00'
draft = false
title = '使用 Vite-Svelte-Tauri 开发前端应用'
categories = ['Main Sections']
tags = ['前端', 'Vite', 'Svelte', 'Tauri']
+++

## 概念介绍

### 前端应用开发

前端是指，在服务端-客户端架构中，运行在客户端的程序。

我们知道，使用 HTML5 ， CSS3 ，以及 JavaScript 可以在浏览器中渲染出页面，它们也被称为“前端三件套”。简单来说， HTML 定义页面结构， CSS 定义页面的样式 ， JavaScript 负责代码逻辑。

在以前， JavaScript 只能在浏览器环境中运行。后来有人开发出了 Node.js ，它独立于浏览器，提供了 JavaScript 的运行环境。再后来又有人提出，可以基于前端三件套（ Web 技术），开发各个平台的 GUI —— 毕竟前端三件套就是为渲染用户界面而生。

### Vite

Vite 是一个现代化的**前端构建工具**，旨在为开发者提供极致的开发体验和高效的构建输出。主要特点有：

1. 快速启动开发服务器
1. 快速热更新

### 前端 UI 框架

Vite 支持各种前端 UI 框架，包括 Vue 、 React 、 Preact 、 Svelte 等等。它们专注于构建用户界面。

### 打包工具

打包工具，把基于 Web 技术的代码，打包为各个操作系统的应用程序。比如：

1. Electorn 和 Wails ，支持桌面端 (Windows, macOS, Linux) 的打包。
1. Capacitor 和 UniApp ，支持移动端 (Android, iOS) 的打包。
1. Tauri ，支持桌面端和移动端的打包。

本文以 Vite + Svelte + Tauri 为例，记录开发环境的搭建以及发布应用的流程。

## 环境

- 操作系统：Windows 11 24H2
- Node.js
- rust（使用 [rustup](https://www.rust-lang.org/zh-CN/tools/install) 安装 toolchain: x86_64-pc-windows-msvc ）
- msvc 套件（使用 [Visual Studio 2022 生成工具](https://visualstudio.microsoft.com/zh-hans/downloads/)，安装 C++ 桌面开发）
- IDE: JetBrains WebStorm

### 安卓环境配置

如果要使用 Tauri 打包安卓应用，还需要安装一系列依赖，非常复杂。

#### rust toolchains:

- aarch64-linux-android
- armv7-linux-androideabi
- i686-linux-android
- x86_64-linux-android

在调式或构建安卓时，会自动安装。

#### Java

配置环境变量 JAVA_HOME 。如果 Java 版本太新，可能也会报错“Java version xx or higher is required. To override this check set SKIP_JDK_VERSION_CHECK”。

#### Android-SDK

直接安装 Android Studio 比较省事，但是为此安装一个 IDE ，代价有点高。所以下面介绍纯命令行工具的安装。

首先下载 Android-SDK 。在下面的连接中，找到并下载“仅限命令行工具”。

{{<link title="下载 Android Studio 和应用工具 - Android 开发者 | Android Developers" link="https://developer.android.com/studio#command-tools" cover="auto">}}

然后解压到任意目录中。解压后结构应为：

```
cmdline-tools\
 ├── bin\
 │   ├── sdkmanager.bat
 │   ├── avdmanager.bat
 │   ├── ...
 ├── lib\
 │   ├── ...
```

配置环境变量 ANDROID_HOME="<floder-path>\cmdline-tools\bin"。

尝试运行命令 `sdkmanager --list` 。如果报错：“Error: Could not determine SDK root. Error: Either specify it explicitly with --sdk_root= or move this package into its expected location: <sdk>\cmdline-tools\latest\”

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

#### AVD (可选)

设置环境变量 ANDROID_AVD_HOME="D:\\.user-home\\.android\\.avd" 。要和环境变量 ANDROID_USER_HOME 对应。

创建一个 AVD (Android Virtual Devices)，以供测试。

下载模拟器：

```
sdkmanager "emulator"
sdkmanager "system-images;android-34;google_apis;x86_64"
```

创建模拟器：

```
avdmanager create avd -n test -k "system-images;android-34;google_apis;x86_64"
```

查看模拟器列表：

```
avdmanager list avd
```

启动模拟器：

```
emulator -avd <name>
```

## 初始化项目

### Vite

输入一下命令创建项目：

```shell
npm create vite@latest
```

根据命令行的引导，创建项目。

创建项目后，安装依赖：

```shell
npm install
```

### Tauri Desktop

安装 Tauri ：

```shell
npm install @tauri-apps/cli @tauri-apps/api
```

初始化 Tauri Desktop 项目：

```shell
npx tauri init
```

根据提示填写配置信息，完成命令后，项目根目录会出现 src-tauri 文件夹。

如果想更改刚刚的配置，可以在 src-tauri/tauri.conf.json 中更改。

### Tauri Android

初始化 Tauri Android 项目：

```shell
npx tauri android init
```

完成命令后，会出现 src-tauri/gen/android 文件夹，是安卓项目的工程文件。

若是“不信任证书”的错误，对此，要修改 ./src-tauri/gen/android/gradle.properties 文件，使用操作系统证书库：

```txt {name="./src-tauri/gen/android/gradle.properties"}
systemProp.javax.net.ssl.trustStores=
systemProp.javax.net.ssl.trustStoreType=Windows-ROOT
```

---

第一次运行要下载 Gradle 。

默认情况下， Gradle 会自动下载到 `用户文件夹\.gradle\` 文件夹下。

设置环境变量 GRADLE_USER_HOME="D:\\.user-home\\.gradle" ，之后下载的 Gradle 会安装到这里。

{{< tabs >}}

<!-- tab 从镜像源安装 Gradle -->

```txt {name="./src-tauri/gen/android/gradle/wrapper/gradle-wrapper.properties"}
distributionUrl=https://mirrors.aliyun.com/macports/distfiles/gradle/gradle-x.x.x-all.zip
```

<!-- tab 从本地安装 Gradle -->

```txt {name="./src-tauri/gen/android/gradle/wrapper/gradle-wrapper.properties"}
distributionUrl=file:///C:/file/path/gradle-x.x.x-all.zip
```

{{</ tabs >}}

---

构建需要下载 maven 包。

{{< tabs >}}

<!-- tab 从镜像源下载 maven 包 -->

```txt {name="./src-tauri/gen/android/gradle.properties"}
......
repositories.grails.default = https://mirrors.ustc.edu.cn/maven/
repositories.grails.default.1 = https://mirrors.tuna.tsinghua.edu.cn/maven/repos/public
repositories.grails.default.2 = https://maven.aliyun.com/repository/public
```

<!-- tab 通过代理下载 maven 包 -->

```txt {name="./src-tauri/gen/android/gradle.properties"}
......
systemProp.http.proxyHost=proxy.example.com
systemProp.http.proxyPort=8080
systemProp.https.proxyHost=proxy.example.com
systemProp.https.proxyPort=8080
systemProp.http.proxyUser=username
systemProp.http.proxyPassword=password
```

{{</ tabs >}}

### 添加脚本

（**非常重要**）：在项目根目录下的 package.json 文件夹下，添加：

```json {name="package.json"}
{
    "scripts": {
        ...
        "tauri": "tauri",
        ...
    }
}

```

否则会出现奇怪的问题。

## 调试

写完代码后，可以对不同的平台进行调试。

- web 端: `npm run dev`
- 桌面端: `npm run tauri dev`
- 安卓端: `npm run tauri android dev` (请先连接虚拟安卓机或物理安卓机)

### 桌面端调试与移动端调试的区别

调试背后的运作原理是， Vite 先启动一个后端，前端从后端获取 HTML, CSS, JS ，然后渲染页面。对于 web 端，前端就是浏览器；对于桌面端，前端就是一个进程；对于安卓端，前端就是运行在安卓操作系统上的应用。在构建生产环境的应用程序时，会把相关的 HTML, CSS, JS 打包进应用程序安装包中。在生产环境中，无需启动后端。

我们观察 ./src-tauri/tauri.conf.json ：

```json {name="./src-tauri/tauri.conf.json"}
{
    "build": {
        "devUrl": "http://localhost:5173"
    },
    ......
}
```

发现它设置了访问后端的 url 。

然而，在进行安卓端的调试时，观察命令行输出，发现它在访问 http://192.168.55.26:5173 。这是因为安卓的虚拟机或者物理机无法访问 localhost ——因为已经是不同的机器了，所以要通过局域网地址访问后端。那么如何在运行桌面端的时候，后端监听 loaclhost ，运行移动端的时候，后端监听局域网呢？

后端是由 Vite 构建的，启动的代码在 ./vite.config.ts 中（这里使用 TypeScript 项目）。

tauri 在启动时，会设置一系列环境变量，其中就有 `TAURI_ENV_PLATFORM` ，表明当前运行的平台。通过这个环境变量，判断当前是否是移动平台，监听对应的 ip 地址。

我们通过 .env.local 文件设置局域网 ip ，避免把局域网 ip 暴露到公共代码库中。 .env.local 存储一系列键值对，在启动程序是加载，模拟环境变量，是一种灵活的机密管理方式。

修改 ./vite.config.ts 文件如下：

```TypeScript {name="./vite.config.ts"}
import {defineConfig, loadEnv} from 'vite'
import {svelte} from '@sveltejs/vite-plugin-svelte'

// https://vite.dev/config/
// @ts-ignore
export default defineConfig(({mode}) => {
  console.log('TAURI_ENV_PLATFORM:', process.env.TAURI_ENV_PLATFORM);
  const envFile = loadEnv(mode, process.cwd(), '');
  let host = 'localhost';
  if (process.env.TAURI_ENV_PLATFORM === undefined ||
      process.env.TAURI_ENV_PLATFORM === 'windows' ||
      process.env.TAURI_ENV_PLATFORM === 'linux' ||
      process.env.TAURI_ENV_PLATFORM === 'macOS') {}
  else{
    host = envFile.VITE_Mobile_HOST;
  }
  return {
    plugins: [svelte()],
    server: {
      host: host,
      port: envFile.VITE_PORT,
      strictPort: true
    }
  };
});

```

## 构建

在构建之前，先在 ./src-tauri/tauri.conf.json 中，进行如下配置：

```json {name="./src-tauri/tauri.conf.json"}
{
    "identifier": "com.xxx.xxx",
    ...
    "build": {
        "frontendDist": "../dist",
        ...
    },
    ...
}
```

- `identifier` 不能是默认的，需要自己设置。
- `frontendDist` 是指打包好的前端界面的文件（HTML, CSS, js）， Vite 会把这些文件生成在 ./dist 目录下。

### Desktop 构建

Tauri 支持两种 Windows 的构建方式， .msi 安装包（使用 Wix 打包）和 .exe 安装包（使用 NSIS 打包）。本文主要介绍 .msi 方式。也可以打包为 macOS 和 Linux 平台的安装包，详细请看官方文档。

在 ./src-tauri/tauri.conf.json 中，进行如下配置：

```json {name="./src-tauri/tauri.conf.json"}
{
    ...
    "bundle": {
        "targets": [
            "msi"
        ],
        "windows": {
            "wix": {
                "upgradeCode": "833ad97c-0000-47cb-0000-2e71181c0584"
            }
        }
        ...
    },
    ...
}
```

- `targets` 是指打包为何种安装包。
- `upgradeCode` ，MSI 安装程序的 GUID 升级代码。此代码在您的所有更新中必须保持不变，否则，Windows 会将您的更新视为不同的应用程序，并且您的用户将拥有应用程序的重复版本。默认情况下， tauri 通过使用字符串 `&lt;productName&gt;.exe.app.x64` 在 DNS 命名空间中生成 Uuid v5 来生成此代码。您可以使用 Tauri 的 CLI 为您生成和打印此代码，运行 `tauri inspect wix-upgrade-code`。建议您在 tauri 配置文件中设置此值，以避免在您想要更改产品名称时意外更改升级代码。

Windows x64 构建指令：

```shell
npx tauri build --target x86_64-pc-windows-msvc
```

构建完成后， .msi 安装包在 ./src-tauri/target/x86_64-pc-windows-msvc/release/bundle/msi 目录中。

#### 网络问题导致的构建失败

第一次构建，会下载打包安装包要用到的工具，比如 Wix 。

在“网络不好”的情况下，下载工具时会遇到证书错误（翻墙有时也不行）。

解决办法是，通过临时环境变量设置代理：

{{< tabs >}}

<!-- tab Shell -->

```shell
export HTTP_PROXY="socks5://127.0.0.1:10808"
export HTTPS_PROXY="socks5://127.0.0.1:10808"
export NO_PROXY="127.0.0.1,::1"
```

<!-- tab PowerShell -->

```powershell
$Env:HTTP_PROXY = "socks5://127.0.0.1:10808"
$Env:HTTPS_PROXY = "socks5://127.0.0.1:10808"
$Env:NO_PROXY = "127.0.0.1,::1"
```

{{</ tabs >}}

> [!warning]
> http 协议是不被支持的。

另外一个办法是使用诸如 Github Action 的 CI/CD 工具，在其他地方构建。

### Android 构建

构建命令：

```shell
npx tauri android build
```

构建完成后，会生成 apk 和 aab 文件，分别在 ./src-tauri/gen/android/app/build/outputs/apk/universal/release/ 目录和 ./src-tauri/gen/android/app/build/outputs/aab/universal/release/ 目录中。

#### 对 apk 签名

生成的 apk 文件名称为 app-universal-release-unsigned.apk ，表明生成的 apk 是没有签名的，无法安装到生产环境中。

要对 apk 签名，参考官方文档 [https://v2.tauri.app/zh-cn/distribute/sign/android/](https://v2.tauri.app/zh-cn/distribute/sign/android/) 。

已经签名的 apk ，文件名中没有“unsigned”。

#### 精简架构（可选）

刚刚的构建命令，默认是多架构的，即安装包中包含多个架构的指令。如果要减少安装包大小，可以精简架构。

使用 Android-SDK 中的 aapt.exe 可以查看安装包包含的架构：

```shell
aapt dump badging your.apk
```

可以看到：

```shell {name="shell output"}
native-code: 'arm64-v8a' 'armeabi-v7a' 'x86' 'x86_64'
```

目前绝大多数手机是 `arm64-v8a` 架构。

查看帮助信息：

```shell
npx tauri android build --help
```

可以看到：

```shell {name="shell output"}
......
  -t, --target [<TARGETS>...]
          Which targets to build (all by default)

          [possible values: aarch64, armv7, i686, x86_64]
......
```

于是，构建 `arm64-v8a` 架构 apk ：

```shell
npx tauri android build --target aarch64
```

## 自动化构建

可以访问 [https://github.com/tsctsc6/sample-svelte](https://github.com/tsctsc6/sample-svelte) ，里面包含 Github Action 的构建代码。
