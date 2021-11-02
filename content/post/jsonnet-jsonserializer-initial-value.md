---
title: "客製 Json.NET 的 JsonConverter - 自動 Initial Value Type 屬性 (使用 JsonSerializer)"
date: 2017-08-02T23:42:00+08:00
lastmod: 2021-11-02T23:42:32+08:00
draft: false
tags: ["套件","csharp"]
slug: "jsonnet-jsonserializer-initial-value"
aliases:
    - /2017/08/jsonnet-jsonserializer-initial-value.html
---
## 客製 Json.NET 的 JsonConverter - 自動 Initial Value Type 屬性 (使用 JsonSerializer)

之前文章 [客製 Json.NET 的 JsonConverter - 自動 Initial Value Type 屬性](/custom-josnconverter-initial-valuetype) 介紹到可以在使用自訂 JsonConverter 在將物件轉為 json 前先進行初始化

後來同事在使用自訂 JsonConverter 時需採用另個 Json.Net 的語法 - JsonSerializer 遇到問題，事實上核心語法都相同，筆記一下用法，之後需要時就可以直接抄 XD

## 基本環境說明

1. 自訂型別

    ```cs
    public class userData<T>
    {
        public string TestName { get; set; }
        public Guid id { get; set; }
        public List<T> name { get; set; }
        public Dictionary<string,int> TestDic { get; set; }
        public IList<T> TestIList { get; set; }
        public IEnumerable<T> TestEnum { get; set; }
        public int[] Products{ get; set; }
    }
    ```

2. 使用方式

    > 以下使用 LINQPad 進行 demo

    ```cs
    userData<string> user = new userData<string> {};
    user.Dump();
    JsonSerializer serializer = new JsonSerializer();
    using (var sw = new StringWriter())
    using (JsonWriter writer = new JsonTextWriter(sw))
    {
        serializer.Serialize(writer,user);
        sw.ToString().Dump();
    }
    ```

    ![1default](https://user-images.githubusercontent.com/3851540/28881733-a93f9dba-77db-11e7-9d02-706acb698804.png)

## 客製 JsonConverter

關於客製 JsonConverter 可以參考 Json.NET 的官方文件 [Custom JsonConverter](http://www.newtonsoft.com/json/help/html/CustomJsonConverter.htm)

```cs
public class InitialJsonConvert : JsonConverter
{
    private readonly Type[] _types;

    public InitialJsonConvert(params Type[] types)
    {
        _types = types;
    }

    public override void WriteJson(JsonWriter writer, object value, JsonSerializer serializer)
    {
        // 取得傳入 class 型別
        Type type = value.GetType();
        // 取得傳 class 的 property 資訊
        PropertyInfo[] propInfos = type.GetProperties(BindingFlags.Public | BindingFlags.Instance);
        //propInfos.Dump();
        // 針對所有 class 的 property 資訊操作
        foreach (var element in propInfos)//.Where(p => p.GetIndexParameters().Length == 0))
        {
            //element.GetIndexParameters().Dump();
            //element.GetIndexParameters().Length.Dump();
            // 檢查傳入值是否為 null
            if (element.GetValue(value) == null)
            {
                //取得 property 的型別
                Type pt = element.PropertyType;
                // string 是 value type 的特例，需要特別處理
                if (pt == typeof(string))
                {
                    // 使用 string.Empty 初始化 property 
                    element.SetValue(value, string.Empty);
                }
                // array 也需要特別處理
                else if (pt.IsArray)
                {
                    //pt.Dump();
                    var arrayType=pt.GetElementType();
                    //arrayType.Dump();
                    element.SetValue(value,Array.CreateInstance(arrayType, 0) );
                }
                else if(pt.IsInterface && type.IsGenericType)
                {
                    var Ttype=pt.GetGenericArguments()[0];
                    element.SetValue(value, Array.CreateInstance(Ttype,0));
                }
                else
                {
                    // 使用 property 的型別來建立實體
                    //pt.Dump();
                    element.SetValue(value, Activator.CreateInstance(pt));
                }
            }
        }
        JToken t = JToken.FromObject(value);

        if (t.Type != JTokenType.Object)
        {
            t.WriteTo(writer);
        }
        else
        {
            JObject o = (JObject)t;
            IList<string> propertyNames = o.Properties().Select(p => p.Name).ToList();

            o.AddFirst(new JProperty("Keys", new JArray(propertyNames)));

            o.WriteTo(writer);
        }
    }

    public override object ReadJson(JsonReader reader, Type objectType, object existingValue, JsonSerializer serializer)
    {
        throw new NotImplementedException("Unnecessary because CanRead is false. The type will skip the converter.");
    }

    public override bool CanRead
    {
        get { return false; }
    }

    public override bool CanConvert(Type objectType)
    {
        return _types.Any(t => t == objectType);
    }
}
```

## 如何使用

1. 指定 JsonSerializer 格式

    ```cs
    serializer.Formatting = Newtonsoft.Json.Formatting.Indented;
    ```

2. 加入自訂 JsonConverter 至 JsonSerializer 中

    ```cs
    serializer.Converters.Add(new InitialJsonConvert(user.GetType()));
    ```

3. 實際效果
    * 修改前與修改後程式碼

        > 以下請使用 LINQPas 執行

        ```cs
        //以下為修改前版本
        //建立 demo 用 instance
        userData<string> user = new userData<string> { };
        //輸出 data
        user.Dump();
        //輸出 serialize 後的 json string
        JsonConvert.SerializeObject(user).Dump();
        //以上為修改前版本

        //以下為修改後版本
        //建立 JsonSerializer instance
        JsonSerializer serializer = new JsonSerializer();
        //指定 JsonSerializer 格式
        serializer.Formatting = Newtonsoft.Json.Formatting.Indented;
        //加入自訂 JsonConvert
        serializer.Converters.Add(new InitialJsonConvert(user.GetType()));
        using (var sw = new StringWriter())
        using (JsonWriter writer = new JsonTextWriter(sw))
        {
            // serialize data
            serializer.Serialize(writer, user);
            // 輸出 data
            sw.ToString().Dump();
        }
        // 輸出經過自訂 serialize 的 data (value type 已被初始化)
        user.Dump();
        ```

    * 輸出結果

        ![2result](https://user-images.githubusercontent.com/3851540/28881734-a9b1e294-77db-11e7-9b76-c881bd4241ab.png)

## 心得

主要程式是延用 [客製 Json.NET 的 JsonConverter - 自動 Initial Value Type 屬性](http://blog.yowko.com/2017/07/custom-josnconverter-initial-valuetype.html) 並改用 JsonSerializer 來進行序列化的動作，適合用在 web request 直接將結果 data 序列化為 json 並透過 StreamWriter 輸出

## 參考資訊

1. [客製 Json.NET 的 JsonConverter - 自動 Initial Value Type 屬性](/custom-josnconverter-initial-valuetype)
2. [Custom JsonConverter](http://www.newtonsoft.com/json/help/html/CustomJsonConverter.htm)
3. [Json.Net fails to serialize to a stream, but works just fine serializing to a string](https://stackoverflow.com/questions/9845741/json-net-fails-to-serialize-to-a-stream-but-works-just-fine-serializing-to-a-st)
