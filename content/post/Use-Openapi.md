+++
date = '2025-11-22T12:33:32+08:00'
draft = true
title = '使用 Openapi 生成 WebAPI 客户端 SDK'
categories = ['Main Sections']
mermaid = true
+++

## OpenAPI 介绍

OpenAPI（原 Swagger 规范） 是一个用来描述 HTTP API 的标准格式（通常是 JSON 或 YAML）。

OpenAPI 的核心用途包含：

- 自动生成接口文档（Swagger UI）
- 各语言自动生成客户端 SDK
- 自动生成服务端框架
- 类型安全：让前后端共享同一份 API 类型定义，避免出现参数不一致的问题，减少沟通成本。
- API 测试

在本文中，我们只要关注“自动生成客户端 SDK”的功能。常用的生成工具有：

- OpenAPI Generator
- NSwag

下面，将展示 asp\.net 编写服务端，生成 CSharp 客户端 SDK 和 TypeScript 客户端 SDK 的例子。

## 编写服务端

其实在 .NET 10 中，只要新建一个 WebAPI 项目，就已经自带 OpenAPI 支持了。下面说明一些细节。

### 注册 OpenAPI

```csharp
builder.Services.AddOpenApi();
```

这行代码，表示注册 OpenAPI 相关依赖。

```csharp
if (app.Environment.IsDevelopment())
{
    app.MapOpenApi();
}
```

这段代码，表示映射 OpenAPI 相关 HTTP 接口，这下文有说明。推荐仅在 Development 环境下映射它们。

### 查看 OpenAPI 文档

运行应用，访问: HTTP GET <baseURL>/openapi/v1.json ，就能看到 OpenAPI 文档了。里面包含了该应用的所有 HTTP 接口的信息，包括请求 URL ，请求方法，请求参数等等。

### 接口写法

```csharp
todosApi.MapGet("/", () => TypedResults.Ok(sampleTodos)).WithName("GetTodos");
```

这段代码是一个简单的 HTTP 接口示例，使用了 Minimal API 。有以下几个关键点：

- 返回值是通过 `TypedResults` 来构建的。与 `Results` 不同的是， `TypedResults` 会带上一些类型信息，以生成 OpenAPI 文档。如果使用 `Results` 返回， OpenAPI 文档上的信息会不完整。
- 返回值必须是确切的类型。如果返回父类，或者接口，那么 OpenAPI 文档上的信息会不完整。
- `WithName` 方法，其实是指定了 OpenAPI 文档中，该请求的 "operationId" 字段。 "operationId" 字段必须指定，且每个请求唯一。

再来看看另一个 HTTP 接口。

```csharp
todosApi
    .MapGet(
        "/{id}",
        Results<Ok<Todo>, NotFound> (int id) =>
            sampleTodos.FirstOrDefault(a => a.Id == id) is { } todo
                ? TypedResults.Ok(todo)
                : TypedResults.NotFound()
    )
    .WithName("GetTodoById");
```

可以看出，如果该请求会返回不同的 HTTP 状态（使用 `TypedResults` 不同的方法返回），就需要手动指定方法的返回值： `Results<Ok<Todo>, NotFound>` 。

#### 未解析的响应类型

在当前的版本(Microsoft.AspNetCore.OpenApi v10.0.0)中，有一些响应是不会写入到 OpenAPI 文档中，需要手动指定。

```csharp {hl_lines=["21"]}
app.MapPost(
        "api/xxx",
        async Task<Results<Ok<XXX>, UnauthorizedHttpResult>> (
            XXXRequest request,
            ClaimsPrincipal claimsPrincipal,
            CancellationToken cancellationToken
        ) =>
        {
            var optionalUserId = claimsPrincipal.GetUserId();
            return optionalUserId switch
            {
                NoneOption<Guid> => TypedResults.Unauthorized(),
                SomeOption<Guid> userId => TypedResults.Ok(
                    // 业务处理的结果
                ),
                _ => throw new ArgumentOutOfRangeException(nameof(optionalUserId)),
            };
        }
    )
    .RequireAuthorization(new AuthorizeAttribute())
    .Produces<UnauthorizedHttpResult>(StatusCodes.Status401Unauthorized)
    .WithName(nameof(AddMusicInfoToMusicList));
```

可以发现， `UnauthorizedHttpResult` 没有写进 OpenAPI 文档中。要添加高亮行代码，即( `.Produces...` )

> [!info]
> 也许以 `HttpResult` 为后缀的响应，是不会自动包含的？

#### 未解析的响应类型-文件流

同样地，返回文件流的响应也不会写入 OpenAPI 文档中，需要手动指定。

```csharp
app.MapGet(
        "api/get-file-stream/{id:guid}",
        async Task<Results<FileStreamHttpResult, NotFound>> (
            Guid id,
            CancellationToken cancellationToken
        ) =>
        {
            var optionalStream = ; // 获取文件流
            return optionalStream switch
            {
                SomeOption<Stream> stream => TypedResults.File(
                    fileStream: stream.Value,
                    contentType: "application/octet-stream",
                    enableRangeProcessing: true
                ),
                _ => TypedResults.NotFound(),
            };
        }
    )
    .AddOpenApiOperationTransformer(
        (operation, context, ct) =>
        {
            operation.Responses!["200"] = new OpenApiResponse
            {
                Description = "OK",
                Content = new Dictionary<string, OpenApiMediaType>
                {
                    ["application/octet-stream"] = new()
                    {
                        Schema = new OpenApiSchema
                        {
                            Type = JsonSchemaType.String,
                            Format = "binary",
                        },
                    },
                },
            };
            operation.Responses!["206"] = new OpenApiResponse
            {
                Description = "Partial Content",
                Content = new Dictionary<string, OpenApiMediaType>
                {
                    ["application/octet-stream"] = new()
                    {
                        Schema = new OpenApiSchema
                        {
                            Type = JsonSchemaType.String,
                            Format = "binary",
                        },
                    },
                },
            };
            return Task.CompletedTask;
        }
    )
    .WithName("GetFileStream");
```

使用 `AddOpenApiOperationTransformer` 方法手动指定 OpenAPI 文档内容。

### 自动生成 OpenAPI 文档

在 WebAPI 项目安装 NuGet 包 Microsoft.Extensions.ApiDescription.Server ，能够在构建项目时，在 obj 文件夹中，生成 OpenAPI 文档。

可以通过设置 MSBuild 属性 OpenApiDocumentDirectory (csproj 中的 `PropertyGroup` 或 编译时 `/p:` 指令) ，设置 OpenAPI 文档生成位置。

## 生成 CSharp 客户端

## 生成 TypeScript 客户端
