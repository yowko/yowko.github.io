---
title: "ASP.NET Core 中 AddMvc() 與 AddMvcCore() 的差別"
date: 2019-06-08T21:30:00+08:00
lastmod: 2021-11-03T21:30:31+08:00
draft: false
tags: ["ASP.NET Core"]
slug: "aspdotnet-core-addmvc-addmvccore"
---

## ASP.NET Core 中 AddMvc() 與 AddMvcCore() 的差別

ASP.NET Core 將過去 ASP.NET MVC 與 ASP.NET Web API 兩套 framework 整合在一起，對於開發人員是種福音：不用再想到底該引用哪個 NameSpace、不用再為該繼承哪個類別煩惱、不用再記兩種開發模式(到底該用 action filter 還是 MessageHandler)，不過實際上在 ASP.NET Core 中還是有 `AddMvc()` 與 `AddMvcCore()` 兩種服務註冊方式

其中差別剛看到時實在沒什麼想法，幸虧 ASP.NET Core 程式碼都是 open source，覺得有疑問馬上可以翻一下，今天就簡單紀錄一下自己看到的差異

## 基本環境說明

1. macOS Mojave 10.14.5
2. .NET Core SDK 2.2.107 (.NET Core Runtime 2.2.5)

## AddMvcCore

完整程式碼請參考 [AddMvcCore](https://github.com/aspnet/AspNetCore/blob/258d34e3828a1870a16d13cd3c62d1b7a65acc4a/src/Mvc/Mvc.Core/src/DependencyInjection/MvcCoreServiceCollectionExtensions.cs#L47)

程式碼節錄如下：

```cs
public static IMvcCoreBuilder AddMvcCore(this IServiceCollection services)
{
    if (services == null)
    {
        throw new ArgumentNullException(nameof(services));
    }
    var partManager = GetApplicationPartManager(services);
    services.TryAddSingleton(partManager);
    ConfigureDefaultFeatureProviders(partManager);
    ConfigureDefaultServices(services);
    AddMvcCoreServices(services);
    var builder = new MvcCoreBuilder(services, partManager);
    return builder;
}
```

## AddMvc

完整程式碼請參考 [AddMvc](https://github.com/aspnet/AspNetCore/blob/0303c9e90b5b48b309a78c2ec9911db1812e6bf3/src/Mvc/Mvc/src/MvcServiceCollectionExtensions.cs#L27)

程式碼節錄如下：

```cs
public static IMvcBuilder AddMvc(this IServiceCollection services)
{
    if (services == null)
    {
        throw new ArgumentNullException(nameof(services));
    }
    services.AddControllersWithViews();
    return services.AddRazorPages();
}
....

public static IMvcBuilder AddControllersWithViews(this IServiceCollection services)
{
    if (services == null)
    {
        throw new ArgumentNullException(nameof(services));
    }
    var builder = AddControllersWithViewsCore(services);
    return new MvcBuilder(builder.Services, builder.PartManager);
}

private static IMvcCoreBuilder AddControllersWithViewsCore(IServiceCollection services)
{
    var builder = AddControllersCore(services)
        .AddViews()
        .AddRazorViewEngine()
        .AddCacheTagHelper();
    AddTagHelpersFrameworkParts(builder.PartManager);
    return builder;
}
....

private static IMvcCoreBuilder AddControllersCore(IServiceCollection services)
{
    // This method excludes all of the view-related services by default.
    return services
        .AddMvcCore()
        .AddApiExplorer()
        .AddAuthorization()
        .AddCors()
        .AddDataAnnotations()
        .AddFormatterMappings();
}
```

## trace code

從 [AddMvc](https://github.com/aspnet/AspNetCore/blob/v2.2.5/src/Mvc/Mvc/src/MvcServiceCollectionExtensions.cs) 追起就可以發現最後殊途同歸：最後還是使用到 [AddMvcCore](https://github.com/aspnet/AspNetCore/blob/258d34e3828a1870a16d13cd3c62d1b7a65acc4a/src/Mvc/Mvc.Core/src/DependencyInjection/MvcCoreServiceCollectionExtensions.cs#L47)

[services.AddControllersWithViews();](https://github.com/aspnet/AspNetCore/blob/0303c9e90b5b48b309a78c2ec9911db1812e6bf3/src/Mvc/Mvc/src/MvcServiceCollectionExtensions.cs#L34)

-->
[public static IMvcBuilder AddControllersWithViews(this IServiceCollection services)](https://github.com/aspnet/AspNetCore/blob/0303c9e90b5b48b309a78c2ec9911db1812e6bf3/src/Mvc/Mvc/src/MvcServiceCollectionExtensions.cs#L176)

-->
[var builder = AddControllersWithViewsCore(services);](https://github.com/aspnet/AspNetCore/blob/0303c9e90b5b48b309a78c2ec9911db1812e6bf3/src/Mvc/Mvc/src/MvcServiceCollectionExtensions.cs#L183)

-->
[private static IMvcCoreBuilder AddControllersWithViewsCore(IServiceCollection services)](https://github.com/aspnet/AspNetCore/blob/0303c9e90b5b48b309a78c2ec9911db1812e6bf3/src/Mvc/Mvc/src/MvcServiceCollectionExtensions.cs#L228)

-->
[var builder = AddControllersCore(services)](https://github.com/aspnet/AspNetCore/blob/0303c9e90b5b48b309a78c2ec9911db1812e6bf3/src/Mvc/Mvc/src/MvcServiceCollectionExtensions.cs#L230)

-->
[private static IMvcCoreBuilder AddControllersCore(IServiceCollection services)](https://github.com/aspnet/AspNetCore/blob/0303c9e90b5b48b309a78c2ec9911db1812e6bf3/src/Mvc/Mvc/src/MvcServiceCollectionExtensions.cs#L141)

-->
[.AddMvcCore()](https://github.com/aspnet/AspNetCore/blob/0303c9e90b5b48b309a78c2ec9911db1812e6bf3/src/Mvc/Mvc/src/MvcServiceCollectionExtensions.cs#L145)

## 心得

以程式碼來看 `AddMvc` 比 `AddMvcCore` 多了下列內容

1. AddRazorPages
2. AddViews
3. AddRazorViewEngine
4. AddCacheTagHelper
5. AddApiExplorer
6. AddAuthorization
7. AddCors
8. AddDataAnnotations
9. AddFormatterMappings
10. AddJsonFormatters

大家在使用時可以評估看看用哪個方法比較合適

## 參考資訊

1. [ASP.NET Core AddMvc() Vs AddMvcCore()](http://ajaynallanagula.blogspot.com/2018/03/aspnet-core-addmvc-vs-addmvccore.html)
