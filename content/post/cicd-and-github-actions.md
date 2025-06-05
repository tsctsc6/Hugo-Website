+++
date = '2025-05-30T22:03:15+08:00'
draft = false
title = 'CI/CD 及其实践'
categories = ['Main Sections']
tags = ['CI/CD', 'Git', 'Github', 'Github Actions']
+++

# 什么是 CI/CD?
CI/CD 是持续集成 (Continuous Integration) 和持续交付/部署 (Continuous Delivery/Deployment) 的缩写，旨在通过自动化的流程和工具，提高软件开发的效率、质量和交付速度。

## 持续集成
持续集成是一种面向开发人员的自动化流程，有助于更频繁地将代码更改合并回共享分支或“主干”。

现代应用开发的目标是让多位开发人员同时处理同一应用的不同功能。如果项目组手动地将不同的代码合并，会非常繁琐和耗时。此外，如果代码出现问题，还要手动地回滚代码，非常繁琐和耗时。

通常，人们使用版本控制系统(Version Control System, VCS)来实现持续集成。最常见的 VCS 是 Git ，代码托管平台 Github 就是基于 Git 的。

## 持续交付/部署
持续交付是指自动执行 CI 中的构建、单元测试和集成测试后，自动将经过验证的代码发布到存储库。

持续部署是持续交付的延伸，可以指自动将开发人员的更改从存储库发布到生产环境，以供客户使用。

GitHub Action 是一个实现 CI/CD 的自动化流程工具，是由 GitHub 提供的一个组件。你可以通过 YAML 文件的配置定义工作流程以构建执行 CI/CD 流水线，并可以触发不同事件时(如代码提交, Pull Request, schedule)自动执行这些工作流程。

# CI/CD 在 Github 平台的实现
## 使用 Github 作为代码托管平台
在 Github 的网页端中，手动创建仓库，根据引导，在本地创建仓库并推送。

或者 IDE 可以自动帮你完成。

## 使用 Git 进行版本控制
使用 Git 作为版本控制系统时，持续集成中所谓的“共享分支”，一般是 main 分支或者 master 分支。如果开发者要开发新功能，改 bug ，需要先从 main 分支拉取一个新分支，如 feat-xx 分支或 fix-xx 分支，在 feat-xx 分支或 fix-xx 分支中修改代码，再合并到 main 分支；或者通过 Pull Request 合并到 main 分支。

> 请自行学习 Git 的基本操作。

GitHub 的 Pull Request 是 GitHub 平台上用于代码审查和协作的核心功能。它本质上是开发者向项目维护者（或其他协作者）发出一个请求，请求对方将自己分支上的代码更改拉取并合并到主分支（或其他目标分支）。也就是说， Github 的 Pull Request 和普通的 Git 分支合并相比，多了一个审查机制。

> Github 的 Pull Request 可以进行跨仓库合并，这常用于对其他人的开源项目做贡献。通常有以下步骤：
> 1. fork 目标仓库。
> 1. 在本地创建分支，修改代码。
> 1. 推送到 fork 的仓库。
> 1. 进入目标仓库，点击 "Pull Request" ，点击 "New Pull Request"，点击 "compare across forks" 。
> 1. 选择目标分支和自己的修改分支，填写 PR 详情（包括标题，描述，审查者等等），确定。
> 1. 审查者可以批准 (Approve)、请求更改 (Request Changes) 或只是评论 (Comment)。
> 1. 审查者合并分支后，可以删除修改分支。

## Gtihub Actions 的基本概念
### 工作流(Workflows)
工作流是 GitHub Actions 执行任务的基本单位，你可以为 Git 上不同的事件(如 main 分支更新等)定义不同的工作流，以响应操作代码的变更。

### 任务(Jobs)
工作流程由一个或多个任务组成，每个任务运行在独立的虚拟环境（虚拟机）中。任务可以是构建、测试、部署等操作。

### 步骤(Steps)
任务由多个步骤组成，每个步骤执行一个操作。一个步骤可以是运行命令、使用某个预定义的操作，或者调用自定义脚本。

### Actions secrets and variables
在仓库页面，点击"Settings"，点击左侧"Secrets and variables"，点击"Actions"，就可以管理在 Github Action 中的机密和变量。比如发布安卓系统的安装包(apk)就需要数字签名，那么就可以使用 Repository secrets 来存储证书，在 Github Action 中使用。

## Github Actions 配置文件基本结构
Github Actions 配置文件必须存储在项目根目录的 `.github/workflows/` 文件夹下，文件后缀 `.yml` 或 `.yaml` 。

