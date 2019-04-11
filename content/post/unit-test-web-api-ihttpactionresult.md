---
title: "Unit Test 如何驗證 ASP.NET Web Api 的 IHttpActionResult"
date: 2017-06-15T23:39:00+08:00
lastmod: 2018-09-23T11:32:21+08:00
draft: false
tags: ["MSTest","NUnit","Unit Test","xUnit"]
slug: "unit-test-web-api-ihttpactionresult"
aliases:
    - /2017/06/unit-test-web-api-ihttpactionresult.html
---
# Unit Test 如何驗證 ASP.NET Web Api 的 IHttpActionResult
經過 91 哥三周的 TDD 訓練，我竟自大地以為我會 unit test 了，新專案到手立馬開啟 TDD 開發模式，不要臉地覺得同事寫的 api 可測試性低相當不可取 @@" ，於是乎動手改造了起來，結果就卡住了 哈哈 果然上課跟實作還是存在著極大的落差呀，順手紀錄一下

## 目標 api 內容

*   接受一個 url 的 query string 參數

    *   沒有傳參數或是參數是空字串就回傳 `BadRequest`
    *   一般正常 string 就回傳 `Ok`

*   範例程式碼

    ```cs
    public class DemoController : ApiController
    {
        public IHttpActionResult Post([FromUri]string name)
        {
            if (string.IsNullOrWhiteSpace(name))
                return BadRequest();
            else
                return Ok();
        }
    }
    ```

## 驗證概念說明

因為本例中沒有回傳其他內容，所以只要驗證 api 回傳型別是否合乎預期，需要注意的只是各個 test framework 語法上的不同

## MSTest 如何驗證

1.  驗證 `BadRequest`

    ```cs
    [TestMethod()]
    public void Post_Name_StringEmpty_Return_BadRequest()
    {
        var target=new DemoController();
        var name = string.Empty;
        var expected = typeof(BadRequestResult);
        //act
        var actualAction = target.Post(name);
        var actual = actualAction as BadRequestResult;
        //assert
        Assert.IsNotNull(actual);
        Assert.IsInstanceOfType(actual, expected);
    }
    ```

2.  驗證 `Ok`

    ```cs
    [TestMethod()]
    public void Post_Name_Yowko_Return_Ok()
    {
        var target = new DemoController();
        var name = "Yowko";
        var expected = typeof(OkResult);
        //act
        var actualAction = target.Post(name);
        var actual = actualAction as OkResult;
        //assert
        Assert.IsNotNull(actual);
        Assert.IsInstanceOfType(actual, expected);
    }

## NUnit 3 如何驗證

1.  驗證 `BadRequest`

    ```cs
    [Test()]
    public void Post_Name_StringEmpty_Return_BadRequest()
    {
        var target = new DemoController();
        var name = string.Empty;
        var expected = typeof(BadRequestResult);
        //act
        var actual = target.Post(name);
        //assert
        Assert.IsInstanceOf( expected,actual);
    }
    ```

2.  驗證 `Ok`

    ```cs
    [Test()]
    public void Post_Name_Yowko_Return_Ok()
    {
        var target = new DemoController();
        var name = "Yowko";
        var expected = typeof(OkResult);
        //act
        var actual = target.Post(name);
        //assert
        Assert.IsInstanceOf(expected, actual);
    }
    ```

## xUnit.net 2.0 如何驗證

1.  驗證 `BadRequest`

    ```cs
    [Fact()]
    public void Post_Name_StringEmpty_Return_BadRequest()
    {
        var target = new DemoController();
        var name = string.Empty;
        var expected = typeof(BadRequestResult);
        //act
        var actual = target.Post(name);
        //assert
        Assert.IsType(expected, actual);
    }
    ```

2.  驗證 `Ok`

    ```cs
    [Fact()]
    public void Post_Name_Yowko_Return_Ok()
    {
        var target = new DemoController();
        var name = "Yowko";
        var expected = typeof(OkResult);
        //act
        var actual = target.Post(name);
        //assert
        Assert.IsType(expected, actual);
    }
    ```

## 心得

這讓我想起第一次上 TDD 時我也覺得我都會了，工作一忙沒機會用，到現在又上完了一次還是不會用，正如 91 哥說的，學習到內化是沒有捷徑的，只能靠不斷的練習，還好這次專案可以讓我練練功，期望這次可以真的學會

# 參考資訊

1.  [How to write unit test case for BadRequest?](https://stackoverflow.com/questions/39115867/how-to-write-unit-test-case-for-badrequest)
2.  [Unit testing a ASP.NET WebAPI 2 controller](https://blogs.msmvps.com/theproblemsolver/2013/11/13/unit-testing-a-asp-net-webapi-2-controller/)
3.  [WebAPI2 Unit Testing with IHttpActionResult](https://gist.github.com/RayKwon/6260978)
4.  [MSTest,NUnit 3,xUnit.net 2.0 比較](//blog.yowko.com/2017/02/MStest-NUnit3-xUnit.net2-Compare.html)
