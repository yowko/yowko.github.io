---
title: "C# reflection getProperty and getValue"
date: 2017-02-14T00:42:34+08:00
lastmod: 2021-11-03T00:42:34+08:00
draft: false
tags: ["csharp"]
slug: "csharp-reflection-getproperty-getvalue"
aliases:
    - /2017/02/c-sharp-reflection-getproperty-and-getvalue.html
---
## C# reflection getProperty and getValue

你曾經想要把 object 中所有的 property name 跟內容都取出來確認嗎？如果每次程式跑 debug 都花個五分鐘，或是你跟我一樣急性子的話，你一定不會想在 debug 下一個一個去確認值是否正確；另外的做法是把手動把值顯示出來，但如果這個 object 有 10 個 property 或是你需要確認多個不同的 object ，想必你也不想一個一個慢慢寫，或許可以參考我的笨方法

## 說明

- 取得目標物件的型別
- 取得目標型別的屬性資訊
- 以目標型別的屬性資訊來取得屬性名稱及屬性值

## console

- 使用 LINQPad 撰寫測試程式

    ```cs
    void Main()
    {

        userData user = new userData { id = Guid.NewGuid(), name = "yowko", tel = "1234567890" };

        Type t = typeof(userData);
        PropertyInfo[] propInfos = t.GetProperties(BindingFlags.Public|BindingFlags.Instance);
        foreach (var element in propInfos)
        {
            element.Name.Dump();
            element.GetValue(user).Dump();
        }
    }

    public class userData
    {
        public Guid id { get; set; }

        public string name { get; set; }

        public string tel { get; set; }
    }
    ```

## ASP.NET MVC

- View

    ```cs
    @using System.Reflection
    @model TestConfiguration.Handler.YowkoConfigSectionHandler
    @{
        ViewBag.Title = "Index";
    }

    <h2>Index</h2>

    @foreach (var item in Model.GetType().GetProperties(BindingFlags.Public | BindingFlags.Instance))
    {
    <div class="row">
        <div class="col-lg-1">@item.Name</div>
        <div class="col-lg-3">@item.GetValue(Model)</div>
    </div>
    }
    ```

## 參考資料

1. [How to Get the Property value using reflection in C# ?](http://abundantcode.com/how-to-get-the-property-value-using-reflection-in-c/)
2. [PropertyInfo.GetValue 方法 (Object)](https://msdn.microsoft.com/zh-tw/library/hh194385.aspx)
3. [Type.GetProperties 方法 (BindingFlags)](https://msdn.microsoft.com/zh-tw/library/kyaxdd3x.aspx)
