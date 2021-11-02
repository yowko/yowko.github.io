---
title: "使用 Moq 來 Mock protected Method"
date: 2017-07-09T20:20:00+08:00
lastmod: 2021-11-02T22:43:59+08:00
draft: false
tags: ["套件","MSTest","Unit Test"]
slug: "moq-mock-protected-method"
aliases:
    - /2017/07/moq-mock-protected-method.html
    - /2017/07/moq-mock-protected-method/
---
## 使用 Moq 來 Mock protected Method

跟同事討論到在進行單元測試時，目標方法使用到其他非 public 方法，而且想要 mock 這個方法的回傳值該怎麼做？

其實這個問題我之前也思考過，一直沒有找到很好的辦法，只印象中 Moq 可以 mock protected method 的回傳，但沒有實作過，剛好趁這個機會紀錄一下

## 基本設定

Restful Web Api 專案，Get 方法需要傳入一個 string 參數，如果未傳入 string 或傳入的參數是空字串就回傳 `BadRequestResult` ，傳入參數是正常 string 就回傳 `OkResult`, 其中檢查 string 的邏輯與動作就獨立包裝在 `detectStringEmpty` function 中，程式碼內容如下

```cs
public class ValuesController : ApiController
{
    public IHttpActionResult Get(string value)
    {
        var isNullorEmpty= detectStringEmpty(value);
        if (isNullorEmpty)
            return BadRequest();
        else
            return Ok();
    }

    private bool detectStringEmpty(string value)
    {
        if (string.IsNullOrWhiteSpace(value))
            return true;
        else
            return false;
    }
}
```

## 修改目標程式碼

> 因為 Moq 只能 mock protected method，所以要將 private method 修改為 protected

1. 將 private 改為 protected

    ```cs
    protected bool detectStringEmpty(string value)
    {
        if (string.IsNullOrWhiteSpace(value))
            return true;
        else
            return false;
    }
    ```

2. 加上 virtual

    > Moq 只能 mock `protected virtual` method

    * 未加 `virtual` 的錯誤訊息

        * 訊息內容

            ```log
            Test Name: Get_StringEmpty_BadRequest
            Test FullName: UnitTestStaticField.Controllers.Tests.ValuesControllerTests.Get_StringEmpty_BadRequest
            Test Source: C:\Users\YowkoTsai\documents\visual studio 2017\Projects\UnitTestStaticField\UnitTestStaticFieldMTests\Controllers\ValuesControllerTests.cs : line 71
            Test Outcome: Failed
            Test Duration: 0:00:00.0531624
            
            Result StackTrace: 
                at Moq.Mock.ThrowIfCantOverride(Expression setup, MethodInfo method)
                at Moq.Mock.<>c__DisplayClass65_0`2.<Setup>b__0()
                at Moq.PexProtector.Invoke[T](Func`1 function)
                at Moq.Mock.Setup[T,TResult](Mock`1 mock, Expression`1 expression, Condition condition)
                at Moq.Protected.ProtectedMock`1.Setup[TResult](String methodName, Boolean exactParameterMatch, Object[] args)
                at Moq.Protected.ProtectedMock`1.Setup[TResult](String methodName, Object[] args)
                at UnitTestStaticField.Controllers.Tests.ValuesControllerTests.Get_StringEmpty_BadRequest() in C:\Users\YowkoTsai\documents\visual studio 2017\Projects\UnitTestStaticField\UnitTestStaticFieldMTests\Controllers\ValuesControllerTests.cs:line 79
            Result Message: 
            Test method UnitTestStaticField.Controllers.Tests.ValuesControllerTests.Get_StringEmpty_BadRequest threw exception: 
            System.NotSupportedException: Invalid setup on a non-virtual (overridable in VB) member: mock => mock.detectStringEmpty(It.IsAny<String>())
            ```

        * 錯誤截圖

            ![1non-virtualerror](https://user-images.githubusercontent.com/3851540/27993755-807e94b4-64e2-11e7-8bee-7714f790d226.png)

3. 修改後程式碼

    ```cs
    protected virtual bool detectStringEmpty(string value)
    {
        if (string.IsNullOrWhiteSpace(value))
            return true;
        else
            return false;
    }
    ```

## 撰寫測試程式

1. 專案使用 NuGet 加入 Moq
2. 引用 namespace

    ```cs
    using Moq;
    using Moq.Protected;
    ```

3. arrange

    * 使用 Moq 語法 mock 目標程式

        ```cs
        var target = new Mock<ValuesController>() { CallBase = true };
        ```

    * mock protected virtual 方法

        > 有兩種寫法

        * 寫法一

            ```cs
            var target = new Mock<ValuesController>();
            target.Protected()
            .Setup<bool>("detectStringEmpty", new object[] { string.Empty })
            .Returns(true);
            ```

        * 寫法二

            ```cs
            var target = new Mock<ValuesController>();
            target.Protected()
            .Setup<bool>("detectStringEmpty",ItExpr.IsAny<string>())
            .Returns(true);
            ```

    * 定義預期結果

        ```cs
        var expected = typeof(BadRequestResult);
        ```

4. act
    * 使用 Moq 語法來執行測試目標程式方法

        ```cs
        var actualAction = target.Object.Get(string.Empty);
        ```

        > 這邊要特別注意，如果前面 mock 測試目標時如果沒有傳入 `{ CallBase = true }`，在執行測試目標程式方法會永遠回傳 null 而造成測試結果與預期有落差

    * 將實際執行結果轉型為預期的 `BadRequestResult`

        ```cs
        var actual = actualAction as BadRequestResult;
        ```

5. assert
    * 檢查執行結果不能是 null

        ```cs
        Assert.IsNotNull(actual);
        ```

    * 驗證執行結果是否與預期相符

        ```cs
        Assert.IsInstanceOfType(actual, expected);
        ```

6. 完整程式碼

    ```cs
    [TestMethod()]
    public void Get_StringEmpty_BadRequest()
    {
        //arrange
        var target = new Mock<ValuesController>() { CallBase = true };
        target.Protected()
            .Setup<bool>("detectStringEmpty", ItExpr.IsAny<string>())
            .Returns(true);
        var expected = typeof(BadRequestResult);
        //act
        var actualAction = target.Object.Get(string.Empty);
        var actual = actualAction as BadRequestResult;
        //assert
        Assert.IsNotNull(actual);
        Assert.IsInstanceOfType(actual, expected);
    }
    ```

## 心得

原本以為 mock framework 用法大同小異，實際嘗試後才發現各個 mock framework 語法與思維差距很大，使用上也不少眉眉角角需要留意，功能當然也各有優缺點，似乎還沒有可以符合各種情境的 framework 來降低使用門檻，聽說付費的 mock framework 功能異常強大，或許該找個時間來測試看看

回到今天的主題：測試目標程式本身只需要將 private 改為 protected virtual 就可以順利達成測試目的，雖然這樣的修改會讓 method 本身對外部程式的可見度提高，但用這點修改換取程式的可測試性應該是可以被接受的，只是如果有辦法完全不修改測試目標程式就完成測試更加完美了，假設有幸找到其他方法，我再來分享

## 參考資訊

1. [Mocking protected members with Moq](http://blogs.clariusconsulting.net/kzu/mocking-protected-members-with-moq/)
2. [moq/moq4](https://github.com/Moq/moq4/wiki/Quickstart)
3. [Calling original method with Moq](https://stackoverflow.com/questions/3073110/calling-original-method-with-moq)
