---
title: "Unit Test 想驗證 private static method 該怎麼做？ - 使用 PrivateType"
date: 2017-06-18T21:26:00+08:00
lastmod: 2018-08-22T21:26:48+08:00
draft: false
tags: ["C#","MSTest","Unit Test"]
slug: "unit-test-static-method"
aliases:
    - /2017/06/unit-test-static-method.html
---
# Unit Test 想驗證 private static method 該怎麼做？ - 使用 PrivateType
TDD 的第一天課程中就提到，以單元測試的角度 private method 不需單獨進行測試，在驗證 public 及 internal method 的過程中自然會涵蓋到 private or protected method，至於 static method 也是相同概念，只要有用到就需要測試

雖然知道透過 public or internal method 呼叫應該可以 cover 到 private static method，但有沒有粒度更小的測試空間呢？

## 基本環境

*   程式碼

    ```cs
    public class ValuesController : ApiController
    {
        private static ILogger logger = LogManager.GetLogger("ValuesController");
        public IHttpActionResult Post([FromBody] string value)
        {
            logger.Debug($"EventTime：{DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss")};Value={value}");
            var msg = GetNowReturn(value);
            if (string.IsNullOrEmpty(value))
                return BadRequest(msg);
            else
                return Ok(msg);
        }
    }
    private static string GetNowReturn(string value)
    {
        return $"{DateTime.Today}-{value}";
    }
    ```

## 如何測試？

> 最簡單的改法應該就是將 `GetNowReturn` 改為 internal

1.  修改 private static 為 internal

    ```cs
    internal string GetNowReturn(string value)
    {
        return $"{DateTime.Today}-{value}";
    }
    ```

2.  修改測試目標程式的 AssemblyInfo.cs 讓測試專案可以看見 internal method

    *   開啟 `Properties` 下的 `AssemblyInfo.cs`
    *   加上 `[assembly: InternalsVisibleTo("{測試專案名稱}")]`


    ![1assmebly](https://user-images.githubusercontent.com/3851540/27260987-0ec37d7a-546c-11e7-9c6e-791a21ccc3dc.png)

3.  測試程式就可以直接呼叫改為 `internal` 的方法來驗證結果

    ```cs
    [TestMethod]
    public void GetNowReturn_Yowko_Return_DateTimeToday_value()
    {
        //arrange 
        var target = new ValuesController();
        string value = "Yowko";
        var expected = $"{DateTime.Today}-{value}";
        //act
        var actual = target.GetNowReturn(value);
        //assert
        Assert.AreEqual(expected, actual);
    }
    ```

## 有其他解決方式嗎？ - 使用 PrivateType

在 [Unit Test 該拿 static 屬性及欄位怎麼辦？ - 使用 PrivateType](//blog.yowko.com/2017/06/unit-test-static-field-property.html) 過程中看官方 api 說明時看到可以直接執行 static method

*   使用 PrivateType 來執行 private static method


1.  <span style="color:red">不用修改</span> 測試目標程式

    ```cs
    public class ValuesController : ApiController
    {
        private static ILogger logger = LogManager.GetLogger("ValuesController");
        public IHttpActionResult Post([FromBody] string value)
        {
            logger.Debug($"EventTime：{DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss")} Value={value}");
            var msg = GetNowReturn(value);
            if (string.IsNullOrEmpty(value))
                return BadRequest(msg);
            else
                return Ok(msg);
        }
    }
    private static string GetNowReturn(string value)
    {
        return $"{DateTime.Today}-{value}";
    }
    ```

2.  建立測試目標程式的 PrivateType 物件

    ```cs
    PrivateType target = new PrivateType(typeof(ValuesController));
    ```

3.  執行測試目標程式的 private static method

    ```cs
    var actual = target.InvokeStatic("GetNowReturn", value);
    ```
4.  最後程式碼

    ```cs
    [TestMethod]
    public void GetNowReturn_Yowko_Return_DateTimeToday_value()
    {
        //arrange 
        PrivateType target = new PrivateType(typeof(ValuesController));
        string value = "Yowko";
        var expected = $"{DateTime.Today}-{value}";
        //act
        var actual = target.InvokeStatic("GetNowReturn", value);
        //assert
        Assert.AreEqual(expected, actual);
    }
    ```

## 心得

使用 PrivateType 來進行 static method 測試，就可以不用動到測試目標程式，相對風險更低，實際測試下來只要是 static 的 method 不論是 private、protected、internal、public 都可以使用 PrivateType 來進行測試，非常方便。

至於該不該單獨為 private method 測試，這就留給大家自行衡量，這邊就介紹 PrivateType 給大家認識，PrivateType 相關 api 可以參考 [PrivateType Class](https://msdn.microsoft.com/en-us/library/microsoft.visualstudio.testtools.unittesting.privatetype%28v=vs.120%29.aspx)

# 參考資訊

1.  [PrivateType Class](https://msdn.microsoft.com/en-us/library/microsoft.visualstudio.testtools.unittesting.privatetype%28v=vs.120%29.aspx)
2.  [Unit Test 該拿 static 屬性及欄位怎麼辦？ - 使用 PrivateType](//blog.yowko.com/2017/06/unit-test-static-field-property.html)
