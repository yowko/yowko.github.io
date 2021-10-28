---
title: "Unit Test 中各個 Test Framework 的初始化及清除用法"
date: 2017-06-03T10:42:00+08:00
lastmod: 2021-10-14T10:42:41+08:00
draft: false
tags: ["NUnit","Unit Test"]
slug: "unit-test-initialize-cleanup"
aliases:
    - /2017/06/unit-test-initialize-cleanup.html
---
## Unit Test 中各個 Test Framework 的初始化及清除用法

最近因為 TDD 課程的關係，比較常使用各個 Test Framework 的功能，在寫作業時發現對於各個 Test Framework 的初始化及清除用法不是那麼清楚，常常會搞錯或是混用，趁著課程空檔，做個紀錄

## MSTest

1. TestInitialize|TestCleanup

    > 以測試方法為單位執行一次(每次測試都會執行)

    * 1-1. 在方法上加 `[TestInitialize]` or `[TestCleanup]` attribute
    * 1-2. 方法需要條件
        * non-static
        * public
        * 沒有回傳值 void
        * 不能有參數
        * 範例：

            ```cs
            [TestInitialize]
            public void Init()
            {
            }
            ```

    * 1-3. 違反方法簽章的錯誤訊息
        * 訊息內容  

            ```txt
            Test Name: ThrowNullReferenceException_realclass
            Test FullName: TDD_Day1.UnitTest_GroupSum.ThrowNullReferenceException_realclass
            Test Source: C:\Users\yowko.tsai\documents\visual studio 2017\Projects\TDD-Day1\TDD-Day1\UnitTest_GroupSum.cs : line 64
            Test Outcome: Failed
            Test Duration: 0:00:00
                        Result Message: Method TDD_Day1.UnitTest_GroupSum.Init has wrong signature. The method must be non-static, public, does not return a value and should not take any parameter. Additionally, if you are using async-await in method then return-type must be Task.
            ```

        * 錯誤截圖

            ![1TestInitializeError](https://cloud.githubusercontent.com/assets/3851540/26749878/b14d8166-4847-11e7-82ff-b93a86c02ab9.png)

2. ClassInitialize|ClassCleanup

    > 以測試 class 為單位執行一次 (每個測試 class 執行一次)

    * 2-1. 在方法上加 `[ClassInitialize]` or `[ClassCleanup]` attribute
    * 2-2. 方法需要條件
        * static
        * public
        * 沒有回傳值 void
        * 需要傳入 `TestContext` 參數
        * 範例：

            ```cs
            [ClassInitialize]
            public static void Init(TestContext testcontext)
            {
            }
            ```

    * 2-3. 違反方法簽章的錯誤訊息
        * 錯誤訊息

            ```txt
            Test Name: ThrowNullReferenceException_realclass
            Test FullName: TDD_Day1.UnitTest_GroupSum.ThrowNullReferenceException_realclass
            Test Source: C:\Users\yowko.tsai\documents\visual studio 2017\Projects\TDD-Day1\TDD-Day1\UnitTest_GroupSum.cs : line 65
            Test Outcome: Failed
            Test Duration: 0:00:00
                        Result Message: Method TDD_Day1.UnitTest_GroupSum.Init has wrong signature. The method must be static, public, does not return a value and should take a single parameter of type TestContext. Additionally, if you are using async-await in method then return-type must be Task.
            ```

        * 錯誤截圖

            ![2ClassInitializeError](https://cloud.githubusercontent.com/assets/3851540/26749879/b173adb4-4847-11e7-83d7-ab1bb838a782.png)

3. AssemblyInitialize|AssemblyCleanup

    > 以測試 project 為單位執行一次 (每個測試專案執行一次)

    * 3-1. 在方法上加 `[AssemblyInitialize]` or `[AssemblyCleanup]` attribute
    * 3-2. 方法需要條件
        * static
        * public
        * 沒有回傳值 void
        * 需要傳入 `TestContext` 參數
        * 範例：

            ```cs
            [AssemblyInitialize]
            public static void Init(TestContext testcontext)
            {
            }
            ```

    * 3-3. 違反方法簽章的錯誤訊息

        * 訊息內容

            ```txt
            Test Name: ThrowNullReferenceException_realclass
            Test FullName: TDD_Day1.UnitTest_GroupSum.ThrowNullReferenceException_realclass
            Test Source: C:\Users\yowko.tsai\documents\visual studio 2017\Projects\TDD-Day1\TDD-Day1\UnitTest_GroupSum.cs : line 65
            Test Outcome: Failed
            Test Duration: 0:00:00
                        Result Message: Method TDD_Day1.UnitTest_GroupSum.Init has wrong signature. The method must be static, public, does not return a value and should take a single parameter of type TestContext. Additionally, if you are using async-await in method then return-type must be Task.
            ```

        * 錯誤截圖

            ![3AssemblyInitializeError](https://cloud.githubusercontent.com/assets/3851540/26749880/b18cdcb2-4847-11e7-9c65-df7895a526a8.png)

4. 其他注意事項

    * 測試 class 需為 plublic

## NUint 3

1. SetUp|TearDown

    > 以測試方法為單位執行一次 (每個測試方法執行一次)

    * 1-1. 在方法上加 `[SetUp]` or `[TearDown]` attribute
    * 1-2. 方法需要條件
        * staic 或是 non-static 皆可
        * public
        * 沒有回傳值 void
        * 不能有參數
        * 範例：

            ```cs
            [SetUp]
            public void Init()
            {
            }
            ```

    * 1-3. 違反方法簽章的錯誤訊息

        * 錯誤訊息

            ```txt
            Test Name: NInitThrowNullReferenceException_realclass_NTest
            Test FullName: TDD_Day1.NUnit.NInit.NInitThrowNullReferenceException_realclass_NTest
            Test Source: C:\Users\yowko.tsai\documents\visual studio 2017\Projects\TDD-Day1\TDD-Day1\NUnit\NInit.cs : line 61
            Test Outcome: Failed
            Test Duration: 0:00:00.0000001
                        Result Message: OneTimeSetUp: Invalid signature for SetUp or TearDown method: InitTest
            ```

        * 錯誤截圖

            ![4setuperror](https://cloud.githubusercontent.com/assets/3851540/26749882/b1945a3c-4847-11e7-9743-1f0441accb0a.png)

2. OneTimeSetUp|OneTimeTearDown

    > 以 class 為單位執行一次 (每個測試 class 執行一次)

    * 2-1. 在方法上加 `[OneTimeSetUp]` or `[OneTimeTearDown]` attribute
    * 2-2. 方法需要條件
        * staic 或是 non-static 皆可
        * public
        * 沒有回傳值 void
        * 不能有參數
        * 可以在同一個測試 fixture 中出現多次
        * 如果標記 `[OneTimeSetUp]` or `[OneTimeTearDown]` 的方法出現 exception 會導致其他測試皆會 fail
        * 在這個方法內容無法輸出資訊至 output (`Console.WriteLine` 與 `TestContext.WriteLine` 皆無法作用，但方法會正確執行，可以透過 debug test 來確認)

            ![6output](https://cloud.githubusercontent.com/assets/3851540/26749884/b19646ee-4847-11e7-9d55-ece12175c7ca.png)

        * 範例：

            ```cs
            [OneTimeSetUp]
            public static void Init()
            {
            }
            ```

    * 2-3. 違反方法簽章的錯誤訊息

        * 訊息內容

            ```txt
            Test Name: NInitThrowNullReferenceException_realclass_NTest
            Test FullName: TDD_Day1.NUnit.NInit.NInitThrowNullReferenceException_realclass_NTest
            Test Source: C:\Users\yowko.tsai\documents\visual studio 2017\Projects\TDD-Day1\TDD-Day1\NUnit\NInit.cs : line 61
            Test Outcome: Failed
            Test Duration: 0:00:00.0000001
                        Result Message: OneTimeSetUp: Invalid signature for SetUp or TearDown method: InitTest
            ```

        * 錯誤截圖

            ![5onetimesetuperror](https://cloud.githubusercontent.com/assets/3851540/26749881/b1939c28-4847-11e7-9177-6ca029d25bdf.png)

3. SetUpFixture

    > 以 namespace 為單位執行一次 (每個 namespace 執行一次)

    * 3-1. 在 class 上加 `[SetUpFixture]` attribute 並在方法上加上 `[OneTimeSetUp]` or `[OneTimeTearDown]`
    * 3-2. 方法需要條件
        * class 層級
        * 不需 public
        * 不得與 `TestFixture` 一併存在
        * 其他規則與 `OneTimeSetUp|OneTimeTearDown` 相同
        * 範例：

            ```cs
            [SetUpFixture]
            public class NUnitInit
            {
                [OneTimeSetUp]
                public void Init()
                {
                    Console.WriteLine("AssemblyInitialize");//不會顧示在 output
                }
            }
            ```

4. 其他注意事項
    * 測試 class 需為 plublic

## xUnit.net 2.0

1. 使用測試 class 的建構子|繼承 IDisposable 實作 Dispose

    > 以測試方法為單位執行一次(每個測試都會執行一次)

    * 每個測試方法執行前動作

        > 使用測試 class 的建構子

        * public

    * 每個測試方法執行後動作

        > 繼承 IDisposable 實作 Dispose

        * public
        * 方法沒有參數

    * 範例

        ```cs
        public class XUnitInit : IDisposable
        {
            public ITestOutputHelper _output;//=new TestOutputHelper();
                        public XUnitInit(ITestOutputHelper output)
            {
                this._output=output;
                _output.WriteLine("constructor");
            }
            public void Dispose()
            {
                _output.WriteLine("dispose");
            }
            [Fact]
            public void MyTest()
            {
                var temp = "my class!";
                _output.WriteLine("This is output from {0}", temp);
            }
        }
        ```

    * 違反方法簽章的錯誤訊息(for 建構子)

        * 錯誤訊息

            ```txt
            Test Name: TDD_Day1.XUnit.XUnitTest_GroupSum.四筆一組取revenue總合_realclass
            Test FullName: TDD_Day1.XUnit.XUnitTest_GroupSum.四筆一組取revenue總合_realclass (2ce5c285b246ba39b822e60e6184ef06ee1c5b9f)
            Test Source: C:\Users\yowko.tsai\documents\visual studio 2017\Projects\TDD-Day1\TDD-Day1\XUnitTest_GroupSum.cs : line 69
            Test Outcome: Failed
            Test Duration: 0:00:00.001
                        Result Message: A test class may only define a single public constructor.
            ```

        * 錯誤截圖

            ![7xunitsingle](https://cloud.githubusercontent.com/assets/3851540/26749883/b195bdfa-4847-11e7-8be8-dc9ebe304332.png)

2. IClassFixture

    > 以 class 為單位執行一次 (每個測試 class 都會執行一次)

    * 2-1. 定義一個執行動作或是其他屬性的 class
    * 2-2. 將定義好的 class 透過 `IClassFixture` 傳入測試 class 中

        ```cs
        public class DatabaseFixture : IDisposable
        {
            public DatabaseFixture()
            {
                Db = new SqlConnection("MyConnectionString");
                // ... initialize data in the test database ...
            }
            public void Dispose()
            {
                // ... clean up test data from the database ...
            }
            public SqlConnection Db { get; private set; }
        }
        public class MyDatabaseTests : IClassFixture<DatabaseFixture>
        {
            DatabaseFixture fixture;
            public MyDatabaseTests(DatabaseFixture fixture)
            {
                this.fixture = fixture;
            }
            // ... write tests, using fixture.Db to get access to the SQL Server ...
        }
        ```

* 其他注意事項
  * xUnit 不需要在測試 class 上加 attribute，但測試 class 仍需要 public
  * xUnit 沒有以 project base 或是 namespace 層級的方法
  * xUnit 無法使用 `Console.WriteLine`，想要輸出文字可以參考下面用法

    ```cs
    public class XUnitTest_GroupSum
    {
        private readonly ITestOutputHelper output;
        public XUnitTest_GroupSum(ITestOutputHelper output)
        {
            output.WriteLine("Init");
        }
    }
    ```

## 心得

xUnit 的使用方式與 MSTest 跟 NUint 差比較多，使用的上需要重新適應，功能也比不上 NUint，資源相對難找，使用便利性比較不足

## 參考資訊

1. [MSTest,NUnit 3,xUnit.net 2.0 比較](/MStest-NUnit3-xUnit.net2-Compare)
2. [使用 MSTest、Nunit 3、xUnit.net 2.0、NSubstitute、FluentAssertions 驗證例外(exception)](/mstestnunit-3xunitnet)
3. [Comparing xUnit.net to other frameworks](http://xunit.github.io/docs/comparisons.html)
4. [Shared Context between Tests](https://xunit.github.io/docs/shared-context.html)
5. [Capturing Output](https://xunit.github.io/docs/capturing-output.html)
