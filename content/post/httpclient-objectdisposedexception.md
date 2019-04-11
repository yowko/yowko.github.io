---
title: "使用 HttpClient 出現 ObjectDisposedException ？！"
date: 2018-07-11T22:52:00+08:00
lastmod: 2018-10-07T00:52:53+08:00
draft: false
tags: ["C#","Debug"]
slug: "httpclient-objectdisposedexception"
aliases:
    - /2018/07/httpclient-objectdisposedexception.html
---
# 使用 HttpClient 出現 ObjectDisposedException ？！
最近某個專案中有個需求需要對 partner 發出 http request，而 user 針對 request 出現 error 時希望加上 retry 機制：`重試一次`，結果就是這個重試一次的要求讓程式出現預期外的 Exception，立馬來看看我犯了什麼錯吧

## 程式碼
1. 模擬用 api
    
    >不會回傳 `ok` 
    
    ```cs
    public class ValuesController : ApiController
    {
        // POST api/values
        public string Post([FromBody]UserModel model)
        {
            return $"Name:{model.Name};Birthday:{model.BOD};Salary:{model.Salary}";
        }
    }

    public class UserModel
    {
        public string Name { get; set; }
        public DateTime BOD { get; set; }
        public int Salary { get; set; }

    }
    ```
2. 實際呼叫端
    
    ```cs
    async Task Main()
    {
        var submitUrl="http://localhost:33173/api/Values";
        //建立 HttpClient
        using (HttpClient client = new HttpClient())
        {
            // 準備送出的 data
            var postData = new UserModel { Name = Guid.NewGuid().ToString(), BOD = DateTime.Now, Salary = 100 };
            // 將 data 轉為 json
            string json = JsonConvert.SerializeObject(postData);
            // 將轉為 string 的 json 依編碼並指定 content type 存為 httpcontent
            HttpContent contentPost = new StringContent(json, Encoding.UTF8, "application/json");
            // 發出 post 並取得結果
            HttpResponseMessage response = await client.PostAsync(submitUrl, contentPost);
            // 將回應結果內容取出並轉為 string
            var result = await response.Content.ReadAsStringAsync();
            //回應內容未包含 "ok" 即重送
            if (!result.Contains("ok"))
            {
                // retry
                response = await client.PostAsync(submitUrl, contentPost);
                //將結果輸出
                $"second response:{response}".Dump();
            }
        }
    }
    public class UserModel
    {
        public string Name { get; set; }
        public DateTime BOD { get; set; }
        public int Salary { get; set; }

    }
    ```

