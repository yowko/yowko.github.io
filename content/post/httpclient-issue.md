---
title: "探討 HttpClient 可能的問題"
date: 2018-12-10T23:45:00+08:00
lastmod: 2020-12-11T23:44:30+08:00
draft: false
tags: ["C#"]
slug: "httpclient-issue"
---
# 探討 HttpClient 可能的問題
印象中前幾年曾經看過有文章提到 HttpClient 雖然是 disposable 但透過 `using` 來使用 HttpClient 卻反而可能出現問題，當時覺得網路文章多數仍是使用 `using`，於是我抱著可能是特殊情境所造成的少數問題，沒有特別留意，最近在看 .NET Core 相關應用時，發現 .NET Core 已針對 HttpClient 使用另外打造新的類別，也讓我重新回想起當年的文章，就趁著這個機會來模擬看看到底會出現什麼問題吧

    
## 原始程式碼
1. 程式碼

    ```cs
    void Main()
    {
        "Starting connections".Dump();
        // 執行多次 http request 取資料
        for (int i = 0; i < 10; i++)
        {
            using (var client = new HttpClient())
            {
                //設定 httpclient 的 base uri
                client.BaseAddress = new Uri("http://localhost");
                //取得 url 內容
                var result = client.GetAsync("/").GetAwaiter().GetResult();
                result.StatusCode.Dump();
            }
        }
        "Connections done".Dump();
    }
    ```
