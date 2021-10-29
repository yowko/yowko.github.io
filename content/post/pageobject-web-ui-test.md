---
title: "使用 PageObject(Page Object Pattern) 建立物件導向的 Web UI 測試程式"
date: 2017-06-09T21:00:00+08:00
lastmod: 2021-10-29T21:00:17+08:00
draft: false
tags: ["套件","TDD","Unit Test"]
slug: "pageobject-web-ui-test"
aliases:
    - /2017/06/pageobject-web-ui-test.html
---
## 使用 PageObject(Page Object Pattern) 建立物件導向的 Web UI 測試程式

從 [使用 Selenium IDE 與 C# 做 Web UI 測試](//blog.yowko.com/2017/06/selenium-ide-c-sharp-web-ui-test.html) 介紹如何使用 Selenium IDE 錄製網頁操作再轉換為 c# 測試案例，讓測試程式也能觸發 Web UI 的測試驗證。

接著為了讓 Selenium 產生出來的測試案例可以有更好的可讀性跟說明需求的能力，[使用 Fluent Automation 與 Selenium 打造語意化 Web UI 測試程式](//blog.yowko.com/2017/06/fluent-automation-selenium-web-ui-test.html) 中使用了 Fluent Automation 來取代 Selenium api。

最後是 91 哥在 TDD 課程中介紹了 Martin Fowler 的概念：[PageObject](https://martinfowler.com/bliki/PageObject.html)，接下來我們就來看看 PageObject 的目的與使用方式

## PageObject 是什麼？

[PageObject GitHub](https://github.com/SeleniumHQ/selenium/wiki/PageObjects) 截錄說明如下

>Within your web app's UI there are areas that your tests interact with. A Page Object simply models these as objects within the test code. This reduces the amount of duplicated code and means that if the UI changes, the fix need only be applied in one place.

![1pageObject](https://user-images.githubusercontent.com/3851540/26961114-ee25a0d4-4d0e-11e7-9771-4e61fac3e3ed.png)

個人解讀如下： 在測試程式中將每個會產生互動的網頁當做一個 object(class) 來處理，這樣就可以降低重複的程式碼，在 UI 變動時也可以讓修改限縮在少數的位置，像是一個頁面上有多個測試時，在 UI 異動時就可以讓測試程式只需修改 page object 而不需每個測試都修改

## 為什麼要用 PageObject ？

1. 讓測試案例與實際 UI 互動抽象化

    * test case 專注於描述及說明需求
    * 實際 UI 可能會經常異動，不該讓 test case 直接相依

2. DRY

    > 讓測試程式更符合物件導向程式設計，不是一直重複寫相同的程式碼

## 如何使用 PageObject？

> 接著 demo 使用到的 PageObject 是 FluentAutomation 的附屬功能，請記得安裝 FluentAutomation

* Demo 案例如下

    > 細節請參考 [使用 Fluent Automation 與 Selenium 打造語意化 Web UI 測試程式](//blog.yowko.com/2017/06/fluent-automation-selenium-web-ui-test.html)

    ```cs
    using Microsoft.VisualStudio.TestTools.UnitTesting;
    using FluentAutomation;
    namespace UnitTestGitHubProject
    {
        [TestClass]
        public class UnitTestGitHubLogin : FluentTest
        {
            private string baseURL = "https://github.com/";
            public UnitTestGitHubLogin()
            {
                SeleniumWebDriver.Bootstrap(
                    SeleniumWebDriver.Browser.Chrome
                );
            }
            [TestMethod]
            public void LoginSuccess()
            {
                I.Open(baseURL + "login")
                    .Enter("{帳號}").In("#login_field")
                    .Enter("{密碼}").In("#password")
                    .Click("input[name='commit']")
                    .Wait(1)
                    .Assert.Url(baseURL);
            }
        }
    }
    ```

> 以下說明會在不同程式間頻繁切換，為了避免混洧會加註程式名稱

1. `測試程式` 中宣告啟動頁面的 PageObject 並傳入 `this`

    ```cs
    GitHubLoginPageObject pageobject = new GitHubLoginPageObject(this);
    ```

2. `測試程式` 中使用 IDE 功能產生 `GitHubLoginPageObject`

    > 意圖導向程式設計的做法，有助於命名及 api 簡化

    ![2genpageobject](https://user-images.githubusercontent.com/3851540/26961115-ee510a08-4d0e-11e7-935b-7ec2519b6868.png)

3. `GitHubLoginPageObject` 中引用 `FluentAutomation`

    ```cs
    using FluentAutomation;
    ```

4. `GitHubLoginPageObject` 中讓 `GitHubLoginPageObject` 繼承 `PageObject<GitHubLoginObject>`

    ```cs
    internal class GitHubLoginPageObject:PageObject<GitHubLoginPageObject>
    ```

5. `GitHubLoginPageObject` 中建立 `GitHubLoginPageObject` 的建構式並繼承 base

    ```cs
    public GitHubLoginPageObject(FluentTest fluenttest):base(fluenttest) 
    {
    }
    ```

6. `GitHubLoginPageObject` 中建構式中指定該頁面所屬 url

    > 這也可以由測試程式傳入，個人覺得訂在 PageObject 比較符合物件導向：每個頁面都有一個 PageObject，一般情境下各個網頁對應的 PageObject 無法共用，因為網頁元素應該不同

    ```cs
    public GitHubLoginPageObject(FluentTest fluenttest):base(fluenttest) 
    {
        this.Url = "https://github.com/";
    }
    ```

7. `測試程式` 中加上 `pagobject.Go()`

    ```cs
    pageobject.Go();
    ```

8. `測試程式` 中加上 pageobject 的執行動作並透過 IDE 產生至 GitHubLoginPageObject 中

    ```cs
    pageobject.Login(username,password);
    ```

9. `GitHubLoginPageObject` 中將原本測試程式執行網頁的操作移至 GitHubLoginPageObject 的動作內

    ```cs
    internal void Login(string username, string password)
    {
        I.Open(this.Url + "login")
            .Enter(username).In("#login_field")
            .Enter(password).In("#password")
            .Click("input[name='commit']")
            .Wait(1);
    }
    ```

10. `測試程式` 中宣告結果頁面的 PageObject 並傳入 `this` 後透過 Visual Studio 建立 `GitHubLoginResult`

    ```cs
    GitHubLoginResult resultpageobject = new GitHubLoginResult(this);
    ```

11. `GitHubLoginResult` 中引用 `FluentAutomation`

    ```cs
    using FluentAutomation;
    ```

12. `GitHubLoginResult` 中讓 `GitHubLoginResult` 繼承 `PageObject<GitHubLoginResult>`

    ```cs
    internal class GitHubLoginResult:PageObject<GitHubLoginResult>
    ```

13. `GitHubLoginResult` 中建立 `GitHubLoginResult` 的建構式並繼承 base

    ```cs
    public GitHubLoginResult(FluentTest fluenttest):base(fluenttest) 
    {
    }
    ```

14. `測試程式` 中加入 result pageobject 驗證方法並使用 Visual Studio 產生至 GitHubLoginResult

    ```cs
    resultpageobject.VerifyRedirectLink("https://github.com/");
    ```

15. `GitHubLoginResult` 中調整驗證方法

    ```cs
    internal void VerifyRedirectLink(string url)
    {
        I.Assert.Url(url);
    }
    ```

16. 接著就可以直接執行測試，並且擁有物件導向測試程式了

## 完整測試程式碼

* 測試程式

    ```cs
    using Microsoft.VisualStudio.TestTools.UnitTesting;
    using FluentAutomation;
    namespace UnitTestGitHubProject
    {
        [TestClass]
        public class UnitTestGitHubLogin : FluentTest
        {
            //private string baseURL = "https://github.com/";
            public UnitTestGitHubLogin()
            {
                SeleniumWebDriver.Bootstrap(
                    SeleniumWebDriver.Browser.Chrome
                    );
            }
            [TestMethod]
            public void LoginSuccess()
            {
                GitHubLoginPageObject pageobject = new GitHubLoginPageObject(this);
                pageobject.Go();
                string username = "{帳號}";
                string password = "{密碼}";
                pageobject.Login(username,password);
                GitHubLoginResult resultpageobject = new GitHubLoginResult(this);
                resultpageobject.VerifyRedirectLink("https://github.com/");
            }
        }
    }
    ```

* GitHubLoginPageObject

    ```cs
    using System;
    using FluentAutomation;
    namespace UnitTestGitHubProject
    {
        internal class GitHubLoginPageObject : PageObject<GitHubLoginPageObject>
        {
            //private string baseURL = "https://github.com/";
            public GitHubLoginPageObject(FluentTest fluenttest):base(fluenttest) 
            {
                this.Url = "https://github.com/";
            }
            internal void Login(string username, string password)
            {
                I.Open(this.Url + "login")
                    .Enter(username).In("#login_field")
                    .Enter(password).In("#password")
                    .Click("input[name='commit']")
                    .Wait(1);
            }
        }
    }
    ```

* GitHubLoginResult

    ```cs
    using System;
    using FluentAutomation;
    namespace UnitTestGitHubProject
    {
        internal class GitHubLoginResult : PageObject<GitHubLoginResult>
        {
            public GitHubLoginResult(FluentTest fluenttest): base(fluenttest)
            {
            }
            internal void VerifyRedirectLink(string url)
            {
                I.Assert.Url(url);
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

## 使用 PageObject 時 assert 該寫在哪裡？

目前有兩派做法：可視實際情境來調整

1. PageObject 也應該包含驗證 ()

    > 包含驗證可以讓測試程式有效減少重複程式碼

2. PageObject 僅提供資料，不該驗證

    > 驗證邏輯應該是跟著需求，而不是 PageObject，應該避免 PageObject 只是 model 的角色不該與邏輯交錯

## 心得

經過使用 PageObject 來修改測試程式後，使得測試程式更符合物件導向的設計，不僅讓測試案例專注於需要描述、PageObject 專責處理 Web UI 的處理，也讓測試案例與 Web UI 的耦合降低，對於測試程式的可讀性及可維護性都有提升，想必對日後測試程式的維護也有幫助

## 參考資訊

1. [PageObject](https://martinfowler.com/bliki/PageObject.html)
2. [PageObject GitHub](https://github.com/SeleniumHQ/selenium/wiki/PageObjects)
