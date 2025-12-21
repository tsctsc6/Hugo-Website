+++
date = '2025-08-29T22:37:55+08:00'
lastmod = '2025-12-21T23:22:34+08:00'
draft = false
title = 'Dotnet 源生成器入门'
categories = ['Main Sections']
tags = ['.NET', 'Source Generator']
+++

## 什么是源生成器

源生成器是一种编译时的代码生成工具，首次引入于 .NET 5 。它允许开发者在编译过程中动态生成代码，从而减少重复性工作并提升性能。源生成器的核心思想是通过分析代码结构，自动生成符合需求的代码片段，避免运行时反射的性能开销。

想要流畅地使用源生成器，建议去了解一下编译原理。源生成器做的工作，就是完成词法语法分析，然后提供一定的接口，供使用者动态生成代码。

## 分部类与分布方法

在正式介绍源生成器的使用之前，先介绍 C# 的分部类与分布方法，这两个概念与源生成器是紧密相关的。

### 分部类

分部类允许将一个类的定义拆分到多个文件中。使用方法如下：

```CSharp {name="MyClass1.cs"}
public partial class MyClass
{
    private int id;
    private string name;
}
```

```CSharp {name="MyClass2.cs"}
public partial class MyClass
{
    public MyClass(int id, string name)
    {
        this.id = id;
        this.name = name;
    }
}
```

我们把 `MyClass` 的定义拆分到两个文件中。

### 分部方法

分部类可以包含分部方法。分部方法允许将一个方法的定义和实现拆分到两个文件中。例如：

```CSharp {name="MyClass1.cs"}
public partial class MyClass
{
    partial void OnNameChanged();
}
```

```CSharp {name="MyClass2.cs"}
public partial class MyClass
{
    partial void OnNameChanged()
    {
        // method body
    }
}
```

分部方法有只能返回 `void` ，可以只定义不实现。如果分部方法没有实现，也可以调用，但不会发生任何事情。

## 示例

### 目标

本文将创建一个源生成器示例，这个示例的目标是：可以为某类的某字段标记 `GenShowMethod` 特性，有此特性的字段，将生成 `public void ShowXxx()` 方法，可以在外部输出类的该字段内容。

### 项目配置

首先我们创建一个 `GeneratorDemo` 的项目，是控制台应用程序。

创建类库项目，**必须选择 .NET Standard 2.0** ，通常的命名规范是以 `Generator` 作为后缀。我们创建一个名为 `MyGenerator` 的项目。为这个项目安装 `Microsoft.CodeAnalysis.CSharp` NuGet 包。

打开 `MyGenerator` 项目的 .csproj 文件，添加以下内容：

```xml {name="MyGenerator.csproj",hl_lines=["3-4"]}
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <IsRoslynComponent>true</IsRoslynComponent>
    <EnforceExtendedAnalyzerRules>true</EnforceExtendedAnalyzerRules>
  </PropertyGroup>
</Project>
```

为 `MyGenerator` 项目安装 nuget 包: `Microsoft.CodeAnalysis.Analyzers` , `Microsoft.CodeAnalysis.CSharp`

打开 `GeneratorDemo` 项目的 .csproj 文件，添加以下内容：

```xml {name="GeneratorDemo.csproj",hl_lines=["3-5"]}
<Project Sdk="Microsoft.NET.Sdk">
  <ItemGroup>
    <ProjectReference Include="..\MyGenerator\MyGenerator.csproj"
                      OutputItemType="Analyzer"
                      ReferenceOutputAssembly="false"/>
  </ItemGroup>
</Project>
```

特别是，这不是单纯的项目引用，需要填写额外字段。

### 编码

#### 准备工作

由于在我们的目标中，要通过 `GenShowMethod` 特性来标记哪些字段要生成方法，因此先定义这个特性。新建一个类库项目 `MyGeneratorAttribute` 。在其中定义 `GenShowMethod` 特性：

```CSharp {name="GenShowMethodAttribute.cs"}
using System;

namespace MyGeneratorAttribute
{
    public class GenShowMethodAttribute : Attribute { }
}
```

编写 `GeneratorDemo` 项目，定义一个类：

```CSharp {name="MyClass.cs"}
using MyGeneratorAttribute;

namespace GeneratorDemo;

public partial class MyClass
{
    [GenShowMethod]
    private int id;

    private string name;

    public MyClass(int id, string name)
    {
        this.id = id;
        this.name = name;
    }
}
```

```CSharp {name="Program.cs"}
namespace GeneratorDemo;

internal class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("Hello, World!");
        var myClass = new MyClass(61, "World");
        // myClass.ShowId();
    }
}
```

我们要通过源生成器生成方法 `ShowId()` 。

#### 编写源生成器

在 `MyGenerator` 项目中编写：

