---
title: "Unit Test 該拿 private 屬性及欄位怎麼辦？ - 使用 PrivateObject"
date: 2017-06-19T00:32:00+08:00
lastmod: 2021-10-14T00:32:37+08:00
draft: false
tags: ["MSTest","Unit Test"]
slug: "unit-test-private-field-property"
aliases:
    - /2017/06/unit-test-private-field-property.html
---
## Unit Test 該拿 private 屬性及欄位怎麼辦？ - 使用 PrivateObject

在 [Unit Test 該拿 static 屬性及欄位怎麼辦？ - 使用 PrivateType](//blog.yowko.com/2017/06/unit-test-static-field-property.html) 使用 NSubstitute 產生假物件後，透過 PrivateType 設定給原本是 private static 的屬性，但文末也提供 PrivateType 是針對 static field 或是 property 才能使用

如果只是一般的 field 或是 property 該如何是好呢？ 這時候可以使用 PrivateObject

## 基本環境

> 有個 restful api，Get 方法會依據 Web.config 的一個變數值來決定回應內容：如果是 localhost 表示 config 異常，其他值就正常回應

* 程式碼

    ```cs
    public class ValuesController : ApiController
    {
        private readonly string connectionString = System.Configuration.ConfigurationManager.AppSettings["connectStr"];
        public IHttpActionResult Get()
        {
            if (connectionString.Equals("localhost"))
                return BadRequest();
            else
                return Ok();
        }
    }
    ```

* Web.config 設定

    ```cs
    <configuration>
        <appSettings>
            <add key="connectStr" value="db.yowko.com"/>
        </appSettings>
    </configuration>
    ```

## 使用 PrivateObject 來模擬 private field or property

* 寫法 一：直接建立測試目標實體

    1. 建立測試目標程式的實體

        ```cs
        var target = new ValuesController();
        ```

    2. 使用測試目標實體建立 PrivateObject

        ```cs
        PrivateObject valueController=new PrivateObject(target);
        ```

    3. 設定測試目標的 field or property

        ```cs
        valueController.SetFieldOrProperty("connectionString", "localhost");
        ```

    4. 呼叫測試目標方法取得回應並進行驗證

        ```cs
        //act
        var actualAction = target.Get();
        var actual = actualAction as BadRequestResult;
        //assert
        Assert.IsNotNull(actual);
        Assert.IsInstanceOfType(actual, expected);
        ```

    5. 完整測試程式碼

        ```cs
        [TestMethod()]
        public void Get_connectionString_is_localhost_Return_BadRequestResult()
        {
            //arrange 
            var target = new ValuesController();
            PrivateObject valueController=new PrivateObject(target);
            valueController.SetFieldOrProperty("connectionString", "localhost");
            var expected = typeof(BadRequestResult);
            //act
            var actualAction = target.Get();
            var actual = actualAction as BadRequestResult;
            //assert
            Assert.IsNotNull(actual);
            Assert.IsInstanceOfType(actual, expected);
        }
        ```

* 寫法 二：使用型別建立 PrivateObject

    1. 使用測試目標實體建立 PrivateObject

        ```cs
        PrivateObject target = new PrivateObject(typeof(ValuesController));
        ```

    2. 設定測試目標的 field or property

        ```cs
        target.SetFieldOrProperty("connectionString", "localhost");
        ```

    3. 呼叫測試目標方法取得回應並進行驗證

        ```cs
        //act
        var actualAction = target.Invoke("Get");
        var actual = actualAction as BadRequestResult;
        //assert
        Assert.IsNotNull(actual);
        Assert.IsInstanceOfType(actual, expected);
        ```

    4. 完整測試程式碼

        ```cs
        [TestMethod()]
        public void Get_connectionString_is_localhost_Return_BadRequestResult()
        {
            //arrange 
            PrivateObject target = new PrivateObject(typeof(ValuesController));
            target.SetFieldOrProperty("connectionString", "localhost");
            var expected = typeof(BadRequestResult);
            //act
            var actualAction = target.Invoke("Get");
            var actual = actualAction as BadRequestResult;
            //assert
            Assert.IsNotNull(actual);
            Assert.IsInstanceOfType(actual, expected);
        }
        ```

## 心得

PrivateType 用來處理 static 資源 (field,property,method)，PrivateObject 則用來處理其他 non-static 資源，還算是清楚，但建立物件方式都不同，是比較容易搞混的

不過我滿欣賞它們不用動到 production code 的特性，讓測試歸測試，重構歸重構，不會為了需要測試得先重構

PrivateObject 的官方說明可以參考 [PrivateObject Class](https://msdn.microsoft.com/en-us/library/microsoft.visualstudio.testtools.unittesting.privateobject%28v=vs.120%29.aspx)

## 參考資訊

1. [Unit Test 該拿 static 屬性及欄位怎麼辦？ - 使用 PrivateType](//blog.yowko.com/2017/06/unit-test-static-field-property.html)
2. [PrivateObject Class](https://msdn.microsoft.com/en-us/library/microsoft.visualstudio.testtools.unittesting.privateobject%28v=vs.120%29.aspx)
3. [Accessing private and protected members - PrivateObject and PrivateType](http://yac.com.pl/mt.texts.vbnet-privateobject-privatetype.en.html)
