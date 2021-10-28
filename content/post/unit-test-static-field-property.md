---
title: "Unit Test 該拿 static 屬性及欄位怎麼辦？ - 使用 PrivateType"
date: 2017-06-17T23:18:00+08:00
lastmod: 2021-10-14T15:27:02+08:00
draft: false
tags: ["MSTest","Unit Test"]
slug: "unit-test-static-field-property"
aliases:
    - /2017/06/unit-test-static-field-property.html
---
## Unit Test 該拿 static 屬性及欄位怎麼辦？ - 使用 PrivateType

自從上完第二次 TDD 課程後，對於新專案的開發充滿著信心，躍躍欲試不算新學到但有新理解的技能，只是過程還是跌跌撞撞、踩雷不斷，也許不該說是踩雷，說是對測試的相關工具還有觀念都沒有很熟悉的關係比較正確。

這也讓我想起有次參加 曹祖聖 老師在一場研討會中，提到有次他在研究 System Center Operations Manager 時，第一次環境架設就一切順利，完全沒遇到問題，但他卻一點開心的感覺也沒有，反而是很失落，因為他知道他沒有真的學到東西，透過錯誤解決以及發想解決方案的過程才會讓他學到更多，同樣的想法也影響著我，透過寫測試時遇到的各式問題不僅讓我更了解測試也讓我更踏實

回到今天的主題，測試時遇到 static field 或是 property 該如何處理？

## 基本環境

* 一個 restful Web Api 只有 Post 有動作，其中引用 nlog 來紀錄收到 request 的時間與收到的參數
* 程式碼

    ```cs
    public class ValuesController : ApiController
    {
        private static ILogger logger = LogManager.GetLogger("ValuesController");
                    public IHttpActionResult Post([FromBody] string value)
        {
            logger.Debug($"EventTime：{DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss")} Value={value}");
            if (string.IsNullOrEmpty(value))
                return BadRequest();
            else
                return Ok();
        }
    }
    ```

## 如何測試？

經過三天 TDD 的訓練後，第一眼看到程式碼，我腦中浮現的測試程式大概長得像下面程式

1. 目標程式加入 constructor

    ```cs
    public ValuesController()
    {
    }
    ```

2. 為 static 資源加入 seam

    > 將 static 資源初始化動作移至 constructor，讓後續有空隙可以將假造物件塞入

    ```cs
    private static ILogger logger;
    public ValuesController()
    {
        logger = LogManager.GetLogger("ValuesController");
    }
    ```

3. 加入允許傳入 ILogger 參數的 constructor 並讓無參數 constructor 呼叫

    > 這邊無參數 constructor 呼叫傳入 ILogger 參數的 constructor 可以讓原本呼叫該 api 的程式碼不用異動

    ```cs
    public ValuesController() : this(LogManager.GetLogger("ValuesController"))
    {
    }
    public ValuesController(ILogger _logger)
    {
        logger = _logger;
    }
    ```

4. 從測試程式傳入 ILogger 物件

    > 搭配 `NSubstitute` 產生虛擬物件，關於如何驗證 IHttpActionResult 細節可以參考 [Unit Test 如何驗證 ASP.NET Web Api 的 IHttpActionResult](/unit-test-web-api-ihttpactionresult)

    ```cs
    [TestMethod]
    public void Post_StringEmpty_Return_BadRequest()
    {
        //arrange 
        var logger = Substitute.For<ILogger>();
        var target = new ValuesController(logger);
        var expected = typeof(BadRequestResult);
        //act
        var actualAction = target.Post(string.Empty);
        var actual = actualAction as BadRequestResult;
        //assert
        Assert.IsNotNull(actual);
        Assert.IsInstanceOfType(actual, expected);
    }
    ```

## 有改善空間嗎？

測試程式本身應該滿簡潔，測試目標程式的修改幅度也不大，說實話我覺得很棒了，但就是掩不住好奇的心想知道其他人是怎麼做的，下面提供另一個做法(我也不知道是好是壞，請大家自行斟酌衡量)

* 使用 PrivateType 來設定 static field 或是 property

1. <span style="color:red">不用修改</span> 測試目標程式

    ```cs
    public class ValuesController : ApiController
    {
        private static ILogger logger = LogManager.GetLogger("ValuesController");
        public IHttpActionResult Post([FromBody] string value)
        {
            logger.Debug($"EventTime：{DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss")};Value={value}");
            if (string.IsNullOrEmpty(value))
                return BadRequest();
            else
                return Ok();
        }
    }
    ```

2. 使用 `NSubstitute` 產生 ILogger 虛擬物件

    ```cs
    var logger = Substitute.For<ILogger>();
    ```

3. 建立測試目標程式的 PrivateType 物件

    ```cs
    PrivateType valueController = new PrivateType(typeof(ValuesController));
    ```

4. 設定 測試目標程式 static field

    ```cs
    valueController.SetStaticFieldOrProperty("logger", logger);
    ```

5. 最終程式碼

    ```cs
    [TestMethod]
    public void Post_StringEmpty_Return_BadRequest()
    {
        //arrange 
        var logger = Substitute.For<ILogger>();
        var target = new ValuesController();
        PrivateType valueController = new PrivateType(typeof(ValuesController));
        valueController.SetStaticFieldOrProperty("logger", logger);
        var expected = typeof(BadRequestResult);
        //act
        var actualAction = target.Post(string.Empty);
        var actual = actualAction as BadRequestResult;
        //assert
        Assert.IsNotNull(actual);
        Assert.IsInstanceOfType(actual, expected);
    }
    ```

## 心得

我測試下來 PrivateType 針對 static 資源有 set、 get 跟 invoke (針對 method) 的 api 可以使用，但有個重點是 `static`，不過不覺得它的名稱取得不好，叫 PrivateType 但事實上 non-private 也可以用(誤)，使用 PrivateType 好處是完全不需要異動到測試目標程式碼，這讓測試程式寫起來更乾淨，提供給大家參考看看，詳細介紹請參考 [PrivateType Class](https://msdn.microsoft.com/en-us/library/microsoft.visualstudio.testtools.unittesting.privatetype%28v=vs.120%29.aspx)(說實話，介紹也沒有很詳細啦)

## 參考資訊

1. [How to access a Static Class Private fields to unit test its methods using Microsoft Fakes in C#](https://stackoverflow.com/questions/30535918/how-to-access-a-static-class-private-fields-to-unit-test-its-methods-using-micro)
2. [PrivateType Class](https://msdn.microsoft.com/en-us/library/microsoft.visualstudio.testtools.unittesting.privatetype%28v=vs.120%29.aspx)
