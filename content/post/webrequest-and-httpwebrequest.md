---
title: "如何使用 WebRequest,HttpWebRequest 來存取(GET,POST,PUT,DELETE,PATCH)網路資源"
date: 2017-03-09T02:42:34+08:00
lastmod: 2021-10-08T00:42:34+08:00
draft: false
tags: ["csharp"]
slug: "webrequest-and-httpwebrequest"
aliases:
    - /2017/03/webrequest-and-httpwebrequest.html
---
## 如何使用 WebRequest,HttpWebRequest 來存取(GET,POST,PUT,DELETE,PATCH)網路資源

現在雲端服務多元，很多系統設計上也都走向 api 化的架構，加上前端工程以及 mobile 裝置的普及，使用後端程式呼叫外部 api 的情境也常遇到，.NET Framework 幫我們封裝了好幾個 api ，讓我們可以快速開發，但也因為有好幾個 api 可以使用而造成不知道該用哪一個，所以就想來釐清其中的差異，首先就先來看看 WebRequest 與 HttpWebRequest，因為兩者有繼承關係所以放在一起看

## WebRequest 基本資訊

用來對 URI 提出 request，是個 abstract class，需使用 `Create` 而不是建構式來初始化 WebRequest instance

- Namespace：System.Net
- Assembly：System (System.dll)
- 基本要求：.NET Framework 1.1 以上
- 包含 `HttpWebRequest`,`FtpWebRequest`,`FileWebRequest`

## HttpWebRequest 基本資訊

基於 WebRequest 實作 HTTP 相關功能，WebRequest 還另外包含 FtpWebRequest 及 FileWebRequest

- Namespace：System.Net
- Assembly：System (System.dll)
- 基本要求：.NET Framework 1.1 以上
- 因為基於 WebRequest 的實作，因此用法上也幾乎與 WebRequest 相同

## 1. GET

```cs
//建立 WebRequest 並指定目標的 uri
WebRequest request = WebRequest.Create("http://jsonplaceholder.typicode.com/posts");
// 使用 HttpWebRequest.Create 實際上也是呼叫 WebRequest.Create
//WebRequest request = HttpWebRequest.Create("http://jsonplaceholder.typicode.com/posts");

//指定 request 使用的 http verb
request.Method = "GET";
//使用 GetResponse 方法將 request 送出，如果不是用 using 包覆，請記得手動 close WebResponse 物件，避免連線持續被佔用而無法送出新的 request
using (var httpResponse = (HttpWebResponse)request.GetResponse())
//使用 GetResponseStream 方法從 server 回應中取得資料，stream 必需被關閉
//使用 stream.close 就可以直接關閉 WebResponse 及 stream，但同時使用 using 或是關閉兩者並不會造成錯誤，養成習慣遇到其他情境時就比較不會出錯
using (var streamReader = new StreamReader(httpResponse.GetResponseStream()))
{
    var result = streamReader.ReadToEnd();
    result.Dump();
}
```

## 2. POST (使用 `application/json`)

- 指定 content type
- 指定 header

    ```cs
    //建立 WebRequest 並指定目標的 uri
    WebRequest request = WebRequest.Create("https://jsonbin.org/me/test");
    //指定 request 使用的 http verb
    request.Method = "POST";
    //準備 post 用資料
    PostData postData = new PostData() { userId = 1, title = "yowko", body = "yowko test body 中文" };
    //指定 request 的 content type
    request.ContentType = "application/json; charset=utf-8";
    //指定 request header
    request.Headers.Add("authorization", "token apikey");
    //將需 post 的資料內容轉為 stream 
    using (var streamWriter = new StreamWriter(request.GetRequestStream()))
    {
        string json = new JavaScriptSerializer().Serialize(postData);
        streamWriter.Write(json);
        streamWriter.Flush();
    }
    //使用 GetResponse 方法將 request 送出，如果不是用 using 包覆，請記得手動 close WebResponse 物件，避免連線持續被佔用而無法送出新的 request
    using (var httpResponse = (HttpWebResponse)request.GetResponse())
    //使用 GetResponseStream 方法從 server 回應中取得資料，stream 必需被關閉
    //使用 stream.close 就可以直接關閉 WebResponse 及 stream，但同時使用 using 或是關閉兩者並不會造成錯誤，養成習慣遇到其他情境時就比較不會出錯
    using (var streamReader = new StreamReader(httpResponse.GetResponseStream()))
    {
        var result = streamReader.ReadToEnd();
        result.Dump();
    }
    ```

## 3. POST(使用 `application/x-www-form-urlencoded`)

