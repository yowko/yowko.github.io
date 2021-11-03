---
title: "為 .NET Core Middleware 加上 Unit Test"
date: 2019-04-28T21:30:00+08:00
lastmod: 2021-11-03T21:30:31+08:00
draft: false
tags: ["dotnet core","Unit Test"]
slug: "dotnet-core-middieware-unit-test"
---
## 為 .NET Core Middleware 加上 Unit Test

.NET Core 的 Middleware 在 .NET Core 的 pipeline 中扮演了非常重要的角色，因此在實際應用上更需要確保功能與結果符合預期。第一次寫 middleware 的 unit test 順手紀錄過程，看看在 unit tese 上有沒有什麼需要留意的，

## 基本環境說明

1. macOS Mojave 10.14.4
2. .NET Core 2.2.101
3. ASP.NET Core Web Application (MVC) 預設專案範本
4. NuGet 套件
    - FluentAssertions 5.6.0
    - Microsoft.AspNetCore.Http 2.2.2
    - nunit 3.11.0
    - NUnit3TestAdapter 3.11.0
    - Microsoft.NET.Test.Sdk 15.9.0

## 加入 Middleware (兩個方式擇一即可)

> 加上檢查 header : request 在未包含指定的 header (auth) 時，即丟出 InvalidOperationException

- 直接將在 `Startup.cs` 的 `Configure` 方法中加入 middleware

    ```cs
    app.Use((context, next) =>
            {
                var auth = context.Request.Headers["auth"];
                if (!string.IsNullOrWhiteSpace(auth))
                {
                    return next();
                }

                throw  new InvalidOperationException("Missing header !!");
            });C
    ```

- 為 middleware 建立獨立類別 `AuthHeaderCheckMiddleware`

    1. 加入 middleware

        ```cs
        public class AuthHeaderCheckMiddleware
        {
            private readonly RequestDelegate _next;

            public AuthHeaderCheckMiddleware(RequestDelegate next)
            {
                _next = next;
            }

            public async Task InvokeAsync(HttpContext context)
            {
                var auth = context.Request.Headers["auth"];
                if (!string.IsNullOrWhiteSpace(auth))
                {
                    await _next(context);
                }

                throw new InvalidOperationException("Missing header !!");
            }
        }
        ```

    2. 註冊 middleware

        > 註冊位置與上個方式相同，但需要留意想要在哪一層處理：愈早註冊就會愈早檢查，只是出現錯誤的可讀性就會愈差

        ```cs
        app.UseMiddleware<AuthHeaderCheckMiddleware>();
        ```

## 撰寫 Middleware 的 Unit Test

1. 建立 mock HttpContext

    > 為了符合 Unit Test 的基本定義， mock HttpContext 的隔離動作不可少，.NET Core 在這點上有長足的進步，直接使用 `DefaultHttpContext` 即可，不需自行 fake (需要參考並 using `Microsoft.AspNetCore.Http`)

    ```cs
    var httpContext = new DefaultHttpContext();
    ```

2. 建立 Unit Test

    - 未提供 auth header 時會出現 InvalidOperationException

        ```cs
        [Test]
        public async Task AthHeaderCheckMiddleware_Without_header_Should_Get_InvalidOperatorException()
        {
            //arrange
            var target = new AuthHeaderCheckMiddleware((innerHttpContext) => null);

            var httpContext = new DefaultHttpContext();

            var expect = new InvalidOperationException("Missing header !!");
            
            //act
            Func<Task> action = () => target.InvokeAsync(httpContext);


            //assert
            action.Should().Throw<InvalidOperationException>()
                .WithMessage(expect.Message);
        }
        ```

    - 提供 auth header 則出現 NullReferenceException (因 next 傳入 null)

        ```cs
        [Test]
        public async Task AthHeaderCheckMiddleware_With_Auht_header_Should_Get_NullReferenceException()
        {
            //arrange
            var target = new AuthHeaderCheckMiddleware((innerHttpContext) => null);
              
            var httpContext = new DefaultHttpContext();
            httpContext.Request.Headers.Add("auth","test");

            //act
            Func<Task> action = () => target.InvokeAsync(httpContext);
            

            //assert
            action.Should().Throw<NullReferenceException>();
            
        }
        ```

## 心得

實際使用起來與過去 ASP.NET Web API 的 Message Handler 測試流程很像，但又更輕鬆點：不用建立 request object 也不用 mock HttpContex，使用體驗很棒，愈用 .NET Core 愈覺得好用呀

像這樣的中介層因為事關重大，沒有測試保護 (不管是 unit test 或是 integration test) 對於新加入團隊的成員難免都是進入障礙，團隊也很難確保穩定的產出品質，就算是團隊成員都很小心，開發時不會犯錯，也無法確保所有成員過了半年後還記得商業邏輯跟程式碼用意，雖然每個人立場不同(也許覺得測試很浪費時間)，但也許該試著看看為什麼有那麼多人推崇測試

## 參考資訊

1. [撰寫自訂的 ASP.NET Core 中介軟體](https://docs.microsoft.com/zh-tw/aspnet/core/fundamentals/middleware/write?view=aspnetcore-2.2&WT.mc_id=DOP-MVP-5002594)
2. [ASP.NET Core 2.1 middlewares part 2: Unit test a custom middleware](http://anthonygiretti.com/2018/09/04/asp-net-core-2-1-middlewares-part2-unit-test-a-custom-middleware/)
