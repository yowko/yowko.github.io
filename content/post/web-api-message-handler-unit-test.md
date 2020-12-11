---
title: "為 Web Api 的 Message Handler 加上單元測試"
date: 2017-06-25T12:18:00+08:00
lastmod: 2020-12-11T12:18:44+08:00
draft: false
tags: ["ASP.NET Web API","Unit Test"]
slug: "web-api-message-handler-unit-test"
aliases:
    - /2017/06/web-api-message-handler-unit-test.html
---
# 為 Web Api 的 Message Handler 加上單元測試
之前在 [為 ASP.NET WEB API 加上簡易的 Token 驗證](/2017/02/aspnet-web-api-fixed-token.html) 中曾經使用過 Message Handler 為 ASP.NET Web Api 加上簡易驗證。

後來在 TDD 課堂中聽到可以為 Message Handler 加上單元測試，於是就來實作看看吧

## 基本環境設定

詳細介紹請參考 [為 ASP.NET WEB API 加上簡易的 Token 驗證](/2017/02/aspnet-web-api-fixed-token.html)，程式碼因為需求異動已有差異，將會使用下方程式碼進行實作

*   流程說明

    > request header 中如果沒有指定的 token 就回傳 401，有指定 token 就繼續執行

*   Web Api 的 Authentication Message Handler 程式碼

    ```cs
    public class AuthenticationHandler : DelegatingHandler
    {
        private IAuthenticationService _authenticationService;
        public AuthenticationHandler(IAuthenticationService authenticationService)
        {
            if (authenticationService != null)
                _authenticationService = authenticationService;
            else
                _authenticationService = new RedisAuthService();
        }
        protected override Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, CancellationToken cancellationToken)
        {
            if (IsAuthenticate(request))
            {
                return base.SendAsync(request, cancellationToken);
            }
            else
            {
                HttpResponseMessage response = request.CreateErrorResponse(HttpStatusCode.Unauthorized, "Token Error !!");
                response.StatusCode = HttpStatusCode.Unauthorized;
                response.Content = new StringContent("Token Error !!");
                return Task.FromResult(response);
            }
        }
        private bool IsAuthenticate(HttpRequestMessage request)
        {
            string authHeader = null;
            var auth = request.Headers.Authorization;
            if (auth != null && auth.Scheme == "Bearer")
                authHeader = auth.Parameter;
            if(_authenticationService.Verify(authHeader))
                return true;
            return false;
        }
    }
    ```

*   IAuthenticationService 程式碼

    ```cs
    public interface IAuthenticationService
    {
        bool Verify(string authHeader);
    }
    ```

## 建立單元測試

