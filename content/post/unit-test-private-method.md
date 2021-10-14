---
title: "Unit Test 想驗證 private method 該怎麼做？ - 使用 PrivateObject"
date: 2017-06-19T01:14:00+08:00
lastmod: 2021-10-14T13:00:53+08:00
draft: false
tags: ["MSTest","Unit Test"]
slug: "unit-test-private-method"
aliases:
    - /2017/06/unit-test-private-method.html
---
## Unit Test 想驗證 private method 該怎麼做？ - 使用 PrivateObject

[Unit Test 該拿 private 屬性及欄位怎麼辦？ - 使用 PrivateObject](//blog.yowko.com/2017/06/unit-test-private-field-property.html) 針對的是 private 而且是 non-static 的 field 或是 property，[Unit Test 想驗證 private static method 該怎麼做？ - 使用 PrivateType](//blog.yowko.com/2017/06/unit-test-static-method.html) 則是針對 private static method，那對於 private 但 non-static method 就一樣需要 PrivateObject 來處理

## 基本環境

> restful api 的 Get 方法依據傳入的參數是否為空字串來回應

* 程式碼

    ```cs
    public IHttpActionResult Get(string value)
    {
        var isNullorEmpty= detectStringEmpty(value);
        if (isNullorEmpty)
            return BadRequest();
        else
            return Ok();
    }
    private bool detectStringEmpty(string value)
    {
        if (string.IsNullOrWhiteSpace(value))
            return true;
        else
            return false;
    }
    ```

## 使用 PrivateObject 來執行 private method

* 寫法一：使用測試目標實體來建立 PrivateObject

    1. 建立測試目標實體

        ```cs
        var target= new ValuesController();
        ```

    2. 使用測試目標實體建立 PrivateObject

        ```cs
        PrivateObject po = new PrivateObject(target);
        ```

    3. 執行測試目標的 private method

        ```cs
        var actual = po.Invoke("detectStringEmpty", new object[] {string.Empty});
        ```

    4. 完整測試程式碼

        ```cs
        [TestMethod()]
        public void detectStringEmpty_Return_True()
        {
            //arrange 
            var target= new ValuesController();
            PrivateObject po = new PrivateObject(target);
            var expected = true;
                        //act
            var actual = po.Invoke("detectStringEmpty", new object[] {string.Empty});
                        //assert
            Assert.AreEqual(actual, expected);
        }
        ```

* 寫法二：使用型別來建立 PrivateObject

    1. 建立 PrivateObject

        ```cs
        PrivateObject target = new PrivateObject(typeof(ValuesController));
        ```

    2. 執行測試目標的 private method

        ```cs
        var actual = target.Invoke("detectStringEmpty", new object[] {string.Empty});
        ```

    3. 完整測試程式

        ```cs
        [TestMethod()]
        public void detectStringEmpty_Return_True()
        {
            //arrange 
            PrivateObject target = new PrivateObject(typeof(ValuesController));
            var expected = true;
            //act
            var actual = target.Invoke("detectStringEmpty", new object[] {string.Empty});
            //assert
            Assert.AreEqual(actual, expected);
        }
        ```

## 心得

透過使用 PrivateObject 來執行 private method 就可以達成驗證 private method 的目地，至於該不該為 private method 一樣得由您自行決定囉

PrivateObject 的官方說明可以參考 [PrivateObject Class](https://msdn.microsoft.com/en-us/library/microsoft.visualstudio.testtools.unittesting.privateobject%28v=vs.120%29.aspx)

PrivateObject 的用法比較靈活，下一篇會介紹 PrivateType 與 PrivateObject 的混合用法

## 參考資訊

1. [Unit Test 該拿 private 屬性及欄位怎麼辦？ - 使用 PrivateObject](//blog.yowko.com/2017/06/unit-test-private-field-property.html)
2. [Unit Test 想驗證 private static method 該怎麼做？ - 使用 PrivateType](//blog.yowko.com/2017/06/unit-test-static-method.html)
3. [Unit test private methods?](https://akurniaga.wordpress.com/tag/unit-test-privateobject-visual-studio/)
4. [How to Unit Test Private Methods in MS Test](https://www.infragistics.com/community/blogs/dhananjay_kumar/archive/2015/07/16/how-to-unit-test-private-methods-in-ms-test.aspx)
