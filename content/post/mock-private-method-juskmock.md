---
title: "如何 Mock Private Method 的回傳值 - 使用 JuskMock"
date: 2017-07-10T01:00:00+08:00
lastmod: 2021-11-02T22:41:58+08:00
draft: false
tags: ["套件","MSTest","Unit Test"]
slug: "mock-private-method-juskmock"
aliases:
    - /2017/07/mock-private-method-juskmock.html
---
## 如何 Mock Private Method 的回傳值 - 使用 JuskMock

前一篇筆記 [使用 Moq 來 Mock protected Method](/moq-mock-protected-method) 文末心得中提到傳言中付費的 mock framewrok 號稱無論是什麼狀況都能 mock，想說改天要找個機會來測試一下，結果自己忍不住好奇心，立馬測試了起來 XD

就來看看該如何使用 JustMock 來 mock private method 吧

## 關於 JustMock

這是 Telerik 出品的 mock framework，Telerik 有多項 UI 相關產品，使用的人也不少，尤其是在 Bootstrap 還沒有那麼風行的年代，工程師普遍對於 UI 苦手，Telerik 的產品著實解救了許多被客戶嫌棄沒有美感的工程師。如果沒參與到那個年代，也許用過或是聽過一套免費的網路封包截錄工具 - Fiddler ，這也是 Telerik 的產品

