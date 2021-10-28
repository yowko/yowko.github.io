---
title: "餵資料集給 SpecFlow 來執行測試及驗證"
date: 2017-06-21T23:16:00+08:00
lastmod: 2021-10-26T23:16:03+08:00
draft: false
tags: ["套件","Unit Test"]
slug: "specflow-outline"
aliases:
    - /2017/06/specflowoutline.html
    - /2017/06/specflow-outline
---
## 餵資料集給 SpecFlow 來執行測試及驗證

之前文章 [使用 SpecFlow 建立人語化的單元測試](/specflow) 已經大致了解如何使用近似人類語言來描述需求跟寫測試案例，透過這樣的方式不僅讓需求更好被理解，也讓測試案例很清楚地被描述，但如果需要使用多筆資料來反覆進行測試驗證，一直 copy and paste 想必身為優秀工程師的大家也無法接受的

今天就來介紹可以達成 NUnit 中 TestCase 、TestCaseSource 功能的 SpecFlow Outline 特性，

## 基礎程式

> 以下內容與 [使用 SpecFlow 建立人語化的單元測試](/specflow) 相同，如果有什麼不清楚來由的，可以參考 [使用 SpecFlow 建立人語化的單元測試](/specflow)

1. Feature 檔內容

    ```cs
    Feature: Calculator
    In order to 減少莫名其妙的錯誤
    As 一個數學白痴
    我想要得到兩數相加的結果
    
    @MSCalculator
    Scenario Outline: 兩數相加
        Given 第一個數字輸入 50
        And 第二個數字輸入 70
        When 按下 add
        Then 結果應該為 120
    ```

2. Step Definition 檔內容

    ```cs
    [Binding]
    public class CalculatorSteps
    {
        private Calculator target;
        [BeforeScenario]
        private void Init()
        {
            target = new Calculator();
        }
        [Given(@"第一個數字輸入 (.*)")]
        public void Given第一個數字輸入(int first)
        {
            ScenarioContext.Current.Set<int>(first, "first");
        }
        [Given(@"第二個數字輸入 (.*)")]
        public void Given第二個數字輸入(int second)
        {
            ScenarioContext.Current.Set<int>(second, "second");
        }
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
        [Then(@"結果應該為 (.*)")]
        public void Then結果應該為(int expected)
        {
            //取得上一步 result 計算的結果
            var actual = ScenarioContext.Current.Get<int>();
            //驗證計算結果是否合乎預期
            Assert.AreEqual(expected, actual);
        }
    }
    ```

3. 測試目標程式(SUT)

    ```cs
    internal class Calculator
    {
        internal int Add(int first, int second)
        {
            return first + second;
        }
    }
    ```

## 想要驗證其他值

1. 驗證 (-50) + 70 = 20

    ```cs
    @MSCalculator
    Scenario: 兩數相加-負數
    Given 第一個數字輸入 -50
    And 第二個數字輸入 70
    When 按下 add
    Then 結果應該為 20
    ```

2. 如果需要驗證其他 10 組、20 組、50 組資料呢？！

    > 要一直 copy and paste 嗎？ 過程驗證的可能是 copy and paste 的正確性而不是程式的正確性了

## 如何解決？

SpecFlow 也想到大家可能會有這樣的需求，因此透過 SpecFlow 的 Outline - Examples 特性就可以簡單的達成目地，修改的內容也只限於 Feature 檔

1. 在 `Scenario` 後面加上 `Outline`

    > `Scenario Outline: 兩數相加`

2. 將 Scenario 中需要傳入的參數位置以 `<變數名稱>` 取代

    ```feature
    Given 第一個數字輸入 <first>
    And 第二個數字輸入 <second>
    When 按下 add
    Then 結果應該為 <result>
    ```

3. 提供 `Examples` 並以變數名稱為 table header 準備傳入值為 table body，各數值間以 `|` 當做分隔符號

    ```cs
    Examples:
    | first | second | result |
    | 50    | 70     | 120    |
    | -50   | 70     | 20     |
    ```

4. 完整 Feature 檔

    ```cs
    Feature: Calculator
        In order to 減少莫名其妙的錯誤
        As 一個數學白痴
        我想要得到兩數相加的結果

    @MSCalculator
    Scenario Outline: 兩數相加
        Given 第一個數字輸入 <first>
        And 第二個數字輸入 <second>
        When 按下 add
        Then 結果應該為 <result>
    
    Examples:
        | first | second | result |
        | 50    | 70     | 120    |
        | -50   | 70     | 20     |
    ```

* 這邊有個關於編輯 Examples table 的小技巧，如果排版跑掉了，請直接刪除最後一個 `|` 符號，再重新補上 `|` 符號 就會自動重新排版了，不過對於中文字無效(會重新排版，只是結果依然不正確)

## 心得

簡單的修改可以讓 SpecFlow 應付相同測試程式而測試參數內容不同的情境，也不用看到一大堆重複的 Scenario 更可免了複製一堆 Scenario 時可能因為眼花、恍神造成參數填錯的狀況。

跟 NUnit 的 TestCase 、TestCaseSource 功能很雷同，如果想要進一步了解 NUnit 的 TestCase 、TestCaseSource 功能，可以參考 [NUnit 幾個參數化測試的方式](/nunit-parameterized-test)

## 參考資訊

1. [使用 SpecFlow 建立人語化的單元測試](/specflow/)
2. [NUnit 幾個參數化測試的方式](/nunit-parameterized-test/)
3. [Scenario Outlines](https://github.com/cucumber/cucumber/wiki/Scenario-outlines)
