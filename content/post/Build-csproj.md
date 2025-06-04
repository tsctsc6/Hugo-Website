+++
date = '2025-06-04T19:11:56+08:00'
draft = false
title = '构建 csproj 项目的笔记'
categories = ['Main Sections']
tags = ['Build']
+++

平时我们在简单编写代码的时候，使用 IDE 生成项目。但是在 CI/CD 流程中，就要使用命令行构建和发布项目了。

# 构建与发布
一般来说，构建命令如下：

```shell
dotnet build .csproj --runtime win-x64 --configuration Debug
```

发布命令如下：

```shell
dotnet publish .csproj --runtime win-x64 --configuration Release --output .\output-dir
```

当然，还有其他参数，具体可以参考[微软官方文档](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet)，或者使用 `-h` 查看。

# 安装依赖
有时，项目需要依赖其他 SDK ，比如发布安卓应用程序，就需要 Android SDK 和 JDK ，可以使用以下命令安装依赖：

```shell
dotnet workload restore .csproj
```

# MSBuild 参数
MSBuild 的参数，包括发布 AOT ，发布单个文件，发布 ReadyToRun 等等。 MSBuild 的参数可以写在 .csproj 文件的 `<PropertyGroup>` 标签中。

具体有哪些属性，可以查看[微软官方文档](https://learn.microsoft.com/zh-cn/visualstudio/msbuild/msbuild-properties)。

在命令行中，传递 MSBuild 参数的形式为：

```
dotnet publish .csproj -p:key0=value0 -p:key1=value1
```

在 IDE Rider 中，在设置 => 构建、执行、部署 => 工具包和构建中，设置 MSBuild 全局属性。
