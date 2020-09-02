---
title: "使用 HttpClient 來存取 GET,POST,PUT,DELETE,PATCH 網路資源"
date: 2017-06-14T21:30:00+08:00
lastmod: 2020-09-01T21:30:31+08:00
draft: false
tags: ["C#"]
slug: "httpclient"
aliases:
    - /2017/06/httpclient.html
---
# 使用 HttpClient 來存取 GET,POST,PUT,DELETE,PATCH 網路資源
之前文章 [如何使用 WebRequest,HttpWebRequest 來存取 (GET,POST,PUT,DELETE,PATCH) 網路資源](//blog.yowko.com/2017/03/webrequest-and-httpwebrequest.html) 紀錄 WebRequest,HttpWebRequest 的用法，[使用 WebClient 來存取 GET,POST,PUT,DELETE,PATCH 網路資源](//blog.yowko.com/2017/06/webclient.html) 則紀錄了 WebClient 的用法，接著就是我所知的最後一個可以用來存取網路資源的 HttpClient

## HttpClient 基本資訊

用來對 URI 提出 request，以及接收 HTTP 的回應

*   Namespace：System.Net.Http
*   Assembly：System.Net.Http (System.Net.Http.dll)
*   基本要求：.NET Framework 4.5 以上


## GET

1.  寫法 1

    ```cs
    //建立 HttpClient
	HttpClient client = new HttpClient() { BaseAddress= new Uri("http://jsonplaceholder.typicode.com/") };
	//使用 async 方法從網路 url 上取得回應
	using (HttpResponseMessage response = await client.GetAsync("posts"))
    // 將網路取得回應的內容設定給 httpcontent，可省略，直接使用 response.Content
    using (HttpContent content = response.Content)
	{
		// 將 httpcontent 轉為 string
		string result = await content.ReadAsStringAsync();
		// linqpad 顯示資料用
		if (result != null)
			result.Dump();
	}
    ```

2.  寫法 2

    ```cs
    //建立 HttpClient
	HttpClient client = new HttpClient() { BaseAddress = new Uri("http://jsonplaceholder.typicode.com/") };

	//使用 async 方法從網路 url 上取得回應
	var response = await client.GetAsync("posts");
	//如果 httpstatus code 不是 200 時會直接丟出 expection
	response.EnsureSuccessStatusCode();
	// 將 response 內容 轉為 string
	string result = await response.Content.ReadAsStringAsync();
	// linqpad 顯示資料用
	result.Dump();
    ```
3.  寫法 3 - 使用 SendAsync

    ```cs
    //建立 HttpClient
	HttpClient client = new HttpClient() { BaseAddress = new Uri("http://jsonplaceholder.typicode.com/") };
	//指定 request 的 method 與 detail url
	using (HttpRequestMessage request = new HttpRequestMessage(HttpMethod.Get, "posts"))
	{
		// 發出 post 並將回應內容轉為 string 再透過 linqpad 輸出
		await client.SendAsync(request)
		.ContinueWith(responseTask =>
		{
			responseTask.GetAwaiter().GetResult().Content.ReadAsStringAsync().Dump();
		});
	}
    ```

## POST

1.  寫法 1 - 使用 PostAsync

    ```cs 
    //建立 HttpClient
	HttpClient client = new HttpClient(){BaseAddress=new Uri("https://jsonbin.org/yowko/")};

	// 指定 authorization header
	client.DefaultRequestHeaders.Add("authorization", "token {api token}");
	
	// 準備寫入的 data
	PostData postData = new PostData() { userId = 123422, title = "yowko 中文", body = "yowko test body 中文" };
	// 將 data 轉為 json
	string json = JsonConvert.SerializeObject(postData);
	// 將轉為 string 的 json 依編碼並指定 content type 存為 httpcontent
	HttpContent contentPost = new StringContent(json, Encoding.UTF8, "application/json");
	// 發出 post 並取得結果
	HttpResponseMessage response = client.PostAsync("test", contentPost).GetAwaiter().GetResult();
	// 將回應結果內容取出並轉為 string 再透過 linqpad 輸出
	response.Content.ReadAsStringAsync().GetAwaiter().GetResult().Dump();
    ```
2.  寫法 2 - 使用 SendAsync

    ```cs
    // 建立 HttpClient
	HttpClient client = new HttpClient();

	// 指定 base url 
	client.BaseAddress = new Uri("https://jsonbin.org/");
	// 指定 authorization header
	client.DefaultRequestHeaders.Add("authorization", "token {api token}");
	// 指定 request 的 method 與 detail url
	HttpRequestMessage request = new HttpRequestMessage(HttpMethod.Post, "yowko/test");
	// 準備寫入的 data
	PostData postData = new PostData() { userId = 1, title = "yowko 中文", body = "yowko test body 中文" };
	// 將 data 轉為 json
	string json = JsonConvert.SerializeObject(postData);
	// 將轉為 string 的 json 依編碼並指定 content type 存為 httpcontent
	request.Content = new StringContent(json, Encoding.UTF8, "application/json");
	// 發出 post 並將回應內容轉為 string 再透過 linqpad 輸出
	await client.SendAsync(request)
	.ContinueWith(responseTask =>
	{
		responseTask.GetAwaiter().GetResult().Content.ReadAsStringAsync().Dump();
	});
    ```

## PUT

> 範例中 PUT 是將 jsonbin 的網址改為 public

1. 寫法 1 - 使用 PutAsync

    ```cs
    //建立 HttpClient
	HttpClient client = new HttpClient() {BaseAddress= new Uri("https://jsonbin.org/yowko/")};

	// 指定 authorization header
	client.DefaultRequestHeaders.Add("authorization", "token {api token}");
	// 發出 post 並取得結果
	HttpResponseMessage response = client.PutAsync("test/_perms", null).Result;
	// 將回應結果內容取出並轉為 string 再透過 linqpad 輸出
	response.Content.ReadAsStringAsync().GetAwaiter().GetResult().Dump();
    ```

2.  寫法 2 - 使用 SendAsync

    ```cs
   // 建立 HttpClient
	HttpClient client = new HttpClient();

	// 指定 base url 
	client.BaseAddress = new Uri("https://jsonbin.org/yowko/");
	// 指定 authorization header
	client.DefaultRequestHeaders.Add("authorization", "token {api token}");
	// 指定 request 的 method 與 detail url
	HttpRequestMessage request = new HttpRequestMessage(HttpMethod.Put, "test/_perms");
	// 發出 post 並將回應內容轉為 string 再透過 linqpad 輸出
	await client.SendAsync(request)
	.ContinueWith(responseTask =>
	{
		responseTask.GetAwaiter().GetResult().Content.ReadAsStringAsync().Dump();
	});
    ```

## DELETE

> 範例中 DELETE 是將 jsonbin 的網址改為 private

1.  寫法 1 - 使用 DeleteAsync

    ```cs
    //建立 HttpClient
	HttpClient client = new HttpClient() { BaseAddress = new Uri("https://jsonbin.org/yowko/") };
	// 指定 authorization header
	client.DefaultRequestHeaders.Add("authorization", "token {api token}");
	// 發出 post 並取得結果
	HttpResponseMessage response = client.DeleteAsync("test/_perms").GetAwaiter().GetResult();
	// 將回應結果內容取出並轉為 string 再透過 linqpad 輸出
	response.Content.ReadAsStringAsync().GetAwaiter().GetResult().Dump();
    ```
2.  寫法 2 - 使用 SendAsync

    ```cs
    // 建立 HttpClient
	HttpClient client = new HttpClient();

	// 指定 base url 
	client.BaseAddress = new Uri("https://jsonbin.org/");
	// 指定 authorization header
	client.DefaultRequestHeaders.Add("authorization", "token {api token}");
	// 指定 request 的 method 與 detail url
	HttpRequestMessage request = new HttpRequestMessage(HttpMethod.Delete, "yowko/test/_perms");
	// 發出 post 並將回應內容轉為 string 再透過 linqpad 輸出
	await client.SendAsync(request)
	.ContinueWith(responseTask =>
	{
		responseTask.GetAwaiter().GetResult().Content.ReadAsStringAsync().Dump();
	});
    ```

## PATCH

> HttpClient 沒有專屬 PATCH 的用法，請參考 PUT

## 7. 使用 proxy

> 有時候程式的 host 環境無法直接上網或是我們想要確認傳出去的相關資訊，就需要設定 proxy

```cs
// 建立 HttpClientHandler
	HttpClientHandler handler = new HttpClientHandler()
	{
		// 指定 proxy uri
		Proxy = new WebProxy("http://127.0.0.1:8888"),
		// 指定 proxy Credentials
		Credentials = new NetworkCredential("{username}", "{password}"),
		// 使用 proxy
		UseProxy = true,
	};
	//建立 HttpClient
	HttpClient client = new HttpClient(handler) { BaseAddress = new Uri("https://jsonbin.org/yowko/") };

	// 指定 authorization header
	client.DefaultRequestHeaders.Add("authorization", "token {api token}");
	// 準備寫入的 data
	PostData postData = new PostData() { userId = 1, title = "yowko1", body = "yowko test body 中文" };
	// 將 data 轉為 json
	string json = JsonConvert.SerializeObject(postData);
	// 將轉為 string 的 json 依編碼並指定 content type 存為 httpcontent
	HttpContent contentPost = new StringContent(json, Encoding.UTF8, "application/json");
	// 發出 post 並取得結果
	HttpResponseMessage response = await client.PostAsync("test", contentPost);
	// 將回應結果內容取出並轉為 string 再透過 linqpad 輸出
	response.Content.ReadAsStringAsync().GetAwaiter().GetResult().Dump();
```

*   以 fiddler 為例

    *   fiddler 的相關設定請參考 [使用 fiddler 內建 proxy 來截錄手機或是程式封包](http://blog.yowko.com/2017/03/use-fiddler-proxy-gather-traffic.html)
    *   截錄到的內容

        ![1result](https://cloud.githubusercontent.com/assets/3851540/23498993/2dcba8e8-ff65-11e6-8aaf-e1cc9ceb6a8c.png)

# 參考資料

1.  [HttpClient 類別](https://msdn.microsoft.com/zh-tw/library/system.net.http.httpclient%28v=vs.110%29.aspx)
2.  [HttpClient](https://www.dotnetperls.com/httpclient)
3.  [Calling a Web API From a .NET Client (C#)](https://docs.microsoft.com/en-us/aspnet/web-api/overview/advanced/calling-a-web-api-from-a-net-client?WT.mc_id=DOP-MVP-5002594)
4.  [Simple C# .NET 4.5 HTTPClient Request Using Basic Auth and Proxy](https://www.snip2code.com/Snippet/13895/Simple-C---NET-4-5-HTTPClient-Request-Us)
