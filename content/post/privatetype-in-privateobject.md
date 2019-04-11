---
title: "驗證 private method 但會使用的 static 資料的做法 - PrivateType 與 PrivateObject 搭配"
date: 2017-06-20T09:00:00+08:00
lastmod: 2018-09-23T18:43:33+08:00
draft: false
tags: ["MSTest","Unit Test"]
slug: "privatetype-in-privateobject"
aliases:
    - /2017/06/privatetype-in-privateobject.html
---
# 驗證 private method 但會使用的 static 資料的做法 - PrivateType 與 PrivateObject 搭配
一連幾篇筆記([Unit Test 該拿 static 屬性及欄位怎麼辦？ - 使用 PrivateType](//blog.yowko.com/2017/06/unit-test-static-field-property.html)、[Unit Test 想驗證 private static method 該怎麼做？ - 使用 PrivateType](//blog.yowko.com/2017/06/unit-test-static-method.html)、[Unit Test 該拿 private 屬性及欄位怎麼辦？ - 使用 PrivateObject](//blog.yowko.com/2017/06/unit-test-private-field-property.html)、[Unit Test 想驗證 private method 該怎麼做？ - 使用 PrivateObject](//blog.yowko.com/2017/06/unit-test-private-method.html)) 認識了如何透過 PrivateType 與 PrivateObject 在撰寫 Unit Test 時解決 private 、 static 資源及方法，當然這兩個類別也可以用在 protected、internal、甚至 public 資源及方法

今天要來介紹混合用法：如果想驗證一個 private 方法，而 private 方法使用了 private static 的資源，這時候該怎麼進行 unit test 呢

## 基本環境

> restful api 在 Post method 會去 call 一個 private method，private method 會用使用 private static field 寫入 log，今天想要針對這個 private method 進行驗證

*   程式碼

    ```cs
    public class ValuesController : ApiController
    {
        private static ILogger logger = LogManager.GetLogger("ValuesController");
        public IHttpActionResult Post([FromBody] string value)
        {
            logger.Debug($"EventTime：{DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss")} Value={value}");
            return GetResponse(value);
        }
        private IHttpActionResult GetResponse(string value)
        {
            logger.Debug($"GetResponse：Receive Value={value}");
            if (string.IsNullOrEmpty(value))
                return BadRequest();
            else
                return Ok();
        }
    }
    ```

## 使用 PrivateType 與 PrivateObject 來測試 private method

1.  使用 NSubstitute 模擬 ILogger

    ```cs
    var logger = Substitute.For<ILogger>();
    ```

2.  使用 PrivateType 將模擬的 PrivateType 設定為測試目標程式的 field

    ```cs
    //建立測試目標程式的 PrivateType
    PrivateType pt = new PrivateType(typeof(ValuesController));
    // 指定 PrivateType 的 logger
    pt.SetStaticFieldOrProperty("logger", logger);
    ```

3.  使用 測試目標程式的 PrivateType 來建立測試目標的 PrivateObject

    *   使用新增測試目標實體來建立 PrivateObject

        ```cs
        PrivateObject target = new PrivateObject(new ValuesController(), pt);
        ```
    *   使用既有測試目標來建立 PrivateObject

        ```cs
        var valuesController = new ValuesController();
        PrivateObject target = new PrivateObject(valuesController, pt);
        ```

4.  完整測試程式

    ```cs
    [TestMethod()]
    public void PostTest()
    {
        //arrange
        var logger = Substitute.For<ILogger>();
        PrivateType pt = new PrivateType(typeof(ValuesController));
        pt.SetStaticFieldOrProperty("logger", logger);
        PrivateObject target = new PrivateObject(new ValuesController(), pt);
        var expected = typeof(BadRequestResult);
        //act
        var actualAction = target.Invoke("GetResponse", new object[] { string.Empty });
        var actual = actualAction as BadRequestResult;
        //assert
        Assert.IsNotNull(actual);
        Assert.IsInstanceOfType(actual, expected);
    }
    ```

## 心得

private method 使用到 private static field (紀錄 log) 的情境是相當常見的，透過 PrivateType 設定 private static field 後將設定後的 PrivateType 當做參數建立 PrivateObject，接著就可以用來執行 private method 進行驗證了。

經過將 PrivateType 與 PrivateObject 混搭使用，這麼一來就可以解決想要驗證 private method 但會使用的 static 資料的問題，是不是很方便呢？！

# 參考資訊

1.  [Unit Test 該拿 static 屬性及欄位怎麼辦？ - 使用 PrivateType](//blog.yowko.com/2017/06/unit-test-static-field-property.html)
2.  [Unit Test 想驗證 private static method 該怎麼做？ - 使用 PrivateType](//blog.yowko.com/2017/06/unit-test-static-method.html)
3.  [Unit Test 該拿 private 屬性及欄位怎麼辦？ - 使用 PrivateObject](//blog.yowko.com/2017/06/unit-test-private-field-property.html)
4.  [Unit Test 想驗證 private method 該怎麼做？ - 使用 PrivateObject](//blog.yowko.com/2017/06/unit-test-private-method.html)
5.  [How can I use PrivateObject to access private members of both my class and its parent?](https://stackoverflow.com/questions/5396996/how-can-i-use-privateobject-to-access-private-members-of-both-my-class-and-its-p)
