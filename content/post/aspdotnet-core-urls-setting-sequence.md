---
title: "ASP.NET Core URLs 設定的套用順序"
date: 2021-08-16T12:30:00+08:00
lastmod: 2021-08-16T12:30:00+08:00
draft: false
tags: ["ASP.NET Core"]
slug: "aspdotnet-core-urls-setting-sequence"
---

## ASP.NET Core URLs 設定的套用順序

最近負責的專案需要為多個 ASP.NET Core 專案指定統一的對外 url (主要是 port)，所以花了點時間好好釐清幾個設定方式的優先順序，為了避免下次又要重新回憶，筆記一下

## 基本環境說明

1. macOS Big Sur 11.5.1
2. .NET Core SDK 5.0.202
3. ASP.NET Core Web Api 預設專案範本

## 套用順序

驗證方式是逐一加上下列設定，可以發現後者會覆蓋前者設定值

1. 預設值：`http://localhost:5000;https://localhost:5001`

    ![0default](https://user-images.githubusercontent.com/3851540/129527030-42a97294-efc4-48d6-b835-a6a8e3a2fcc1.png)

2. launchSettings.json 中的 `applicationUrl`

    ```json
    "profiles": {
        "WebApplication": {
            "applicationUrl": "https://localhost:6001;http://localhost:6000"
        }
    }
    ```

    ![1applicationUrl](https://user-images.githubusercontent.com/3851540/129527038-5c93838f-6bde-4f8c-8996-e717a2b8cbf7.png)

3. environment `ASPNETCORE_URLS`

    > environment 設定方式很多種，以下示範使用 launchSettings.json

    ```json
    "profiles": {
        "WebApplication": {
            "applicationUrl": "https://localhost:6001;http://localhost:6000",
            "environmentVariables":{
                "ASPNETCORE_URLS": "https://localhost:7001;http://localhost:7000"
            }
        }
    }
    ```

    ![2env](https://user-images.githubusercontent.com/3851540/129527041-a8d30ee1-0ce6-4186-ad01-860d994a95a5.png)

4. WebHostBuilder 的 `UseUrls`

    > Program.cs 中的 CreateHostBuilder 方法

    ```cs
     public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                    webBuilder.UseUrls("http://localhost:8000", "https://localhost:8001");
                });
    ```

    ![3useurls](https://user-images.githubusercontent.com/3851540/129527042-5733fff7-fe19-4417-9edf-d11afe5e1b05.png)

5. dotnet run 參數 `--urls`

    ```bash
    dotnet run --urls http://localhost:9000;https://localhost:9001 KestrelEndpointsDemo.dll
    ```

    ![4runurls](https://user-images.githubusercontent.com/3851540/129527044-d0bde957-98fc-4c7c-b5c6-63645f3461df.png)

6. WebHostBuilder 的 `UseKestrel`

    > Program.cs 中的 CreateHostBuilder 方法

    ```cs
    public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>().UseKestrel(opts =>
                    {
                        opts.ListenLocalhost(10000, opts =>opts.Protocols= HttpProtocols.Http1);
                        opts.ListenLocalhost(10001, opts => opts.UseHttps());
                    });
                    // UseUrls 相關設定會被 UseKestrel 覆蓋
                    webBuilder.UseUrls("http://localhost:8000", "https://localhost:8001");
                });
    ```

    ![5usekestrel](https://user-images.githubusercontent.com/3851540/129527045-c336a638-3ab6-4ea5-967d-5b38c68dad34.png)

- appsettings.json 的 `Kestrel:Endpoints:Http:Url` 與 `Kestrel:Endpoints:Https:Url`

    > 這個設定很特別，不會覆蓋先前的任何設定，而是強制 application 依設定值加開監聽 url

    ```json
    {
        "Kestrel": {
            "Endpoints": {
              "Http": {
                "Url": "http://localhost:20000"
              },
              "Https": {
                "Url": "https://localhost:20001"
              }
            }
          }
    }
    ```

    ![6kestrelendpoint](https://user-images.githubusercontent.com/3851540/129527048-c185d9ef-f835-43d7-8444-fba73c6ebb6f.png)

## 心得

過去印象一直停留在官網的說明 [Microsoft Docs:設定 ASP.NET Core Kestrel web 伺服器的端點](https://docs.microsoft.com/zh-tw/aspnet/core/fundamentals/servers/kestrel/endpoints?view=aspnetcore-5.0&WT.mc_id=DOP-MVP-5002594) 中，偶然之間看到 Andrew Lock 的 [5 ways to set the URLs for an ASP.NET Core app](https://andrewlock.net/5-ways-to-set-the-urls-for-an-aspnetcore-app/) 主要是針對 ASP.NET Core 1.0，所以興起了確認 .NET 5 的行為是否一致的想法，不確定是微軟調整了實作方式還是 macOS 行為不同，但目前測試起來是跟 官網說明 與 Andrew Lock 文章提到的略有不同

完整程式碼請參考：[yowko/KestrelEndpointsDemo](https://github.com/yowko/KestrelEndpointsDemo)

## 參考資訊

1. [Microsoft Docs:設定 ASP.NET Core Kestrel web 伺服器的端點](https://docs.microsoft.com/zh-tw/aspnet/core/fundamentals/servers/kestrel/endpoints?view=aspnetcore-5.0&WT.mc_id=DOP-MVP-5002594)
2. [5 ways to set the URLs for an ASP.NET Core app](https://andrewlock.net/5-ways-to-set-the-urls-for-an-aspnetcore-app/)
