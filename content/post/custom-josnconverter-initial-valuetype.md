---
title: "客製 Json.NET 的 JsonConverter - 自動 Initial Value Type 屬性"
date: 2017-07-07T21:00:00+08:00
lastmod: 2021-11-03T14:17:40+08:00
draft: false
tags: ["套件","Debug"]
slug: "custom-josnconverter-initial-valuetype"
aliases:
    - /2017/07/custom-josnconverter-initial-valuetype.html
---
## 客製 Json.NET 的 JsonConverter - 自動 Initial Value Type 屬性

同事負責的專案原本使用 XML 做為資料傳遞的媒介，為了縮小網路傳輸封包，所以改用 json，而這個動作讓原本正常運行的功能出現問題

問題描述：使用 XML 在 Value Type 未給值時，程式會自動初始化才寫入 XML (這個部份是同事轉述，我沒有實際測試，說錯請指教)，後來改用 json 後未給值的 Value Type 則是會直接使用 null 進行寫入，這讓接收端未檢查 property 是否 null 的程式出現 NullPointerException

第一個念頭：在接收端使用 `Null 條件運算子(Null-conditional / Elvis operator – ?.)`，但要改的程式碼非常多，立馬放棄

第二個念頭：修改 Json.NET 的 JsonConverter，直接在寫入 json 前先進行初始化動作，最後就是這個方案雀屏中選了，就來看看該怎麼做吧

## 基本環境說明

1. 自訂型別

    ```cs
    public class userData<T>
    {
        public string TestName { get; set; }
        public Guid id { get; set; }
        public List<T> name { get; set; }
        public Dictionary<string,int> TestDic { get; set; }
        //2017/07/10 同事反應需要 IList 加入
        public IList<T> TestIList { get; set; }
        //2017/07/10 同事反應需要 IList 順便測試 IEnumerable
        public IEnumerable<T> TestEnum { get; set; }
        public int[] Products{ get; set; }
    }
    ```

2. 使用方式

    > 以下使用 LINQPad demo

    ```cs
    userData<string> user = new userData<string> { };
    user.Dump();
    JsonConvert.SerializeObject(user).Dump();
    ```

    ![1originuse](https://user-images.githubusercontent.com/3851540/27953763-9a9b90ee-633f-11e7-8519-cea4b7ca7b6b.png)

## 建立 class 繼承 JsonConverter

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

## 開始客製

1. 使用 reflection 檢查內容

    ```cs
    // 取得傳入 class 型別
    Type type = value.GetType();
    // 取得傳 class 的 property 資訊
    PropertyInfo[] propInfos = type.GetProperties(BindingFlags.Public | BindingFlags.Instance);
    // 針對所有 property 資訊操作
    foreach (var element in propInfos)
    {
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
                var arrayType=pt.GetElementType();
                element.SetValue(value,Array.CreateInstance(arrayType, 0) );
            }
            //2017/07/10 同事反應需要 IList 加入
            else if(pt.IsInterface && type.IsGenericType)
            {
                var Ttype=pt.GetGenericArguments()[0];
                element.SetValue(value, Array.CreateInstance(Ttype,0));
            }
            else
            {
                // 使用 property 的型別來建立實體
                element.SetValue(value, Activator.CreateInstance(pt));
            }
        }
    }
    ```

2. 完整程式碼

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
            // 針對所有 class 的 property 資訊操作
            foreach (var element in propInfos)
            {
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
                        var arrayType=pt.GetElementType(); 
                        element.SetValue(value,Array.CreateInstance(arrayType, 0) );
                    }
                    //2017/07/10 同事反應需要 IList 加入
                    else if(pt.IsInterface && type.IsGenericType)
                    {
                        var Ttype=pt.GetGenericArguments()[0];
                        element.SetValue(value, Array.CreateInstance(Ttype,0));
                    }
                    else
                    {
                        // 使用 property 的型別來建立實體
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

1. 在 SerializeObject 時使用 `Formatting` 與自訂 `JsonConvert`

    > 以下使用 LINQPad demo

    ```cs
    JsonConvert.SerializeObject(user, Newtonsoft.Json.Formatting.Indented, new InitialJsonConvert(user.GetType())).Dump();
    ```

2. 實際效果
    * 修改前後程式碼對比

        > 請使用 LINQPad 執行

        ```cs
        // demo 用 instance
        userData<string> user = new userData<string> { };
        //輸出 user
        user.Dump();
        //輸出 serialize 後的 user string
        JsonConvert.SerializeObject(user).Dump();
        // 輸出自訂 serialize 的 user string
        JsonConvert.SerializeObject(user, Newtonsoft.Json.Formatting.Indented, new InitialJsonConvert(user.GetType())).Dump();
        // 輸出經過自訂 serialize 的 user (value type 已被初始化)
        user.Dump();
        ```

    * 輸出結果

        ![2result](https://user-images.githubusercontent.com/3851540/27953764-9a9eb2f6-633f-11e7-88a2-a9c60a1468b3.png)

## 心得

我覺得應該有更簡單的做法，這個需求應該很常見才是，但因為是 production issue 時間緊迫，先求可以解決問題，有空再來找更好的解法，如果大家知道有哪個設定可以達成目的，拜託請告訴我，讓小弟學習一下，感謝

## 參考資訊

1. [Custom JsonConverter](http://www.newtonsoft.com/json/help/html/CustomJsonConverter.htm)
2. [c# reflection getProperty and getValue](/2017/02/c-sharp-reflection-getproperty-and-getvalue.html)
3. [How to check if a variable is Array or Object?](https://stackoverflow.com/questions/10118324/how-to-check-if-a-variable-is-array-or-object)
4. [How do I create a C# array using Reflection and only type info?](https://stackoverflow.com/questions/3419456/how-do-i-create-a-c-sharp-array-using-reflection-and-only-type-info)
5. [Get a new object instance from a Type](https://stackoverflow.com/questions/752/get-a-new-object-instance-from-a-type)
