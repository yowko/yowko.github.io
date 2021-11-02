---
title: "NUnit 幾個參數化測試的方式"
date: 2017-04-17T23:30:00+08:00
lastmod: 2021-11-01T19:13:37+08:00
draft: false
tags: ["NUnit","Unit Test"]
slug: "nunit-parameterized-test"
aliases:
    - /2017/04/nunit-parameterized-test.html
---
## NUnit 幾個參數化測試的方式

QA 同事最近正在進行一個龐大計劃：打算將上百個單元測試，整併為一個。剛聽到這個需求，我當然是持反對意見，將那麼多測試整併成一個，一定會需要為多個測試做取最大相容性的處理，難免在測試中就會必須加上條件判斷，雖然測試看起來變少，但整併後的測試複雜度一定會變高，結果同事說那上百個測試只是 data 跟 result 不同，只是一開始寫的人沒有 OOP 的概念，所以全部都拆開，導致現在難以維護，聽完說明我就立馬改為支持整併了XD

改寫過程使用了 TestCase 、 TestCaseSource、Values、ValueSource 多種給值的做法，紀錄一下避免忘記

## 基本設定

1. NUnit framework 3.6.1
2. ASP.NET WebAPI 5.2.3 預設專案範本
    * 稍微修改一下回傳內容

        ```cs
        public string Get(string name)
        {
            return $"{name}:{name.Length}";
        }
        ```

## 基本 TestCase 用法

* 測試程式

    ```cs
    [TestFixture()]
    public class ValuesControllerTests
    {
        [Test]
        [TestCase("Yowko", 5)]
        [TestCase("Admin", 5)]
        [TestCase("test", 4)]
        public void GetById(string name, int length)
        {
            // Arrange
            ValuesController controller = new ValuesController();
            // Act
            string result = controller.Get(name);
            // Assert
            Assert.AreEqual($"{name}:{length}", result);
        }
    }
    ```

* 測試結果

    > Test Explorer 會將測試方法代入測試參數全部列出來，一目了然

    ![1testcaseresult](https://cloud.githubusercontent.com/assets/3851540/25095623/0ab3431e-23cf-11e7-9123-ea5a0e864575.png)

## 使用 TestFixture 改寫

* 測試程式

    ```CS
    [TestFixture("YowkoTsai",9)]
    [TestFixture("SuperAdmin",10)]
    [TestFixture("TestUser",8)]
    public class ValuesControllerTests
    {
        private readonly string name;
        private readonly int length;
        public ValuesControllerTests(string name, int length)
        {
            this.name = name;
            this.length = length;
        }
        public void GetById()
        {
            // Arrange
            ValuesController controller = new ValuesController();
            // Act
            string result = controller.Get(name);
            // Assert
            Assert.AreEqual($"{name}:{length}", result);
        }
    }
    ```

* 改寫結果
  * Test Explorer 會列出測試方式及執行次數，但不會代入參數值
  * 如果有多個測試方法可以全部套用

    ![2texttextureresult](https://cloud.githubusercontent.com/assets/3851540/25095624/0adf6318-23cf-11e7-9ed1-03f7f75b6f43.png)

## 使用 TestCaseSource 改寫

* 測試程式

    ```cs
    [TestFixture()]
    public class ValuesControllerTests
    {
        private static IEnumerable<TestCaseData> DataSources()
        {
            yield return new TestCaseData("Yowko", 5);
            yield return new TestCaseData("Admin", 5);
            yield return new TestCaseData("test", 4);
        }
                
        [Test, TestCaseSource("DataSources")]
        public void GetById(string name, int length)
        {
            // Arrange
            ValuesController controller = new ValuesController();
            // Act
            string result = controller.Get(name);
            // Assert
            Assert.AreEqual($"{name}:{length}", result);
        }
    }
    ```

* 改寫結果

    ![3testcasesource](https://cloud.githubusercontent.com/assets/3851540/25095627/0af0b28a-23cf-11e7-8b97-d7a361a74997.png)

## 使用 Values 改寫

* 測試程式

    ```cs
    [TestFixture()]
    public class ValuesControllerTests
    {
        [Test]
        public void GetById(
            [Values("Yowko", "LOL", "test")] string name,
            [Values(5, 3, 4)]int length)
        {
            // Arrange
            ValuesController controller = new ValuesController();
            // Act
            string result = controller.Get(name);
            // Assert
            Assert.AreEqual($"{name}:{length}", result);
        }
    }
    ```

* 測試結果

    ![4valueresult](https://cloud.githubusercontent.com/assets/3851540/25095626/0aeef10c-23cf-11e7-92cf-36dd75c65085.png)

* 測試案例會列出所有組合，如果需要一對一測試組合就要加上`Sequential`

    ```cs
    [TestFixture()]
    public class ValuesControllerTests
    {
        [Test, Sequential]
        public void GetById(
            [Values("Yowko", "LOL", "test")] string name,
            [Values(5, 3, 4)]int length)
        {
            // Arrange
            ValuesController controller = new ValuesController();
            // Act
            string result = controller.Get(name);
            // Assert
            Assert.AreEqual($"{name}:{length}", result);
        }
    }
    ```

* 測試結果

    ![5valueresult](https://cloud.githubusercontent.com/assets/3851540/25095629/0af6c0da-23cf-11e7-829a-0e5c38709762.png)

## 使用 ValueSource 改寫

* 測試程式

    ```cs
    [TestFixture()]
    public class ValuesControllerTests
    {
        private static string[] TestNames()
        {
            return new string[] { "Yowko", "Lol", "test" };
        }
        private static int[] TestLengths()
        {
            return new int[] { 5, 3, 4 };
        }
        [Test]
        public void GetById([ValueSource("TestNames")] string name,
            [ValueSource("TestLengths")] int length)
        {
            // Arrange
            ValuesController controller = new ValuesController();
            // Act
            string result = controller.Get(name);
            // Assert
            Assert.AreEqual($"{name}:{length}", result);
        }
    }
    ```

* 測試結果

    ![6valuesourceresult](https://cloud.githubusercontent.com/assets/3851540/25095625/0aeec182-23cf-11e7-992d-3edda63a11a5.png)

* 測試案例會列出所有組合，如果需要一對一測試組合就要加上`Sequential`

    ```cs
    [TestFixture()]
    public class ValuesControllerTests
    {
        private static string[] TestNames()
        {
            return new string[] { "Yowko", "Lol", "test" };
        }
        private static int[] TestLengths()
        {
            return new int[] { 5, 3, 4 };
        }
        [Test,Sequential]
        public void GetById([ValueSource("TestNames")] string name,
            [ValueSource("TestLengths")] int length)
        {
            // Arrange
            ValuesController controller = new ValuesController();
            // Act
            string result = controller.Get(name);
            // Assert
            Assert.AreEqual($"{name}:{length}", result);
        }
    }
    ```

* 測試結果

    ![7valuesourceresult](https://cloud.githubusercontent.com/assets/3851540/25095628/0af6b54a-23cf-11e7-9e86-17eabce13fc6.png)

## 參考資訊

1. [Parameterized Tests Made Simple](http://www.smaclellan.com/posts/parameterized-tests-made-simple/)
2. [NUnit's Test Case Source](http://www.c-sharpcorner.com/UploadFile/0f29c1/nunit-test-case-source/)
3. [Nunit test setup method with argument](http://stackoverflow.com/questions/16276278/nunit-test-setup-method-with-argument)
