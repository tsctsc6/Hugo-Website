+++
date = '2025-06-04T19:11:56+08:00'
lastmod = '2025-11-26T12:57:56+08:00'
draft = false
title = '构建 csproj 项目的笔记'
categories = ['Main Sections']
tags = ['.NET', 'Build']
+++

平时我们在简单编写代码的时候，使用 IDE 生成项目。但是在 CI/CD 流程中，就要使用命令行构建和发布项目了。

## 构建与发布

一般来说，构建命令如下：

```shell
dotnet build .csproj --runtime win-x64 --configuration Debug
```

发布命令如下：

```shell
dotnet publish .csproj --runtime win-x64 --configuration Release --output .\output-dir
```

当然，还有其他参数，具体可以参考[微软官方文档](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet)，或者使用 `-h` 查看。

## 安装依赖

有时，项目需要依赖其他 SDK ，比如发布安卓应用程序，就需要 Android SDK 和 JDK ，可以使用以下命令安装依赖：

```shell
dotnet workload restore .csproj
```

## MSBuild 属性

MSBuild 属性，包括发布 AOT ，发布单个文件，发布 ReadyToRun 等等。 MSBuild 属性可以在两个地方指定：

- .csproj 文件的 `<PropertyGroup>` 标签中。
- 发布命令的 `-p:` 参数中。优先级比 .csproj 文件高。

具体有哪些属性，可以查看微软官方文档。

{{<link title="MSBuild 属性" link="https://learn.microsoft.com/zh-cn/visualstudio/msbuild/msbuild-properties" cover="auto">}}

在命令行中，指定 MSBuild 属性的形式为：

```
dotnet publish .csproj -p:key0=value0 -p:key1=value1
```

> [!info]
> 除了 \.NET 官方提供的 MSBuild 属性以外，第三方库也可以定义自己的 MSBuild 属性。

### 常用的 MSBuild 属性

|             MSBuild 属性             |                                         含义                                          | 类型 |
| :----------------------------------: | :-----------------------------------------------------------------------------------: | :--: |
|          PublishReadyToRun           |                                  ReadyToRun 编译模式                                  | bool |
|          PublishSingleFile           |                                      单文件发布                                       | bool |
| IncludeNativeLibrariesForSelfExtract |                             把原生的 dll 也包含进单文件中                             | bool |
|   IncludeAllContentForSelfExtract    | 控制是否将所有内容文件（Content Files）打包到单文件中，并在运行时自动提取到临时目录中 | bool |
|              DebugType               |                      生成 PDB 文件的选项（具体查看微软官方文档）                      | 枚举 |
|             DebugSymbols             |                                   是否生成 PDB 文件                                   | bool |

### 例子

指定单文件发布，以下两种方式等效，编译命令优先级更高：

{{< tabs >}}

<!-- tab 在 .csproj 中指定 -->

```xml
<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
        <PublishSingleFile>true</PublishSingleFile>
    </PropertyGroup>
</Project>
```

<!-- tab 编译命令中指定 -->

```powershell
dotnet publish -p:PublishSingleFile=true
```

{{</ tabs >}}

***

此外，在 IDE Rider 中，在设置 => 构建、执行、部署 => 工具包和构建中，设置 MSBuild 全局属性。