1.  第一個遇到的問題就是 `SendAsync` 是 `protected` 無法直接使用 `Create Unit Tests` 建立測試

    ![1portected](https://user-images.githubusercontent.com/3851540/27513602-9b3d005a-599e-11e7-8b1e-c11d72d183cf.png)

    *   解決方式有兩個：

        * A. 自行建立測試專案

        * B. 先將 `protected` 改為 `public` 建立完再改回來

            > 記得改回 `protected`，否則 override 的簽章會不一致

            ![2overrideerror](https://user-images.githubusercontent.com/3851540/27513603-9b6497dc-599e-11e7-91d8-6b681a7a3e4e.png)

2.  將測試方法改為語意清楚、一眼可以看出問題的名稱

    ```cs
    [TestMethod()]
    public void 驗證AuthenticationHandler_未傳入token_回傳401()
    {
    }
    
    [TestMethod()]
    public void 驗證AuthenticationHandler_傳入token_回傳200()
    {
    }
    ```

3.  撰寫驗證 `未通過` 測試
    *   建立 request 物件
        
        > 因為是 web api，方法會被呼叫到都是透過接受 web request

        ```cs
        var request = new HttpRequestMessage();
        ```

    *   使用 NSubstitute 模擬外部驗證 token 的 service

        ```cs
        var authService = Substitute.For<IAuthenticationService>();
        authService.Verify(string.Empty).ReturnsForAnyArgs(false);
        ```

    *   建立測試目標程式的實體

        ```cs
        var authHandler = new AuthenticationHandler(authService);
        ```

    *   建立 AuthenticationHandler PrivateObject 物件

        > 因需要呼叫 protected 方法，PrivateObject 用法請參考 [Unit Test 想驗證 private method 該怎麼做？ - 使用 PrivateObject](http://blog.yowko.com/2017/06/unit-test-private-method.html)

        ```cs
        PrivateObject target = new PrivateObject(authHandler);
        ```

    *   預期回傳 401

        ```cs
        var expected = HttpStatusCode.Unauthorized;
        ```

    *   執行 AuthenticationHandler 中的 SendAsync 方法

        > 記得傳入 request 及 CancellationToken 物件

        ```cs
        var response = target.Invoke("SendAsync", new object[] { request, new CancellationToken() });
        ```

    *   取得回應結果

        ```cs
        var actual = response as Task<HttpResponseMessage>;
        ```

    *   驗證是否通過

        ```cs
        Assert.AreEqual(expected, actual.Result.StatusCode);
        ```
    *   完整程式碼

        ```cs
        [TestMethod()]
        public void 驗證AuthenticationHandler_未傳入token_回傳401()
        {
            //建立 request 物件
            var request = new HttpRequestMessage();
            //模擬外部驗證 token 的 service
            var authService = Substitute.For<IAuthenticationService>();
            authService.Verify(string.Empty).ReturnsForAnyArgs(false);
            // 建立 AuthenticationHandler 實體
            var authHandler = new AuthenticationHandler(authService);
            //建立 AuthenticationHandler PrivateObject 物件(因需要呼叫 protected 方法)
            PrivateObject target = new PrivateObject(authHandler);
            //預期回傳 401
            var expected = HttpStatusCode.Unauthorized;
            //act
            //傳入 request 及 CancellationToken 物件 執行 SendAsync 方法
            var response = target.Invoke("SendAsync", new object[] { request, new CancellationToken() });
            //取得回應結果
            var actual = response as Task<HttpResponseMessage>;
            //assert
            //驗證是否通過
            Assert.AreEqual(expected, actual.Result.StatusCode);
        }
        ```

4.  撰寫驗證 `通過` 測試
    * 新增一個 `StubAuthenticationHandler` 繼承 `DelegatingHandler` 並覆寫 `SendAsync`

        > 動作與建立 `AuthenticationHandler` 相同，但回傳內容修改為回應我們要的 HttpStatusCode.OK，為什麼需要這麼做等等會說明

    *   回應寫法有兩種

        ```cs
        internal class StubAuthenticationHandler : DelegatingHandler
        {
            protected override Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, CancellationToken cancellationToken)
            {
                //寫法一
                //return new TaskFactory<HttpResponseMessage>().StartNew(() => new HttpResponseMessage(HttpStatusCode.OK), cancellationToken);
                //寫法二
                return Task.FromResult(new HttpResponseMessage(HttpStatusCode.OK));
            }
        }
        ```

    *   建立 request 物件

        ```cs
        var request = new HttpRequestMessage();
        ```
    *   為 request 加入 Authorization header

        ```cs
        var token = "7e255df6-e953-438f-b51c-653a1939d103";
        request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", token);
        ```
    *   使用 NSubstitute 模擬外部驗證 token 的 service

        ```cs
        var authService = Substitute.For<IAuthenticationService>();
        authService.Verify(token).ReturnsForAnyArgs(true);
        ```

    *   建立 AuthenticationHandler ，並指定 InnerHandler 為我們新增的 stub handler

        ```cs
        var authHandler = new AuthenticationHandler(authService)
        {
            InnerHandler = new StubAuthenticationHandler()
        };
        ```

        *   這邊如果不是使用我們新增虛擬通過驗證的回應會出現下列錯誤

            ```
            System.InvalidOperationException occurred
            HResult=0x80131509
            Message=The inner handler has not been assigned.
            Source=System.Net.Http
            StackTrace:
            <Cannot evaluate the exception stack trace>
            ```

            ![3innerhandlererror](https://user-images.githubusercontent.com/3851540/27513604-9b856ff2-599e-11e7-9c98-c2d6264891f2.png)

        *   原因是通過驗證後，會去執行 `return base.SendAsync(request, cancellationToken);` 但我們建立用來測試 request 物件並沒有指定 target url 而造成沒有 handler 可以用來回應 request

        *   解決方式就是透過建立 `AuthenticationHandler` 時將 `InnerHandler` 指定為我們新增用來測試的 stub handler

            > 因為 InnerHandler 是 HttpMessageHandler 物件並指向下一層 pipeline，會使用這個物件當做 base.SendAsync 的執行對象，詳細資訊請參考 [蔡煥麟老師的 ASP.NET Web API 訊息處理器](http://www.huanlintalk.com/2013/01/aspnet-web-api-message-handlers.html) 一文

    *   建立 AuthenticationHandler PrivateObject 物件

        > 因需要呼叫 protected 方法，PrivateObject 用法請參考 [Unit Test 想驗證 private method 該怎麼做？ - 使用 PrivateObject](http://blog.yowko.com/2017/06/unit-test-private-method.html)

        ```cs
        PrivateObject target = new PrivateObject(authHandler);
        ```
    *   預期回傳 200

        ```cs
        var expected = HttpStatusCode.OK;
        ```
    
    *   傳入 request 及 CancellationToken 物件 執行 SendAsync 方法

        ```cs
        var response = target.Invoke("SendAsync", new object[] { request, new CancellationToken() });
        ```
    *   取得回應結果

        ```cs
        var actual = response as Task<HttpResponseMessage>;
        ```
    *   驗證是否通過

        ```cs
        Assert.AreEqual(expected, actual.Result.StatusCode);
        ```
    *   完整程式碼

        ```cs
        internal class StubAuthenticationHandler : DelegatingHandler
        {
            protected override Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, CancellationToken cancellationToken)
            {
                //return new TaskFactory<HttpResponseMessage>().StartNew(() => new HttpResponseMessage(HttpStatusCode.OK), cancellationToken);
                return Task.FromResult(new HttpResponseMessage(HttpStatusCode.OK));
            }
        }
        [TestMethod()]
        public void 驗證AuthenticationHandler_傳入token_回傳200()
        {
            //arrange
            //建立 request 物件
            var request = new HttpRequestMessage();
            //為 request 加入 Authorization header
            var token = "7e255df6-e953-438f-b51c-653a1939d103";
            request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", token);
            //模擬外部驗證 token 的 service
            var authService = Substitute.For<IAuthenticationService>();
            authService.Verify(token).ReturnsForAnyArgs(true);
            // 建立 AuthenticationHandler ，並指定 InnerHandler 為我們新增的 stub handler
            var authHandler = new AuthenticationHandler(authService)
            {
                InnerHandler = new StubAuthenticationHandler()
            };
            //建立 AuthenticationHandler PrivateObject 物件(因需要呼叫 protected 方法)
            PrivateObject target = new PrivateObject(authHandler);
            //預期回傳 200
            var expected = HttpStatusCode.OK;
            //act
            //傳入 request 及 CancellationToken 物件 執行 SendAsync 方法
            var response = target.Invoke("SendAsync", new object[] { request, new CancellationToken() });
            //取得回應結果
            var actual = response as Task<HttpResponseMessage>;
            //assert
            //驗證是否通過
            Assert.AreEqual(expected, actual.Result.StatusCode);
        }
        ```

## 心得

從 Web Api 2012 年問世至今也好幾年了，實際開發應用的專案少說也有數十個，使用到 Message Handler 的也不在少數(大部份用在驗證及 log)，但從來沒想過 Message Handler 也可以做測試，甚至還不覺得要測試，但定神細想 Message Handler 的確非常需要測試保護，一來重要性很高，二來有邏輯，在為 Message Handler 加上測試後，對於 Message Handler 程式碼的可維護性感覺提昇不少，真棒

在查資料過程中發現，從 Web Api 問世當時就有不少人針對 Message Handler 撰寫單元測試了XD。直到 2017 年的今天，如果我沒再去上 TDD 也就失去這個知道 Message Handler 單元測試的機會，不曉得還要井底窺天多久 @@"

這讓我聯想到程式設計師絕對不能自恃經驗豐富而自滿，要抱著永遠都有更好解決方式的想法來督促自己持續努力跟進步，人外有人、天外有天，只有放下先入為主的想法、適時清空自己才能學到更多

# 參考資訊

1.  [HTTP Message Handlers in ASP.NET Web API](https://docs.microsoft.com/en-us/aspnet/web-api/overview/advanced/http-message-handlers?WT.mc_id=DOP-MVP-5002594)
2.  [Unit testing Web API message handlers](http://blog.chatekar.com/unit-testing-web-api-message-handlers/)
3.  [ASP.NET Web API Series - Part 6: MessageHandler explained](http://byterot.blogspot.tw/2012/05/aspnet-web-api-series-messagehandler.html)
4.  [ASP.NET Web API Series - Part 7: Real world Message Handlers](http://byterot.blogspot.tw/2012/05/aspnet-web-api-messagehandler-real.html)
5.  [為 ASP.NET WEB API 加上簡易的 Token 驗證](/2017/02/aspnet-web-api-fixed-token.html)
6.  [ASP.NET Web API 訊息處理器](http://www.huanlintalk.com/2013/01/aspnet-web-api-message-handlers.html)
7.  [Unit Test 想驗證 private method 該怎麼做？ - 使用 PrivateObject](http://blog.yowko.com/2017/06/unit-test-private-method.html)
