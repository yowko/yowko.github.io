---
title: "關於 ASP.NET Core ListenAnyIP"
date: 2021-08-18T12:30:00+08:00
lastmod: 2021-08-18T12:30:00+08:00
draft: false
tags: ["ASP.NET Core"]
slug: "aspdotnet-core-listenanyip"
---

## 關於 ASP.NET Core ListenAnyIP

之前筆記 [ASP.NET Core URLs 設定的套用順序](/aspdotnet-core-urls-setting-sequence/) 紀錄到 ASP.NET Core URL 幾種設定方式的套用順序，其中 WebHostBuilder 的 `UseKestrel` 方法，筆記使用的是 `opts.ListenLocalhost(10000, opts =>opts.Protocols= HttpProtocols.Http1);` 與 `opts.ListenLocalhost(10001, opts => opts.UseHttps());`，相信一定也看過 `opts.ListenAnyIP(12345);` 這樣指定任意 ip 的寫法，今天就來紀錄一下該如何使用

## 基本環境說明

1. macOS Big Sur 11.5.1
2. .NET Core SDK 5.0.202
3. ASP.NET Core Web Api 預設專案範本

    > 這邊會延續之前筆記 [ASP.NET Core URLs 設定的套用順序](/aspdotnet-core-urls-setting-sequence/) 的內容

    - Program.cs 程式碼

        ```cs
        public class Program
        {
            public static void Main(string[] args)
            {
                CreateHostBuilder(args).Build().Run();
            }

            public static IHostBuilder CreateHostBuilder(string[] args) =>
                Host.CreateDefaultBuilder(args)
                    .ConfigureWebHostDefaults(webBuilder =>
                    {
                        webBuilder.UseStartup<Startup>().UseKestrel(opts =>
                        {
                            opts.ListenAnyIP(12345);
                        });
                    });
        }
        ```

## 發現與使用方式

1. 使用 `opts.ListenAnyIP(12345);` 會 listen `http://[::]:12345`

    ![1listen](https://user-images.githubusercontent.com/3851540/129881759-ea1c984a-b324-4887-9ec2-bf6939a4d17c.png)

2. 關於 `ListenAnyIP`

    > 完整內容可以參考官方文件 [Microsoft Docs:KestrelServerOptions.ListenAnyIP Method](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.server.kestrel.core.kestrelserveroptions.listenanyip?view=aspnetcore-5.0&WT.mc_id=DOP-MVP-5002594)，但請務必看英文版，中文意思完全不對了
    >
    > - 中文版
    > ![3zh](https://user-images.githubusercontent.com/3851540/129882670-2d57c06b-843b-40ed-a49c-d4cf15f3dcec.png)
    > - 英文版
    > ![4en](https://user-images.githubusercontent.com/3851540/129882687-5c430a51-2064-487a-bf5d-9c898fb93494.png)

    預設使用 IPv6 的 `[::]` 監聽所有 IP，如果 IPv6 無法使用的話就使用 IPv4 的 `0.0.0.0` 監聽所有 IP

3. 使用 IPv4 無法開啟

    同事反應他使用 `localhost`、`127.0.0.1`、`內網 ip` 皆無法正確開啟網頁，但我自己是都可以正常開啟

4. 使用 IPv6 開啟

    這個比較偏向對於 IPv6 格式的理解，IPv6 的 localhost 為 `[::1]`，想要透過 IPv6 開啟 12345 port 中的 swagger 就是透過 `http://[::1]:12345/swagger/index.html`

    ![2ipv6](https://user-images.githubusercontent.com/3851540/129881771-6c3a55e0-3afe-4dda-84f8-ab65a3f9e00b.png)

## 心得

我沒找出為什麼在使用 ListenAnyIP 的預設監聽 IPv6 所有 ip 的設定下，我可以直接使用 IPv4 的 ip 直接存取，也沒找出為什麼同事無法開啟，最大的收獲可能是搞清楚了 IPv6 的 localhost 是 `[::1]` 哈哈

## 參考資訊

1. [ASP.NET Core URLs 設定的套用順序](/aspdotnet-core-urls-setting-sequence/)
2. [Microsoft Docs:KestrelServerOptions.ListenAnyIP Method](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.server.kestrel.core.kestrelserveroptions.listenanyip?view=aspnetcore-5.0&WT.mc_id=DOP-MVP-5002594)
3. [How do I get the kestrel web server to listen to non-localhost requests?](https://stackoverflow.com/a/34221690)
4. [What is IPV6 for localhost and 0.0.0.0?](https://stackoverflow.com/a/46711717)