```cs
WebRequest request = HttpWebRequest.Create("https://jsonbin.org/me/test");
request.Method = "POST";
//使用 application/x-www-form-urlencoded
request.ContentType = "application/x-www-form-urlencoded; charset=utf-8";
request.Headers.Add("authorization", "token apikey");
//要傳送的資料內容(依字串表示)
string postData = "id=9&name=yowko&body=yowko中文";
//將傳送的字串轉為 byte array
byte[] byteArray = Encoding.UTF8.GetBytes(postData);
//告訴 server content 的長度
request.ContentLength = byteArray.Length;
//將 byte array 寫到 request stream 中 
using (Stream reqStream = request.GetRequestStream())
{
    reqStream.Write(byteArray, 0, byteArray.Length);
}
using (var httpResponse = (HttpWebResponse)request.GetResponse())
using (var streamReader = new StreamReader(httpResponse.GetResponseStream()))
{
    var result = streamReader.ReadToEnd();
    result.Dump();
}
```

## 4. PUT

```cs
WebRequest request = WebRequest.Create("https://jsonbin.org/yowko/test/_perms");
request.Method = "PUT";
request.Headers.Add("authorization", "token apikey");
using (var httpResponse = (HttpWebResponse)request.GetResponse())
using (var streamReader = new StreamReader(httpResponse.GetResponseStream()))
{
    var result = streamReader.ReadToEnd();
    result.Dump();
}
```

## 5. DELETE

```cs
WebRequest request = WebRequest.Create("https://jsonbin.org/yowko/test/_perms");
request.Method = "DELETE";
request.Headers.Add("authorization", "token apikey");
using (var httpResponse = (HttpWebResponse)request.GetResponse())
using (var streamReader = new StreamReader(httpResponse.GetResponseStream()))
{
    var result = streamReader.ReadToEnd();
    result.Dump();
}
```

## 6. PATCH

```cs
WebRequest request = WebRequest.Create("https://jsonbin.org/me/test");
request.Method = "PATCH";
PostData postData = new PostData() { body = "yowko test body 中文2" };
request.ContentType = "application/json; charset=utf-8";
request.Headers.Add("authorization", "token apikey");
using (var streamWriter = new StreamWriter(request.GetRequestStream()))
{
    string json = new JavaScriptSerializer().Serialize(postData);
    streamWriter.Write(json);
    streamWriter.Flush();
}

using (var httpResponse = (HttpWebResponse)request.GetResponse())
using (var streamReader = new StreamReader(httpResponse.GetResponseStream()))
{
    var result = streamReader.ReadToEnd();
    result.Dump();
}
```

## 7. 使用 proxy

> 有時候程式的 host 環境無法直接上網或是我們想要確認傳出去的相關資訊，就需要設定 proxy

```cs
WebRequest request = WebRequest.Create("https://jsonbin.org/me/test");
request.Method = "POST";
PostData postData = new PostData() { userId = 1, title = "yowko1", body = "yowko test body 中文" };
request.ContentType = "application/json; charset=utf-8";
request.Headers.Add("authorization", "token apikey");
//指令 proxy address
string proxyAddress="http://127.0.0.1:8888";
//建立 proxy
WebProxy myProxy=new WebProxy(new Uri(proxyAddress));
//建立 proxy 的認證資訊
myProxy.Credentials=new NetworkCredential("{username}","{password}");
//將 proxy 指定給 request 使用
request.Proxy=myProxy;
using (var streamWriter = new StreamWriter(request.GetRequestStream()))
{
    string json = new JavaScriptSerializer().Serialize(postData);
    streamWriter.Write(json);
    streamWriter.Flush();
}
using (var httpResponse = (HttpWebResponse)request.GetResponse())
using (var streamReader = new StreamReader(httpResponse.GetResponseStream()))
{
    var result = streamReader.ReadToEnd();
    result.Dump();
}

```

- 以 fiddler 為例
  - fiddler 的相關設定請參考 [使用 fiddler 內建 proxy 來截錄手機或是程式封包](/use-fiddler-proxy-gather-traffic)
  - 截錄到的內容

      ![1result](https://cloud.githubusercontent.com/assets/3851540/23498993/2dcba8e8-ff65-11e6-8aaf-e1cc9ceb6a8c.png)

## 參考資料

1. [WebRequest 類別](https://msdn.microsoft.com/zh-tw/library/system.net.webrequest%28v=vs.110%29.aspx)
2. [如何：使用 WebRequest 類別傳送資料](https://msdn.microsoft.com/zh-tw/library/debx8sh9%28v=vs.110%29.aspx)
3. [如何：使用 WebRequest 類別要求資料](https://msdn.microsoft.com/zh-tw/library/456dfw4f%28v=vs.110%29.aspx)
4. [HttpWebRequest 類別](https://msdn.microsoft.com/zh-tw/library/system.net.httpwebrequest%28v=vs.110%29.aspx)
