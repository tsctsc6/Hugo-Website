+++
date = '2025-07-31T19:45:58+08:00'
draft = false
title = '在 ASP.NET Web API 中使用 ASP.NET Identity'
categories = ['Main Sections']
mermaid = true
+++

> 前置知识：
> 
> * [Entity Framework Core](https://learn.microsoft.com/zh-cn/ef/core/) 的简单使用
> * [ASP.NET Web API](https://learn.microsoft.com/zh-cn/aspnet/web-api/) 的简单使用

## ASP\.NET Identity 介绍
ASP\.NET Identity 是一个用于 ASP\.NET 应用程序的身份认证系统，提供了用户管理、角色管理、声明、令牌等功能。它支持用户注册、登录、注销以及外部登录提供程序。

[官方文档](https://learn.microsoft.com/zh-cn/aspnet/core/security/authentication/identity)

默认情况下， ASP\.NET Identity 会自动创建 7 个有关身份认证的表，表的 ER 图如下（不显示字段）：

```mermaid
erDiagram
    AspNetUserRoles }|--|| AspNetUsers : ""
    AspNetUserClaims }|--|| AspNetUsers : ""
    AspNetUsers ||--|{ AspNetUserLogins : ""
    AspNetUsers ||--|{ AspNetUserTokens : ""
    AspNetRoleClaims }|--|| AspNetRoles : ""
    AspNetUserRoles }|--|| AspNetRoles : ""
```

以下是各个表的字段的介绍。

> * Type 一栏，不是准确的数据库的类型，在不同的数据库会有所不同。
> * "NN" 指 "NOT NULL"

### AspNetUsers 表
| Type | Name | PK | FK | NN | Remark |
| :--: | :--: | :--: | :--: | :--: | :--: |
| string | Id | &#10004; |  | &#10004; | 格式为 UUID |
| string | UserName |  |  |  |  |
| string | NormalizedUserName |  |  |  | UserName 字段的全大写形式 |
| string | Email |  |  |  |  |
| string | NormalizedEmail |  |  |  | Email 字段的全大写形式 |
| boolean | EmailConfirmed |  |  | &#10004; | 邮箱是否有效（比如通过邮箱验证码认证） |
| string | PasswordHash |  |  |  |  |
| string | SecurityStamp |  |  |  | 格式为 Base64 |
| string | ConcurrencyStamp |  |  |  | 格式为 UUID |
| string | PhoneNumber |  |  |  |  |
| boolean | PhoneNumberConfirmed |  |  | &#10004; |  |
| boolean | TwoFactorEnabled |  |  | &#10004; |  |
| timestamp | LockoutEnd |  |  |  | 账户​​锁定的结束时间 |
| boolean | LockoutEnabled |  |  | &#10004; |  |
| integer | AccessFailedCount |  |  | &#10004; | 记录用户​​连续登录失败的次数​​，成功的登录会重置该计数器为 0 |

#### PasswordHash 字段
具体作用，请查看"[根据用户名和口令进行身份验证-加盐](./Basic-Computer-Theory/Introduction-to-Information-Security/Authentication-based-on-username-and-password/index.md#加盐)"，这里把口令的哈希值和盐值存储在同一字段中。

#### SecurityStamp 字段
当用户的关键安全信息发生变化​​（如更改密码、锁定用户等），此值​​会被更新，以吊销凭证，使该用户在​​其他所有设备或浏览器​​上的现有会话失效。

在基于令牌认证（[JWT](./Basic-Computer-Theory/Introduction-to-Information-Security/Session-state-management/index.md#jwt)）的场景中， SecurityStamp 可以映射到 JWT 的 jti (JWT ID) 声明，并在令牌验证时检查有效性。更新 SecurityStamp 等价于撤销之前颁发的所有 JWT 。

#### ConcurrencyStamp 字段
用于实现​​数据库层面的乐观并发控制 (Optimistic Concurrency)​​。

乐观并发是指，允许多个进程或用户尝试同时更新同一条用户记录，而不锁定记录。它假设冲突不常发生。

记录在读取时会获取当前的 ConcurrencyStamp 值。当尝试更新记录时， ED Core (或底层的存储提供程序) 会将传入记录的 ConcurrencyStamp 值​​与数据库中当前该记录的 ConcurrencyStamp 值进行比较​​。

* 如果值​​匹配​​：说明记录自上次读取后​​没有被其他操作修改过​​，更新允许进行，并在更新​​后会自动生成一个新的 ConcurrencyStamp​​（通常是新的 GUID）存入数据库。
* 如果值​​不匹配​​：说明记录自上次读取后​​已经被其他操作修改过​​（例如，管理员同时在后台修改了该用户）。此时 EF Core 会抛出 DbUpdateConcurrencyException 异常，开发者需要捕获此异常并决定如何处理冲突（如提示用户、重试操作、合并变更等）。

### AspNetRoles 表
| Type | Name | PK | FK | NN | Remark |
| :--: | :--: | :--: | :--: | :--: | :--: |
| string | Id | &#10004; |  | &#10004; | 格式为 UUID |
| string | Name |  |  |  |  |
| string | NormalizedName |  |  |  |  |
| string | ConcurrencyStamp |  |  |  | 格式为 UUID |

### AspNetUserClaims 表
| Type | Name | PK | FK | NN | Remark |
| :--: | :--: | :--: | :--: | :--: | :--: |
| integer | Id | &#10004; |  | &#10004; | 自增 Id |
| string | UserId |  | &#10004; | &#10004; | 格式为 UUID |
| string | ClaimType |  |  |  |  |
| string | ClaimValue |  |  |  |  |

### AspNetRoleClaims 表
| Type | Name | PK | FK | NN | Remark |
| :--: | :--: | :--: | :--: | :--: | :--: |
| integer | Id | &#10004; |  | &#10004; | 自增 Id |
| string | RoleId |  | &#10004; | &#10004; | 格式为 UUID |
| string | ClaimType |  |  |  |  |
| string | ClaimValue |  |  |  |  |

### AspNetUserLogins 表
| Type | Name | PK | FK | NN | Remark |
| :--: | :--: | :--: | :--: | :--: | :--: |
| string | LoginProvider | &#10004; |  | &#10004; |  |
| string | ProviderKey | &#10004; |  | &#10004; |  |
| string | ProviderKey |  | &#10004; | &#10004; | 格式为 UUID |
| string | ProviderDisplayName |  |  |  |  |

### AspNetUserTokens 表
| Type | Name | PK | FK | NN | Remark |
| :--: | :--: | :--: | :--: | :--: | :--: |
| string | UserId | &#10004; | &#10004; | &#10004; | 格式为 UUID |
| string | LoginProvider | &#10004; |  | &#10004; |  |
| string | Name | &#10004; |  | &#10004; |  |
| string | Value |  |  |  |  |


### AspNetUserRoles 表
| Type | Name | PK | FK | NN | Remark |
| :--: | :--: | :--: | :--: | :--: | :--: |
| string | UserId | &#10004; | &#10004; | &#10004; | 格式为 UUID |
| string | RoleId | &#10004; | &#10004; | &#10004; | 格式为 UUID |

## 为 ASP\.NET Web API 配置 ASP\.NET Identity + JWT
目前， ASP\.NET Identity 只有在 Blazor Web App, Razor Pages, MVC 应用中有模板。如果我们需要为 ASP\.NET Web API 配置 ASP\.NET Identity ，只能自己动手了。

创建一个 ASP\.NET Web API 项目，不要选择任何身份验证方式。

安装 NuGet 包：

```shell
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
dotnet add package Microsoft.EntityFrameworkCore.Tools
```

还有你喜欢的 Entity Framework Core 数据库驱动。

### 项目结构
项目使用 MVC 架构（Web API 没有 View），即分为 Controller 层、 Service 层(本例集成在 Controller 层)、 Repository 层(本例使用 EF Core  代替)、 Model 层。具体可以看 [SpringBoot 架构介绍](./ProgrammingLanguageNote/SpringBoot/index.md)。

### 定义实体和数据库上下文
```CSharp {name="Models/ApplicationUser.cs"}
using Microsoft.AspNetCore.Identity;

public class ApplicationUser : IdentityUser
{
    // 如有需要可添加更多属性
    // public DateOnly Birthday { get; set; }
}
```

```CSharp {name="Data/ApplicationDbContext.cs"}
using Microsoft.AspNetCore.Identity.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore;

public class ApplicationDbContext : IdentityDbContext<ApplicationUser>
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options) { }

    // 如有其它 DbSet<...> 可在此添加
}
```

### 配置
在 `var app = builder.Build();` 之前，添加：

```CSharp {name="Program.cs"}
// 1) 添加 DbContext
builder.Services.AddDbContext<ApplicationDbContext>(options =>
{
    options.UseSqlite(builder.Configuration.GetConnectionString("DefaultConnection"));
    // 初始化角色
    options.UseSeeding((context, _) =>
    {
        context.Add<IdentityRole>(new()
        {
            Id = Guid.NewGuid().ToString(),
            Name = "Admin",
            NormalizedName = "ADMIN",
            ConcurrencyStamp = Guid.NewGuid().ToString()
        });
        context.SaveChanges();
    });
});

// 2) 添加 Identity
builder.Services
    .AddIdentity<ApplicationUser, IdentityRole>(options =>
    {
        options.Password.RequiredLength = 6;
        options.Password.RequireNonAlphanumeric = false;
        // 其它密码/锁定/用户设置...
    })
    .AddEntityFrameworkStores<ApplicationDbContext>()
    .AddDefaultTokenProviders();

// 3) 添加 JWT 认证
var jwtKey = builder.Configuration["Jwt:Key"]!;
var keyBytes = Encoding.UTF8.GetBytes(jwtKey);
builder.Services
    .AddAuthentication(options =>
    {
        options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
        options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
    })
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            ValidateAudience = true,
            ValidAudience = builder.Configuration["Jwt:Audience"],
            ValidateIssuerSigningKey = true,
            IssuerSigningKey = new SymmetricSecurityKey(keyBytes),
            ValidateLifetime = true,
            ClockSkew = TimeSpan.Zero
        };
    });

// 4) 添加授权（如果需要角色策略等）
builder.Services.AddAuthorization();

// 5) 添加控制器
builder.Services.AddControllers();
```

在 `var app = builder.Build();` 之后，添加：

```CSharp {name="Program.cs"}
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

在 appsettings.json 或 appsettings.Development.json 中配置好连接字符串和 JWT ：

```json {name="appsettings.json"}
{
  "ConnectionStrings": {
    "DefaultConnection": "DataSource=app.db;Cache=Shared"
  },
  "Jwt": {
    "Key": "至少32字符的随机密钥串",
    "Issuer": "MyApi",
    "Audience": "MyApiClient"
  }
}
```

### 生成数据库迁移并更新
```shell
dotnet ef migrations add InitIdentity
dotnet ef database update
```

> 在使用 `dotnet ef database update` 时，仅触发 `UseSeeding` 方法，不触发  `UseAsyncSeeding` 方法。

### 编写注册和登录 API
定义 DTO:

```CSharp {name="DTOs/RegisterDto.cs"}
public class RegisterDto
{
    public string UserName { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public string Password { get; set; } = string.Empty;
}
```

```CSharp {name="DTOs/LoginDto.cs"}
public class LoginDto
{
    public string UserName { get; set; } = string.Empty;
    public string Password { get; set; } = string.Empty;
}
```

编写 Controller:

```CSharp {name="Controllers/AuthController.cs"}
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Text;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;
using Microsoft.IdentityModel.Tokens;
using TestJWT.DTOs;
using TestJWT.Models;

[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private readonly UserManager<ApplicationUser> _userMgr;
    private readonly SignInManager<ApplicationUser> _signInMgr;
    private readonly IConfiguration _config;

    public AuthController(
        UserManager<ApplicationUser> userMgr,
        SignInManager<ApplicationUser> signInMgr,
        IConfiguration config
    )
    {
        _userMgr = userMgr;
        _signInMgr = signInMgr;
        _config = config;
    }

    [HttpPost("register")]
    public async Task<IActionResult> Register(RegisterDto dto)
    {
        var user = new ApplicationUser { UserName = dto.UserName, Email = dto.Email };
        var result = await _userMgr.CreateAsync(user, dto.Password);
        if (!result.Succeeded)
            return BadRequest(result.Errors);

        // （可选）给用户分配默认角色
        await _userMgr.AddToRoleAsync(user, "Admin");

        return Ok(new { Message = "注册成功" });
    }

    [HttpPost("login")]
    public async Task<IActionResult> Login(LoginDto dto)
    {
        var user = await _userMgr.FindByNameAsync(dto.UserName);
        if (user == null)
            return Unauthorized("用户名不存在");

        var signInRes = await _signInMgr.CheckPasswordSignInAsync(user, dto.Password, false);
        if (!signInRes.Succeeded)
            return Unauthorized("密码错误");
        
        // 生成 JWT
        var claims = new List<Claim>
        {
            new(JwtRegisteredClaimNames.Sub, user.Id),
            new(JwtRegisteredClaimNames.UniqueName, user.UserName)
        };
        // 加入角色声明
        var roles = await _userMgr.GetRolesAsync(user);
        claims.AddRange(roles.Select(r => new Claim("role", r)));

        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_config["Jwt:Key"]!));
        var creds  = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);
        var token  = new JwtSecurityToken(
            issuer: _config["Jwt:Issuer"],
            audience: _config["Jwt:Audience"],
            claims: claims,
            expires: DateTime.UtcNow.AddHours(1),
            signingCredentials: creds
        );
        var tokenStr = new JwtSecurityTokenHandler().WriteToken(token);

        return Ok(new { token = tokenStr, expires = token.ValidTo });
    }
}
```

### 保护 API
在需要登录才可访问的控制器或方法上加上 `[Authorize]` ，也可指定角色：

```CSharp {name="Controllers/ValuesController.cs"}
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

[ApiController]
[Route("api/[controller]")]
[Authorize]
public class ValuesController : ControllerBase
{
    [HttpGet("hello")]
    public IActionResult Hello()
    {
        var userId = User.Claims.FirstOrDefault(c => c.Type == ClaimTypes.NameIdentifier)?.Value;
        var userName = User.Identity?.Name;
        return Ok($"Hello, {userId}, {userName}!");
    }

    [HttpGet("admin")]
    [Authorize(Roles = "Admin")]
    public IActionResult AdminOnly()
    {
        var role = User.Claims.FirstOrDefault(c => c.Type == ClaimTypes.Role)?.Value;
        var userId = User.Claims.FirstOrDefault(c => c.Type == ClaimTypes.NameIdentifier)?.Value;
        var userName = User.Claims.FirstOrDefault(c => c.Type == ClaimTypes.Name)?.Value;
        return Ok($"Hello, {role}, {userId}, {userName}!");
    }
}
```

## 测试
### 注册
访问 POST /api/auth/register ,

请求头加入: `Content-Type: application/json` ,

请求体：

```json
{
  "userName": "test",
  "email": "test@test.com",
  "password": "Admin@123"
}
```

如果不出意外，服务器返回：

```json
{
    "message": "注册成功"
}
```

### 登录
访问 POST /api/auth/login ,

请求头加入: `Content-Type: application/json` ,

请求体：

```json
{
  "userName": "test",
  "password": "Admin@123"
}
```

如果不出意外，服务器返回：

```json
{
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI5MTkzMzQzZS03YzBlLTQ0Y2YtODQ3Ny0yZmFiYTUxZjllZWYiLCJ1bmlxdWVfbmFtZSI6InRlc3QiLCJyb2xlIjoiQWRtaW4iLCJleHAiOjE3NTQxMTQ4NzMsImlzcyI6IlRlc3RKV1QiLCJhdWQiOiJUZXN0SldUX0NsaWVudCJ9.njYXzAmvXvkxJRxo7m4qmD4j2Ls0KcbE5TiMfWXmxJk",
    "expires": "2025-08-02T06:07:53Z"
}
```

### 访问 API
访问 GET /api/values/hello ,

请求头加入: `Authorization: Bearer eyJhbGc...`

如果不出意外，服务器返回：

```txt
Hello, 9193343e-7c0e-44cf-8477-2faba51f9eef, test!
```

访问 GET /api/values/admin ,

请求头加入: `Authorization: Bearer eyJhbGc...`

如果不出意外，服务器返回：

```txt
Hello, Admin, 9193343e-7c0e-44cf-8477-2faba51f9eef, test!
```

## 分析 JWT
如果把 [JWT](./Basic-Computer-Theory/Introduction-to-Information-Security/Session-state-management/index.md#jwt) 的 Payload 解码，我们会看到（已美观格式化）：

```json
{
    "sub": "9193343e-7c0e-44cf-8477-2faba51f9eef",
    "unique_name": "test",
    "role": "Admin",
    "exp": 1754114873,
    "iss": "TestJWT",
    "aud": "TestJWT_Client"
}
```

如果在 `ValuesController` 的 `Hello` 方法的开头打上断点，调试。其中， `User.Claims` 的类型是 `IEnumerable<Claim>` ，其内容是：

| Index | 属性 `Type` 的值 | 属性 `Value` 的值 |
| :--: | :--: | :--: |
| 0 | http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier | 9193343e-7c0e-44cf-8477-2faba51f9eef |
| 1 | http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name | test |
| 2 | http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier | Admin |
| 3 | exp | 1754108924 |
| 4 | iss | TestJWT |
| 5 | aud | TestJWT_Client |

那么，为什么 JWT Payload 的值，和 C# 代码内的属性 `Type` 的值，会是 uri 呢？因为 ASP\.NET 会自动把 JWT Payload 中的标准的 JWT claim 名称，映射为 `ClaimTypes` 类型字段。即，把 `sub` 映射为 `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier` , 把 `unique_name` 映射为 `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name` ，等等。

具体的映射表，可以这样查看：

```CSharp
var x = JwtSecurityTokenHandler.DefaultInboundClaimTypeMap
```

如果想关闭映射，可以在 Program.cs 中配置：

```CSharp {name="Programs.cs"}
builder.Services
    .AddJwtBearer(options =>
    {
        options.MapInboundClaims = false;
        options.TokenValidationParameters = new TokenValidationParameters
        {
            // 告诉授权系统“角色”就是 JWT 里的 "role"
            RoleClaimType = "role",
            // 告诉系统“用户名”就是 JWT 里的 "unique_name"
            NameClaimType = JwtRegisteredClaimNames.UniqueName
            // 其他配置 ...
        }
        // 其他配置 ...
    });
```

并且，由于关闭了映射，所以 `ValuesController` 中，获取用户 Id 的方法不再有效，应该改为：

```CSharp
var role = User.Claims.FirstOrDefault(c => c.Type == "role")?.Value;
var userId = User.Claims.FirstOrDefault(c => c.Type == JwtRegisteredClaimNames.Sub)?.Value;
var userName = User.Claims.FirstOrDefault(c => c.Type == JwtRegisteredClaimNames.UniqueName)?.Value;
return Ok($"Hello, {role}, {userId}, {userName}!");
```

> * `JwtRegisteredClaimNames` 是静态类，里面包含了一系列标准的 JWT claim 名称，在 `AuthController` 的 `Login` 方法，生成 JWT 的时候也有用到。
> * `ClaimTypes` 是静态类，里面包含了一系列映射之后的声明值。
