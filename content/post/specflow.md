---
title: "使用 SpecFlow 建立人語化的單元測試"
date: 2017-06-13T01:00:00+08:00
lastmod: 2021-10-26T11:19:56+08:00
draft: false
tags: ["套件","NUnit","Unit Test"]
slug: "specflow"
aliases:
    - /2017/06/specflow.html
---
## 使用 SpecFlow 建立人語化的單元測試

如何寫出良好可讀性的程式是個持續性的挑戰，也可能是永無止盡的挑戰，而測試程式雖然目的是測試，但終究是程式，也需要具備良好的可讀性，加上測試程式擔負了需求描述及需求說明的功能，可讀性的要求比一般還要高，今天就來紀錄一下該如何使用 SpecFlow 建立人語化的單元測試，挑戰非開發人員也可以瞭解測試程式所代表的意思

## 什麼是 SpecFlow ？

* 官網 - [SpecFlow](http://specflow.org/) 說明如下：

    >Use SpecFlow to define, manage and automatically execute human-readable acceptance tests in .NET projects. Writing easily understandable tests is a cornerstone of the BDD paradigm and also helps build up a living documentation of your system.
    >SpecFlow is open source and provided under a BSD license. As part of the Cucumber family, SpecFlow uses the official Gherkin parser and supports the .NET framework, Xamarin and Mono.
    >SpecFlow integrates with Visual Studio, but can be also used from the command line (e.g. on a build server). SpecFlow supports popular testing frameworks: MSTest, NUnit (2 and 3), xUnit 2 and MbUnit.
    >SpecFlow+ adds additional functionality to SpecFlow, such as Visual Studio Test Explorer integration, a dedicated test runner with advanced test execution options, execution reports (HTML, XML, JSON) and much more.

* 個人理解重點是：

    > SpecFlow 可以用來開發非發人員也可以理解的驗證測試，這是 BDD 成功的基礎，還可以用來建立系統的即時文件

## 如何使用

參考官方文件 - [SpecFlow+ Getting Started](http://specflow.org/getting-started/) 加上個人安裝紀錄而來的，官網上範例是 VS 2013，我則會用 VS 2017

1. 安裝 VS 套件

    * Visual Studio 主選單的 Tools --> Extensions and Updates

        ![1specflow](https://user-images.githubusercontent.com/3851540/27064125-691fbcfe-5028-11e7-8d3c-2f1454a75682.png)

    * `Online` tab --> 搜尋 `specflow` --> Download

        ![2downspecflow](https://user-images.githubusercontent.com/3851540/27064130-6943a1be-5028-11e7-9f60-db92176a377a.png)

    * 下載完成後請關閉 Visual Studio 2017 進行安裝

        ![3needrestart](https://user-images.githubusercontent.com/3851540/27064131-6943cfae-5028-11e7-8522-e4533dc9a63d.png)

        ![4install1](https://user-images.githubusercontent.com/3851540/27064134-6947940e-5028-11e7-9242-f074dc05d923.png)

        ![5install2](https://user-images.githubusercontent.com/3851540/27064132-6945497e-5028-11e7-96ee-3245104bb46e.png)

        ![6install3](https://user-images.githubusercontent.com/3851540/27064135-69497490-5028-11e7-8dbd-e405cac4b175.png)

        ![7install4](https://user-images.githubusercontent.com/3851540/27064133-6946a5da-5028-11e7-99c8-df66882834ef.png)

    * 完裝完成後 Visual Studio 會加入 SpecFlow 的 template

        ![8specflowtemplate](https://user-images.githubusercontent.com/3851540/27064138-6969b7a0-5028-11e7-8a44-642d90423c98.png)

2. 專案加入 SpecFlow

    * 專案上按右鍵 --> Manage NuGet Packages...

        ![12nuget](https://user-images.githubusercontent.com/3851540/27064140-696c0b0e-5028-11e7-9243-89c1bc3c44ff.png)

    * Browse --> 搜尋 `specflow` --> 依 test framework 來安裝 (MSTest 只需安裝 SpecFlow 即可)

        ![13specunit](https://user-images.githubusercontent.com/3851540/27064141-69729dfc-5028-11e7-8f97-962d29a486e9.png)

    * 有相依套件，直接安裝即可

        ![14dependency](https://user-images.githubusercontent.com/3851540/27064142-69897c2a-5028-11e7-8922-8eb95f6f40cf.png)

3. 調整使用的 Test Framework

    > SpecFlow 支援主流 Test Framework：MSTest, NUnit (2 and 3), xUnit 2 and MbUnit.預設使用 NUnit，如果要用其他 Test Framework ，就需要調整測試專案的 app.config 設定，以下的 demo 會以 NUint 為例

    * MSTest

        ```xml
        <specFlow>
            <unitTestProvider name="MsTest" />
        </specFlow>
        ```

    * xUnit.net 2.0

        ```xml
        <specFlow>
            <unitTestProvider name="xUnit" />
        </specFlow>
        ```

    * 其他 test framework 設定值可以參考 [Unit test providers](https://github.com/techtalk/SpecFlow/wiki/Unit-test-providers)
    * app.config 完整設定範例，官方文件請參考 [Configuration](https://github.com/techtalk/SpecFlow/wiki/Configuration)

        ```xml
        <?xml version="1.0" encoding="utf-8" ?>
        <configuration>
            <configSections>
                <section name="specFlow"
                type="TechTalk.SpecFlow.Configuration.ConfigurationSectionHandler, TechTalk.SpecFlow"/>
            </configSections>
            <specFlow>
                <unitTestProvider name="MsTest" />
            </specFlow>
        </configuration>
        ```

4. 加入 Feature 檔

    > 在完成 SpecFlow 安裝後，Visual Studio 就擁有各式 SpecFlow 的 template，我們可以透過這些 template 來產生 feature 檔

    * 在專案按下右鍵 --> Add --> New Item...

        ![9newitem](https://user-images.githubusercontent.com/3851540/27064136-6966dd78-5028-11e7-9b98-3f928bc3e4bd.png)

    * 選擇 SpecFlow Feature File template--> 調整 feature 檔名 --> Add

        ![10addfeature](https://user-images.githubusercontent.com/3851540/27064139-696b0894-5028-11e7-8411-5c7826a1f6e2.png)

5. 依需求修改 Feature 檔

    ```feature
    Feature: Calculator
        In order to 減少莫名其妙的錯誤
        As 一個數學白痴
        我想要得到兩數相加的結果
    
    @myCalculator
    Scenario: 兩數相加
        Given 第一個數字輸入 50
        And 第二個數字輸入 70
        When 按下 add
        Then 結果應該為 120
    ```

    * `Feature` 是情境描述，就是 user story
    * `@myCalculator` 是用來為 scenario 加上識別標記
    * `Scenario` 是實際用來執行的 test case，會以 regular expression 來 parse 參數

6. 產生 Step Definitions

    > 這就是測試程式的主體，
    >
    > * `Given` 是單元測試 3A 中的 `Arrange`
    > * `When` 是單元測試 3A 中的 `Act`
    > * `Then` 是單元測試 3A 中的 `Assert`

    * 產生前 Scenario 中的參數說明都會是 `紫` 色的

        ![11purple](https://user-images.githubusercontent.com/3851540/27064137-6969b0ca-5028-11e7-8c47-1d82d21837fb.png)

    * 在 `Scenario` 按右鍵 --> Generate Step Definitions

        >如果這邊沒有 `Generate Step Definitions` 選項，記得專案要安裝 SpecFlow

        ![15gensetp](https://user-images.githubusercontent.com/3851540/27064143-698c8320-5028-11e7-9d4e-0d8d75141f39.png)

    * 按下 `Generate` 產生檔案

        * 複製產生結果

            > 如果已產生過檔案，再產生一次檔案會直接覆蓋，複製的功能在此時就很好用了

        * 將結果產生至檔案

            > 預設路徑不是當前路徑，要注意別放錯位置

        ![16generate](https://user-images.githubusercontent.com/3851540/27064144-698f546a-5028-11e7-852c-2766565dbb7f.png)

    * 產生 Step Definitions 後 Scenario 中的參數說明都會變成 `白` 色的，參數部部則是 `灰` 色

        ![17white](https://user-images.githubusercontent.com/3851540/27064128-69226972-5028-11e7-8977-eb4400c10e58.png)

7. 實作測試程式 (Step Definitions)

    * 加上測試目標

        ```cs
        private Calculator target;
        ```

    * 使用 IDE 產生 Calculator class (意圖導向開發)
    * 建立初始化 target 並加上 `[BeforeScenario]`

        ```cs
        [BeforeScenario]
        private void Init()
        {
            target = new Calculator();
        }
        ```

    * 修改 `Given第一個數字輸入` 方法參數名稱使其有意義

        ```cs
        public void Given第一個數字輸入(int first)
        ```

    * 修改 `Given第一個數字輸入` 方法實作

        * 因為要讓值跨不同 method 使用，所以需要註冊 key 的及內容存進 ScenarioContext

        ```cs
        ScenarioContext.Current.Set<int>(first, "first");
        ```

    * 比照上述二步驟調整 `Given第二個數字輸入` 方法

        ```cs
        [Given(@"第二個數字輸入 (.*)")]
        public void Given第二個數字輸入(int second)
        {
            ScenarioContext.Current.Set<int>(second, "second"); 
        }
        ```

    * 修改 `When按下Add` 方法實作內容

        ```cs
        [When(@"按下 add")]
        public void When按下Add()
        {
            //取得 first
            var first = ScenarioContext.Current.Get<int>("first");
            //取得 second
            var second = ScenarioContext.Current.Get<int>("second");
            // 呼叫 target 的 add 方法，這個 Add 可以用 ide 自動產生至 Calculator class 中
            int actual = target.Add(first, second);
            //將結果存入 ScenarioContext，不給 key 也可以當做 key
            ScenarioContext.Current.Set<int>(actual);
        }
        ```

    * 實作 Calculator 的 Add 方法

        ```cs
        internal int Add(int first, int second)
        {
            return first + second;
        }
        ```

    * 修改 `Then結果應該為` 方法參數名稱及實作

        ```cs
        [Then(@"結果應該為 (.*)")]
        public void Then結果應該為(int expected)
        {
            //取得上一步 result 計算的結果
            var actual = ScenarioContext.Current.Get<int>();
            //驗證計算結果是否合乎預期
            Assert.AreEqual(expected, actual);
        }
        ```

8. 可以執行或是 debug 也可以下中斷點

    ![18scenario](https://user-images.githubusercontent.com/3851540/27064124-692004b6-5028-11e7-92ae-221198629b7c.png)

9. 可以自由在 feature 的 scenario 與 step definition 切換

    * 從 feature 的 scenario 跳至 step definition

        ![19scenariotostep](https://user-images.githubusercontent.com/3851540/27064129-6922c3fe-5028-11e7-91bc-43a86abe48e5.png)

    * 從 step definition 跳至 scenario

        ![20steptoscenario](https://user-images.githubusercontent.com/3851540/27064126-69209746-5028-11e7-848f-3f68f7ded0ae.png)

        * 如果 step 被多個 scenario 使用會出現選擇視窗

            ![21option](https://user-images.githubusercontent.com/3851540/27064127-6920eb06-5028-11e7-8782-68030aaa5d90.png)

## 心得

透過 SpecFlow 就可以讓測試程式擁有說人話的能力，也更能充份說明需求，讓程式與需求的隔閡縮小，加上同樣模式的驗證都可以利用加入 scenario 後直接套用原本的測試程式就好，不必多寫測試程式，讓撰寫測試程式的 effort 縮小很多，是不是很棒呀？！

## 參考資訊

1. [SpecFlow](http://specflow.org/)
2. [Unit test providers](https://github.com/techtalk/SpecFlow/wiki/Unit-test-providers)
3. [Configuration](https://github.com/techtalk/SpecFlow/wiki/Configuration)
