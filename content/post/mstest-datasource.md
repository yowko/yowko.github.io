---
title: "讓 MSTest 支援使用資料集進行重覆測試"
date: 2017-08-01T23:52:00+08:00
lastmod: 2021-11-02T10:52:52+08:00
draft: false
tags: ["MSTest","Unit Test"]
slug: "mstest-datasource"
aliases:
    - /2017/08/mstest-datasource.html
---
## 讓 MSTest 支援使用資料集進行重覆測試

之前文章 [NUnit 幾個參數化測試的方式](/nunit-parameterized-test) 介紹到如何在 NUnit 單元測試中使用 `TestCase`、`TestCaseSource`、`Values`、`ValueSource` 來進行參數化測試，也曾在 [餵資料集給 SpecFlow 來執行測試及驗證](/specflowoutline) 中介紹到如何使用 SpecFlow Outline 達成相同目的

公司最近專案的測試 framework 使用 MSTest，不過 MSTest 原生並沒有支援像 NUnit 及 SpecFlow 直接指定值的參數化測試方式，嘗試了幾個方式後，筆記一下個人覺得不錯的做法

## 安裝 NuGet 套件 - MSTestHacks

1. 使用 Nuget Package Explorer
    * 搜尋 `MSTestHacks`

        ![1INSTALL](https://user-images.githubusercontent.com/3851540/28834212-3c810096-7714-11e7-9ce8-1eab989d557e.png)

2. 使用 Package Manager Console

    > `Install-Package MSTestHacks`

## 修改測試程式

1. 引用 namesapce

    ```cs
    using MSTestHacks;
    ```

2. 讓測試 class 繼承 `TestBase`

    ```cs
    [TestClass]
    public class UnitTest1 : TestBase
    {
    }
    ```

3. 建立測試用的資料集合

    * 可以使用 `Property`、`Field`、`Method`

        * Prperty

            ```cs
            private IEnumerable<string> Projects
            {
                get
                {
                    return new List<string> { "Yowko Test", "Unit Test", "Yowko Demo" };
                }
            }
            ```

        * Method

            ```cs
            private IEnumerable<int> GetAges()
            {
                return new List<int> { 1,2,3};
            }
            ```

    * 需回傳 `IEnumerable`

4. 將測試用資料集指定給測試方法

    > 在測試方法加上 `DataSource` ，並指定 `"{namespace}.{testclass}.{property|field|method}"`

    * Property

        ```cs
        [TestMethod]
        [DataSource("DemoMSTest.Tests.UnitTest1.Projects")]
        public void TestMethod2()
        {
        }
        ```

    * Method

        ```cs
        [TestMethod]
        [DataSource("DemoMSTest.Tests.UnitTest1.GetAges")]
        public void TestMethod1()
        {
        }
        ```

5. 逐一取出資料集內容並轉型為正確型別即可進行驗證

    > this.TestContext.GetRuntimeDataSourceObject<{目標型別}>();

    * Property

        ```cs
        [TestMethod]
        [DataSource("DemoMSTest.Tests.UnitTest1.Projects")]
        public void TestMethod2()
        {
            var project = this.TestContext.GetRuntimeDataSourceObject<string>();
            Console.WriteLine(project);
            Assert.IsTrue(project.Contains("Test"));
        }
        ```

    * Method

        ```cs
        [TestMethod]
        [DataSource("DemoMSTest.Tests.UnitTest1.GetAges")]
        public void TestMethod1()
        {
            var age = this.TestContext.GetRuntimeDataSourceObject<int>();
            Console.WriteLine(age);
            Assert.IsFalse(age<0);
        }
        ```

6. 測試結果
    * 會顯示在同一個測試方法中，有多個測試結果

        ![4result1](https://user-images.githubusercontent.com/3851540/28834218-3d0ecd90-7714-11e7-884d-4de67cd16b3b.png)

        ![5result2](https://user-images.githubusercontent.com/3851540/28834217-3d09663e-7714-11e7-8824-d8a12fa7205a.png)

    * 有一測試資料讓測試 fail，則該測試方法就會被標記為 fail

        ![2result1](https://user-images.githubusercontent.com/3851540/28834215-3cf7c230-7714-11e7-9516-af27c2e28f86.png)

        ![3result2](https://user-images.githubusercontent.com/3851540/28834216-3d04205c-7714-11e7-936a-06b8939a1c64.png)

## 如何驗證例外

曾經在 [使用 MSTest、Nunit 3、xUnit.net 2.0、NSubstitute、FluentAssertions 驗證例外(exception)](/mstestnunit-3xunitnet) 介紹過可以經由安裝 `MSTestExtensions` 就可以直接使用 `Assert.Throws<T>()` 來驗證例外，不需在測試方法上加上 `[ExpectedException(typeof({exception 型別}))]` 以避免失真，但 `MSTestExtensions` 與今天介紹的 `MSTestHacks` 都需要讓測試程式繼承自訂的 class，不過 c# 的語言特性只允許繼承一個 class，幸虧 `MSTestHacks` 已經預想到這個問題 - 它有提供驗證 exception 的語法

1. 僅驗證特定的 exception

    ```cs
    ExceptionAssert.Throws<ArgumentNullException>(()=>target.Get());
    ```

2. 驗證 exception 及錯誤訊息

    ```cs
    ExceptionAssert.Throws<ArgumentException>(() => target.Post(),expectedMessage);
    ```

## 心得

原本想要使用 MSTest 的 DataSource，但預設只支援 xml、csv 跟 database., 嘗試更新到 MSTest V2 測試 DataRow 後，發現使用方法提供資料的支援度不佳，幸虧後來找到 `MSTestHacks`，省了不少事，加上也涵蓋了 `MSTestExtensions` 簡易的 exception 驗證方式，非常方便

## 參考資訊

1. [NUnit 幾個參數化測試的方式](/nunit-parameterized-test)
2. [餵資料集給 SpecFlow 來執行測試及驗證](/specflowoutline)
3. [使用 MSTest、Nunit 3、xUnit.net 2.0、NSubstitute、FluentAssertions 驗證例外(exception)](/mstestnunit-3xunitnet)
4. [MSTestHacks](https://github.com/Thwaitesy/MSTestHacks)
