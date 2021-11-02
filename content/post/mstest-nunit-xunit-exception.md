---
title: "使用 MSTest、Nunit 3、xUnit.net 2.0、NSubstitute、FluentAssertions 驗證例外(exception)"
date: 2017-05-27T01:00:00+08:00
lastmod: 2021-11-02T00:35:20+08:00
draft: false
tags: ["NUnit","Unit Test","MSTest","xUnit"]
slug: "mstest-nunit-xunit-exception"
aliases:
    - /2017/05/mstestnunit-3xunitnet.html
---
## 使用 MSTest、Nunit 3、xUnit.net 2.0、NSubstitute、FluentAssertions 驗證例外(exception)

程式總是會有出乎預期的操作行為，或是使用者有心挑戰系統強健性時，難免會讓程式出現例外(exeption)，而讓 end user 看到錯誤畫面對商譽是重大傷害，所以例外處理也是程式開發中重要的一環。為了掌握程式正常運作的基準，我們可以透過測試程式來模擬錯誤，確保讓原本應該錯誤的錯誤操作不會因為程式的修修改改而造成應該出錯的程式不再出錯，重點就是 `該錯的一定要錯，不該錯一定不能錯`

測試過程中試了幾個 test framewrok，也試用常見好用的套件，隨手紀錄

## 基本設定

1. 使用 ASP.NET Web Api 預設專案範本

    ```cs
    public class ValuesController : ApiController
    {
        // GET api/values
        public IEnumerable<string> Get()
        {
            return new string[] { "value1", "value2" };
        }
        // GET api/values/5
        public string Get(int id)
        {
            return "value";
        }
        // POST api/values
        public void Post([FromBody]string value)
        {
        }
        // PUT api/values/5
        public void Put(int id, [FromBody]string value)
        {
        }
        // DELETE api/values/5
        public void Delete(int id)
        {
        }
    }
    ```

2. 有回傳值方法：修改其中的 `Get(int id)` 使其丟出 exception

    ```cs
    public string Get(int id)
    {
        if (id <= 0)
        {
            throw new ArgumentException();
        }
        return "value";
    }
    ```

3. 無回傳值方法：修改其中的 `Delete(int id)` 使其丟出 exception

    ```cs
    public void Delete(int id)
    {
        if (id <= 0)
        {
            throw new ArgumentException();
        }
    }
    ```

## MSTest 驗證 exception

* 方法一：在測試方法加上 `[ExpectedException(typeof({exception 型別}))]`

    ```cs
    [TestMethod]
    [ExpectedException(typeof(ArgumentException))]
    public void 測試Get_傳入0_得到ArgumentException()
    {
        //arrange
        var target = new ValuesController();
        //act
        var actual = target.Get(0);
        //assert-expected exception
    }
    ```

  * 缺點是如果在執行目標方法前就遇到相同的錯誤時。測試會 pass

* 方法二：安裝 `MSTestExtensions`

    1. `Install-Package MSTestExtensions`
    2. `using MSTestExtensions`
    3. 測試 class 繼承 `BaseTest`

        ```cs
        [TestClass()]
        public class ValuesControllerMSTests:BaseTest
        ```

    4. 使用 `Assert.Throws<T>()` 驗證

        ```cs
        [TestMethod]
        public void 測試Get_傳入0_得到ArgumentException()
        {
            //arrange
            var target = new ValuesController();
            //act
            //assert-expected exception
            Assert.Throws<ArgumentException>(() => target.Get(0));
        }
        ```

## NUnit 3 驗證 exception

1. 方法一：使用 delegate 來呼叫目標方法並取得 `ActualValueDelegate<object>`

    * 需 `using NUnit.Framework.Constraints`
    * 範例程式碼

        ```cs
        [Test]
        public void 測試Get_傳入0_得到ArgumentException()
        {
            //arrange
            var target = new ValuesController();
            //act
            ActualValueDelegate<object> actual = () => target.Get(0);
            //assert-expected exception
            Assert.That(actual, Throws.TypeOf<ArgumentException>());
        }
        ```

2. 方法二：直接在驗證時使用 delegate 來呼叫目標方法

    * 範例程式碼

        ```cs
        [Test]
        public void 測試Get_傳入0_得到ArgumentException()
        {
            //arrange
            var target = new ValuesController();
            //act
            //assert-expected exception
            Assert.Throws<ArgumentException>(() => target.Get(0));
        }
        ```

## xUnit.net 2.0 驗證 exception

1. 直接在驗證時使用 delegate 來呼叫目標方法

    * 範例程式碼

        ```cs
        [Fact]
        public void 測試Get_傳入0_得到ArgumentException()
        {
            //arrange
            var target = new ValuesController();
                        //act
                        //assert-expected exception
            Assert.Throws<ArgumentException>(() => target.Get(0));
        }
        ```

