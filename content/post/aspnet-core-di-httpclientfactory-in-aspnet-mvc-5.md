---
title: "在 ASP.NET MVC 5 中使用 ASP.NET Core Dependency Injection 與 HttpClientFactory"
date: 2019-01-14T23:45:00+08:00
lastmod: 2020-06-01T09:44:30+08:00
draft: false
tags: ["C#","dotnet core","ASP.NET MVC"]
slug: "aspnet-core-di-httpclientfactory-in-aspnet-mvc-5"
---
# 在 ASP.NET MVC 5 中使用 ASP.NET Core Dependency Injection 與 HttpClientFactory
習慣了 ASP.NET Core DI 的寫法後，回到 ASP.NET MVC 5 後突然覺得不太適應，沒有 HttpClientFactory 都覺得 HttpClient 好像很容易出錯，於是試著研究研究，順手紀錄一下，

仔細想想好像沒什麼人會這麼用，要嘛直上 ASP.NET Core 要嘛直接用過去 ASP.NET MVC 的 HttpClient 寫法並留意不要踩到每次 dispose 跟 dns 未立即生效的問題似乎也不是很嚴重


## 基本環境說明
1. .NET Framework 4.7.2
2. ASP.NET MVC 5.2.4
3. OWIN 4
4. Microsoft.Extensions.DependencyInjection 2.2

## 1. 安裝 `Microsoft.Extensions.DependencyInjection` NuGet 套件
- Package Manager

    ```
    Install-Package Microsoft.Extensions.DependencyInjection 
    ``` 
- ~~.NET CLI~~

    > <span style="color:red">update:</span> 2020/06/01 經保哥提醒，才發現我從別篇筆記抄 install package template 過來，只改了 package name，沒留意到 .NET Framework 不能使用 .NET CLI 安裝套件

    ~~dotnet add package Microsoft.Extensions.DependencyInjection~~

## 2. 在 `Startup.cs` 中加入 `ConfigureServices` 方法

```cs
public void ConfigureServices(IServiceCollection services)
{
}
```

## 3. 建立 `DefaultDependencyResolver` 的類別實作 `IDependencyResolver`

> 建立依賴項目的解析器

```cs
public class DefaultDependencyResolver : IDependencyResolver
{
    protected IServiceProvider serviceProvider;

    public DefaultDependencyResolver(IServiceProvider serviceProvider)
    {
        this.serviceProvider = serviceProvider;
    }

    public object GetService(Type serviceType)
    {
        return this.serviceProvider.GetService(serviceType);
    }

    public IEnumerable<object> GetServices(Type serviceType)
    {
        return this.serviceProvider.GetServices(serviceType);
    }
}
```

## 4. 修改 `Startup.cs` 的 `Configuration` 方法
- 修改前
    
    ```cs
    public void Configuration(IAppBuilder app)
    {
        ConfigureAuth(app);
    }
    ```
- 修改後

    ```cs
    public void Configuration(IAppBuilder app)
    {
        var services = new ServiceCollection();
        ConfigureAuth(app);
        ConfigureServices(services);
        var resolver = new DefaultDependencyResolver(services.BuildServiceProvider());
        DependencyResolver.SetResolver(resolver);
    }
    ```

## 5. 建立 `ServiceProviderExtensions` 類別

> 用來將 Controller 註冊進 DI container 中為一服務

```cs
public static class ServiceProviderExtensions
{
   public static IServiceCollection AddControllersAsServices(this IServiceCollection services,
      IEnumerable<Type> controllerTypes)
   {
      foreach (var type in controllerTypes)
      {
         services.AddTransient(type);
      }

      return services;
   }
}
```

## 6. 實際在 `ConfigureServices` 方法中將 Controller 註冊為 service 

```cs
services.AddControllersAsServices(typeof(Startup).Assembly.GetExportedTypes()
   .Where(t => !t.IsAbstract && !t.IsGenericTypeDefinition)
   .Where(t => typeof(IController).IsAssignableFrom(t) 
      || t.Name.EndsWith("Controller", StringComparison.OrdinalIgnoreCase)));
```

## 7. 使用範例：HttpClientFactory
1. 安裝 `Microsoft.Extensions.Http` NuGet 套件

    > 這邊使用 `Microsoft.Extensions.Http 2.2.0`

    - Package Manager

        ```
        Install-Package Microsoft.Extensions.Http
        ``` 
    - .NET CLI

        ```
        dotnet add package Microsoft.Extensions.Http
        ``` 
2. 在 `ConfigureServices` 方法中註冊 HttpClient

    ```cs
    services.AddHttpClient();
    ``` 
3. 在使用處的 constructor 加入 `IHttpClientFactory`

    ```cs
    private readonly IHttpClientFactory _clientFactory;

    public HomeController(IHttpClientFactory clientFactory)
    {
        _clientFactory = clientFactory;
    }
    ```
4. 實際呼叫

    ```cs
    var client = _clientFactory.CreateClient();
    client.BaseAddress = new Uri("https://blog.yowko.com");
    var result = await client.GetAsync("/");
    ViewBag.BlogContent = await result.Content.ReadAsStringAsync();
    ```

## 心得
其實想在 .NET Framework 環境使用 HttpClientFactory 很久了，主要是想避開 HttpClient 過去的一些問題，但近期工作專案都改用 .NET Core 相較之下花在 .NET Framework 這邊的心力少了許多，這次剛好趁著想要好好紀錄 HttpClientFactory 的相關用法，特別找出適用的做法，雖然用處有限但還是滿有趣的


# 參考資訊
1. [Integrating ASP.NET Core Dependency Injection in MVC 4](https://scottdorman.github.io/2016/03/17/integrating-asp.net-core-dependency-injection-in-mvc-4/)