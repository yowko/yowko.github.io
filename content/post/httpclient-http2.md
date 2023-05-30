---
title: "如何在 .NET6 上指定 HttpClient 使用 HTTP/2"
date: 2023-05-30T00:30:00+08:00
lastmod: 2023-05-30T00:30:31+08:00
draft: false
tags: ["csharp","dotnet6","dotnet","httpclient"]
slug: "httpclient-http2"
---

## 如何在 .NET6 上指定 HttpClient 使用 HTTP/2

之前筆記 [gRPC 在 ASP.NET Core 7 的 JSON 轉碼功能](/grpc-aspdotnetcore7-json) 紀錄到如何使用 ASP.NET Core 7 加入的 JSON 轉碼功能：可以讓 gRPC service 也可以透過 rest api 的方式來呼叫，所以就有了 HttpClient 直接使用 rest api 呼叫 gRPC 服務的需求，這才發現 HttpClient 預設使用 `HTTP/1.1` protocol，今天就來紀錄一下該如何在 .NET6 上指定 HttpClient 使用 HTTP/2

HTTP/2 的出現是為了解決 HTTP/1.1 的幾個問題，其中一個就是 head-of-line blocking 限制而造成的效能問題，但在設定前額外了解一下 HTTP/2 的版本差異

- h2

    `h2` 指的是使用傳輸層安全協議 (TLS) 的 HTTP/2 連線。為了通過 TLS 建立 HTTP/2 連線，application 需要依賴  TLS application-layer protocol negotiation (ALPN) extension。 這種機制簡化了 negotiation 建立，因為 HTTP/2 negotiation 直接整合到 TLS negotiation 中。這就是為什麼大多數瀏覽器僅支持使用 TLS 的 HTTP/2 的原因之一，因為它比基於明文的 HTTP/2 更容易實現。

- h2c

    `h2c` 指的是基於明文的 HTTP/2 連接，意思是通過簡單的 TCP 連接。 基本上，此連接中的所有數據都在常規 TCP 連接中以明文形式交換。 在這種情況下，HTTP 版本 negotiation 是通過利用 HTTP/1.1 升級標頭實現的。 連接以 HTTP/1.1 連接開始，然後客戶端向服務器發送升級標頭以將連接升級到 HTTP/2。 無法保證服務器會接受升級連接。 因此，建立 h2c 連接要比建立 h2 連接複雜。

從 .NET 5.0 開始，Microsoft 為開發人員提供了更大的靈活性，可以將 HttpClient 配置為使用 h2 或 h2c。 此外，也可以選擇決定如何處理連接升級或降級程序。 以下是可用於此目的的 2 個屬性：

- DefaultRequestVersion

    指定預設的 HTTP 版本。 預設值為 HTTP 1.1，因此如果要使用協議版本 2，則需要將其更改為 `HttpVersion.Version20`。

- DefaultVersionPolicy

    指定要使用的 HttpVersionPolicy。 此屬性有 3 個選項：
    1. RequestVersionOrLower
    2. RequestVersionOrHigher
    3. RequestVersionExact

    預設值為 RequestVersionOrLower，這意味著 HttpClient 將嘗試使用請求的版本，如果不行，將降級到較低版本。

上述內容翻譯自 [How to use HTTP/2 with HttpClient in .NET 6.0](https://www.siakabaro.com/use-http-2-with-httpclient-in-net-6-0/)，如有疑問建議直接看原文

## 基本環境說明

1. macOS Ventura 13.4
2. .NET SDK 6.0.400
3. JetBrains Rider 2023.1.2

## 設定方式

1. 建立 HttpClient 時，指定

    - `DefaultRequestVersion` 為 `HttpVersion.Version20`
    - `DefaultVersionPolicy` 為 `HttpVersionPolicy.RequestVersionOrLower`

        > 這允許 HttpClient 在網站不支援 `HTTP/2` 可以改用 `HTTP/1.1` 進行連線

2. 程式碼

    ```cs
    using System.Net;

    HttpClient myHttpClient = new HttpClient
    {
        DefaultRequestVersion = HttpVersion.Version20,
        DefaultVersionPolicy = HttpVersionPolicy.RequestVersionOrLower
    };
    string requestUrl = "https://http2.pro/api/v1";
    try
    {
        Console.WriteLine($"GET {requestUrl}.");
        HttpResponseMessage response = await myHttpClient.GetAsync(requestUrl);
        response.EnsureSuccessStatusCode();
        Console.WriteLine($"Response HttpVersion: {response.Version}");
        string responseBody = await response.Content.ReadAsStringAsync();
        Console.WriteLine($"Response Body Length is: {responseBody.Length}");
        Console.WriteLine($"------------Response Body------------");
        Console.WriteLine(responseBody);
        Console.WriteLine($"------------End of Response Body------------");
    }
    catch (HttpRequestException e)
    {
        Console.WriteLine($"HttpRequestException : {e.Message}");
    }
    Console.WriteLine($"Press Enter to exit....");
    Console.ReadLine();
    ```

3. 實際效果

    - 預設值

        ![1default](https://github.com/yowko/picsbed/assets/3851540/8ccb3603-375b-4d66-a42e-d55e97e969ef)

    - 指定 `HttpVersion.Version20`

        ![2http2](https://github.com/yowko/picsbed/assets/3851540/1939e547-8186-4d2b-b684-b57971897469)

## 心得

主要內容是從 [How to use HTTP/2 with HttpClient in .NET 6.0](https://www.siakabaro.com/use-http-2-with-httpclient-in-net-6-0/) `參考`來的，但 url 的部份則是從保哥文章 [設定 .NET 的 HttpClient 使用 HTTP/2 通訊協定發出 HTTP 要求](https://blog.miniasp.com/post/2023/01/17/How-to-use-HTTP-Version-2-with-HttpClient) 中來的，保哥提供的 url `https://http2.pro/api/v1` 回應比較直覺簡短，使用上方便很多更加一目瞭然

## 參考資訊

1. [How to use HTTP/2 with HttpClient in .NET 6.0](https://www.siakabaro.com/use-http-2-with-httpclient-in-net-6-0/)
2. [設定 .NET 的 HttpClient 使用 HTTP/2 通訊協定發出 HTTP 要求](https://blog.miniasp.com/post/2023/01/17/How-to-use-HTTP-Version-2-with-HttpClient)