```CSharp {name="FiledGenerator.cs"}
namespace MyGenerator
{
    [Generator(LanguageNames.CSharp)]
    public class FiledGenerator : IIncrementalGenerator
    {
        public void Initialize(IncrementalGeneratorInitializationContext context)
        {

        }
    }
}
```

通过继承 `IIncrementalGenerator` 接口，以及添加特性 `Generator` 定义一个源生成器。

> 也可以通过继承 `ISourceGenerator` 接口定义一个源生成器，不过该接口效率较低，不太推荐。

在 `Initialize` 方法中，可以注册以下方法：

- 编译器每扫描一个语法节点，要进行的操作（通常是筛选节点）。
- 编译器扫描所有语法节点后，要进行的操作（通常是进一步筛选节点和生成代码）。

于是我们编写 `Initialize` 方法：

```CSharp {name="FiledGenerator.cs"}
public void Initialize(IncrementalGeneratorInitializationContext context)
{
    // 编译器每扫描一个语法节点：
    var targetFields = context
        .SyntaxProvider.CreateSyntaxProvider(
            predicate: (node, _) => IsTargetNode(node),
            transform: (ctx, _) => TransformNodeToSyntax(ctx)
        )
        .Where(n => !(n is null));
    // 编译器扫描所有语法节点后
    context.RegisterSourceOutput(
        context.CompilationProvider.Combine(targetFields.Collect()),
        (spc, source) => Execute(spc, source.Left, source.Right)
    );
}
```

```CSharp {name="FiledGenerator.cs"}
/// <summary>
/// 语法节点初步筛选条件
/// </summary>
/// <param name="node"></param>
/// <returns></returns>
private static bool IsTargetNode(SyntaxNode node)
{
    if (
        node is FieldDeclarationSyntax fieldDecl
        && fieldDecl.Parent is ClassDeclarationSyntax classDecl
        && classDecl.Modifiers.Any(m => m.IsKind(SyntaxKind.PartialKeyword))
    )
    {
        return true;
    }
    return false;
}
```

```CSharp {name="FiledGenerator.cs"}
/// <summary>
/// 类型转换
/// 语法节点的进一步筛选条件，不符合条件的返回 null
/// </summary>
/// <param name="ctx"></param>
/// <returns></returns>
private static FieldDeclarationSyntax TransformNodeToSyntax(GeneratorSyntaxContext ctx)
{
    if (!(ctx.Node is FieldDeclarationSyntax fieldDecl))
    {
        return null;
    }
    if (
        fieldDecl
            .AttributeLists.SelectMany(a => a.Attributes)
            .Select(a => ctx.SemanticModel.GetSymbolInfo(a).Symbol as IMethodSymbol)
            .Any(s =>
                s.ContainingType.ToDisplayString()
                == "MyGeneratorAttribute.GenShowMethodAttribute"
            )
    )
    {
        return ctx.Node as FieldDeclarationSyntax;
    }
    return null;
}
```

```CSharp {name="FiledGenerator.cs"}
/// <summary>
/// 类型转换
/// 语法节点的进一步筛选条件，不符合条件的返回 null
/// </summary>
/// <param name="ctx"></param>
/// <returns></returns>
private static FieldDeclarationSyntax TransformNodeToSyntax(GeneratorSyntaxContext ctx)
{
    if (!(ctx.Node is FieldDeclarationSyntax fieldDecl))
    {
        return null;
    }
    if (
        fieldDecl
            .AttributeLists.SelectMany(a => a.Attributes)
            .Select(a => ctx.SemanticModel.GetSymbolInfo(a).Symbol as IMethodSymbol)
            .Any(s =>
                s.ContainingType.ToDisplayString()
                == "MyGeneratorAttribute.GenShowMethodAttribute"
            )
    )
    {
        return ctx.Node as FieldDeclarationSyntax;
    }
    return null;
}
```

```CSharp {name="FiledGenerator.cs"}
private static void Execute(
    SourceProductionContext context,
    Compilation compilation,
    ImmutableArray<FieldDeclarationSyntax> fields
)
{
    // 按 class 来分组，每个 class 一个文件
    foreach (
        var group in fields.GroupBy(f => ((ClassDeclarationSyntax)f.Parent).Identifier.Text)
    )
    {
        var classSymbol =
            compilation
                .GetSemanticModel(group.First().SyntaxTree)
                .GetDeclaredSymbol(group.First().Parent) as INamedTypeSymbol;
        var sb = new StringBuilder(4096);

        sb.AppendLine("// <Auto-generated/>");
        sb.AppendLine($"namespace {classSymbol.ContainingNamespace};");
        sb.AppendLine();
        sb.AppendLine($"partial class {classSymbol.Name}");
        sb.AppendLine("{");

        // 对每个符合要求的字段生成代码
        foreach (var field in group)
        {
            sb.AppendLine(
                $"\tpublic void Show{FirstCharToUpper(field.Declaration.Variables.First().Identifier.ValueText)}()"
            );
            sb.AppendLine("\t{");
            sb.AppendLine(
                $"\t\tConsole.WriteLine({field.Declaration.Variables.First().Identifier.ValueText});"
            );
            sb.AppendLine("\t}");
            sb.AppendLine();
        }
        sb.AppendLine("}");

        // 源生成器生成的代码，后缀名一般为 .g.cs
        context.AddSource(
            $"{group.Key}.g.cs",
            SourceText.From(sb.ToString(), Encoding.UTF8)
        );
    }
}

private static string FirstCharToUpper(string s)
{
    var firstChar = s[0].ToString().ToUpper();
    return firstChar + s.Substring(1);
}
```

