---
title: "如何取得 NUnit 當前執行的測試方法名稱"
date: 2019-03-10T23:00:00+08:00
lastmod: 2019-03-10T23:00:31+08:00
draft: false
tags: ["Unit Test","NUnit"]
slug: "nunit-current-test-name"
---
# 如何取得 NUnit 當前執行的測試方法名稱
看到 `如何取得 NUnit 當前執行的測試方法名稱` 這個標題，相信不少有 Unit Test 經驗的開發者都會疑惑：為什麼會需要當下的測試方法名稱：Unit Test 沒有寫 log 的需求也不能有邏輯判斷，取得執行當下的測試方法有什麼意義嗎？

沒錯，的確不是為了 Unit Test 所需，而是 Integration Test，不知道是不是 test case 設計不好，在執行平行測試時，常常會出現測試 fail，但單獨執行又正常的現象，仔細追查原因後發現是 Integration Test 在平行測試下同時存取到相同資源造成的：A 測試想要刪資料、B 測試想要改資料，使得測試時好時壞

## 基本環境說明

## 模擬情境
1. 模擬程式碼

    - 測試一：新增資料至 redis，成功後即刪除資料
    - 測試二：新增資料後修改資料，確認取得更新的結果與預期相符，即刪除資料
    - 測試三：新增資料後，執行刪除，確認成功

    ```cs
    public class Tests
    {
        private static ConnectionMultiplexer redis = ConnectionMultiplexer.Connect("localhost");
        private static IDatabase db = redis.GetDatabase();

        private string key;
        private string value;

        [SetUp]
        public void InitData() {
            key = "Name";
            value = "Yowko";
        }

        [Test]
        [Parallelizable]
        public void AddData()
        {
            //arrange
            var expect = true;
            
            //act
            var actual = SetDataToRedis(key, value);

            #region - Remove Data -
            var deleteActual = DeleteDataFromRedis(key);
            #endregion

            //assert
            actual.Should().Be(expect);
            deleteActual.Should().Be(expect);
        }

        [Test]
        [Parallelizable]
        public void UpdateData()
        {
            //arrange
            var expect = "Yowko_update";
            SetDataToRedis(key,value);
            SetDataToRedis(key, expect);

            //act
            var actual = db.StringGet(key);

            #region - Remove Data -
            var deleteActual = DeleteDataFromRedis(key);
            #endregion

            //assert
            actual.Should().Be(expect);
            deleteActual.Should().BeTrue();
        }


        [Test]
        [Parallelizable]
        public void DeleteData()
        {
            //arrange
            var expect = true;
            SetDataToRedis(key, value);

            //act
            var actual = DeleteDataFromRedis(key);

            
            //assert
            actual.Should().Be(expect);
        }

        private bool SetDataToRedis(string key, string value)
        {
            return db.StringSet(key, value);
        }
        private bool DeleteDataFromRedis(string key)
        {
            return db.KeyDelete(key);
        }
    }
    ```
2. 平行執行會出錯

    ![1parallelfail](https://user-images.githubusercontent.com/3851540/54087643-c79f4900-438f-11e9-8e44-ea29a918415d.png)

3. 單獨測試則正常

    ![2singlepass](https://user-images.githubusercontent.com/3851540/54087644-c79f4900-438f-11e9-974c-91783f8fa12d.png)

## 解決方式：使用當前測試方法名稱當做測試變數
Integration Test 資料互相影響是在所難免，所以透過當前測試方法名稱當做測試變數以隔離每個測試間的交互關係

`NUnit.Framework.TestContext.CurrentContext.Test.Name` 或 `NUnit.Framework.TestContext.CurrentContext.Test.FullName`


```cs
public class Tests
{
    private static ConnectionMultiplexer redis = ConnectionMultiplexer.Connect("localhost");
    private static IDatabase db = redis.GetDatabase();
    private string key;
    private string value;
    [SetUp]
    public void InitData() {
        key = "Name";
        value = "Yowko";
    }
    [Test]
    [Parallelizable]
    public void AddData()
    {
        //arrange
        var expect = true;
        var key = TestContext.CurrentContext.Test.Name;
        
        //act
        var actual = SetDataToRedis(key, value);
        #region - Remove Data -
        var deleteActual = DeleteDataFromRedis(key);
        #endregion
        //assert
        actual.Should().Be(expect);
        deleteActual.Should().Be(expect);
    }
    [Test]
    [Parallelizable]
    public void UpdateData()
    {
        //arrange
        var expect = "Yowko_update";
        var key = TestContext.CurrentContext.Test.Name;
        SetDataToRedis(key,value);
        SetDataToRedis(key, expect);
        //act
        var actual = db.StringGet(key);
        #region - Remove Data -
        var deleteActual = DeleteDataFromRedis(key);
        #endregion
        //assert
        actual.Should().Be(expect);
        deleteActual.Should().BeTrue();
    }
    [Test]
    [Parallelizable]
    public void DeleteData()
    {
        //arrange
        var expect = true;
        var key = TestContext.CurrentContext.Test.Name;
        SetDataToRedis(key, value);
        //act
        var actual = DeleteDataFromRedis(key);
        
        //assert
        actual.Should().Be(expect);
    }
    private bool SetDataToRedis(string key, string value)
    {
        return db.StringSet(key, value);
    }
    private bool DeleteDataFromRedis(string key)
    {
        return db.KeyDelete(key);
    }
}
```

![3allsuccess](https://user-images.githubusercontent.com/3851540/54087645-c79f4900-438f-11e9-833b-b2f41426690f.png)

# 心得
規劃是希望每個 test case 執行結束後自行刪除建立的資料，以免遺留髒資料，但就是因為這個動作讓不同的 test case 對相同資料操作而造成問題，也才想到透過使用 test method name 來隔離，不確定是不是個好做法  但可以確定的是可以解決問題

原本是想透過 c# StackTrace 拿到 method，查了資料才發現 NUnit 有提供更簡單的方式：`NUnit.Framework.TestContext.CurrentContext.Test.Name` 或 `NUnit.Framework.TestContext.CurrentContext.Test.FullName`  可能前人跟我有類似需求  哈哈

# 參考資訊
1. [How to access the NUnit test name programmatically?](https://stackoverflow.com/a/14340993)