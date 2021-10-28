---
title: "使用 WebClient 來存取 GET,POST,PUT,DELETE,PATCH 網路資源"
date: 2017-06-13T21:00:00+08:00
lastmod: 2021-10-08T21:00:14+08:00
draft: false
tags: ["csharp"]
slug: "webclient"
aliases:
    - /2017/06/webclient.html
---
## 使用 WebClient 來存取 GET,POST,PUT,DELETE,PATCH 網路資源

之前在專案中看到許多不同風格的程式，這種現象很常見，尤其在由來已久、團隊成員來來去去的專案中更是常發生，我並沒有太多想法，但就取得外部網路資源的寫法也有好幾套這就讓我比較驚訝了，幾個常見呼叫網路資源的 api 都用了，所以心血來潮想要整理一下，後來又發現一部份還有加上自行封裝的用法，但封裝後的功能不夠全面，造成有些情境不得不改用原生 api

之前簡單紀錄過 WebRequest,HttpWebRequest 的用法，有興趣可以參考 [如何使用 WebRequest,HttpWebRequest 來存取 (GET,POST,PUT,DELETE,PATCH) 網路資源](//blog.yowko.com/2017/03/webrequest-and-httpwebrequest.html)，今天就來看看 WebClient 的用法吧

## WebClient 基本資訊

提供通用方法使用 WebRequest 類別傳送及接收 URI (支援 `http:`, `https:`, `ftp:`,和 `file:` ) 的資源

* Namespace：System.Net
* Assembly：System (System.dll)
* 基本要求：.NET Framework 1.1 以上
* WebClient 預設僅會傳送必要的 http header
* 缺點：無法指定 Timeout ,另外保哥文章 [利用 WebClient 類別模擬 HTTP POST 表單送出的注意事項](http://blog.miniasp.com/post/2010/01/23/Emulate-Form-POST-with-WebClient-class.aspx)有提到 `不適合用來下載大量的檔案，高負載的網站也不適合這樣用，即便你用非同步的方式撰寫，也會讓 WebClient 因為佔據過多 Threads 而導致效能降低` 這我不知道怎麼模擬，請大家參考保哥文章

## GET

1. 寫法 1

    ```cs
    using (WebClient webClient = new WebClient())
    // 從 url 讀取資訊至 stream
    using(Stream stream = webClient.OpenRead("http://jsonplaceholder.typicode.com/posts")    )
    // 使用 StreamReader 讀取 stream 內的字元
    using(StreamReader reader= new StreamReader(stream))
    {
        // 將 StreamReader 所讀到的字元轉為 string
        string request = reader.ReadToEnd();
        request.Dump();
    }
    ```

2. 寫法 2

    ```cs
    // 建立 webclient
    using(WebClient webClient = new WebClient())
    {
        // 指定 WebClient 的編碼
        webClient.Encoding = Encoding.UTF8;
        // 指定 WebClient 的 Content-Type header
        webClient.Headers.Add(HttpRequestHeader.ContentType, "application/json");
        // 從網路 url 上取得資料
        var body = webClient.DownloadString("http://jsonplaceholder.typicode.com/posts");
        body.Dump();
    }
    ```

## POST

WebClient 共有四種 POST 相關的方法

1. UploadString(string)

    > 將 String 傳送至資源。

    * 以下 demo 會搭配 `application/json`

        ```cs
        // 建立 WebClient
        using (WebClient webClient = new WebClient())
        {
            // 指定 WebClient 編碼
            webClient.Encoding = Encoding.UTF8;
            // 指定 WebClient 的 Content-Type header
            webClient.Headers.Add(HttpRequestHeader.ContentType, "application/json");
            // 指定 WebClient 的 authorization header
            webClient.Headers.Add("authorization", "token {apitoken}");
            // 準備寫入的 data
            PostData postData = new PostData() { userId = 1123456, title = "yowko", body = "yowko test body 中文" };
            // 將 data 轉為 json
            string json = JsonConvert.SerializeObject(postData);
            // 執行 post 動作
            var result = webClient.UploadString("https://jsonbin.org/yowko/test", json);
            // linqpad 將 post 結果輸出
            result.Dump();
        }
        ```

2. UploadData(byte[])

    > 將位元組陣列傳送至資源，並傳回含有任何回應的 Byte 陣列。

    * 以下 demo 會搭配 `application/x-www-form-urlencoded`

        ```cs
        // 建立 WebClient
        using (WebClient webClient = new WebClient())
        {
            // 指定 WebClient 編碼
            webClient.Encoding = Encoding.UTF8;
            // 指定 WebClient 的 Content-Type header
            webClient.Headers.Add(HttpRequestHeader.ContentType,             application/x-www-form-urlencoded");
            // 指定 WebClient 的 authorization header
            webClient.Headers.Add("authorization", "token {apitoken}");
            //要傳送的資料內容(依字串表示)
            string postData = "id=12354&name=yowko&body=yowko test body 中文";
            //將傳送的字串轉為 byte array
            byte[] byteArray = Encoding.UTF8.GetBytes(postData);
            // 執行 post 動作
            var result = webClient.UploadData("https://jsonbin.org/yowko/test", byteArray);
            // linqpad 將 post 結果輸出
            result.Dump();
        }
        ```

3. UploadValues (byte[])

    > 將 NameValueCollection 傳送至資源，並傳回含有任何回應的 Byte 陣列。

    * 不需指定 content type

        ```cs
        // 建立 WebClient
        using (WebClient webClient = new WebClient())
        {
            // 指定 WebClient 編碼
            webClient.Encoding = Encoding.UTF8;
            // 指定 WebClient 的 authorization header
            webClient.Headers.Add("authorization", "token {apitoken}");
            //要傳送的資料內容
            NameValueCollection nameValues = new NameValueCollection();
            nameValues["userId"] = "456";
            nameValues["title"] = "yowko";
            nameValues["body"]="yowko test body 中文";
                        // 執行 post 動作
            var result = webClient.UploadValues("https://jsonbin.org/yowko/test",     nameValues);
            //將 post 結果轉為 string
            string resultstr = Encoding.UTF8.GetString(result);
            // linqpad 將 post 結果輸出
            resultstr.Dump();
        }
        ```

4. UploadFile(byte[]) <span style="color:red">今天不會介紹</span>

    > 將本機檔案傳送至資源，並傳回含有任何回應的 Byte 陣列。

## PUT

> 方法與 POST 相同，只需在 url 與 data 間多傳一個 method 的參數，範例中 PUT 是將 jsonbin 的網址改為 public

```cs
// 建立 WebClient
using (WebClient webClient = new WebClient())
{
    // 指定 WebClient 編碼
    webClient.Encoding = Encoding.UTF8;
    // 指定 WebClient 的 Content-Type header
    webClient.Headers.Add(HttpRequestHeader.ContentType, "application/json");
    // 指定 WebClient 的 authorization header
    webClient.Headers.Add("authorization", "token {apitoken}");
    // 執行 PUT 動作
    var result = webClient.UploadString("https://jsonbin.org/yowko/test/_perms","PUT",     "");
    // linqpad 將 post 結果輸出
    result.Dump();
}
```

## DELETE

> 方法與 POST 相同，只需在 url 與 data 間多傳一個 method 的參數，範例中 DELETE 是將 jsonbin 的網址改為 private

```cs
// 建立 WebClient
using (WebClient webClient = new WebClient())
{
    // 指定 WebClient 編碼
    webClient.Encoding = Encoding.UTF8;
    // 指定 WebClient 的 Content-Type header
    webClient.Headers.Add(HttpRequestHeader.ContentType, "application/json");
    // 指定 WebClient 的 authorization header
    webClient.Headers.Add("authorization", "token {apitoken}");
    // 執行 DELETE 動作
    var result = webClient.UploadString("https://jsonbin.org/yowko/test/_perms",    "DELETE", "");
    // linqpad 將 post 結果輸出
    result.Dump();
}
```

## PATCH

> 方法與 POST 相同，只需在 url 與 data 間多傳一個 method 的參數

```cs
// 建立 WebClient
using (WebClient webClient = new WebClient())
{
    // 指定 WebClient 編碼
    webClient.Encoding = Encoding.UTF8;
    // 指定 WebClient 的 Content-Type header
    webClient.Headers.Add(HttpRequestHeader.ContentType, "application/json");
    // 指定 WebClient 的 authorization header
    webClient.Headers.Add("authorization", "token {api token}");
    // 準備寫入的 data
    PostData postData = new PostData() { title = "yowko 中文", body = "yowko body 中文"     };
    // 將 data 轉為 json
    string json = JsonConvert.SerializeObject(postData);
    // 執行 PATCH 動作
    var result = webClient.UploadString("https://jsonbin.org/yowko/test","PATCH", json);
    // linqpad 將 post 結果輸出
    result.Dump();
}
```

## 使用 proxy

> 有時候程式的 host 環境無法直接上網或是我們想要確認傳出去的相關資訊，就需要設定 proxy

```cs
// 建立 WebClient
using (WebClient webClient = new WebClient())
{
    // 指定 WebClient 編碼
    webClient.Encoding = Encoding.UTF8;
    // 指定 WebClient 的 Content-Type header
    webClient.Headers.Add(HttpRequestHeader.ContentType, "application/json");
    // 指定 WebClient 的 authorization header
    webClient.Headers.Add("authorization", "token {api token}");
    //指令 proxy address
    string proxyAddress = "http://127.0.0.1:8888";
    //建立 proxy
    WebProxy myProxy = new WebProxy(new Uri(proxyAddress));
    //建立 proxy 的認證資訊
    myProxy.Credentials = new NetworkCredential("{username}", "{password}");
    //將 proxy 指定給 request 使用
    webClient.Proxy = myProxy;
    // 準備寫入的 data
    PostData postData = new PostData() { userId=1, title = "yowko1", body = "yowko test     body 中文" };
    // 將 data 轉為 json
    string json = JsonConvert.SerializeObject(postData);
    // 執行 post 動作
    var result = webClient.UploadString("https://jsonbin.org/yowko/test", json);
    // linqpad 將 post 結果輸出
    result.Dump();
}
```

* 以 fiddler 為例

  * fiddler 的相關設定請參考 [使用 fiddler 內建 proxy 來截錄手機或是程式封包](http://blog.yowko.com/2017/03/use-fiddler-proxy-gather-traffic.html)
  * 截錄到的內容

    ![1result](https://cloud.githubusercontent.com/assets/3851540/23498993/2dcba8e8-ff65-11e6-8aaf-e1cc9ceb6a8c.png)

## 參考資料

1. [WebClient 類別](https://msdn.microsoft.com/zh-tw/library/system.net.webclient%28v=vs.110%29.aspx)
2. [利用 WebClient 類別模擬 HTTP POST 表單送出的注意事項](http://blog.miniasp.com/post/2010/01/23/Emulate-Form-POST-with-WebClient-class.aspx)
3. [如何使用 WebRequest,HttpWebRequest 來存取 (GET,POST,PUT,DELETE,PATCH) 網路資源](//blog.yowko.com/2017/03/webrequest-and-httpwebrequest.html)
4. [使用 fiddler 內建 proxy 來截錄手機或是程式封包](http://blog.yowko.com/2017/03/use-fiddler-proxy-gather-traffic.html)