至於 JustMock 的特色，請直接參考官網介紹 - [JustMock](http://www.telerik.com/products/mocking.aspx)，功能非常強大，缺點是它必需要付費使用，好消息是它提供免費 30 天的試用可以拿來測試

JustMock 有提供 JustMock Lite 的 open source 免費版，功能少了很多(最需要的 non-public 就不支援)，官網上有提供功能比較，詳細內容請參考 [JustMock Lite](http://www.telerik.com/justmock/free-mocking)

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

> 以上內容皆與 [使用 Moq 來 Mock protected Method](/moq-mock-protected-method) 相同

## 撰寫測試程式

1. 安裝 JustMock - [下載點](https://www.telerik.com/download-trial-file/v2/justmock)
2. 引用 namespace

    ```cs
    using Telerik.JustMock;
    ```

3. 啟用 JustMock
    * Visual Studio 主選單 JustMock --> Enable Profiler

        ![1enable](https://user-images.githubusercontent.com/3851540/28023469-9458239a-65c0-11e7-835a-b95a1e2e0a74.png)

    * 未啟用錯誤訊息

        * 訊息內容

            ```log
            Test Name: Get_StringEmpty_BadRequestResult
            Test FullName: UnitTestStaticField.Controllers.Tests.ValuesControllerTests.Get_StringEmpty_BadRequestResult
            Test Source: C:\Users\YowkoTsai\documents\visual studio 2017\Projects\UnitTestStaticField\UnitTestStaticFieldMTests\Controllers\ValuesControllerTests.cs : line 111
            Test Outcome: Failed
            Test Duration: 0:00:00.1605095
                        
            Result StackTrace: 
                at Telerik.JustMock.Core.ProfilerInterceptor.ThrowElevatedMockingException(MemberInfo member)
                at Telerik.JustMock.Core.MocksRepository.CheckMethodInterceptorAvailable(IMatcher instanceMatcher, MethodBase method)
                at Telerik.JustMock.Core.MocksRepository.AddArrange(IMethodMock methodMock)
                at Telerik.JustMock.Core.MocksRepository.Arrange[TMethodMock](Object instance, MethodBase method, Object[] arguments, Func`1 methodMockFactory)
                at Telerik.JustMock.Expectations.NonPublicExpectation.<>c__DisplayClass22`1.<Arrange>b__20()
                at Telerik.JustMock.Core.ProfilerInterceptor.GuardInternal[T](Func`1 guardedAction)
                at Telerik.JustMock.Expectations.NonPublicExpectation.Arrange[TReturn](Object target, String memberName, Object[] args)
                at UnitTestStaticField.Controllers.Tests.ValuesControllerTests.Get_StringEmpty_BadRequestResult() in C:\Users\YowkoTsai\documents\visual studio 2017\Projects\UnitTestStaticField\UnitTestStaticFieldMTests\Controllers\ValuesControllerTests.cs:line 114
            Result Message: 
            Test method UnitTestStaticField.Controllers.Tests.ValuesControllerTests.Get_StringEmpty_BadRequestResult threw exception: 
            Telerik.JustMock.Core.ElevatedMockingException: Cannot mock 'Boolean detectStringEmpty(System.String)'. The profiler must be enabled to mock, arrange or execute the specified target.
            ```

        * 錯誤截圖

            ![2error](https://user-images.githubusercontent.com/3851540/28023471-94849434-65c0-11e7-8af1-df5a6364bd09.png)

4. arrange
    * 使用 JuskMock 語法 mock 測試目標

        ```cs
        var target = Mock.Create<ValuesController>();
        ```

    * 使用 JuskMock 語法 mock 測試目標方法回傳值

        >* 重點 一：non-public 的 member 需使用 `NonPublic` api
        >* 重點 二：執行目標方法具有回傳值，Arrange 要使用泛型版本並指定回傳型別
        >* 重點 三：其他用法請參考官方文件 [Mocking Non-public Members and Types](http://docs.telerik.com/help/justmock/advanced-usage-mocking-non-public-members-and-types.html)

        ```cs
        Mock.NonPublic.Arrange<bool>(target, "detectStringEmpty", new object[] { string.Empty }).Returns(true);
        ```

    * 通知 JuskMock 需執行原測試目標方法

        > 如果不加上 `.CallOriginal()` 會執行 JustMock 建立出來的 instance 方法，結果會跟預期完全不同，要特別留意

        ```cs
        Mock.Arrange(() => target.Get(string.Empty)).CallOriginal();
        ```

    * 定義預期結果

        ```cs
        var expected = typeof(BadRequestResult);
        ```

5. act
    * 執行測試目標程式方法

        ```cs
        var actualAction = target.Get(string.Empty);
        ```

    * 將實際執行結果轉型為預期的 `BadRequestResult`

        ```cs
        var actual = actualAction as BadRequestResult;
        ```

6. assert

    * 檢查執行結果不能是 null

        ```cs
        Assert.IsNotNull(actual);
        ```

    * 驗證執行結果是否與預期相符

        ```cs
        Assert.IsInstanceOfType(actual, expected);
        ```

7. 完整程式碼

    ```cs
    [TestMethod()]
    public void Get_StringEmpty_BadRequestResult()
    {
        //arrange
        var target = Mock.Create<ValuesController>();
        Mock.NonPublic.Arrange<bool>(target, "detectStringEmpty", new object[] { string.Empty }).Returns(true);
        Mock.Arrange(() => target.Get(string.Empty)).CallOriginal();
        var expected = typeof(BadRequestResult);
        //act
        var actualAction = target.Get(string.Empty);
        var actual = actualAction as BadRequestResult;
                
        //assert
        Assert.IsNotNull(actual);
        Assert.IsInstanceOfType(actual, expected);
    }
    ```

## 心得

使用 JustMock 的確達成我理想中完全不需調整測試目標程式的概念，但要不要實際採用 JustMock，我個人倒是持保留態度，原因詳述如下：

1. 速度慢

    > 執行測試時明顯感受到執行速度比未使用 JustMock 的版本緩慢，我自己測試下來常常超過 500 ms，而且偵錯時會慢更多，這會讓整個開發效率降低而不利測試程式的開發

2. 價格不斐

    > 如果是只購買 JustMock 一個人是 399 美金，我沒看到時間限制應該是買斷的，如果有助於整體開發這錢絕點值得花，但軟體的導入需要配合其他團隊成員及 CI server，這樣一來就不是想用自己買就可以解決的了

排除上述原因，JustMock 真的非常方便，尤其在面對 legacy code 時，讓開發人員可以專心在測試程式的撰寫上，而不需分心想著該如何重構才能讓程式具備可測試性，對於測試程式的開發實在是一大利器

## 參考資訊

1. [使用 Moq 來 Mock protected Method](/moq-mock-protected-method)
2. [JustMock](http://www.telerik.com/products/mocking.aspx)
3. [JustMock Lite](http://www.telerik.com/justmock/free-mocking)
4. [Telerik JustMock Documentation](http://docs.telerik.com/help/justmock/introduction.html)
5. [Mocking Non-public Members and Types](http://docs.telerik.com/help/justmock/advanced-usage-mocking-non-public-members-and-types.html)