至此，一个源生成器就编写好了。

### 运行项目

更改 `GeneratorDemo` 项目的代码：

```CSharp {name="Program.cs"}
namespace GeneratorDemo;

internal class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("Hello, World!");
        var myClass = new MyClass(61, "World");
        myClass.ShowId();
    }
}
```

运行，结果为：

```Plane
Hello, World!
61
```

成功调用了 `ShowId` 方法。

### 查看生成的代码

#### Visual Studio

项目 `GeneratorDemo` -> 依赖项 -> 分析器 -> `MyGenerator` -> `MyGenerator.FiledGenerator` -> .g.cs 文件

#### JetBrains Rider

项目 `GeneratorDemo` -> 依赖项 -> 源生成器 -> `MyGenerator.FiledGenerator` -> .g.cs 文件

### 调试

#### 断点调试

需要安装 .NET Compiler Platform SDK .

{{<link title=".NET Compiler Platform SDK (Roslyn API) - C# | Microsoft Learn" link="https://learn.microsoft.com/zh-cn/dotnet/csharp/roslyn-sdk/" cover="auto">}}

在 `MyGenerator` 项目的跟目录添加 `Properties/launchSettings.json` ，填写以下内容：

```json {name="Properties/launchSettings.json"}
{
  "$schema": "http://json.schemastore.org/launchsettings.json",
  "profiles": {
    "Generators": {
      "commandName": "DebugRoslynComponent",
      "targetProject": "../GeneratorDemo/GeneratorDemo.csproj"
    }
  }
}
```

> 其中， `targetProject` 字段，表示的是你想将这个源生成器作用于哪个项目。

此时， IDE 出现了新的启动项： "MyGenerator: Generators" 。打上断点，运行，即可进行断点调试。

#### 诊断报告

诊断报告显示在"问题"一栏，也就是你平时查看编译错误的地方。可以通过以下方式发送诊断报告：

```CSharp
context.ReportDiagnostic(
    Diagnostic.Create(
        new DiagnosticDescriptor(
            id: "MY01",
            title: "",
            messageFormat: $"有一个消息",
            category: "",
            defaultSeverity: DiagnosticSeverity.Info,
            isEnabledByDefault: true
        ),
        Location.Create(field.SyntaxTree, field.Span)
    )
);
```

这里的 `context` 的类型是 `SourceProductionContext` ，不是 `Initialize` 方法中的 `IncrementalGeneratorInitializationContext` 。

可以在 `Initialize` 方法中的 `context.RegisterSourceOutput` 中，通过 `action` 参数传递的方法，得到 `SourceProductionContext` 。也就是上述例子中的 `Execute` 方法。

#### 可视化语法树

还没弄懂。

### 构建 Nuget 包

在 `MyGenerator.csproj` 中，添加以下内容：

```xml {name="MyGenerator.csproj"}
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <!-- Other content -->
    <!-- ... -->
    <!-- NuGet info -->
    <PackageId>MyGenerator</PackageId>
    <Version>1.0.0</Version>
    <Authors>xxx</Authors>
    <Description>xxx</Description>
    <!-- Declare Analyzer -->
    <IncludeBuildOutput>false</IncludeBuildOutput>
    <IsRoslynAnalyzer>true</IsRoslynAnalyzer>
    <!-- not generate lib/ref -->
    <SuppressDependenciesWhenPacking>true</SuppressDependenciesWhenPacking>
  </PropertyGroup>
  <ItemGroup>
    <!-- Put the DLL into analyzers -->
    <None Include="$(OutputPath)\$(AssemblyName).dll"
          Pack="true"
          PackagePath="analyzers/dotnet/cs" />
  </ItemGroup>
</Project>
```

打包命令：

```
dotnet pack -c Release
```

生成位置： bin/Release/MyGenerator.1.0.0.nupkg

不要忘记也把 MyGeneratorAttribute 项目也打包，放在同一个包源中。

接下来是上传 .nupkg 包，无论是放在本地包源，还是自定义包源，还是官方包源都可以，步骤略。
