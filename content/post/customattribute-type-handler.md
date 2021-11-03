---
title: "透過自訂 Attribute 標示屬性讓 Dapper 進行 json 轉換"
date: 2019-01-17T21:30:00+08:00
lastmod: 2021-11-03T21:30:31+08:00
draft: false
tags: ["csharp","Dapper"]
slug: "dapper-customattribute-typehandler"
---
## 透過自訂 Attribute 標示屬性讓 Dapper 進行 json 轉換

之前在筆記 [使用 Dapper 將 json string 轉換為 object](/dapper-json-string-to-object/) 紀錄到可以透過自訂 TypeHandler 讓 Dapper 可以將 db 內的值處理成需要的樣子(目前的用法是 db 欄位直接存 json 字串，但實際使用時需要將 json 轉回 c# object)

雖然自訂 TypeHandler 是當下最佳的解決方案，但實際使用上卻顯得有些礙手礙腳：使用時需要直接將轉換的目標型別傳至 db access layer 進行設定 - 非常不直覺，於是同事就提出使用 custom attribute 的方式，讓 json 轉換為 c# object 的行為改由 property 的 attribute 來設定，轉換型別也直接動態取用自 property 的型別定義，權責分離得更乾淨，也不用擔心忘記註冊轉換的問題，覺得解法超棒 (我怎麼沒想到XD)  紀錄一篇以加深印象

## 建立 Interface

> 主要是用來讓 reflection 時可以加速搜尋到需要執行 Dapper 轉換的 class，只是純標示用沒有實際用途

```cs
public interface IConvertType
{
}
```

## 建立客製 Attribute

> 用來標記需要透過 dapper 轉換的 property，用來取得 property 名稱及型別

```cs
[AttributeUsage(AttributeTargets.Property)]
public class JsonTypeAttribute : Attribute
{
}
```

## 客製 Dapper 的 TypeHandler

> 實際 db 欄位與 c# object 間的轉換邏輯

```cs
public class UserTypeHandler : SqlMapper.ITypeHandler
{
    //將 json string 從 db 取出時做轉型
    public object Parse(Type destinationType, object value)
    {
        //將 DB 的 json string 內容轉為目標的型別
        return JsonConvert.DeserializeObject(value.ToString(), destinationType);
    }
    //將值存回 db 時由 object 轉為 json
    public void SetValue(IDbDataParameter parameter, object value)
    {
        parameter.Value = (value == null) ? (object)DBNull.Value : JsonConvert.SerializeObject(value);
        parameter.DbType = DbType.String;
    }
}
```

## 程式初始化時自動依設定註冊 TypeHandler

> 將 class 掃描動作加至程式啟動處：application_start 或是 startup、static constructor 這類只會被執行一次的位置以避免無謂效能耗損

```cs
//載入當下執行 assembly 所參考到的所有 assembly
var assemblies = Assembly
    .GetExecutingAssembly()
    .GetReferencedAssemblies()
    .Select(Assembly.Load);

var convertClasses = AppDomain
    .CurrentDomain
    .GetAssemblies()
    .Where(a => a.FullName.StartsWith("Yowko", StringComparison.InvariantCultureIgnoreCase))
    .SelectMany(x => x.DefinedTypes.Where(type => typeof(IConvertType).GetTypeInfo().IsAssignableFrom(type.AsType())))//取得有實作 IConvertType 的類別
    //.SelectMany(a => a.GetTypes().Where(p => p.GetInterfaces().Contains(typeof(IConvertType))))//效果與上句相同，據說上句較快，但個人覺得這句語意比較清楚且效能差異只有一次應可忽略
    .SelectMany(a => a.GetProperties(BindingFlags.Public | BindingFlags.NonPublic | BindingFlags.Instance).Where(b => b.GetCustomAttribute<JsonTypeAttribute>() != null))//取出有 JsonTypeAttribute 標記的 property
    .Select(c => c.PropertyType)
    .ToList();

    // 將需要轉型的 property 型別註冊至 Dapper 使用 TypeHandler 做型別轉換
    convertClasses.ForEach(a => SqlMapper.AddTypeHandler(a, new UserTypeHandler()));

```

## 實際使用

```cs
var users = new List<User>();
// db 連線
using (var conn = new SqlConnection("Data Source=.;database=YowkoTest;Integrated Security=SSPI;app=LINQPad"))
{
    // dapper 取得資料
    users = conn.Query<User>("SELECT * FROM dbo.[User]").ToList();
}

//顯示結果
users.ForEach(a => Console.WriteLine(JsonConvert.SerializeObject(a)));
Console.ReadKey();
```

## 完整程式碼

[完整程式碼](https://github.com/yowko/CustomTypeHandlerWithAttribute.git)

## 心得

透過自訂 attribute 標記的方式讓程式碼更簡潔，意圖也更明確，完全是不同層次的思維，相較之下我原本解法顯得有夠遜，有強大同事可以互相討論學習實在太棒了

## 參考資訊

1. [使用 Dapper 將 json string 轉換為 object](/dapper-json-string-to-object/)