2. 執行結果

    ![1sourceresult](https://user-images.githubusercontent.com/3851540/49747856-4af0c600-fcdf-11e8-815d-280b2f4b6afd.png)

## 可能問題與解決方式
### 1. 造成 `通訊端耗盡 (sockets exhaustion)`
    
> using 區段工作完成後，會呼叫 `dispose` 方法來清除物件，根據於 TCP 通訊協定的內容，在完全關閉連線前有 `TIME-WAIT` 的緩衝來等待 `2 MSL` - MSL (Maximum Segment Lifetime) 時間以確保通訊的另一端已關閉連接。根據 [RFC: 793 協定](https://tools.ietf.org/html/rfc793) MSL 為 `2` 分鐘，`2 MSL 即為 4 分鐘`

- 已完成 web call ，透過 `netstat` 確認狀態仍為 `TIME-WAIT`
    ![2state](https://user-images.githubusercontent.com/3851540/49747858-4b895c80-fcdf-11e8-9b6c-69b79e1bed69.png)
- 模擬耗盡 sockets
    - 程式碼
        
        ```cs
        void Main()
        {
            "Starting connections".Dump();
            //嘗試全數耗盡 65536 port
            for (int i = 0; i < 70000; i++)
            {
                using (var client = new HttpClient())
                {
                    client.BaseAddress=new Uri("http://localhost");
                    var result = client.GetAsync("/").GetAwaiter().GetResult();
                    //列出每個執行動作的 index 與結果
                    $"{i} : {result.StatusCode}".Dump();
                }
            }
            "Connections done".Dump();
        }
        ``` 
    - 錯誤訊息
        
        ```
        Unable to connect to the remote server
        InnerException
        Only one usage of each socket address (protocol/network address/port) is normally permitted 127.0.0.1:80 
        ``` 
- 解決方式：使用 Singleton 或 static 方式建立 HttpClient 物件
    
    > 官方建議針對一個 domain 建立一個 HttpClient instance
    - Singleton
        - ~~double-check locking~~
            
            
            ~~public class HttpClientServiceA~~
            ~~{~~
                ~~private HttpClientServiceA() { }~~
                ~~private static HttpClient httpClient;~~
                ~~private static readonly object alock = new object();~~
                ~~public static HttpClient GetHttpClient()~~
                ~~{~~
                    ~~if (httpClient == null)~~
                    ~~{~~
                        ~~lock (alock)~~
                        ~~{~~
                            ~~if (httpClient == null)~~
                            ~~{~~
                                ~~httpClient = new HttpClient();~~
                                ~~httpClient.BaseAddress = new Uri("/");~~
                            ~~}~~
                        ~~}~~
                    ~~}~~
                    ~~return httpClient;~~
                ~~}~~
            ~~}~~
            
            > 經黑大提醒，重新閱讀 [Implementing the Singleton Pattern in C#](http://csharpindepth.com/articles/general/singleton.aspx) : 作者建議不要使用 `double-check locking` ，建議作法使用 `Lazy<T>`
        
        - Safety through initialization

            > 如果無法使用 `Lazy<T>`，可以考慮使用這個做法

            ```cs
            public sealed class HttpClientServiceA
            {
                private static readonly HttpClient instance = new HttpClient() { BaseAddress = new Uri("/") };

                static HttpClientServiceA()
                {

                }

                private HttpClientServiceA()
                {

                }

                public static HttpClient Instance
                {
                    get
                    {
                        return instance;
                    }
                }
            }
            ```


        - Lazy<T>
            
            ```cs
            public sealed class HttpClientServiceA
            {
                private static readonly Lazy<HttpClient> lazy = new Lazy<HttpClient>(
                () => { 
                        var result = new HttpClient();
                        result.BaseAddress = new Uri("/");
                        return result;
                        });
                public static HttpClient Instance { get { return lazy.Value; } }
                private HttpClientServiceA() {}
            }
            ``` 
    - static
        
        ```cs
        class HttpClientServiceA
        {
            private static readonly HttpClient _httpClient;
            static HttpClientServiceA()
            {
                _httpClient = new HttpClient();
                _httpClient.BaseAddress = new Uri("/");
            }
            public HttpClient HttpclientInstance = _httpClient;
        }
        ```
    - 實際使用
        - singleton
            
            ```cs
            var httpclient = HttpClientServiceA.Instance;
            var result = httpclient.GetAsync("").GetAwaiter().GetResult();
            ``` 
        - static
            
            ```cs
            HttpClientServiceA httpclient = new HttpClientServiceA();
            var result = httpclient.HttpclientInstance.GetAsync("").GetAwaiter().GetResult();
            ```

### 2. 共用的 HttpClient 可能會無法即時反應 DNS 的異動
- 重現問題流程
    - 分別透過 using 與 singleton HttpClient 取得 `/` 內容
    - 修改 hosts file 將 `blog.yowko.com` 主機 ip 指向本機 (原理與修改方式可以參考之前筆記 [在 Windows 環境將特定網址指向不同 IP](/windows-host-file))
    - 重新透過 using 與 singleton HttpClient 取得 `/` 內容
- 程式碼
    - 使用 singleton HttpClient
        
        ```cs
        public IActionResult About()
        {
            var httpclient = HttpClientServiceB.Instance;
            var result = httpclient.GetAsync("").GetAwaiter().GetResult();
            ViewData["Message"] = result.Content.ReadAsStringAsync().GetAwaiter().GetResult();
            return View();
        }
        ```
    - 使用 using HtttpClient
        
        ```cs
        public IActionResult Contact()
        {
            using (var httpclient = new HttpClient())
            {
                httpclient.BaseAddress = new Uri("http://blog.yowko.com/");
                var result = httpclient.GetAsync("").GetAwaiter().GetResult();
                ViewData["Message"] = result.Content.ReadAsStringAsync().GetAwaiter().GetResult();
            }
            return View();
        }
        ```
- 未修改 hosts file : `兩者行為相同`
    - 使用 singleton HttpClient
        
        ![4singletonbefore](https://user-images.githubusercontent.com/3851540/49747860-4b895c80-fcdf-11e8-9e10-505e7544a5f3.png) 
    - 使用 using HtttpClient
        
        ![5usingbefore](https://user-images.githubusercontent.com/3851540/49747861-4c21f300-fcdf-11e8-88f3-5559d37ae4b2.png)

- 修改 hosts file : `singleton HttpClient 未能即時反應 DNS 異動`
    - 使用 singleton HttpClient
        
        ![6singletonafter](https://user-images.githubusercontent.com/3851540/49747863-4c21f300-fcdf-11e8-960c-a34ece6a1455.png)
    - 使用 using HtttpClient
        
        ![7usingafter](https://user-images.githubusercontent.com/3851540/49747864-4c21f300-fcdf-11e8-9f39-c1e3ebf5dcd9.png)

- 解決方式
    
    >將 HttpClient 的 `DefaultRequestHeaders.ConnectionClose` 屬性設定為 `true`，也就是將 HTTP 的 keep-alive header 設為 `false`，讓 socket 在每次處理完 request 即關閉
    
    - singleton
        
        ```cs
        public sealed class HttpClientServiceB
        {
            private static readonly Lazy<HttpClient> lazy = new Lazy<HttpClient>(
                () => {
                    var result = new HttpClient();
                    result.BaseAddress = new Uri("http://blog.yowko.com/");
                    result.DefaultRequestHeaders.ConnectionClose = true;
                    return result;
                });
            public static HttpClient Instance { get { return lazy.Value; } }
            private HttpClientServiceB() { }
        }
        ```
    - static
        
        ```cs
        public class HttpClientServiceA
        {
            private static readonly HttpClient _httpClient;
            static HttpClientServiceA()
            {
                _httpClient = new HttpClient();
                _httpClient.BaseAddress = new Uri("http://blog.yowko.com/");
                _httpClient.DefaultRequestHeaders.ConnectionClose = true;
            }
            public HttpClient HttpclientInstance = _httpClient;
        }
        ```

    > 2018/12/31 補充，重新檢視 [iisue - Singleton HttpClient doesn't respect DNS changes](https://github.com/dotnet/corefx/issues/11224) 後發現漏了一段內容：將 `DefaultRequestHeaders.ConnectionClose` 設為 `true` (也就是將 `keep-alive` header 設為 `false`) 會造成每次 request 結束後都關閉 socket，而增加大約 35 ms 的時間耗損，也失去了重複使用 socket 的好處，比較適用於每次 request 損耗 35 ms 不會造成影響的情境
    
    >- 修改 `ConnectionLeaseTimeout` 時間 : 用來管理 TCP socket 保持開啟的時間，預設為 `-1` 永遠開啟
    >- 修改 `DnsRefreshTimeout` 時間: 用來管理 DNS 更新間隔，預設為 `120000` (兩分鐘)
    >- 兩者皆應視實際使用情境調整

    - singleton 改良版
        
        ```cs
        public sealed class HttpClientServiceB
        {
            private static readonly Lazy<HttpClient> lazy = new Lazy<HttpClient>(
                () =>
                {
                    var baseUri = new Uri("http://blog.yowko.com");
                    var result = new HttpClient();
                    result.BaseAddress = baseUri;
                    //設定 1 分鐘沒有活動即關閉連線，預設 -1 (永不關閉)
                    ServicePointManager.FindServicePoint(baseUri).ConnectionLeaseTimeout = (int)TimeSpan.FromMinutes(1).TotalMilliseconds;
                    //設定 1 分鐘更新 DNS，預設 120000 (2 分鐘)
                    ServicePointManager.DnsRefreshTimeout = (int)TimeSpan.FromMinutes(1).TotalMilliseconds;
                    return result;
                });
            public static HttpClient Instance { get { return lazy.Value; } }
            private HttpClientServiceB() { }
        }
        ```
    - static
        
        ```cs
        public class HttpClientServiceA
        {
            private static readonly HttpClient _httpClient;
            static HttpClientServiceA()
            {
                var baseUri = new Uri("http://blog.yowko.com");
                _httpClient = new HttpClient();
                _httpClient.BaseAddress = baseUri;
                //設定 1 分鐘沒有活動即關閉連線，預設 -1 (永不關閉)
                ServicePointManager.FindServicePoint(baseUri).ConnectionLeaseTimeout = (int)TimeSpan.FromMinutes(1).TotalMilliseconds;
                //設定 1 分鐘更新 DNS，預設 120000 (2 分鐘)
                ServicePointManager.DnsRefreshTimeout = (int)TimeSpan.FromMinutes(1).TotalMilliseconds; ;
            }
            public HttpClient HttpclientInstance = _httpClient;
        }
        ```


## 其他延伸現象
1. 未耗盡 65536 ports ？！

    > Windows 可使用的 port 設定可以查詢 `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\MaxUserPort`，以我的 Windows 10 環境而言，預設值為 `15000`

    ```ps1
    Get-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters -name:MaxUserPort
    ```
        
    ![3maxuserport](https://user-images.githubusercontent.com/3851540/49747859-4b895c80-fcdf-11e8-9884-427873c3a8e3.png)

2. 嘗試縮短 `TIME-WAIT` 時間

    > Windows 環境可以透過設定 `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\TcpTimedWaitDelay` 來修改

## 心得
過去沒有真的遇到 HttpClient 的問題，主要原因應該就是過去經手的系統使用量還不足以引發問題，趁著理解 .NET Core 的新類別重新學習 HttpClient 可能的潛在問題與解決方式，只是出乎意料地花了很多時間來模擬與測試，所幸終於試出點心得了


# 參考資訊
1. [YOU'RE USING HTTPCLIENT WRONG AND IT IS DESTABILIZING YOUR SOFTWARE](https://aspnetmonsters.com/2016/08/2016-08-27-httpclientwrong/)  
2. [C#: HttpClient should NOT be disposed](https://medium.com/@nuno.caneco/c-httpclient-should-not-be-disposed-or-should-it-45d2a8f568bc)
3. [Best practices for using HttpClient on Services](https://blogs.msdn.microsoft.com/shacorn/2016/10/21/best-practices-for-using-httpclient-on-services/)
4. [Single instance of reusable HttpClient](https://codereview.stackexchange.com/questions/69950/single-instance-of-reusable-httpclient)
5. [HttpClient, it lives, and it is glorious](http://www.bizcoder.com/httpclient-it-lives-and-it-is-glorious)
6. [netstat 指令用法,及狀態說明](http://blog.jangmt.com/2010/10/netstat.html) 
7. [HttpClient Class](https://docs.microsoft.com/zh-tw/dotnet/api/system.net.http.httpclient?WT.mc_id=DOP-MVP-5002594)
8. [在 Windows 上遇到非常多 TIME_WAIT 連線時應如何處理](https://blog.miniasp.com/post/2010/11/17/How-to-deal-with-TIME_WAIT-problem-under-Windows.aspx)
9. [在 Windows 環境將特定網址指向不同 IP](/windows-host-file)