## 錯誤訊息
1. 訊息內容
    
    ```
    Data	Data
    HelpLink	null
    HResult	-2146233088
    InnerException	System.ObjectDisposedException: Cannot access a disposed object.
    Object name: 'System.Net.Http.StringContent'.
    at System.Net.Http.HttpContent.CheckDisposed()
    at System.Net.Http.HttpContent.CopyToAsync(Stream stream, TransportContext context)
    at System.Net.Http.HttpClientHandler.GetRequestStreamCallback(IAsyncResult ar)
    InnerExceptions	ReadOnlyCollection`1
    Message	One or more errors occurred.
    Source	mscorlib
    StackTrace	   at System.Threading.Tasks.Task.ThrowIfExceptional(Boolean includeTaskCanceledExceptions)
    at System.Threading.Tasks.Task`1.GetResultCore(Boolean waitCompletionNotification)
    at System.Threading.Tasks.Task`1.get_Result()
    at UserQuery.Main() in C:\Users\yowko.tsai\AppData\Local\Temp\LINQPad5\_qjoirzcd\query_semfzk.cs:line 56
    at LINQPad.ExecutionModel.ClrQueryRunner.Run()
    at LINQPad.ExecutionModel.Server.RunQuery(QueryRunner runner)
    at LINQPad.ExecutionModel.Server.StartQuery(QueryRunner runner)
    at LINQPad.ExecutionModel.Server.<>c__DisplayClass153_0.<ExecuteClrQuery>b__0()
    at LINQPad.ExecutionModel.Server.SingleThreadExecuter.Work()
    at System.Threading.ThreadHelper.ThreadStart_Context(Object state)
    at System.Threading.ExecutionContext.RunInternal(ExecutionContext executionContext, ContextCallback callback, Object state, Boolean preserveSyncCtx)
    at System.Threading.ExecutionContext.Run(ExecutionContext executionContext, ContextCallback callback, Object state, Boolean preserveSyncCtx)
    at System.Threading.ExecutionContext.Run(ExecutionContext executionContext, ContextCallback callback, Object state)
    at System.Threading.ThreadHelper.ThreadStart()
    TargetSite	TargetSite
    ``` 
2. 錯誤截圖
    
    ![1error](https://user-images.githubusercontent.com/3851540/42524074-08d47d58-84a2-11e8-8852-00e6162b3be6.png)

## 發生原因解析
1. 由 StackTrace 得知出現問題的來源：`System.Net.Http.HttpContent.CheckDisposed()`
    
    ![2stacktrace](https://user-images.githubusercontent.com/3851540/42524075-090498bc-84a2-11e8-9048-dde04f416ff2.png)
2. 針對 `HttpContent.cs` 原始碼偵錯
    
    > 請參考 [HttpContent.cs](https://www.symbolsource.org/Public/Metadata/NuGet/Project/HttpClient/0.5.0/Release/.NETFramework,Version=v4.0/Microsoft.Net.Http/Microsoft.Net.Http/System/Net/Http/HttpContent.cs) 
    
    - `CheckDisposed()` 方法位於 ln.457
        
        ![3checkdispose](https://user-images.githubusercontent.com/3851540/42524077-092e3f28-84a2-11e8-9003-09ca8b1342e6.png)
    - `disposed` 屬性於 ln.435 被修改
        
        ![4disposed](https://user-images.githubusercontent.com/3851540/42524078-096032f8-84a2-11e8-8413-e18887430e12.png)
3. 找出 HttpContent 何時被 disposed
    
    > 請參考 [HttpClient.cs](https://www.symbolsource.org/Public/Metadata/NuGet/ProjectHttpClient/0.5.0/Release/.NETFramework,ersion=v4.0/Microsoft.Net.Http/Microsoft.Net.ttp/System/Net/Http/HttpClient.cs)
    
    - 程式呼叫 ln.389 的 `PostAsync()`
        
        ![5PostAsync](https://user-images.githubusercontent.com/3851540/42524079-098c1166-84a2-11e8-8eaf-f75f7a94e23a.png)
    - 轉 call ln.256 的 `SendAsync` 
        
        ![6SendAsync](https://user-images.githubusercontent.com/3851540/42524081-09e1905a-84a2-11e8-822a-41e46d0d3de2.png)
    - http request 完成後 call ` DisposeRequestContent`
        
        ![7 DisposeRequestContent](https://user-images.githubusercontent.com/3851540/42524070-087268f2-84a2-11e8-9599-82cbed0a9668.png)
    - `DisposeRequestContent` 中 dispose httpcontent
        
        ![8disposcontent](https://user-images.githubusercontent.com/3851540/42524071-08a137d6-84a2-11e8-8154-095192e5051d.png)

## 心得
過去如果需要在 http request 中加入 retry 機制，都是透過迴圈或是遞迴方式來達成，而此次 user 明確指出只需 retry once，讓我偷懶直接重新 post，而未加入過去慣用的 retry 機制，因此剛好有這個機會瞭解到原來 HttpClient 為了確保 request content 只會被發送一次，會直接 dispose request content

透過此次問題偵錯的過程深深地感受到自己還有好多開發基礎知識掌握度不夠，也發覺過去的成功經驗可能會因自己偷懶未完成複製而成為失敗的主因呀

# 參考資訊
1. [Cannot access a disposed object. ‘System.Net.Http.StringContent’ While having retry logic.](https://amoghnatu.net/2017/01/12/cannot-access-a-disposed-object-system-net-http-stringcontent-while-having-retry-logic/)
2. [HttpContent.cs](https://www.symbolsource.org/Public/Metadata/NuGet/Project/HttpClient/0.5.0/Release/.NETFramework,Version=v4.0/Microsoft.Net.Http/Microsoft.Net.Http/System/Net/Http/HttpContent.cs) 
3. [HttpClient.cs](https://www.symbolsource.org/Public/Metadata/NuGet/ProjectHttpClient/0.5.0/Release/.NETFramework,ersion=v4.0/Microsoft.Net.Http/Microsoft.Net.ttp/System/Net/Http/HttpClient.cs)