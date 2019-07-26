---
title: "Test 中驗證 Object 是否相同的方法"
date: 2019-07-23T21:30:00+08:00
lastmod: 2019-07-23T21:30:31+08:00
draft: false
tags: ["UnitTest","csharp"]
slug: "object-compare-in-test"
---

## Test 中驗證 Object 是否相同的方法

最近花了不少時間在重構先前專案中的 Unit Test 與 Integration Test，其中對於 reference type 的物件比對有幾種不同的寫法

當然我個人大多配合團隊規範，不會堅持某些做法，就是邊寫邊看邊學，如果有機會看到更好的方式就偷師幾招，今天就筆記一下個人在進行測試時驗證 `期望物件` 是否與 `實際物件` 相同的幾個做法

## 基本環境說明

1. macOS Mojave 10.14.5
2. .NET Core 2.2.301
3. NuGet package

    - ExpectedObjects 2.3.4
    - FluentAssertions 5.7.0

4. 測試用資料

    ```cs
    public class TestObject
    {
        public int Id { get; set; }

        public string Name { get; set;}

        public DateTime Birthday { get; set;}
    }
    ```

## 1. 覆寫 `Equals`

> 不建議只為了 `測試` 而覆寫 `Object.Equals`，這是本末倒置：`不該為了測試目的而修改了 prodcution code 的行為`，以下只是學術研究

1. 繼承並實作 `IEquatable<T>`

    ```cs
    public class TestObject : IEquatable<TestObject>
    ```

2. 實作 `Equals`

    ```cs
    public bool Equals(TestObject other)
    {
        if (ReferenceEquals(null, other)) return false;
        if (ReferenceEquals(this, other)) return true;

       return this.Id == other.Id && this.Birthday == other.Birthday && this.Name == other.Name;
    }
    ```

3. 覆寫 `Object.Equals` 與 `GetHashCode`

    > 以 `測試` 這個目的，這個步驟可以忽略，不會影響實際效果

    ```cs
    public override bool Equals(object obj)
    {
        if (ReferenceEquals(null, obj)) return false;
        if (ReferenceEquals(this, obj)) return true;

        return obj.GetType() == this.GetType() && Equals((TestObject) obj);
    }

    public override int GetHashCode()
    {
        unchecked
        {
            return this.Id * 1983 * this.Birthday.GetHashCode() * 7 * this.Name.GetHashCode() * 29;
        }
    }
    ```

4. 實際使用

    ```cs
    // Arrange
    var expected = new TestObject
    {
        Id = 1,
        Name = "Test",
        Birthday = new DateTime(1983, 7, 29)
    };


    // Act
    var actual = new TestObject
    {
        Id = 1,
        Name = "Test",
        Birthday = new DateTime(1983, 7, 29)
    };

    // Assert
    Assert.AreEqual(expected, actual);
    ```

## 2. 自訂比對

1. 建立比對方法

    ```cs
    public static class CompareExtension
    {
        public static string ToStringNullSafe(this object obj)
        {
            return obj != null ? obj.ToString() : String.Empty;
        }
        public static bool Compare<T>(T actual, T expected, params string[] ignore)
        {
            var actualProps = actual.GetType().GetProperties();
            var expectedProps = expected.GetType().GetProperties();
            var count = actualProps.Length;
            for (var i = 0; i < count; i++)
            {
                var actualValue = actualProps[i].GetValue(actual, null).ToStringNullSafe();
                var expectedValue = expectedProps[i].GetValue(expected, null).ToStringNullSafe();
                if (actualValue != expectedValue && ignore.All(x => x != actualProps[i].Name))
                {
                    return false;
                }
            }
            return true;
        }
    }
    ```

2. 實際使用

    ```cs
    // Arrange
    var expected = new TestObject
    {
        Id = 1,
        Name = "Test",
        Birthday = new DateTime(1983, 7, 29)
    };


    // Act
    var actual = new TestObject
    {
        Id = 1,
        Name = "Test",
        Birthday = new DateTime(1983, 7, 29)
    };

    // Assert
    Assert.IsTrue( CompareExtension.Compare(actual,expected));
    ```

## 3. 使用第三方套件

> 這是普遍推薦的作法，差異在於 assert fail 時會丟出詳細錯誤細節，對於 debug 相對友善

- ExpectedObject

    ```cs
    // Arrange
    var expected = new TestObject
    {
        Id = 1,
        Name = "Test",
        Birthday = new DateTime(1983, 7, 29)
    };


    // Act
    var actual = new TestObject
    {
        Id = 1,
        Name = "Test",
        Birthday = new DateTime(1983, 7, 29)
    };

    // Assert
    expected.ToExpectedObject().ShouldEqual(actual);
    ```

- FluentAssertions

    ```cs
    // Arrange
    var expected = new TestObject
    {
        Id = 1,
        Name = "Test",
        Birthday = new DateTime(1983, 7, 29)
    };


    // Act
    var actual = new TestObject
    {
        Id = 1,
        Name = "Test",
        Birthday = new DateTime(1983, 7, 29)
    };

    // Assert
    actual.Should().BeEquivalentTo(expected);
    ```

## 心得

以下是實際出現 assert fail 的訊息截圖

1. 實作 `IEquatable<T>`

    ![_output_3objectequal](https://user-images.githubusercontent.com/3851540/61721133-8e91cf00-ad9a-11e9-964d-1e0750326d8e.png)

2. 自訂比對

    ![_output_4customcompare](https://user-images.githubusercontent.com/3851540/61721134-8e91cf00-ad9a-11e9-848a-5888b05359ed.png)

3. ExpectedObject

    ![_output_1expectedobject](https://user-images.githubusercontent.com/3851540/61721130-8df93880-ad9a-11e9-89b0-421250240095.png)

4. FluentAssertions

    ![_output_2fluentassertions](https://user-images.githubusercontent.com/3851540/61721131-8df93880-ad9a-11e9-8ceb-3b7ab19ddad7.png)

看完 assert fail 的訊息後，我立馬就放棄了 `實作 IEquatable<T>` 與 `自訂比對`，提示訊息很不友善，對於快速定位問題源頭沒有幫助

至於兩個 NuGet package ： `ExpectedObject` 與 `FluentAssertions`，我個人是比較常用 `FluentAssertions`，因為 `FluentAssertions` 還可以用來將 assert 做語意化的呈現，而 `ExpectedObject` 就是只用來比較 object 是否相同，功能相對比較侷限

## 參考資訊

1. [Asserting Equality in your C# unit tests](https://medium.com/@pjbgf/asserting-equality-in-your-c-unit-tests-837b423024bf)
2. [How to Compare Object Instances in your Unit Tests Quickly and Easily](https://buildplease.com/pages/testing-deep-equalilty/)