以下配置文件做的事情是：当 main 分支更新时，自动构建 Release 版本，打包zip，计算sha256，发布到 Github 仓库的 Releases 中。

```yaml
name: Publish win-x64

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
  workflow_dispatch:

env:
  DOTNET_VERSION: 8.0.x

jobs:
  build:
    runs-on: windows-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Setup .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: ${{ env.DOTNET_VERSION }}

    - name: Publish win-x64 application
      run: |
        dotnet publish your-project.csproj --configuration Release --runtime win-x64 --output .\your-project-win-x64 --no-self-contained -p:PublishSingleFile=true -p:PublishReadyToRun=true

    - name: Create win-x64 Release ZIP
      run: |
        New-Item -ItemType Directory -Path ".\release"
        Compress-Archive -LiteralPath .\your-project-win-x64 -DestinationPath .\release\your-project-win-x64.zip

    - name: Calculate win-x64 ZIP SHA256
      id: sha256
      run: |
        Get-FileHash .\release\your-project-win-x64.zip  | Format-List > .\release\your-project-sha256.txt
    
    - name: Create GitHub Release
      id: create_release
      uses: softprops/action-gh-release@v2
      with:
        tag_name: v1.0.0
        name: v1.0.0
        body_path: ./release/your-project-sha256.txt
        draft: false
        prerelease: false
        fail_on_unmatched_files: true
        files: |
          ./release/your-project-win-x64.zip
      env:
        GITHUB_TOKEN: ${{ secrets.Workflow }}
```

### 配置文件中的关键字说明
| 关键字 | 作用 | 示例 |
| :--: | :--: | :--: |
| `on` | 定义触发条件 | `workflow_dispatch:` |
| `runs-on` | 指定运行环境 | `ubuntu-latest`, `windows-latest`, `macos-latest` |
| `uses` | 引用预定义 Action | `actions/checkout@v4` |
| `with` | 向 Action 传递输入参数 | `key: value`, `dotnet-version: 8.0.x` |
| `run` | 执行 shell 命令 | `dotnet publish ...` |
| `env` | 定义环境变量（支持全局/作业级/步骤级） | `DEBUG: true` |
| `secrets` | 使用仓库机密 | `${{ secrets.API_TOKEN }}` |
| `vars` | 使用仓库变量 | `${{ vars.NAME }}` |
| `needs` | 定义作业依赖关系 | `needs: [build-job, test-job]` |
| `if` | 条件执行（支持表达式） | `if: ${{ github.ref == 'refs/heads/main' }}` |

#### Shell 种类设置
可以在配置文件的最外层使用以下代码来指定使用哪种 Shell ：

```
defaults:
  run:
    shell: bash
```

也可以在每一个 step 下，使用以下代码来使用特定的 Shell ：

```
steps:
  - name: Display the path
    run: echo %PATH%
    shell: cmd
```

默认情况下，各个运行平台所使用的 Shell 如下：

| 平台 | 默认 Shell | 说明 |
| :--: | :--: | :--: |
| Windows | `pwsh` | PowerShell 7 |
| 非 Windows 平台 | `bash` | 在非 Windows 平台上默认使用 shell ，并回退到 sh 。在 Windows 上指定 bash shell 时，将使用 Git for Windows 附带的 bash shell 。 |

支持但需明确指定的非默认 Shell 如下：

| 平台 | Shell | 说明 |
| :--: | :--: | :--: |
| Windows / Linux | `python` |  |
| Windows / Linux | `pwsh` | PowerShell 7 |
| Windows / Linux | `bash` | 非 Windows 平台上默认使用 shell ，并回退到 sh 。在 Windows 上指定 bash shell 时，将使用 Git for Windows 附带的 bash shell 。 |
| macOS / Linux | `sh` | 如果未提供 shell 且在路径中找不到 bash 时，非 Windows 平台的默认回退行为。 |
| Windows | `cmd` |  |
| Windows | `PowerShell` |  |

#### 预定义 Action 查找
可以在 [Github Marketplace](https://github.com/marketplace?type=actions) 中搜索有哪些预定义 Action 可以使用，参数有哪些。

## 另请参阅
大规模软件是如何开发的？(What does larger scale software development look like?)，[Bilibili 中文翻译](https://www.bilibili.com/video/BV1ZZjjzvEFD/)，[Youtube 英文原版](https://www.youtube.com/watch?v=Dl-BdxNRUqs)。