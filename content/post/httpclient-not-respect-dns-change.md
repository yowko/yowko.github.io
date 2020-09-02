---
title: "HttpClient 無法反應 DNS 異動的解決方式"
date: 2019-01-05T23:45:00+08:00
lastmod: 2018-01-05T23:44:30+08:00
draft: false
tags: ["C#","Benchmark"]
slug: "httpclient-not-respect-dns-change"
---
# HttpClient 無法反應 DNS 異動的解決方式
之前筆記 [探討 HttpClient 可能的問題](https://blog.yowko.com/httpclient-issue/) 提到使用 HttpCLient 時避免 socket 耗盡的方式就是只建立一個 HttpClient instance (透過 static or singleton)，但這樣的方式卻會造成 DNS 紀錄出現變動被忽略進而影響系統正確運行

筆記中有提到可以透過將 HttpClient 的 `DefaultRequestHeaders.ConnectionClose` 屬性設定為 `true`，也就是將 HTTP 的 keep-alive header 設為 `false`，讓 socket 在每次處理完 request 即關閉，這幾天查資料時發現還有其他做法可以使用，一併紀錄一下

## 前提設定
1. 使用 staticHttpClient instance 來取得 `https://blog.yowko.com/` 資料

    ```cs
    public class StaticHttpClientService
    {
        private static readonly HttpClient _httpClient;
        static StaticHttpClientService()
        {
            var baseUri = new Uri("http://blog.yowko.com");
            _httpClient = new HttpClient();
            _httpClient.BaseAddress = baseUri;
        }

        public HttpClient HttpclientInstance = _httpClient;
    }
    ```
2. 透過修改 hosts 檔案來模擬 DNS 修改

    > 修改方式請參考 [在 Windows 環境將特定網址指向不同 IP](https://blog.yowko.com/windows-host-file)

3. 使用環境
    - Visual Studio 2017  15.9.4
    - .NET Framework 4.7.2
    - BenchmarkDotNet 0.11.3

        > 為避免外部網路干擾，透過修改 hosts file 將 `blog.yowko.com` 指向 127.0.0.1

## 解決方式
1. dispose HttpClient

    > 每次 request 結束即 dispose 的做法存在與 TCP 協定及 OS 層實作的效能問題，應重複使用 HttpClient 並自行管理 dispose 時間

    ```cs
    public class StaticHttpClientService
    {
        private static HttpClient _httpClient;
        private static DateTime _TTL;
        private static void createInstance()
        {
            _httpClient = new HttpClient();
            _httpClient.BaseAddress = new Uri("http://blog.yowko.com");
            //設定 dispose HttpClient 的時間
            _TTL = DateTime.UtcNow.AddMinutes(1);
        }
        static StaticHttpClientService()
        {
            createInstance();
        }
        public HttpClient HttpclientInstance
        {
            get
            {
                if (DateTime.UtcNow > _TTL)
                {
                    _httpClient.Dispose();
                    //重新建立 HttpClient
                    createInstance();
                }

                return _httpClient;
            }
        }
    }
    ```

2. 關閉 socket 連線

    將 HttpClient 的 `DefaultRequestHeaders.ConnectionClose` 屬性設定為 `true`，也就是將 HTTP 的 keep-alive header 設為 `false`，讓 socket 在每次處理完 request 即關閉

    ```cs
    public class StaticHttpClientService
    {
        private static readonly HttpClient _httpClient;
        static StaticHttpClientService()
        {
            var baseUri = new Uri("http://blog.yowko.com");
            _httpClient = new HttpClient();
            _httpClient.BaseAddress = baseUri;
            _httpClient.DefaultRequestHeaders.ConnectionClose = true;
        }

        public HttpClient HttpclientInstance = _httpClient;
    }
    ```

3. 設定釋放 socket 連線時間

    > 避免每次皆關閉 socket 而造成無謂的效能損耗
    >- 修改 `ConnectionLeaseTimeout` 時間 : 用來管理 TCP socket 保持開啟的時間，預設為 `-1` 永遠開啟
    >- 修改 `DnsRefreshTimeout` 時間: 用來管理 DNS 更新間隔，預設為 `12000` (兩分鐘)

    ```cs
    public class StaticHttpClientService
    {
        private static readonly HttpClient _httpClient;
        static StaticHttpClientService()
        {
            var baseUri = new Uri("http://blog.yowko.com");
            _httpClient = new HttpClient();
            _httpClient.BaseAddress = baseUri;
            //設定 1 分鐘沒有活動即關閉連線，預設 -1 (永不關閉)
            ServicePointManager.FindServicePoint(baseUri)
            .ConnectionLeaseTimeout = (int)TimeSpan.FromMinutes(1).TotalMilliseconds;
            //設定 1 分鐘更新 DNS，預設 12000 (2 分鐘)
            ServicePointManager.DnsRefreshTimeout = (int)TimeSpan.FromMinutes(1).TotalMilliseconds; ;
        }

        public HttpClient HttpclientInstance = _httpClient;
    }
    ```

## 效能比較

1. 第一次

    Method |     Mean |     Error |    StdDev |
    ----------------------- |---------:|----------:|----------:|
    Dispose | 407.8 us |  7.849 us |  8.060 us |
    ConnectionClose | 691.4 us | 15.557 us | 45.869 us |
    ConnectionLeaseTimeout | 423.3 us |  8.316 us | 16.609 us |

    ![test1](https://user-images.githubusercontent.com/3851540/50735047-20f9c900-11e3-11e9-82b5-fb197e1a41f0.png)

2. 第二次

    Method |     Mean |     Error |    StdDev |
    ----------------------- |---------:|----------:|----------:|
    Dispose | 410.3 us |  6.086 us |  5.395 us |
    ConnectionClose | 732.0 us | 14.589 us | 21.384 us |
    ConnectionLeaseTimeout | 415.8 us |  5.225 us |  4.887 us |

    ![test2](https://user-images.githubusercontent.com/3851540/50735048-20f9c900-11e3-11e9-9165-ac68585d78fa.png)    

3. 第三次

    Method |     Mean |     Error |    StdDev |
    ----------------------- |---------:|----------:|----------:|
    Dispose | 416.6 us |  2.951 us |  2.760 us |
    ConnectionClose | 759.2 us |  7.041 us |  6.586 us |
    ConnectionLeaseTimeout | 422.3 us | 11.351 us | 14.355 us |

    ![test3](https://user-images.githubusercontent.com/3851540/50735046-20613280-11e3-11e9-97b9-a567a1866879.png)


## 心得
以效能數據來看，雖說三者執行時間都非常快，但每次 request 都關閉 socket 執行時間是另外兩者的 1.8 倍以上，我相信這在高流量環境下是不被允許的，差距太大了

至於自行管理 HttpClient instance 及使用 ServicePointManager 兩者差距就微乎其微了

# 參考資訊
1. [探討 HttpClient 可能的問題](https://blog.yowko.com/httpclient-issue/)
2. [Singleton HttpClient doesn't respect DNS changes](https://github.com/dotnet/corefx/issues/11224#issuecomment-271195770)
3. [system.net.http.httpclient does not respect dns update in a timely manner](https://social.msdn.microsoft.com/Forums/sqlserver/en-US/39af7077-fbb5-4a8c-a4b9-42a73aa96b8a/systemnethttphttpclient-does-not-respect-dns-update-in-a-timely-manner?forum=wcf)
4. [Beware of the .NET HttpClient](http://www.nimaara.com/2016/11/01/beware-of-the-net-httpclient/)
5. [ServicePoint.ConnectionLeaseTimeout Property](https://docs.microsoft.com/en-us/dotnet/api/system.net.servicepoint.connectionleasetimeout?redirectedfrom=MSDN&view=netframework-4.7.2&WT.mc_id=DOP-MVP-5002594#System_Net_ServicePoint_ConnectionLeaseTimeout)
6. [ServicePointManager.DnsRefreshTimeout Property](https://docs.microsoft.com/en-us/dotnet/api/system.net.servicepointmanager.dnsrefreshtimeout?view=netframework-4.7.2&WT.mc_id=DOP-MVP-5002594)