---
title: "將 Object 序列化為 Json 時，攤平 Json string 欄位"
date: 2019-05-05T21:30:00+08:00
lastmod: 2019-05-05T21:30:31+08:00
draft: false
tags: ["C#"]
slug: "not-serialize-json-string"
---
# 將 Object 序列化為 Json 時，攤平 Json string 欄位

在不同系統間透過 Json 來交換資料是很常見的設計，甚至在某些系統上還可以見到將 Json string 儲存在屬性中，當然有人會質疑這樣就失去強型別的好處，不過會這樣使用就是希望達到 schema-free 的彈性：在一開始未能完整設計但又是非核心常用的內容可以被保留下來。

最近專案就用到了類似的設計：將某些述敘用內容轉為 Json string 放在屬性值中保存，在系統間的溝通完全沒有問題，不再會因為上下游系統想要改屬性名稱或是增減屬性定義就需要每個系統跟著調整，不過在 log 時就不是那麼好閱讀，Json 有一部份的好處就是人眼可以直接看得懂，但物件屬性值如果是 Json string 在 log 輸出時會被重複 serialize，而加上跳脫符號(`\`)，這個問題過去也遇過就趁這機會筆記一下吧

## 基本環境說明

1. macOS Mojave 10.14.4
2. .NET Core 2.2.101
3. NuGet package
   - Newtonsoft.Json 12.0.2
   - Serilog 2.8.0
   - Serilog.Sinks.File 4.0.0
4. 基本 model 及使用方式

    ```cs
     class People
    {
        public string Name { get; set; }
        public int UserId { get; set; }

        public DateTime BirthDay { get; set; }
        public string JobsString { get; set; }
    }

    internal class Job
    {
        public string CompanyName { get; set; }
        public decimal Salary { get; set; }
    }
    var Jobs = new List<Job>()
    {
        new Job
        {
            CompanyName = "C1",
            Salary = 1000
        },
        new Job
        {
            CompanyName = "C2",
            Salary = 2000
        }
    };

    var people = new People
    {
        Name = "Yowko",
        UserId = 1,
        BirthDay = new DateTime(1983, 7, 29),
        JobsString = JsonConvert.SerializeObject(Jobs)
    };
    ```


## 重現問題

1. 將上述 `people` 透過 serilog 輸出

    ```cs
    Log.Information(JsonConvert.SerializeObject(people));
    ```

2. 實際 log 內容

    ```
    2019-05-05 23:41:09.770 +08:00 [INF] {"Name":"Yowko","UserId":1,"BirthDay":"1983-07-29T00:00:00","JobsString":"[{\"CompanyName\":\"C1\",\"Salary\":1000.0},{\"CompanyName\":\"C2\",\"Salary\":2000.0}]"}
    ```

## 解決方式

1. 自訂 method : 用來檢查 value 是否可以 parse 成 json

    > 用來確認屬性值是否為 Json string

    ```cs
    public static bool ValidateJSON(out JToken value, string s)
    {
        value = null;
        try
        {
            value = JToken.Parse(s);
            return true;
        }
        catch (JsonReaderException ex)
        {
            return false;
        }
    }
    ```

2. 處理物件屬性是否包含 Json string

    ```cs
    //將物件轉為 JObject
    var tmpJsonObject = JObject.FromObject(people);
    //處理每個屬性
    foreach (var tmpJobject in tmpJsonObject)
    {
        //確認屬性值是否為 Json string
        if (ValidateJSON( out var result,tmpJobject.Value.ToString()))
        {
            //如果屬性值為 Json string 就使用 JToken 轉換並塞回該 JObject 屬性中
            tmpJsonObject.Property(tmpJobject.Key).Value = result; 
        }
    }
    ```

3. 輸出 log

    ```cs
    Log.Information(JsonConvert.SerializeObject(tmpJsonObject));
    ```

4. 實際 log 內容

    ```
    2019-05-05 23:43:09.847 +08:00 [INF] {"Name":"Yowko","UserId":1,"BirthDay":"1983-07-29T00:00:00","JobsString":[{"CompanyName":"C1","Salary":1000.0},{"CompanyName":"C2","Salary":2000.0}]}
    ```

## 心得

我查到資料 [How to make sure that string is valid JSON using JSON.NET](https://stackoverflow.com/a/20218426)

1. JObject.Parse

    > 用來處理 Json object

2. JArray.Parse

    > 用來處理 Json array

3. JContainer.Parse

    > 可以同時用來處理 Json object 及 Json array

其中因為 Container 是衍伸類別，所以改用 JToken

# 參考資訊

1. [How to make sure that string is valid JSON using JSON.NET](https://stackoverflow.com/a/20218426)
2. [Access to a static member of a type via a derived type](https://confluence.jetbrains.com/pages/viewpage.action?pageId=37232484)