2. 附上官方測試範例(用 try - catch) [xunit](https://github.com/xunit/xunit/blob/master/test/test.xunit1/xunit/ThrowsTests.cs)

    ```cs
    [Fact]
    public void ExpectExceptionButCodeThrowsDerivedException()
    {
        try
        {
            Assert.Throws<Exception>(delegate { throw new InvalidOperationException(); });
        }
        catch (AssertException exception)
        {
            Assert.Equal("Assert.Throws() Failure", exception.UserMessage);
        }
    }
    ```

## NSubstitute 驗證 exception

> 簡單易用的 Mock library

1. 針對有回傳值方法
    * MSTest

        * 測試只能 mock interface，mock 實體 class 會有問題，原因待查
        * 範例程式碼 1 - 使用 `MSTestExtensions`

            ```cs
            [TestMethod]
            public void 測試Get_傳入0_得到ArgumentException()
            {
                //arrange
                var target = Substitute.For<IValuesController>();
                target.Get(0).Returns(x => { throw new ArgumentException(); });
                //act
                //assert-expected exception
                Assert.Throws<ArgumentException>(() => target.Get(0));
            }
            ```

        * 範例程式碼 2

            ```cs
            [TestMethod]
            [ExpectedException(typeof(ArgumentException))]
            public void 測試Get_傳入0_得到ArgumentException()
            {
                //arrange
                var target = Substitute.For<IValuesController>();
                target.Get(0).Returns(x => { throw new ArgumentException(); });
                //act
                var actual = target.Get(0);
                //assert
            }
            ```

    * NUnit 3
        * 範例程式碼

            ```cs
            [Test]
            public void 測試Get_傳入0_得到ArgumentException()
            {
                //arrange
                var target = Substitute.For<IValuesController>();
                target.Get(0).Returns(x => { throw new ArgumentException(); });
                //act
                //assert-expected exception
                Assert.Throws<ArgumentException>(() => target.Get(0));
            }
            ```

    * xUnit.net 2.0

        * 範例程式碼

            ```cs
            [Fact]
            public void 測試Get_傳入0_得到ArgumentException()
            {
                //arrange
                var target = Substitute.For<IValuesController>();
                target.Get(0).Returns(x => { throw new ArgumentException(); });
                //act
                //assert-expected exception
                Assert.Throws<ArgumentException>(() => target.Get(0));
            }
            ```

2. 針對無回傳 (void) 方法
    * MSTest

        * 範例程式碼 1 - 使用 `MSTestExtensions`

            ```cs
            [TestMethod]
            public void 測試Delete_傳入0_得到ArgumentException()
            {
                //arrange
                var target = Substitute.For<IValuesController>();
                target.When(x => x.Delete(0)).Do(x => { throw new ArgumentException(); });
                //act
                Assert.Throws<ArgumentException>(() => target.Delete(0));
            }
            ```

        * 範例程式碼 2

            ```cs
            [TestMethod]
            [ExpectedException(typeof(ArgumentException))]
            public void 測試Delete_傳入0_得到ArgumentException()
            {
                //arrange
                var target = Substitute.For<IValuesController>();
                target.When(x => x.Delete(0)).Do(x => { throw new ArgumentException(); });
                //act
                target.Delete(0);
                //assert
            }
            ```

    * NUnit 3

        * 範例程式碼

            ```cs
            [Test]
            public void 測試Delete_傳入0_得到ArgumentException()
            {
                //arrange
                var target = Substitute.For<IValuesController>();
                target.When(x => x.Delete(0)).Do(x => { throw new ArgumentException(); });
                            //act
                            //assert-expected exception
                Assert.Throws<ArgumentException>(() => target.Delete(0));
            }
            ```

    * xUnit.net 2.0
        * 範例程式碼

            ```cs
            [Fact]
            public void 測試Delete_傳入0_得到ArgumentException()
            {
                //arrange
                var target = Substitute.For<IValuesController>();
                target.When(x => x.Delete(0)).Do(x => { throw new ArgumentException(); });
                //act
                //assert-expected exception
                Assert.Throws<ArgumentException>(() => target.Delete(0));
            }
            ```

## FluentAssertions 驗證 exception

> 使用語意化表達式，讓測試更好閱讀

1. 方法一：直接呼叫方法

    * MSTest

        * 直接使用實體 class

            ```cs
            [TestMethod]
            public void 測試Get_傳入0_得到ArgumentException()
            {
                //arrange
                var target = new ValuesController();
                        
                //act
                //assert
                target.Invoking(a => a.Get(0)).ShouldThrow<ArgumentException>();
            }
            ```

        * 搭配 NSubstitute

            ```cs
            [TestMethod]
            public void 測試Get_傳入0_得到ArgumentException()
            {
                //arrange
                var target = Substitute.For<IValuesController>();
                target.Get(0).Returns(x => { throw new ArgumentException(); });
                //act
                //assert
                target.Invoking(a => a.Get(0)).ShouldThrow<ArgumentException>();
            }
            ```

    * NUnit 3

        * 直接使用實體 class

            ```cs
            [Test]
            public void 測試Get_傳入0_得到ArgumentException()
            {
                //arrange
                var target = new ValuesController();
                            //act
                            //assert-expected exception
                target.Invoking(a => a.Get(0)).ShouldThrow<ArgumentException>();
            }
            ```

        * 搭配 NSubstitute

            ```cs
            [Test]
            public void 測試Get_傳入0_得到ArgumentException()
            {
                //arrange
                var target = Substitute.For<IValuesController>();
                target.Get(0).Returns(x => { throw new ArgumentException(); });
                            //act
                            //assert-expected exception
                target.Invoking(a => a.Get(0)).ShouldThrow<ArgumentException>();
            }
            ```

    * xUnit.net 2.0

        * 直接使用實體 class

            ```cs
            [Fact]
            public void 測試Get_傳入0_得到ArgumentException()
            {
                //arrange
                var target = new ValuesController();
                //act
                //assert-expected exception
                target.Invoking(a => a.Get(0)).ShouldThrow<ArgumentException>();
            }

        * 搭配 NSubstitute

            ```cs
            [Fact]
            public void 測試Get_傳入0_得到ArgumentException()
            {
                //arrange
                var target = Substitute.For<IValuesController>();
                target.Get(0).Returns(x => { throw new ArgumentException(); });
                //act
                //assert-expected exception
                target.Invoking(a => a.Get(0)).ShouldThrow<ArgumentException>();
            }
            ```

2. 方法二：使用 delegate 呼叫

    * MSTest

        * 直接使用實體 class

            ```cs
            [TestMethod]
            public void 測試Get_傳入0_得到ArgumentException()
            {
                //arrange
                var target = new ValuesController();
                            //act
                Action act = () => target.Get(0);
                            //assert
                act.ShouldThrow<ArgumentException>();
            }
            ```

        * 搭配 NSubstitute

            ```cs
            [TestMethod]
            public void 測試Get_傳入0_得到ArgumentException()
            {
                //arrange
                var target = Substitute.For<IValuesController>();
                target.Get(0).Returns(x => { throw new ArgumentException(); });
                //act
                Action act = () => target.Get(0);
                //assert
                act.ShouldThrow<ArgumentException>();
            }
            ```

    * NUnit 3

        * 直接使用實體 class

            ```cs
            [Test]
            public void 測試Get_傳入0_得到ArgumentException()
            {
                //arrange
                var target = new ValuesController();
                //act
                Action act = () => target.Get(0);
                //assert-expected exception
                act.ShouldThrow<ArgumentException>();
            }
            ```

        * 搭配 NSubstitute

            ```cs
            [Test]
            public void 測試Get_傳入0_得到ArgumentException()
            {
                //arrange
                var target = Substitute.For<IValuesController>();
                target.Get(0).Returns(x => { throw new ArgumentException(); });
                //act
                Action act = () => target.Get(0);
                //assert-expected exception
                act.ShouldThrow<ArgumentException>();
            }
            ```

    * xUnit.net 2.0

        * 直接使用實體 class

            ```cs
            [Fact]
            public void 測試Get_傳入0_得到ArgumentException()
            {
                //arrange
                var target = new ValuesController();
                //act
                Action act = () => target.Get(0);
                //assert-expected exception
                act.ShouldThrow<ArgumentException>();
            }
            ```

        * 搭配 NSubstitute

            ```cs
            [Fact]
            public void 測試Get_傳入0_得到ArgumentException()
            {
                //arrange
                var target = Substitute.For<IValuesController>();
                target.Get(0).Returns(x => { throw new ArgumentException(); });
                //act
                Action act = () => target.Get(0);
                //assert-expected exception
                act.ShouldThrow<ArgumentException>();
            }
            ```

## 參考資訊

1. [MSTestExtensions](https://github.com/bbraithwaite/mstestextensions)
2. [NUnit 3.0 and Assert.Throws](https://stackoverflow.com/questions/33897323/nunit-3-0-and-assert-throws)
3. [Testing exceptions with xUnit](http://hadihariri.com/2008/10/17/testing-exceptions-with-xunit/)
4. [NSubstitute Throwing exceptions](http://nsubstitute.github.io/help/throwing-exceptions/)
5. [Fluent Assertions - Exceptions](http://fluentassertions.com/documentation.html#exceptions)
