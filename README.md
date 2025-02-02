[合集 \- .NET(1\)](https://github.com)1\.\[.NET] 使用客户端缓存提高API性能01\-10收起
# 使用客户端缓存提高API性能


## 摘要


在现代应用程序中，性能始终是一个关键的考虑因素。无论是提高响应速度，降低延迟，还是减轻服务器负载，开发者都在寻找各种方法来优化他们的API。在Web开发中，利用客户端缓存是一种有效的方法，可以显著提高API的性能。本文将结合[Replicant](https://github.com)和[Delta](https://github.com)，深入探讨如何在.NET中使用客户端缓存，巧妙地提升API的响应速度。


## 问题概述


尽管数据库已经过优化，但在实际应用中，我们仍然会遇到一些性能瓶颈。例如，当数据库中有数百万条记录时，即使添加了索引，某些查询（如对用户名的部分匹配搜索）仍可能导致响应延迟。在API集成中，每次请求都需要花费较长的时间，这不仅影响用户体验，还会增加服务器的负载。


## 解决方案概述


为了应对这些性能挑战，我们可以采用以下策略：


1. **利用客户端缓存**：通过在客户端缓存API响应，减少重复请求，提高响应速度。
2. **使用HTTP缓存头**：利用HTTP的缓存控制头信息（如`ETag`、`Cache-Control`）来管理缓存，实现高效的数据更新检测。
3. **引入辅助库**：使用诸如[Replicant](https://github.com)和[Delta](https://github.com)这样的库，简化缓存的实现方式，降低开发成本。


接下来，我们将详细探讨如何在.NET中实现上述策略，并提供具体的代码示例。


## 客户端缓存概述


### 什么是HTTP缓存头？


HTTP提供了一系列头信息，用于管理缓存行为。这些头信息包括：


* **ETag**：实体标签，用于标识资源的版本。当资源发生变化时，`ETag`也会随之更新。
* **Cache\-Control**：指定缓存策略，如`no-cache`、`max-age`等。
* **Last\-Modified**：指示资源的最后修改时间。


通过合理地设置这些头信息，客户端和服务器可以协同工作，实现高效的缓存机制。


### 浏览器中的缓存机制


当浏览器发送请求时，如果服务器返回了`ETag`或`Last-Modified`等头信息，浏览器会缓存响应。当再次请求相同资源时，浏览器会携带`If-None-Match`或`If-Modified-Since`等条件请求头，询问服务器资源是否更新。如果资源未更新，服务器返回`304 Not Modified`，浏览器直接使用本地缓存的数据。


### 在HTTP客户端中实现缓存


在非浏览器环境（如使用`HttpClient`进行API调用）中，实现类似的缓存机制需要额外的工作。幸运的是，[Replicant](https://github.com):[蓝猫加速器官网](https://baoshidao.com)库提供了便利的解决方案。


## 使用Delta优化服务器端缓存


### 什么是Delta？


Delta是一个开源的.NET库，通过在数据库中添加一个版本列（如`RowVersion`）或数据库自带的追踪数据，结合浏览器或客户端的缓存机制，实现高效的数据更新检测。它可以自动处理`ETag`的生成和验证，简化服务器端的开发工作。


### 如何使用Delta？


1. **安装NuGet包**



```


|  | Install-Package Delta |
| --- | --- |


```
2. **配置中间件**


在`Startup.cs`或`Program.cs`中，添加Delta的中间件：



```


|  | app.UseDelta(); //Delta内置了对EFCore的支持，只要是基于ADO.NET的ORM都可以使用Delta |
| --- | --- |


```
3. **修改数据库**


* **SQL Server**


在相关的数据库表中，添加一个时间戳或版本列，用于跟踪数据的变化。例如，使用`RowVersion`列：



```


|  | [Timestamp] |
| --- | --- |
|  | public byte[] RowVersion { get; set; } |


```
* **PostgreSQL**
启用`track_commit_timestamp`功能


### Delta的工作原理


是否否是响应304 未修改请求根据时间戳从 WebAssembly 和 SQL计算当前 ETag是否有If\-None\-MatchHeader?当前ETag与If\-None\-Match匹配?将当前 ETag添加到响应头* **ETag生成**：Delta会根据数据库中的`RowVersion`或时间戳列，自动生成`ETag`。
* **条件请求**：当客户端发送请求时，携带`If-None-Match`头信息，Delta会根据`ETag`判断数据是否发生变化。
* **响应优化**：如果数据未变化，服务器直接返回`304 Not Modified`，客户端可以使用缓存的数据。


### 使用Delta的优势


* **非侵入式**：无需对现有的API进行大量修改，只需简单配置即可。
* **性能提升**：在数据未变化的情况下，避免了不必要的数据库查询和数据传输。
* **适用性强**：适用于各种类型的API，包括RESTful API、GraphQL等。


## 使用Replicant实现HTTP客户端缓存


### 什么是Replicant？


Replicant是由Simon Cropp开发的一个开源.NET库，用于实现HTTP客户端的缓存机制。它利用了与浏览器相同的HTTP缓存头（如`ETag`、`Cache-Control`），可以缓存HTTP响应，避免重复请求。


### 使用Replicant的步骤


1. **安装NuGet包**


在项目中安装Replicant包：



```


|  | Install-Package Replicant |
| --- | --- |


```
2. **替换HttpClient**


Replicant提供了一个包装的`HttpClient`，在创建客户端时使用`HttpCache`：



```


|  | HttpCache cachedClient = HttpCache.Default; |
| --- | --- |


```
3. **发送请求**


使用`HttpCache`发送请求，与普通的`HttpClient`用法相同：



```


|  | var response = await httpClient.ResponseAsync("https://api.example.com/products"); |
| --- | --- |
|  | var content = await response.Content.ReadAsStringAsync(); |


```


### Replicant的工作原理


* **首次请求**：当第一次请求某个资源时，Replicant会将响应的内容和相关的缓存头信息（如`ETag`）存储在本地缓存中。
* **后续请求**：再次请求相同资源时，Replicant会检查本地缓存的有效性。如果缓存有效，且资源未更新，Replicant会直接返回缓存的内容，避免实际的HTTP请求。


## 综合示例：提升API集成请求性能


下面，我们将结合Replicant和Delta，提供一个完整的示例，展示如何利用客户端和服务器端缓存，提高API的性能。


### 后端代码示例


以下是使用Delta库的后端代码示例。该示例中，我们构建了一个简单的博客应用，包含了博客（Blog）和帖子（Post）两个实体。


#### 1\. 配置`Program.cs`



```


|  | using Delta; |
| --- | --- |
|  | using Microsoft.EntityFrameworkCore; |
|  | using System.Text.Json; |
|  | using System.Text.Json.Serialization; |
|  | using TutorialClientCache.Components; |
|  |  |
|  | var builder = WebApplication.CreateBuilder(args); |
|  | builder.AddNpgsqlDbContext("mydb"); |
|  |  |
|  | // 添加服务到容器 |
|  | builder.Services.AddRazorComponents() |
|  | .AddInteractiveWebAssemblyComponents(); |
|  |  |
|  | builder.Services.AddBootstrapBlazor(); |
|  |  |
|  | var app = builder.Build(); |
|  |  |
|  | // 使用Delta中间件 |
|  | app.UseDelta(); |
|  |  |
|  | // 定义API端点 |
|  | app.MapGet("/posts", async (string? title, BloggingContext db) => |
|  | { |
|  | var query = db.Posts |
|  | .Where(p => title == null || p.Title.Contains(title)) |
|  | .OrderByDescending(p => p.Title) |
|  | .ThenBy(p => p.Content) |
|  | .Take(10); |
|  |  |
|  | return await query.ToListAsync(); |
|  | }); |
|  |  |
|  | // 配置HTTP请求管道 |
|  | if (app.Environment.IsDevelopment()) |
|  | { |
|  | app.UseWebAssemblyDebugging(); |
|  | } |
|  | else |
|  | { |
|  | app.UseExceptionHandler("/Error", createScopeForErrors: true); |
|  | } |
|  |  |
|  | app.UseAntiforgery(); |
|  |  |
|  | app.MapStaticAssets(); |
|  | app.MapRazorComponents() |
|  | .AddInteractiveWebAssemblyRenderMode() |
|  | .AddAdditionalAssemblies(typeof(TutorialClientCache.Client._Imports).Assembly); |
|  |  |
|  | app.Run(); |


```

#### 2\. 定义数据上下文和实体


**数据上下文 `BloggingContext`**



```


|  | public class BloggingContext : DbContext |
| --- | --- |
|  | { |
|  | public BloggingContext(DbContextOptions options) |
|  | : base(options) |
|  | { |
|  | } |
|  |  |
|  | public DbSet Blogs { get; set; } |
|  | public DbSet Posts { get; set; } |
|  | } |


```

**实体类 `Blog` 和 `Post`**



```


|  | public class Blog |
| --- | --- |
|  | { |
|  | public int BlogId { get; set; } |
|  | public string Url { get; set; } |
|  |  |
|  | public List Posts { get; } = new List(); |
|  | } |
|  |  |
|  | public class Post |
|  | { |
|  | public int PostId { get; set; } |
|  | public string Title { get; set; } |
|  | public string Content { get; set; } |
|  |  |
|  | public int BlogId { get; set; } |
|  | public Blog Blog { get; set; } |
|  | } |


```

### 客户端代码示例


由于浏览器`HttpClient`已经内置客户端缓存逻辑，这里提供一个可以在其他Runtime环境内使用缓存的例子：



```


|  | // See https://aka.ms/new-console-template for more information |
| --- | --- |
|  | using Replicant; |
|  | using System.Diagnostics; |
|  |  |
|  | Console.WriteLine("Hello, World!"); |
|  |  |
|  | string url = "http://localhost:5225/posts?title=CSS"; |
|  |  |
|  | //warm up |
|  | await MeasureHttpClientPerformance(url); |
|  | await MeasureHttpCachePerformance(url); |
|  |  |
|  | for (int i = 0; i < 5; i++) |
|  | { |
|  | var httpClientTime = await MeasureHttpClientPerformance(url); |
|  | Console.WriteLine($"HttpClient Time: {httpClientTime} ms"); |
|  | } |
|  | for (int i = 0; i < 5; i++) |
|  | { |
|  | var httpCacheTime = await MeasureHttpCachePerformance(url); |
|  | Console.WriteLine($"HttpCache Time: {httpCacheTime} ms"); |
|  | } |
|  |  |
|  |  |
|  | static async Task<long> MeasureHttpClientPerformance(string url) |
|  | { |
|  | using var httpClient = new HttpClient(); |
|  | var stopwatch = Stopwatch.StartNew(); |
|  |  |
|  | var response = await httpClient.GetAsync(url); |
|  | response.EnsureSuccessStatusCode(); |
|  | var content = await response.Content.ReadAsStringAsync(); |
|  |  |
|  | stopwatch.Stop(); |
|  | return stopwatch.ElapsedMilliseconds; |
|  | } |
|  |  |
|  | static async Task<long> MeasureHttpCachePerformance(string url) |
|  | { |
|  | HttpCache cachedClient = HttpCache.Default; |
|  | var stopwatch = Stopwatch.StartNew(); |
|  |  |
|  | var response = await cachedClient.ResponseAsync(url); |
|  | response.EnsureSuccessStatusCode(); |
|  | var content = await response.Content.ReadAsStringAsync(); |
|  |  |
|  | stopwatch.Stop(); |
|  | return stopwatch.ElapsedMilliseconds; |
|  | } |


```

### 效果


![image](https://img2024.cnblogs.com/blog/3358435/202501/3358435-20250110173130525-1797296321.png)
![image](https://img2024.cnblogs.com/blog/3358435/202501/3358435-20250110173135876-873570712.png)
![image](https://img2024.cnblogs.com/blog/3358435/202501/3358435-20250110173159319-427215224.png)


## 结论


通过结合使用Delta和Replicant，我们可以在.NET应用程序中有效地利用客户端缓存机制，显著提升API的性能。利用`ETag`和`304 Not Modified`等HTTP特性，我们能够减少不必要的网络请求和数据传输，提高应用的响应速度和用户体验。


## 参考链接


* [Replicant GitHub 仓库](https://github.com)
* [Delta GitHub 仓库](https://github.com)
* [HTTP 缓存机制详解](https://github.com)
* [ASP.NET Core 使用中间件](https://github.com)


