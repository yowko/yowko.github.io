---
title: "使用 Fluent Automation 與 Selenium 打造語意化 Web UI 測試程式"
date: 2017-06-08T00:14:00+08:00
lastmod: 2021-11-03T16:33:28+08:00
draft: false
tags: ["套件","TDD","Tools","Unit Test"]
slug: "fluent-automation-selenium-web-ui-test"
aliases:
    - /2017/06/fluent-automation-selenium-web-ui-test.html
---
## 使用 Fluent Automation 與 Selenium 打造語意化 Web UI 測試程式

之前文章 [使用 Selenium IDE 與 C# 做 Web UI 測試](/selenium-ide-c-sharp-web-ui-test)，介紹了如何使用 Selenium IDE 錄製網頁操作並將過程 export 成 c# 測試程式，雖然很方便但 Selenium api 太像程式碼，一般而言測試案例是用來描述需求，這樣會讓測試案例的可讀性降低，一旦測試的可讀性降低，漸漸的就會造成維護不易，時間一久就愈來愈少人願意維護而造成 code coverage 不增反降，這對於程式碼品質絕對有負面影響。

今天來紀錄一下 91 哥在 twMVC 活動中介紹的 Fluent Automation 套件，看它可以如何讓我們的 Web UI 測試有更好的可讀性

## 基本設定

案例會持續延用 [使用 Selenium IDE 與 C# 做 Web UI 測試](/selenium-ide-c-sharp-web-ui-test)，錄製登入 GitHub

* 主要測試程式如下(如果你懶得錄，可以使用下面的版本，但請記得改帳號跟密碼)

    ```cs
    using System;
    using System.Text;
    using System.Text.RegularExpressions;
    using System.Threading;
    using NUnit.Framework;
    using OpenQA.Selenium;
    using OpenQA.Selenium.Firefox;
    using OpenQA.Selenium.Support.UI;
    using FluentAutomation;
    
    namespace SeleniumTests
    {
        [TestFixture]
        public class Ngithub
        {
            private IWebDriver driver;
            private StringBuilder verificationErrors;
            private string baseURL;
            private bool acceptNextAlert = true;
            
            [SetUp]
            public void SetupTest()
            {
                driver = new FirefoxDriver();
                baseURL = "https://github.com/";
                verificationErrors = new StringBuilder();
            }
            
            [TearDown]
            public void TeardownTest()
            {
                try
                {
                    driver.Quit();
                }
                catch (Exception)
                {
                    // Ignore errors if unable to close the browser
                }
                Assert.AreEqual("", verificationErrors.ToString());
            }
            
            [Test]
            public void TheNgithubTest()
            {
                driver.Navigate().GoToUrl(baseURL + "/login");
                driver.FindElement(By.Id("login_field")).Clear();
                driver.FindElement(By.Id("login_field")).SendKeys("{帳號}");
                driver.FindElement(By.Id("password")).Clear();
                driver.FindElement(By.Id("password")).SendKeys("{密碼}");
                driver.FindElement(By.Name("commit")).Click();
                // 這邊要稍等一下，讓 github 完成 redirect
                Thread.Sleep(1000);
                //驗證網址是否正確
                Assert.AreEqual(baseURL, driver.Url);
            }
            
            private bool IsElementPresent(By by)
            {
                try
                {
                    driver.FindElement(by);
                    return true;
                }
                catch (NoSuchElementException)
                {
                    return false;
                }
            }
            
            private bool IsAlertPresent()
            {
                try
                {
                    driver.SwitchTo().Alert();
                    return true;
                }
                catch (NoAlertPresentException)
                {
                    return false;
                }
            }
            
            private string CloseAlertAndGetItsText()
            {
                try
                {
                    IAlert alert = driver.SwitchTo().Alert();
                    string alertText = alert.Text;
                    if (acceptNextAlert)
                    {
                        alert.Accept();
                    }
                    else
                    {
                        alert.Dismiss();
                    }
                    return alertText;
                }
                finally
                {
                    acceptNextAlert = true;
                }
            }
        }
    }
    ```

## 安裝 NuGet 套件

1. FluentAutomation.SeleniumWebDriver
    * 專案上按右鍵 --> Manage NuGet Packages

        ![1nuget](https://user-images.githubusercontent.com/3851540/26888548-f0aae4b8-4bdd-11e7-8cf4-1279dac0bc85.png)

    * Browse --> 搜尋 `FluentAutomation.SeleniumWebDriver` --> Install

        ![2install](https://user-images.githubusercontent.com/3851540/26888549-f0d1bf98-4bdd-11e7-8e95-64b54034cad8.png)

    * 會一併安裝其他相關套件

        ![3otherpacks](https://user-images.githubusercontent.com/3851540/26888550-f0e78d32-4bdd-11e7-8781-fa4c7a354e9b.png)

2. Selenium.WebDriver.ChromeDriver
    * 專案上按右鍵 --> Manage NuGet Packages

        ![1nuget](https://user-images.githubusercontent.com/3851540/26888548-f0aae4b8-4bdd-11e7-8cf4-1279dac0bc85.png)

    * Browse --> 搜尋 `Selenium.WebDriver.ChromeDriver` --> Install

        ![4install](https://user-images.githubusercontent.com/3851540/26888553-f0ee5446-4bdd-11e7-80d7-3ca8b766dde1.png)

    * 這邊留意一下套件名稱，名稱很容易混洧

        ![5mass](https://user-images.githubusercontent.com/3851540/26888551-f0ea08c8-4bdd-11e7-9470-bcb17818579e.png)

    * 這個套件我試了好久才試出來，如果沒裝會出現錯誤訊息
        * 訊息內容

            ```log
            Test Name: Test1
            Test FullName: TestVSIXN3Tests.SampleTests.Test1
            Test Source: C:\Users\yowko.tsai\Documents\Visual Studio 2017\Projects\TestVSIX\TestVSIXN3Tests\SampleTests.cs : line 24
            Test Outcome: Failed
            Test Duration: 0:00:01.732
                        Result StackTrace: 
            at FluentAutomation.BaseCommandProvider.<>c__DisplayClass9.<WaitUntil>b__8()
            at FluentAutomation.BaseCommandProvider.Act(CommandType commandType, Action action)
            at FluentAutomation.BaseCommandProvider.WaitUntil(Expression`1 conditionAction, TimeSpan timeout)
            at FluentAutomation.BaseCommandProvider.Act(CommandType commandType, Action action)
            at FluentAutomation.CommandProvider.Navigate(Uri url)
            at FluentAutomation.ActionSyntaxProvider.Open(Uri url)
            at FluentAutomation.ActionSyntaxProvider.Open(String url)
            at TestVSIXN3Tests.SampleTests.Test1() in C:\Users\yowko.tsai\Documents\Visual Studio 2017\Projects\TestVSIX\TestVSIXN3Tests\SampleTests.cs:line 25
            --InvalidOperationException
            at OpenQA.Selenium.Remote.RemoteWebDriver.UnpackAndThrowOnError(Response errorResponse)
            at OpenQA.Selenium.Remote.RemoteWebDriver.Execute(String driverCommandToExecute, Dictionary`2 parameters)
            at OpenQA.Selenium.Remote.RemoteCookieJar.DeleteAllCookies()
            at FluentAutomation.CommandProvider.<>c__DisplayClass2.<.ctor>b__0()
            at System.Lazy`1.CreateValue()
            at System.Lazy`1.LazyInitValue()
            at System.Lazy`1.get_Value()
            at FluentAutomation.CommandProvider.get_webDriver()
            at FluentAutomation.CommandProvider.<>c__DisplayClass5.<Navigate>b__4()
            at lambda_method(Closure )
            at FluentAutomation.BaseCommandProvider.<>c__DisplayClass9.<WaitUntil>b__8()
            Result Message: 
            FluentAutomation.Exceptions.FluentException : An unexpected exception was thrown inside WaitUntil(Action). See InnerException for details.
            ----> System.InvalidOperationException : unknown error: Runtime.executionContextCreated has invalid 'context': {"auxData":{"frameId":"9064.1","isDefault":true},"id":1,"name":"","origin":"://"}
            (Session info: chrome=58.0.3029.110)
            (Driver info: chromedriver=2.9.248315,platform=Windows NT 6.1 SP1 x86_64)
            ```

        * 錯誤截圖

            ![6error](https://user-images.githubusercontent.com/3851540/26888552-f0ec8ecc-4bdd-11e7-9c83-f8b9dbc95bff.png)

## 如何使用

1. 引用 `FluentAutomation`

    ```cs
    using FluentAutomation;
    ```

2. 測試 class 繼承 `FluentTest`

    ```cs
    public class Ngithub : FluentTest
    ```

3. 建構式中指定使用的瀏覽器

    ```cs
    SeleniumWebDriver.Bootstrap(SeleniumWebDriver.Browser.Firefox);
    ```

    > 可指定多個瀏覽器

4. 使用語意化語法

    ```cs
    I.Open(baseURL+"login")
    .Enter("{帳號}")
    .In("#login_field")
    .Enter("{密碼}")
    .In("#password")
    .Click("input[name='commit']")
    .Wait(1).Assert.Url(baseURL);
    ```

5. 完整程式

    ```cs
    using NUnit.Framework;
    using FluentAutomation;
    namespace SeleniumTests
    {
        [TestFixture]
        public class Ngithub : FluentTest
        {
            private string baseURL;
            public Ngithub()
            {
                baseURL = "https://github.com/";
                SeleniumWebDriver.Bootstrap(
                SeleniumWebDriver.Browser.Firefox
                );
            }
            [Test]
            public void TheNgithubTest()
            {
                    I.Open(baseURL+"login")
                    .Enter("{帳號}}").In("#login_field")
                    .Enter("{密碼}").In("#password")
                    .Click("input[name='commit']")
                    .Wait(1)
                    .Assert.Url(baseURL);
            }
        }
    }
    ```

## 其他驗證方式

* 驗證在 selector 中是否有特定 css class

    ```cs
    I.Assert.Class("{class name}").Of("{selector}");
    ```

* 驗證特定的 selector 數量

    ```cs
    I.Assert.Count(1).Of("{selector}");
    ```

* 驗證頁面上是否有特定 selector

    ```cs
    I.Assert.Exists("{selector}");
    ```

* 驗證匿名函式結果是否為 false

    ```cs
    var element = I.Find("input");
    I.Assert.False(() => element().IsSelect);
    ```

* 驗證 selector 內容是否為指定文字

  * 可以用於有 `innerHTML` 或是可以提供 `文字` 值的 DOM element 上

    ```cs
    I.Assert.Text("FluentAutomation").In("{selector}");
    ```

  * 可以用來驗證回傳 `true`|`false` 的匿名函式

    ```cs
    I.Assert.Text((text) => text.Length > 50).In("{selector}");
    ```

* 驗證匿名函數應該出現 exception

    > 用來做反向驗證 --> 驗證某個 selector 不該存在

    ```cs
    //登入成功沒有這個 div 會 pass ，登入失敗出現這個 div 會 fail
    I.Assert.Throws(() => I.Assert.Exists("#js-flash-container .flash.flash-full.flash-error .container"));
    ```

* 驗證 `value` 是否合乎預期

  * 可以使用 selector 選定的 `<INPUT>`、`<TEXTAREA>`、`<SELECT>`

    ```cs
    I.Assert.Value(10).In("{selector}");
    ```

  * 可以用來驗證回傳 `true`|`false` 的匿名函式

    ```cs
    I.Assert.Value((value) => value.StartsWith("M")).In("{selector}");
    ```

* 驗證匿名函數結果應該為 `true`

    ```cs
    var element = I.Find("select");
    I.Assert.True(() => element().IsSelect);
    ```

* 驗證瀏覽器是否有 url

  * 可以驗證 `string` 或是 `Uri`

    ```cs
    I.Assert.Url(url);
    ```

  * 可以用來驗證回傳 `true`|`false` 的匿名函式

    ```cs
    I.Assert.Url((uri) => uri.Scheme == "https");
    ```

## 心得

Fluent Automation 從 2015 至今都沒有更新紀錄，而且連 Fluent Automation 的官網已經失效了：[http://fluent.stirno.com](http://fluent.stirno.com)，幸好 GitHub 上還有相關資訊，官方文件比較不好找 文件可以參考 [FluentAutomation Docs](https://github.com/stirno/FluentAutomation/tree/dev/Docs/v3)，雖然使用上便利性稍差但功能本身非常好用，前後比較一下使用 Fluent Automation 是不是覺得意圖一目瞭然，可讀性超高呀？！

## 參考資訊

1. [使用 Selenium IDE 與 C# 做 Web UI 測試](/selenium-ide-c-sharp-web-ui-test)
2. [FluentAutomation Docs](https://github.com/stirno/FluentAutomation/tree/dev/Docs/v3)
3. [GitHub - FluentAutomation](https://github.com/stirno/FluentAutomation)
