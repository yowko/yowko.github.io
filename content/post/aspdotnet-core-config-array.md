---
title: "在 ASP.NET Core Configuration 中使用 array"
date: 2022-01-20T00:30:00+08:00
lastmod: 2022-01-20T00:30:31+08:00
draft: false
tags: ["ASP.NET Core","csharp"]
slug: "aspdotnet-core-config-array"
---

## 在 ASP.NET Core Configuration 中使用 array

這是之前專案遇到的需求：在 config 中設定多個值來供 application 使用，印象中之前有用過但沒找到筆記，順便嘗試一下不同做法，筆記一下

## 基本環境說明

1. macOS Monterey 12.1
2. .NET SDK 6.0.100
3. 範例 config

    ```json
    {
        "ConfigArray": ["Monday","Tuesday","Wednesday","Thursday","Friday","Saturday","Sunday"]
    }
    ```

## 使用方式

1. 使用 `IConfiguration`

    ```cs
    var configs= app.Configuration.GetSection("ConfigArray").Get<string[]>();
    ```

    - 完整程式碼 `Program.cs`

        ```cs
        var builder = WebApplication.CreateBuilder(args);

        // Add services to the container.

        builder.Services.AddControllers();

        // Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
        builder.Services.AddEndpointsApiExplorer();
        builder.Services.AddSwaggerGen();

        var app = builder.Build();

        var configs= app.Configuration.GetSection("ConfigArray").Get<string[]>();

        foreach (var config in configs)
        {
            Console.WriteLine(config);
        }

        // Configure the HTTP request pipeline.
        if (app.Environment.IsDevelopment())
        {
            app.UseSwagger();
            app.UseSwaggerUI();
        }

        app.UseHttpsRedirection();

        app.UseAuthorization();

        app.MapControllers();

        app.Run();
        ```

2. 使用 `Options pattern`

    - 建立對應的 poco

        > 需要讓 porperty name 與 config key 一致才能正確綁定

        ```cs
        namespace ArrayConfig;

        namespace ArrayConfig;

        public class Weekdays
        {
            public string[] ConfigArray { get; set; } = Array.Empty<string>();
        }
        ```

    - 註冊 config

        ```cs
        builder.Services.Configure<Weekdays>(builder.Configuration);
        ```

    - 取用 config array

        ```cs
        var configs=app.Services.GetRequiredService<IOptions<Weekdays>>()?.Value.ConfigArray;
        ```

    - 完整程式碼 `Program.cs`

        ```cs
        using ArrayConfig;
        using Microsoft.Extensions.Options;

        var builder = WebApplication.CreateBuilder(args);

        // Add services to the container.

        builder.Services.AddControllers();

        // Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
        builder.Services.AddEndpointsApiExplorer();
        builder.Services.AddSwaggerGen();

        builder.Services.Configure<Weekdays>(builder.Configuration);

        var app = builder.Build();

        var configs=app.Services.GetRequiredService<IOptions<Weekdays>>()?.Value.ConfigArray;
        foreach (var config in configs)
        {
            Console.WriteLine(config);
        }

        // Configure the HTTP request pipeline.
        if (app.Environment.IsDevelopment())
        {
            app.UseSwagger();
            app.UseSwaggerUI();
        }

        app.UseHttpsRedirection();

        app.UseAuthorization();

        app.MapControllers();

        app.Run();
        ```

## 心得

就程式碼數量來看絕對是直接從 `IConfiguration` 取用完勝，但 `Options pattern` 有強型別的好處也不失為種好做法，但兩者在使用上需要注意個問題，詳細內容請參考 [ASP.NET Core Configuration 中的 array 沒有正確覆寫](/aspdotnet-core-config-array-not-override)

完整程式碼請參考 [yowko/ArrayConfig](https://github.com/yowko/ArrayConfig)

## 參考資訊

1. [Get JSON Array using IConfiguration in ASP.NET Core](https://www.thecodebuzz.com/get-json-array-using-iconfiguration-apsettings-json-asp-net-core/)
2. [Microsoft Docs: Options pattern in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options?view=aspnetcore-6.0&WT.mc_id=DOP-MVP-5002594)
3. [ASP.NET Core Get Json Array using IConfiguration](https://stackoverflow.com/questions/41329108/asp-net-core-get-json-array-using-iconfiguration)
4. [ASP.NET Core Configuration 中的 array 沒有正確覆寫](/aspdotnet-core-config-array-not-override)
5. [yowko/ArrayConfig](https://github.com/yowko/ArrayConfig)
